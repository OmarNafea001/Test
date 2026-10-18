#!/bin/bash

$ORACLE_HOME/bin/sqlplus -s <<END
RP7VFEPIDAT/RP7VFEPIDAT@$GLOBAL_DB_ORACLE_SERVICE
INSERT INTO rd_t_pcrf_expiry_date_mismatch
SELECT  /*+ PARALLEL(16) */
DISTINCT to_date(pcrf.RUN_DATE, 'YYYYMMDD') RUN_DATE,
pcrf.msisdn , LISTAGG(service_name, ',') WITHIN GROUP (ORDER BY service_name)SERVICE_NAME,
Trunc(TO_DATE(VALID_DATE , 'YYYYMMDDHH24MISS'))VALID_DATE,
Trunc(TO_DATE(EXPIRY_DATE, 'YYYYMMDDHH24MISS'))EXPIRY_DATE
FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED pcrf
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG billing
ON pcrf.MSISDN = billing.msisdn
WHERE ((billing."TYPE" like '%PRE%' AND service_name NOT LIKE '%Th%' AND service_name NOT LIKE '%OnHold%')
OR (billing."TYPE" like '%POST%' AND service_name IN (SELECT DISTINCT pcrf_service_name
FROM RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE WHERE (FREQUENCY LIKE '%OnDemand%'
AND (prod_Type_name = 'Addon' OR prod_Type_name = 'DSL Promotion') AND PCRF_SERVICE_NAME NOT LIKE '%_R%' AND PCRF_TYPE LIKE '%POSTPAID%'))))
AND ( EXPIRY_DATE IS NULL OR TO_DATE(EXPIRY_DATE , 'YYYYMMDDHH24MISS') < TRUNC(SYSDATE)-1)
AND PCRF.CUSTOMER_STATUS = '1'
AND service_name NOT LIKE '%Th%' AND service_name NOT LIKE '%OnHold%'
AND BILLING.PRICE_GROUP NOT IN ('TEST PHONES',
                                                                'Pre-paid Test SIM',
                                                                'Test Individuals',
                                                                'Dummy PG'
                                                           )
GROUP BY pcrf.RUN_DATE , pcrf.MSISDN,VALID_DATE,EXPIRY_DATE;
COMMIT;
exit;
END
