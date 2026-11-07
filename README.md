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


i want to add coulmn to this from 

SELECT QUOTA_USAGE/1024/1024 FROM RP7VFEPIDAT.PFM_PCRF_EDRS_STG_CONSUM 
WHERE TIME = date'2026-10-17'
AND (MSISDN, SERVICE_NAME) IN (SELECT SUBSTR(MSISDN,2),SERVICE_NAME   FROM RP7VFEPIDAT.RD_T_PCRF_EXPIRY_DATE_MISMATCH
WHERE RUN_DATE = date'2026-10-17')


to add the total consumption on that day.  consumption

and create a script duckdb as before name it expiry_date_mismatch

to select from dumps: 

pcrf_edrs_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_edrs*.csv"  to read from all the files dont put date
pcrf_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_{run_date}*"
bscs_dump = f"/RAIDdata/PI/PI_PRJ/exports/bscs_profiles_adsl_{run_date}*"

i will tell you the coumn i will need?

from pcrf_dump  RUN_DATE, MSISDN, SERVICE_NAME, VALID_DATE, EXPIRY_DATE, CUSTOMER_STATUS

bscs_dump  RUN_DATE ,  MSISDN, TYPE, PRICE_GROUP
pcrf_edrs_dump : QUOTA_USAGE ,EDR_DATE  

EDR_DATE format 2026-10-13 






