-- =====================================================================
-- PCRF base customers that do NOT exist on the TE invoice
--   Invoice run date : 20261001  (start of month)
--   Invoice month    : 01/09/2026 -> 30/09/2026
--   PCRF base        : every MSISDN with a Main Bundle service in PCRF
--                      from 01/09 to 30/09, one row each = the FIRST day
--                      it has that service
--
--   Dates are written as literals (not a params table) so Oracle can
--   prune partitions directly. To run another month, find & replace:
--       20260901  -> first day of the invoice month
--       20261001  -> invoice run date (first day of the next month)
-- =====================================================================


-- =====================================================================
-- QUERY 1 : DETAIL  (one row per missing MSISDN)
-- =====================================================================
WITH
-- 1) PCRF base: first day in the month with a Main Bundle service
--    (aggregate KEEP FIRST instead of ROW_NUMBER -> no big window sort)
pcrf_base AS (
    SELECT /*+ MATERIALIZE PARALLEL(p 64) FULL(p) */
           p.MSISDN,
           MIN(p.RUN_DATE) AS FIRST_PCRF_DATE,
           MAX(p.SERVICE_NAME) KEEP (DENSE_RANK FIRST ORDER BY p.RUN_DATE) AS PCRF_SERVICE_NAME,
           MAX(UPPER(DECODE(p.CUSTOMER_STATUS, 0, 'DeActive', 1, 'Active',
                                               2, 'Suspended', 3, 'On-Hold',
                                               p.CUSTOMER_STATUS)))
               KEEP (DENSE_RANK FIRST ORDER BY p.RUN_DATE) AS PCRF_STATUS
    FROM RP7VFEPIDAT.PFM_PCRF_FILTERED p
    WHERE p.RUN_DATE >= TO_DATE('20260901','YYYYMMDD')
      AND p.RUN_DATE <  TO_DATE('20261001','YYYYMMDD')
      AND p.SERVICE_NAME IS NOT NULL
      AND p.PACKAGE_TYPE = 'Main Bundle'
      AND p.SERVICE_NAME NOT IN ('Serv_DSL_Generic_Promo',
                                 'Serv_DSL_Compensation_Gift',
                                 'Serv_DSL_Throttling',
                                 'ADSL_OnHold',
                                 'Serv_DSL_Double_Quota_Pre')
      AND p.SERVICE_NAME NOT LIKE '%Add%'
      AND p.SERVICE_NAME NOT LIKE '%DoubleQuota%'
    GROUP BY p.MSISDN
),

-- 2) SDSL and Cloud customers to exclude (tariff description from BSCS)
sdsl_cloud AS (
    SELECT /*+ MATERIALIZE PARALLEL(b 64) */ DISTINCT b.MSISDN
    FROM RP7VFEPIDAT.PFM_BSCS_ADSL b
    WHERE b.RUN_DATE = 20261001
      AND (b.TMCODE_DES LIKE 'SDSL%' OR b.TMCODE_DES LIKE 'Cloud%')
),

-- 3) Everyone on the TE invoice during the invoice month
te_invoice AS (
    SELECT /*+ MATERIALIZE PARALLEL(t 64) FULL(t) */ DISTINCT t.MSISDN
    FROM RP7VFEPIDAT.RD_T_TE_DATA_BASE_HISTORICAL t
    WHERE t.RUN_DATE >= TO_DATE('20260901','YYYYMMDD')
      AND t.RUN_DATE <  TO_DATE('20261001','YYYYMMDD')
),

-- 4) Base customers missing from the TE invoice (hash anti-joins)
missing AS (
    SELECT /*+ MATERIALIZE PARALLEL(64) */
           pb.MSISDN, pb.FIRST_PCRF_DATE, pb.PCRF_SERVICE_NAME, pb.PCRF_STATUS
    FROM pcrf_base pb
    WHERE NOT EXISTS (SELECT /*+ HASH_AJ */ 1 FROM sdsl_cloud s WHERE s.MSISDN = pb.MSISDN)
      AND NOT EXISTS (SELECT /*+ HASH_AJ */ 1 FROM te_invoice te WHERE te.MSISDN = pb.MSISDN)
),

-- 5) Service name from QUOTA_NAME_LKP on the run date (01/10, the only date it has)
quota AS (
    SELECT /*+ PARALLEL(q 64) */ q.MSISDN, MAX(q.QUOTA_NAME) AS QUOTA_NAME
    FROM RP7VFEPIDAT.QUOTA_NAME_LKP q
    WHERE q."TIME" = TO_DATE('20261001','YYYYMMDD')
    GROUP BY q.MSISDN
),

