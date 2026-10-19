#!/bin/bash
# =====================================================================
# PCRF Expiry Date Mismatch - النسخة المقروءة (نفس اللوجيك بتاع الأصلية)
#
# الفكرة: عملاء Active خدماتهم لسه شغالة على PCRF
#         رغم إن تاريخ انتهائها عدّى (أو مفيش تاريخ انتهاء أصلاً)
# الناتج: يتسجل في rd_t_pcrf_expiry_date_mismatch
#
# خريطة الخطوات:
#   1. POSTPAID_ADDONS  ← قائمة الخدمات اللي بنفحصها للـ Postpaid
#   2. الـ FROM/JOIN    ← الخدمات من PCRF + بيانات الـ Billing
#   3. شرط نوع العميل   ← Prepaid (كل الخدمات) / Postpaid (الـ Add-ons بس)
#   4. شرط الانتهاء     ← مفيش تاريخ، أو التاريخ قبل امبارح
#   5. الاستبعادات      ← Active بس، بدون Throttle/OnHold/خطوط التست
#   6. GROUP BY         ← صف واحد لكل عميل، والخدمات جنب بعض بفاصلة
#
# ملاحظة: مفيش سطور فاضية جوه الـ query لأن SQL*Plus بيعتبرها نهاية الجملة
# =====================================================================
$ORACLE_HOME/bin/sqlplus -s <<END
RP7VFEPIDAT/RP7VFEPIDAT@$GLOBAL_DB_ORACLE_SERVICE
INSERT INTO rd_t_pcrf_expiry_date_mismatch
WITH
--------------------------------------------------------------------
-- 1) POSTPAID_ADDONS: الخدمات اللي بنفحصها للعملاء الـ Postpaid
--    (إضافات بتتشترى عند الطلب ولها صلاحية محددة)
--------------------------------------------------------------------
POSTPAID_ADDONS AS (
  SELECT DISTINCT pcrf_service_name
  FROM   RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE
  WHERE  FREQUENCY LIKE '%OnDemand%'                          -- بتتشترى عند الطلب
    AND  prod_type_name IN ('Addon', 'DSL Promotion')         -- إضافة أو برومو
    AND  pcrf_service_name NOT LIKE '%_R%'                    -- ملاحظة: الـ _ هنا بتعني اي حرف
    AND  pcrf_type LIKE '%POSTPAID%'                          -- نوعها Postpaid
)
--------------------------------------------------------------------
-- الـ SELECT الرئيسي
--------------------------------------------------------------------
SELECT /*+ PARALLEL(16) */                                    -- تشغيل متوازي لتسريع الـ query
       TO_DATE(pcrf.run_date, 'YYYYMMDD')                     AS run_date,
       pcrf.msisdn                                            AS msisdn,
       -- كل خدمات العميل في عمود واحد، مفصولة بفاصلة
       LISTAGG(service_name, ',') WITHIN GROUP (ORDER BY service_name) AS service_name,
       TRUNC(TO_DATE(valid_date,  'YYYYMMDDHH24MISS'))        AS valid_date,
       TRUNC(TO_DATE(expiry_date, 'YYYYMMDDHH24MISS'))        AS expiry_date
--------------------------------------------------------------------
-- 2) المصادر: الخدمات على PCRF + بيانات الـ Billing
--------------------------------------------------------------------
FROM   RP7VFEPIDAT.PFM_PCRF_STG_FILTERED pcrf
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG billing
       ON pcrf.msisdn = billing.msisdn
WHERE
  ------------------------------------------------------------------
  -- 3) شرط نوع العميل
  ------------------------------------------------------------------
  (
    -- (أ) عميل Prepaid: كل خدماته عدا Throttle و OnHold
    (    billing."TYPE" LIKE '%PRE%'
     AND service_name NOT LIKE '%Th%'
     AND service_name NOT LIKE '%OnHold%')
    OR
    -- (ب) عميل Postpaid: الخدمات اللي في قائمة الـ Add-ons بس
    (    billing."TYPE" LIKE '%POST%'
     AND service_name IN (SELECT pcrf_service_name FROM POSTPAID_ADDONS))
  )
  ------------------------------------------------------------------
  -- 4) شرط الـ Mismatch: مفيش تاريخ انتهاء، أو انتهى قبل امبارح
  --    (TRUNC(SYSDATE)-1 = امبارح الساعة 12 بالليل، يعني فيه فترة سماح)
  ------------------------------------------------------------------
  AND (   expiry_date IS NULL
       OR TO_DATE(expiry_date, 'YYYYMMDDHH24MISS') < TRUNC(SYSDATE) - 1)
  ------------------------------------------------------------------
  -- 5) الاستبعادات
  ------------------------------------------------------------------
  AND pcrf.customer_status = '1'                              -- العملاء الـ Active بس
  AND service_name NOT LIKE '%Th%'                            -- استبعاد Throttle
  AND service_name NOT LIKE '%OnHold%'                        -- استبعاد OnHold
  AND billing.price_group NOT IN ('TEST PHONES',              -- استبعاد خطوط التست
                                  'Pre-paid Test SIM',
                                  'Test Individuals',
                                  'Dummy PG')
--------------------------------------------------------------------
-- 6) التجميع: صف واحد لكل (تاريخ تشغيل + عميل + تاريخي الصلاحية)
--------------------------------------------------------------------
GROUP BY pcrf.run_date,
         pcrf.msisdn,
         valid_date,
         expiry_date;
COMMIT;
exit;
END
