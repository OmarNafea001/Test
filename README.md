-- =====================================================================
-- PCRF Quota Mismatch - النسخة المنظمة (نفس اللوجيك بتاع الأصلية)
-- الناتج: يتسجل في RD_T_PCRF_QUOTA_CONSUM_CASES
--
-- خريطة الخطوات:
--   1. CATALOGUE         ← نوع الـ PCRF ونوع المنتج لكل خدمة
--   2. ACTIVE_CUSTOMERS  ← العملاء الـ Active بس
--   3. MAIN_PLUS_DOUBLE  ← مجموع كوتا (Main + Double) لكل عميل
--   4. MAIN_ONLY         ← كوتا الـ Main لوحدها لكل عميل
--   5. DOUBLE_SUM        ← الرقمين (3) و (4) جنب بعض في صف واحد   <-- الـ JOIN المهم
--   6. DOUBLE_EXACT      ← عملاء الـ Double اللي كوتاهم = ضعف الـ Main بالظبط
--   7. BASE              ← حساب الكوتا الفعلية والـ VARIANCE والتصنيفات
--   8. SELECT النهائي    ← الفلاتر وإدخال الـ Cases
--
-- ملاحظة: مفيش سطور فاضية جوه الـ query لأن SQL*Plus بيعتبرها نهاية الجملة
-- =====================================================================
INSERT INTO RD_T_PCRF_QUOTA_CONSUM_CASES
WITH
--------------------------------------------------------------------
-- 1) CATALOGUE: بيانات كل خدمة (Prepaid/Postpaid و Main Bundle وغيره)
--------------------------------------------------------------------
CATALOGUE AS (
  SELECT DISTINCT
         pcrf_service_name,
         pcrf_type,
         prod_type_name
  FROM   RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE
),
--------------------------------------------------------------------
-- 2) ACTIVE_CUSTOMERS: العملاء الـ Active فقط (customer_status = 1)
--------------------------------------------------------------------
ACTIVE_CUSTOMERS AS (
  SELECT msisdn
  FROM   RP7VFEPIDAT.PFM_PCRF_STG_FILTERED
  WHERE  customer_status = 1
),
--------------------------------------------------------------------
-- 3) MAIN_PLUS_DOUBLE: مجموع كوتا (Main Bundle + Double) لكل عميل
--    HAVING COUNT(*) = 2 ← يعني العميل عنده صفين بس (Main واحد + Double واحد)
--------------------------------------------------------------------
MAIN_PLUS_DOUBLE AS (
  SELECT con.msisdn,
         ROUND(SUM(con.quota_sum)/1024/1024, 1) AS main_double_quota_sum
  FROM   RP7VFEPIDAT.PFM_PCRF_STG_CONSUM_FILTERED con
  LEFT JOIN CATALOGUE cat
         ON con.service_name = cat.pcrf_service_name
  WHERE  con.service_name = 'Serv_DSL_Double_Quota_Pre'
      OR cat.prod_type_name = 'Main Bundle'
  GROUP BY con.msisdn
  HAVING COUNT(*) = 2
),
--------------------------------------------------------------------
-- 4) MAIN_ONLY: كوتا الـ Main Bundle لوحدها لكل عميل
--------------------------------------------------------------------
MAIN_ONLY AS (
  SELECT con.msisdn,
         ROUND(con.quota_sum/1024/1024, 1) AS main_quota_sum
  FROM   RP7VFEPIDAT.PFM_PCRF_STG_CONSUM_FILTERED con
  JOIN   CATALOGUE cat
         ON con.service_name = cat.pcrf_service_name
  WHERE  cat.prod_type_name = 'Main Bundle'
),
--------------------------------------------------------------------
-- 5) DOUBLE_SUM: بنجمع (3) و (4) على MSISDN فيطلع لكل عميل صف فيه:
--       main_double_quota_sum = Main + Double
--       main_quota_sum        = Main لوحدها
--    (الأصل كان FULL OUTER JOIN، لكن النتيجة النهائية مفيهاش فرق عن LEFT JOIN
--     لأن أي عميل مش موجود في (3) مكانش بيتربط بالجدول الأساسي أصلاً)
--------------------------------------------------------------------
DOUBLE_SUM AS (
  SELECT d.msisdn,
         d.main_double_quota_sum,
         m.main_quota_sum
  FROM   MAIN_PLUS_DOUBLE d
  LEFT JOIN MAIN_ONLY m
         ON d.msisdn = m.msisdn
),
--------------------------------------------------------------------
-- 6) DOUBLE_EXACT: العملاء اللي (Main + Double) = 2 × Main بالظبط (يعني سليمين)
--------------------------------------------------------------------
DOUBLE_EXACT AS (
  SELECT msisdn
  FROM   DOUBLE_SUM
  WHERE  main_double_quota_sum = 2 * main_quota_sum
),
--------------------------------------------------------------------
-- 7) BASE: الحسابات الأساسية لكل MSISDN + Service
--------------------------------------------------------------------
BASE AS (
  SELECT DISTINCT
         TO_DATE(consum.run_date, 'YYYYMMDD')           AS run_date,
         consum.msisdn                                  AS msisdn,
         consum.service_name                            AS service_name,
         ROUND(consum.quota_sum/1024/1024, 1)           AS quota_sum_gb,   -- الكوتا الفعلية
         consum.expected_org_quota                      AS expected_org_quota,
         consum.expected_tax_quota                      AS expected_tax_quota,
         ------------------------------------------------------------
         -- VARIANCE = الفعلي - المتوقع (أول شرط يتحقق هو اللي بيتنفذ)
         -- موجب = العميل أخد زيادة / سالب = أخد أقل
         ------------------------------------------------------------
         CASE
           -- (أ) الفعلي = المتوقع الأصلي ← سليم
           WHEN ROUND(consum.quota_sum/1024/1024, 1) = consum.expected_org_quota
             THEN 0
           -- (ب) Prepaid + Main Bundle والفعلي = Org + Tax ← سليم
           WHEN ROUND(consum.quota_sum/1024/1024, 1) = consum.expected_org_quota + consum.expected_tax_quota
                AND cat.pcrf_type LIKE '%PREPAID%'
                AND cat.prod_type_name = 'Main Bundle'
             THEN 0
           -- (ج) Prepaid + Main Bundle والفعلي مختلف عن Org + Tax ← الفرق عن (Org + Tax)
           WHEN ROUND(consum.quota_sum/1024/1024, 1) != consum.expected_org_quota + consum.expected_tax_quota
                AND cat.pcrf_type LIKE '%PREPAID%'
                AND cat.prod_type_name = 'Main Bundle'
             THEN ROUND(consum.quota_sum/1024/1024, 1) - (consum.expected_org_quota + consum.expected_tax_quota)
           -- (د) أي حالة تانية ← الفرق عن Org بس
           ELSE ROUND(consum.quota_sum/1024/1024, 1) - consum.expected_org_quota
         END                                            AS variance,
         -- نوع الـ PCRF (لو فاضي نكتب NA)
         NVL(cat.pcrf_type, 'NA')                       AS pcrf_type,
         -- لو Org و Tax و Throttle كلهم 0 ← الخدمة مش متغطية في الـ Lookup
         CASE
           WHEN consum.expected_org_quota   = '0'
            AND consum.expected_tax_quota   = '0'
            AND consum.expected_throt_quota = '0'
             THEN 'EXCLUDED'
           ELSE 'INCLUDED'
         END                                            AS lookup_inclusion,
         ------------------------------------------------------------
         -- تصنيف الـ Double Quota: بنقارن (Main + Double) بـ (2 × Main)
         ------------------------------------------------------------
         CASE
           WHEN consum.service_name = 'Serv_DSL_Double_Quota_Pre'
                AND ds.main_double_quota_sum < ds.main_quota_sum * 2
             THEN 'Customer Experience'      -- أخد أقل من الضعف
           WHEN consum.service_name = 'Serv_DSL_Double_Quota_Pre'
                AND ds.main_double_quota_sum > ds.main_quota_sum * 2
             THEN 'Revenue Loss'             -- أخد أكتر من الضعف
           ELSE 'NOT Double Quota Case'      -- مش Double، أو الضعف بالظبط
         END                                            AS double_quota_cases
  FROM   RP7VFEPIDAT.PFM_PCRF_STG_CONSUM_FILTERED consum
  LEFT JOIN CATALOGUE cat
         ON consum.service_name = cat.pcrf_service_name
  LEFT JOIN DOUBLE_SUM ds
         ON consum.msisdn = ds.msisdn
  WHERE  consum.msisdn IN (SELECT msisdn FROM ACTIVE_CUSTOMERS)
)
--------------------------------------------------------------------
-- 8) الـ SELECT النهائي: الفلاتر (اللي بيدخل الجدول هو اللي بيعدي كل الشروط)
--------------------------------------------------------------------
SELECT run_date,
       msisdn,
       service_name,
       quota_sum_gb,
       expected_org_quota,
       expected_tax_quota,
       variance,
       pcrf_type,
       lookup_inclusion,
       double_quota_cases
FROM   BASE
WHERE  variance != 0                              -- المختلف فقط
  AND  service_name NOT LIKE '%Thrott%'           -- استبعاد Throttle
  AND  service_name NOT LIKE '%Double%'           -- استبعاد Double  <-- انتبه: ده بيشيل خدمة الـ Double نفسها
  AND  service_name NOT LIKE '%Gift%'             -- استبعاد Gift
  AND  service_name NOT LIKE '%Promo%'            -- استبعاد Promo
  -- Repurchase Normal Cases: العميل اشترى نفس الباقة من 2 لـ 5 مرات (سلوك طبيعي)
  AND  quota_sum_gb NOT IN (expected_org_quota*2, expected_org_quota*3,
                            expected_org_quota*4, expected_org_quota*5)
  -- استبعاد عملاء الـ Double السليمين (كوتاهم = ضعف الـ Main بالظبط)
  AND  NOT (service_name = 'Serv_DSL_Double_Quota_Pre'
            AND msisdn IN (SELECT msisdn FROM DOUBLE_EXACT));
COMMIT;