-- 6) Consumption during the invoice month, only for the missing MSISDNs
--    EDRS MSISDN has no '20' prefix, so it is added here to match PCRF/TE
consumption AS (
    SELECT /*+ PARALLEL(e 64) FULL(e) */
           '20' || e.MSISDN   AS MSISDN,
           SUM(e.QUOTA_USAGE) AS TOTAL_USAGE
    FROM RP7VFEPIDAT.PFM_PCRF_EDRS_STG_CONSUM e
    WHERE e.RUN_DATE >= TO_DATE('20260901','YYYYMMDD')
      AND e.RUN_DATE <  TO_DATE('20261001','YYYYMMDD')
      AND e.SERVICE_NAME NOT LIKE '%Promo%'
      AND e.SERVICE_NAME NOT LIKE '%Gift%'
      AND e.SERVICE_NAME NOT LIKE '%Add%'
      AND EXISTS (SELECT /*+ HASH_SJ */ 1 FROM missing m WHERE m.MSISDN = '20' || e.MSISDN)
    GROUP BY '20' || e.MSISDN
)

SELECT /*+ PARALLEL(64) USE_HASH(q c) */
    m.MSISDN,
    m.FIRST_PCRF_DATE,
    m.PCRF_SERVICE_NAME,
    m.PCRF_STATUS,
    NVL(q.QUOTA_NAME, '_N')                         AS SERVICE_NAME,
    REGEXP_SUBSTR(q.QUOTA_NAME, '[0-9]+M', 1, 1)    AS SPEED,
    ROUND(NVL(c.TOTAL_USAGE, 0) / POWER(1024, 3))   AS CONSUMPTION_GB
FROM missing m
LEFT JOIN quota       q ON q.MSISDN = m.MSISDN
LEFT JOIN consumption c ON c.MSISDN = m.MSISDN
ORDER BY CONSUMPTION_GB DESC;



-- =====================================================================
-- QUERY 2 : SUMMARY  (one row: base vs TE invoice, difference, %)
-- =====================================================================
WITH
pcrf_base AS (
    SELECT /*+ MATERIALIZE PARALLEL(p 64) FULL(p) */ DISTINCT p.MSISDN
    FROM RP7VFEPIDAT.PFM_PCRF_FILTERED p
    WHERE p.RUN_DATE >= TO_DATE('20260901','YYYYMMDD')
      AND p.RUN_DATE <  TO_DATE('20261001','YYYYMMDD')
      AND p.SERVICE_NAME IS NOT NULL
      AND p.PACKAGE_TYPE = 'Main Bundle'
      AND p.SERVICE_NAME NOT IN ('Serv_DSL_Generic_Promo',
                                 'Serv_DSL_Compensation_Gift',
                                 'Serv_DSL_Throttling',
                                 'ADSL_OnHold',
                                 'Serv_DSL_Double_Quota_Pre')
      AND p.SERVICE_NAME NOT LIKE '%Add%'
      AND p.SERVICE_NAME NOT LIKE '%DoubleQuota%'
),

sdsl_cloud AS (
    SELECT /*+ MATERIALIZE PARALLEL(b 64) */ DISTINCT b.MSISDN
    FROM RP7VFEPIDAT.PFM_BSCS_ADSL b
    WHERE b.RUN_DATE = 20261001
      AND (b.TMCODE_DES LIKE 'SDSL%' OR b.TMCODE_DES LIKE 'Cloud%')
),

te_invoice AS (
    SELECT /*+ MATERIALIZE PARALLEL(t 64) FULL(t) */ DISTINCT t.MSISDN
    FROM RP7VFEPIDAT.RD_T_TE_DATA_BASE_HISTORICAL t
    WHERE t.RUN_DATE >= TO_DATE('20260901','YYYYMMDD')
      AND t.RUN_DATE <  TO_DATE('20261001','YYYYMMDD')
),

base AS (
    SELECT /*+ PARALLEL(64) */ pb.MSISDN
    FROM pcrf_base pb
    WHERE NOT EXISTS (SELECT /*+ HASH_AJ */ 1 FROM sdsl_cloud s WHERE s.MSISDN = pb.MSISDN)
)

SELECT /*+ PARALLEL(64) USE_HASH(te) */
    TO_DATE('20261001','YYYYMMDD')                                      AS RUN_DATE,
    COUNT(*)                                                            AS TOTAL_PCRF_BASE,
    COUNT(te.MSISDN)                                                    AS ON_TE_INVOICE,
    COUNT(*) - COUNT(te.MSISDN)                                         AS NOT_ON_TE_INVOICE,
    ROUND((COUNT(*) - COUNT(te.MSISDN)) * 100 / NULLIF(COUNT(*), 0), 2) AS NOT_ON_TE_PCT
FROM base b
LEFT JOIN te_invoice te ON te.MSISDN = b.MSISDN;
