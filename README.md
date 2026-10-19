WITH params AS (
    SELECT TRUNC(MAX(RUN_DATE)) AS ref_date
    FROM   RP7VFEPIDAT.PFM_PCRF_CONSUM_FILTERED
),

cycles AS (
    SELECT '01' AS bc, 31 AS e FROM dual UNION ALL
    SELECT '02',       15      FROM dual UNION ALL
    SELECT '04',        3      FROM dual UNION ALL
    SELECT '07',        6      FROM dual UNION ALL
    SELECT '09',       20      FROM dual UNION ALL   -- please confirm
    SELECT '11',       10      FROM dual UNION ALL
    SELECT '25',       24      FROM dual
),

cycle_cand AS (
    SELECT c.bc, c.e, p.ref_date,
           CASE WHEN c.e = 31
                THEN LAST_DAY(TRUNC(p.ref_date,'MM'))
                ELSE TRUNC(p.ref_date,'MM') + c.e - 1 END                 AS end_this_month,
           CASE WHEN c.e = 31
                THEN LAST_DAY(ADD_MONTHS(TRUNC(p.ref_date,'MM'),-1))
                ELSE ADD_MONTHS(TRUNC(p.ref_date,'MM'),-1) + c.e - 1 END  AS end_prev_month
    FROM   cycles c CROSS JOIN params p
),

cycle_win AS (
    SELECT bc,
           cycle_end,
           CASE WHEN e = 31 THEN TRUNC(cycle_end,'MM')
                ELSE ADD_MONTHS(cycle_end,-1) + 1 END AS cycle_start,
           ref_date
    FROM (
        SELECT bc, e, ref_date,
               CASE WHEN end_this_month <= ref_date
                    THEN end_this_month ELSE end_prev_month END AS cycle_end
        FROM cycle_cand
    )
),

billing AS (
    SELECT MSISDN, BILLCYCLE, PCRF_SERVICE_NAME_EXPECTED
    FROM (
        SELECT b.MSISDN, b.BILLCYCLE, b.PCRF_SERVICE_NAME_EXPECTED,
               b.TYPE, b.STATUS, b.PRICE_GROUP,
               ROW_NUMBER() OVER (PARTITION BY b.MSISDN ORDER BY b.RUN_DATE DESC) AS rn
        FROM   RP7VFEPIDAT.PFM_BSCS_ADSL b
    )
    WHERE rn = 1
    AND   TYPE   = 'POST'
    AND   STATUS = 'ACTIVE'
    AND   NVL(PRICE_GROUP,'x') NOT IN ('TEST PHONES','Pre-paid Test SIM',
                                       'Test Individuals','Dummy PG')
),

catalogue AS (
    SELECT PCRF_SERVICE_NAME,
           MAX(PCRF_QOUTA_VALUE_GB) AS main_quota_gb
    FROM   RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE
    WHERE  PROD_TYPE_NAME IN ('Main Bundle','DSL Package')
    GROUP  BY PCRF_SERVICE_NAME
),

subs AS (
    SELECT b.MSISDN, b.BILLCYCLE,
           w.cycle_start, w.cycle_end, w.ref_date,
           c.main_quota_gb
    FROM   billing b
    JOIN   catalogue c ON c.PCRF_SERVICE_NAME = b.PCRF_SERVICE_NAME_EXPECTED
    JOIN   cycle_win w ON w.bc = b.BILLCYCLE
),

/* ---------- Source 1: PFM_PCRF_CONSUM_FILTERED (snapshot on last day) ---------- */
win_rows AS (
    SELECT s.MSISDN,
           f.RUN_DATE,
           f.QUOTA_CONSUMPTION,
           CASE WHEN UPPER(f.SERVICE_NAME) LIKE '%DOUBLEQUOTA%'
                THEN 'DOUBLE' ELSE 'MAIN' END AS quota_type
    FROM   subs s
    JOIN   RP7VFEPIDAT.PFM_PCRF_CONSUM_FILTERED f
           ON  f.MSISDN   = s.MSISDN
           AND f.RUN_DATE BETWEEN s.cycle_start AND s.cycle_end
    WHERE  UPPER(f.SERVICE_NAME) NOT LIKE '%THROTTLING%'
),

snap AS (
    SELECT MSISDN,
           MAX(RUN_DATE)                                          AS snap_date,
           MAX(CASE WHEN quota_type = 'DOUBLE' THEN 1 ELSE 0 END) AS has_double
    FROM   win_rows
    GROUP  BY MSISDN
),

usage_agg AS (
    SELECT w.MSISDN,
           SUM(w.QUOTA_CONSUMPTION) / 1048576 AS consumed_gb_filtered
    FROM   win_rows w
    JOIN   snap s ON s.MSISDN = w.MSISDN AND s.snap_date = w.RUN_DATE
    GROUP  BY w.MSISDN
),

