-- =====================================================================
-- PCRF base customers that do NOT exist on the TE invoice
--   Invoice run date : 20261001  (start of month)
--   Invoice month    : 01/09/2026 -> 30/09/2026
--   To run another month, change the date in the params CTE only.
-- =====================================================================
WITH params AS (
    SELECT TO_DATE('20261001','YYYYMMDD')                 AS run_dt,     -- invoice run date
           ADD_MONTHS(TO_DATE('20261001','YYYYMMDD'), -1) AS inv_start,  -- 01/09/2026
           TO_DATE('20261001','YYYYMMDD')                 AS inv_end     -- < 01/10/2026
    FROM dual
),

-- 1) Our base: PCRF on the run date only (prepaid + postpaid, all statuses)
--    GROUP BY MSISDN -> exactly one row per MSISDN.
--    If one MSISDN has several statuses that day, keep the best one:
--    Active > Suspended > On-Hold > DeActive
pcrf_base AS (
    SELECT /*+ PARALLEL(16) */
           p.MSISDN,
           MAX(UPPER(DECODE(p.CUSTOMER_STATUS, 0, 'DeActive', 1, 'Active',
                                               2, 'Suspended', 3, 'On-Hold',
                                               p.CUSTOMER_STATUS)))
               KEEP (DENSE_RANK FIRST ORDER BY
                     DECODE(p.CUSTOMER_STATUS, 1, 1, 2, 2, 3, 3, 0, 4, 5)) AS PCRF_STATUS
    FROM RP7VFEPIDAT.PFM_PCRF_FILTERED p, params
    WHERE p.RUN_DATE = params.run_dt
    GROUP BY p.MSISDN
),

-- 2) SDSL and Cloud customers to exclude (tariff description from BSCS)
sdsl_cloud AS (
    SELECT /*+ PARALLEL(16) */ DISTINCT b.MSISDN
    FROM RP7VFEPIDAT.PFM_BSCS_ADSL b, params
    WHERE b.RUN_DATE = TO_NUMBER(TO_CHAR(params.run_dt,'YYYYMMDD'))
      AND (b.TMCODE_DES LIKE 'SDSL%' OR b.TMCODE_DES LIKE 'Cloud%')
),

base AS (
    SELECT pb.MSISDN, pb.PCRF_STATUS
    FROM pcrf_base pb
    WHERE NOT EXISTS (SELECT 1 FROM sdsl_cloud s WHERE s.MSISDN = pb.MSISDN)
),

-- 3) Everyone on the TE invoice during the invoice month
te_invoice AS (
    SELECT /*+ PARALLEL(16) */ DISTINCT t.MSISDN
    FROM RP7VFEPIDAT.RD_T_TE_DATA_BASE_HISTORICAL t, params
    WHERE t.RUN_DATE >= params.inv_start
      AND t.RUN_DATE <  params.inv_end
),

-- 4) Base customers missing from the TE invoice
missing AS (
    SELECT b.MSISDN, b.PCRF_STATUS
    FROM base b
    WHERE NOT EXISTS (SELECT 1 FROM te_invoice te WHERE te.MSISDN = b.MSISDN)
),

-- 5) Service name on the run date
quota AS (
    SELECT q.MSISDN, MAX(q.QUOTA_NAME) AS QUOTA_NAME
    FROM RP7VFEPIDAT.QUOTA_NAME_LKP q, params
    WHERE q."TIME" = params.run_dt
    GROUP BY q.MSISDN
),

-- 6) Consumption during the invoice month
--    EDRS MSISDN has no '20' prefix, so it is added here to match PCRF/TE
consumption AS (
    SELECT /*+ PARALLEL(16) */
           '20' || e.MSISDN   AS MSISDN,
           SUM(e.QUOTA_USAGE) AS TOTAL_USAGE
    FROM RP7VFEPIDAT.PFM_PCRF_EDRS_STG_CONSUM e, params
    WHERE e.RUN_DATE >= params.inv_start
      AND e.RUN_DATE <  params.inv_end
      AND e.SERVICE_NAME NOT LIKE '%Promo%'
      AND e.SERVICE_NAME NOT LIKE '%Gift%'
      AND e.SERVICE_NAME NOT LIKE '%Add%'
    GROUP BY '20' || e.MSISDN
),

totals AS (
    SELECT (SELECT COUNT(*) FROM base)    AS TOTAL_BASE,
           (SELECT COUNT(*) FROM missing) AS MISSING_COUNT
    FROM dual
)

SELECT
    m.MSISDN,
    m.PCRF_STATUS,
    NVL(q.QUOTA_NAME, '_N')                               AS SERVICE_NAME,
    REGEXP_SUBSTR(q.QUOTA_NAME, '[0-9]+M', 1, 1)          AS SPEED,
    ROUND(NVL(c.TOTAL_USAGE, 0) / POWER(1024, 3))         AS CONSUMPTION_GB,
    t.MISSING_COUNT,
    t.TOTAL_BASE,
    ROUND(t.MISSING_COUNT * 100 / NULLIF(t.TOTAL_BASE, 0), 2) AS MISSING_PCT
FROM missing m
LEFT JOIN quota       q ON q.MSISDN = m.MSISDN
LEFT JOIN consumption c ON c.MSISDN = m.MSISDN
CROSS JOIN totals t
ORDER BY CONSUMPTION_GB DESC;
