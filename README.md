
Truncate table pfm_adsl_msisdn;
Truncate table pfm_adsl_msisdn_dups;
COMMIT;

INSERT INTO PFM_ADSL_MSISDN
SELECT MSISDN,RUN_DATE FROM PFM_BSCS_ADSL_STG;

COMMIT;

INSERT INTO PFM_ADSL_MSISDN
SELECT MSISDN,TO_NUMBER(RUN_DATE) FROM RD_T_IN_ADSL;

COMMIT;

INSERT INTO PFM_ADSL_MSISDN
SELECT MSISDN,RUN_DATE FROM PFM_RADIUS_STG;

COMMIT;

INSERT INTO PFM_ADSL_MSISDN
SELECT MSISDN,RUN_DATE FROM PFM_PCRF_STG;

COMMIT;


INSERT INTO PFM_ADSL_MSISDN_DUPS
SELECT MSISDN,RUN_DATE FROM PFM_ADSL_MSISDN
GROUP BY MSISDN,RUN_DATE
HAVING count(1) > 1;

COMMIT;

DELETE FROM PFM_ADSL_MSISDN
WHERE MSISDN IN (
SELECT MSISDN FROM PFM_ADSL_MSISDN_DUPS);

COMMIT;


INSERT INTO PFM_ADSL_MSISDN
SELECT MSISDN,RUN_DATE FROM PFM_ADSL_MSISDN_DUPS;

COMMIT;


exec dbms_stats.gather_table_stats(ownname=>'RP7VFEPIDAT', tabname=>'PFM_ADSL_MSISDN', estimate_percent => 15, degree => 



i want to creat ethe following as duckdb script that reads from the following files and exports as file as well.

PFM_BSCS_ADSL_STG: name it bscs : /RAIDdata/PI/PI_PRJ/exports//project/PLATFORM_INTEGRITY/exports/bscs_profiles_adsl_{run_date}

2- RD_T_IN_ADSL charging_dump = f"/project/PLATFORM_INTEGRITY/exports/charging_adsl_{run_date}*"

3- PFM_RADIUS_STG  radius= f"/project/PLATFORM_INTEGRITY/exports/radius_{run_date}*"

4- PFM_PCRF_STG  pcrf  = f"/project/PLATFORM_INTEGRITY/exports/pcrf_{run_date}*"


read from the dources above as tables then create file to export with 
/project/PLATFORM_INTEGRITY/exports/pfm_adsl_msisdn_{run_date}

is thi clear tell me what did u understood? 







4, cascade => true, method_opt => 'FOR ALL COLUMNS SIZE AUTO');

EXIT;