/* ---------- Source 2: PFM_PCRF_EDRS_CONSUM (sum of sessions across the cycle) ---------- */
edr_agg AS (
    SELECT s.MSISDN,
           SUM(e.QUOTA_USAGE) / 1048576 AS consumed_gb_edr,
           COUNT(*)                     AS edr_sessions
    FROM   subs s
    JOIN   RP7VFEPIDAT.PFM_PCRF_EDRS_CONSUM e
           ON  e.MSISDN = SUBSTR(s.MSISDN, 2)          -- EDR has no leading '2'
           AND e.TIME  >= s.cycle_start
           AND e.TIME   < s.cycle_end + 1
    WHERE  UPPER(e.SERVICE_NAME) NOT LIKE '%THROTTLING%'
    AND   (UPPER(e.SERVICE_NAME) LIKE 'SERV_ADSL%'
        OR UPPER(e.SERVICE_NAME) LIKE '%DOUBLE%QUOTA%')
    GROUP  BY s.MSISDN
),

calc AS (
    SELECT s.MSISDN, s.BILLCYCLE, s.cycle_start, s.cycle_end, s.ref_date,
           s.main_quota_gb,
           sn.snap_date,
           NVL(sn.has_double,0)                            AS has_double,
           s.main_quota_gb * (1 + NVL(sn.has_double,0))    AS allocated_gb,
           NVL(u.consumed_gb_filtered,0)                   AS cons_f,
           NVL(e.consumed_gb_edr,0)                        AS cons_e,
           NVL(e.edr_sessions,0)                           AS edr_sessions
    FROM   subs s
    LEFT JOIN snap       sn ON sn.MSISDN = s.MSISDN
    LEFT JOIN usage_agg  u  ON u.MSISDN  = s.MSISDN
    LEFT JOIN edr_agg    e  ON e.MSISDN  = s.MSISDN
)

SELECT MSISDN,
       BILLCYCLE,
       cycle_start,
       cycle_end,
       TO_CHAR(cycle_end,'YYYY-MM')                                   AS cycle_month,
       TO_CHAR(cycle_end,'FMMonth YYYY','NLS_DATE_LANGUAGE=ENGLISH')  AS cycle_month_name,
       snap_date                                                      AS snapshot_date,
       CASE WHEN snap_date = cycle_end THEN 'Yes' ELSE 'No' END       AS snapshot_is_cycle_end,
       ROUND(main_quota_gb, 2)                                        AS main_quota_gb,
       CASE WHEN has_double = 1 THEN 'Yes' ELSE 'No' END              AS double_quota,
       ROUND(allocated_gb, 2)                                         AS total_allocated_gb,

       -- Source 1: CONSUM_FILTERED
       ROUND(cons_f, 2)                                               AS consumed_gb_filtered,
       ROUND(cons_f / NULLIF(allocated_gb,0) * 100, 2)                AS consumption_pct_filtered,
       ROUND(allocated_gb - cons_f, 2)                                AS variance_gb_filtered,
       CASE WHEN cons_f >= allocated_gb THEN 'Finisher'
            ELSE 'Non-Finisher' END                                   AS finisher_filtered,
       CASE
            WHEN cons_f >= allocated_gb                      THEN 'Finisher'
            WHEN cons_f / NULLIF(allocated_gb,0) < 0.35      THEN 'Cat 1: < 35%'
            WHEN cons_f / NULLIF(allocated_gb,0) < 0.50      THEN 'Cat 2: 35 - <50%'
            WHEN cons_f / NULLIF(allocated_gb,0) < 0.75      THEN 'Cat 3: 50 - <75%'
            ELSE                                                  'Cat 4: 75 - <100%'
       END                                                            AS category_filtered,

       -- Source 2: EDR
       edr_sessions,
       ROUND(cons_e, 2)                                               AS consumed_gb_edr,
       ROUND(cons_e / NULLIF(allocated_gb,0) * 100, 2)                AS consumption_pct_edr,
       ROUND(allocated_gb - cons_e, 2)                                AS variance_gb_edr,
       CASE WHEN cons_e >= allocated_gb THEN 'Finisher'
            ELSE 'Non-Finisher' END                                   AS finisher_edr,
       CASE
            WHEN cons_e >= allocated_gb                      THEN 'Finisher'
            WHEN cons_e / NULLIF(allocated_gb,0) < 0.35      THEN 'Cat 1: < 35%'
            WHEN cons_e / NULLIF(allocated_gb,0) < 0.50      THEN 'Cat 2: 35 - <50%'
            WHEN cons_e / NULLIF(allocated_gb,0) < 0.75      THEN 'Cat 3: 50 - <75%'
            ELSE                                                  'Cat 4: 75 - <100%'
       END                                                            AS category_edr,

       -- Comparison between the two sources
       ROUND(cons_f - cons_e, 2)                                      AS source_diff_gb,
       ROUND((cons_f - cons_e) / NULLIF(cons_e,0) * 100, 2)           AS source_diff_pct
FROM   calc
ORDER  BY BILLCYCLE, MSISDN;
