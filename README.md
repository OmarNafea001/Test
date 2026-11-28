this is the query im using the script" import os
import sys

PROJECT_ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__), ".."))
sys.path.insert(0, PROJECT_ROOT)

import argparse
from datetime import datetime
from config.duckdb_connection import duckdb_connection
from utils.logger import get_logger
from utils.export_to_oracle import prepare_ctl
from utils.export_to_oracle import run_sqlldr

EXPORTS = "/project/PLATFORM_INTEGRITY/exports"


def clean(col, alias=None):
    """Empty / blank text -> NULL, like Oracle treats ''."""
    return f"NULLIF(TRIM({col}), '') AS {alias or col}"


def export_adsl_status_recon(run_date: str, con=None):

    logger = get_logger(
        flow="adsl_status_recon",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### adsl_status_recon for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        adsl_dump = f"{EXPORTS}/pfm_adsl_msisdn_{run_date}*"      
        bscs_dump = f"{EXPORTS}/bscs_profiles_adsl_{run_date}*"   
        charging_dump = f"{EXPORTS}/charging_adsl_{run_date}*"    
        pcrf_dump = f"{EXPORTS}/pcrf_filtered_{run_date}*"                 
        radius_dump = f"{EXPORTS}/radius_{run_date}*"             

        # ---------------------------------------------------------------
        # 1. Sources
        # ---------------------------------------------------------------
        con.execute(f"""
            CREATE OR REPLACE TABLE adsl_src AS
            SELECT {clean('MSISDN')}
            FROM read_csv_auto('{adsl_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE billing_src AS
            SELECT {clean('RUN_DATE')}, {clean('MSISDN')}, {clean('"TYPE"', 'TYPE')}, {clean('STATUS')},
                   {clean('AUTO_ACTIVATION_FLAG')},
                   {clean('TM_CODE_DESC')},
                   {clean('PRICE_GROUP')}
            FROM read_csv_auto('{bscs_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE charging_src AS
            SELECT {clean('RUN_DATE')}, {clean('MSISDN')}, {clean('STATUS')},
                   {clean('PACKAGE_TYPE')}
            FROM read_csv_auto('{charging_dump}', all_varchar=true, ignore_errors=True)
            
        """)

        # SELECT DISTINCT * FROM PFM_PCRF_STG_FILTERED (only the columns the query uses)
        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT DISTINCT {clean('RUN_DATE')}, {clean('MSISDN')}, {clean('SERVICE_NAME')}, {clean('CUSTOMER_STATUS')}
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE radius_src AS
            SELECT {clean('RUN_DATE')}, {clean('MSISDN')}, {clean('STATUS')}
            FROM read_csv_auto('{radius_dump}', all_varchar=true, ignore_errors=True)
        """)

        for t in ("adsl_src", "billing_src", "charging_src", "pcrf_src", "radius_src"):
            n = con.execute(f"SELECT COUNT(*) FROM {t}").fetchone()[0]
            logger.info(f"{t}: {n} rows")

        # ---------------------------------------------------------------
        # 2. Status helpers (replace the repeated DECODE / CASE blocks)
        # ---------------------------------------------------------------
        # UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold',Customer_STATUS))
        con.execute("""
            CREATE OR REPLACE TEMP MACRO pcrf_status_decode(s) AS
                UPPER(CASE s WHEN '0' THEN 'DeActive'
                             WHEN '1' THEN 'Active'
                             WHEN '2' THEN 'Suspended'
                             WHEN '3' THEN 'On-Hold'
                             ELSE s END)
        """)

        # UPPER(DECODE(RADIUS.STATUS,'0','DeActive','1','Active','2','Suspended',RADIUS.STATUS))
        con.execute("""
            CREATE OR REPLACE TEMP MACRO radius_status_decode(s) AS
                UPPER(CASE s WHEN '0' THEN 'DeActive'
                             WHEN '1' THEN 'Active'
                             WHEN '2' THEN 'Suspended'
                             ELSE s END)
        """)

        # case when upper(STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG=0 then 'NoAutoActivation' else upper(STATUS) end
        con.execute("""
            CREATE OR REPLACE TEMP MACRO billing_status_decode(status, flag) AS
                CASE WHEN UPPER(status) = 'ACTIVE' AND TRY_CAST(flag AS INTEGER) = 0
                     THEN 'NoAutoActivation'
                     ELSE UPPER(status) END
        """)

        # ---------------------------------------------------------------
        # 3. Recon
        # ---------------------------------------------------------------
        con.execute("""
            CREATE OR REPLACE TABLE adsl_status_recon AS
            WITH
            -- one row per MSISDN with the status from every system
            -- (the inner query of the Oracle INSERT, used twice there)
            recon_base AS (
                SELECT
                    COALESCE(billing.RUN_DATE, charging.RUN_DATE,
                             pcrf.RUN_DATE, radius.RUN_DATE)                     AS run_date,
                    COALESCE(billing.MSISDN, charging.MSISDN, pcrf.MSISDN, radius.MSISDN) AS msisdn,
                    billing."TYPE"                                               AS billing_type,
                    billing.PRICE_GROUP                                          AS price_group,
                    billing_status_decode(billing.STATUS, billing.AUTO_ACTIVATION_FLAG) AS billing_status,
                    UPPER(charging.STATUS)                                       AS charging_status,
                    CASE WHEN pcrf.SERVICE_NAME IS NULL AND pcrf.CUSTOMER_STATUS IS NOT NULL
                         THEN 'NoPCRFService'
                         ELSE pcrf_status_decode(pcrf.CUSTOMER_STATUS)
                    END                                                          AS pcrf_status,
                    radius_status_decode(radius.STATUS)                          AS radius_status,
                    CASE WHEN billing."TYPE" = 'PRE'  THEN UPPER(charging.STATUS)
                         WHEN billing."TYPE" = 'POST' THEN billing_status_decode(billing.STATUS, billing.AUTO_ACTIVATION_FLAG)
                    END                                                          AS adsl_status,
                    CASE WHEN pcrf.SERVICE_NAME IS NULL AND pcrf.CUSTOMER_STATUS IS NOT NULL
                         THEN pcrf_status_decode(pcrf.CUSTOMER_STATUS)
                    END                                                          AS nopcrfservice_status
                FROM adsl_src adsl
                LEFT JOIN billing_src  billing  ON adsl.MSISDN = billing.MSISDN
                LEFT JOIN charging_src charging ON adsl.MSISDN = charging.MSISDN
                LEFT JOIN pcrf_src     pcrf     ON adsl.MSISDN = pcrf.MSISDN
                LEFT JOIN radius_src   radius   ON adsl.MSISDN = radius.MSISDN
                WHERE (charging.PACKAGE_TYPE = 'DSL Status' OR charging.PACKAGE_TYPE IS NULL)
                  AND (radius.STATUS IN ('1', '2', '0', '3') OR radius.STATUS IS NULL)
                  AND (billing.TM_CODE_DESC NOT LIKE '%SDSL%' OR billing.TM_CODE_DESC IS NULL)
                  AND (billing.MSISDN NOT LIKE '%CP%' OR billing.MSISDN IS NULL)
            ),

            -- MSISDNs whose status combination is a known, accepted case -> removed
            -- (the "WHERE msisdn NOT IN (...)" part; no PRICE_GROUP filter, same as Oracle)
            excluded_msisdn AS (
                SELECT msisdn
                FROM recon_base
                WHERE (billing_type IS NULL  AND billing_status IS NULL        AND charging_status IS NULL        AND pcrf_status = 'SUSPENDED' AND radius_status IS NULL)
                   OR (billing_type = 'PRE'  AND billing_status = 'ACTIVE'     AND charging_status = 'ACTIVE'     AND pcrf_status = 'ACTIVE'    AND radius_status = 'ACTIVE')
                   OR (billing_type = 'PRE'  AND billing_status = 'SUSPENDED'  AND charging_status = 'SUSPENDED'  AND pcrf_status = 'SUSPENDED' AND radius_status = 'ACTIVE')
                   OR (billing_type = 'PRE'  AND billing_status = 'SUSPENDED'  AND charging_status IS NULL        AND pcrf_status IS NULL       AND radius_status = 'ACTIVE')
                   OR (billing_type = 'PRE'  AND billing_status = 'ACTIVE'     AND charging_status IS NULL        AND pcrf_status IS NULL       AND radius_status = 'DEACTIVE')
                   OR (billing_type = 'POST' AND billing_status = 'ACTIVE'     AND charging_status IS NULL        AND pcrf_status IS NULL       AND radius_status = 'DEACTIVE')
                   OR (billing_type = 'POST' AND billing_status = 'ACTIVE'     AND charging_status IS NULL        AND pcrf_status = 'ACTIVE'    AND radius_status = 'ACTIVE')
                   OR (billing_type IS NULL  AND billing_status IS NULL        AND charging_status IS NULL        AND pcrf_status IS NULL       AND radius_status = 'DEACTIVE')
                   OR (billing_type IS NULL  AND billing_status IS NULL        AND charging_status IS NULL        AND pcrf_status IS NULL       AND radius_status IS NULL)
            ),

            -- remaining MSISDNs, without test price groups
            included AS (
                SELECT *
                FROM recon_base
                WHERE (price_group NOT IN ('TEST PHONES', 'Pre-paid Test SIM', 'Test Individuals', 'Dummy PG')
                       OR price_group IS NULL)
                  AND msisdn NOT IN (SELECT msisdn FROM excluded_msisdn)
            ),

            -- impact classification (on the raw statuses, before '_N' replacement)
            classified AS (
                SELECT *,
                    CASE WHEN (billing_type IS NULL AND charging_status IS NULL)
                           OR (billing_type = 'PRE' AND charging_status IS NULL)
                         THEN '_N'
                         ELSE adsl_status
                    END AS adsl_status_out,

                    CASE
                        WHEN pcrf_status = 'ACTIVE' AND radius_status = 'ACTIVE'
                             AND (   ((charging_status IN ('SUSPENDED', 'DEACTIVE') OR charging_status IS NULL) AND billing_type = 'PRE')
                                  OR (billing_status IN ('SUSPENDED', 'DEACTIVE', 'NoAutoActivation') AND billing_type = 'POST')
                                  OR (billing_status IS NULL AND charging_status IS NULL AND billing_type IS NULL))
                            THEN 'Revenue Loss'
                        WHEN radius_status = 'ACTIVE' AND adsl_status = 'DEACTIVE'
                             AND (pcrf_status = 'ACTIVE' OR pcrf_status IS NULL)
                            THEN 'Revenue Loss'
                        WHEN radius_status = 'ACTIVE' AND (adsl_status = 'SUSPENDED' OR adsl_status IS NULL)
                             AND pcrf_status IS NULL
                            THEN 'No Impact'
                        WHEN radius_status = 'ACTIVE' AND (adsl_status = 'SUSPENDED' OR adsl_status IS NULL)
                             AND pcrf_status = 'ACTIVE'
                            THEN 'Revenue Loss'
                        WHEN radius_status = 'ACTIVE' AND pcrf_status = 'NoPCRFService'
                             AND (nopcrfservice_status = 'ACTIVE' OR nopcrfservice_status = 'SUSPENDED')
                             AND adsl_status = 'ACTIVE'
                            THEN 'Customer Experience'
                        WHEN radius_status = 'DEACTIVE' AND adsl_status = 'ACTIVE'
                             AND (pcrf_status IN ('SUSPENDED', 'ON-HOLD') OR pcrf_status IS NULL)
                            THEN 'Customer Experience'
                        WHEN radius_status IS NULL AND adsl_status = 'ACTIVE'
                             AND (pcrf_status = 'ON-HOLD' OR pcrf_status IS NULL)
                            THEN 'Customer Experience'
                        WHEN radius_status = 'ACTIVE' AND adsl_status = 'ACTIVE'
                             AND (pcrf_status = 'SUSPENDED' OR pcrf_status IS NULL)
                            THEN 'Customer Experience'
                        WHEN radius_status = 'ACTIVE' AND adsl_status = 'ACTIVE'
                             AND (pcrf_status IN ('SUSPENDED', 'ON-HOLD') OR pcrf_status IS NULL)
                            THEN 'Customer Experience'
                        ELSE 'No Impact'
                    END AS impact_case
                FROM included
            )

            SELECT DISTINCT
                run_date                         AS RUN_DATE,
                msisdn                           AS MSISDN,
                COALESCE(billing_type,    '_N')  AS BILLING_TYPE,
                COALESCE(billing_status,  '_N')  AS BILLING_STATUS,
                COALESCE(charging_status, '_N')  AS CHARGING_STATUS,
                COALESCE(pcrf_status,     '_N')  AS PCRF_STATUS,
                COALESCE(radius_status,   '_N')  AS RADIUS_STATUS,
                adsl_status_out                  AS ADSL_STATUS,
                nopcrfservice_status             AS NOPCRFSERVICE_STATUS,
                impact_case                      AS IMPACT_CASE
            FROM classified
        """)

        count = con.execute("SELECT COUNT(*) FROM adsl_status_recon").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"adsl_status_recon produced 0 rows for RUN_DATE= {run_date}; check source files and filters")

        logger.info(f"Exporting adsl_status_recon_{run_date}.csv ..")
        con.execute(f"""COPY adsl_status_recon TO '{EXPORTS}/adsl_status_recon_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into ADSL_STATUS_RECON table ..")
        ctl_file = prepare_ctl("adsl_status_recon", run_date)
        run_sqlldr("adsl_status_recon", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("adsl_status_recon job finished")
        logger.info(f"adsl_status_recon total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"adsl_status_recon failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_adsl_status_recon(run_date=args.run_date)












this is the real query i want to match the wesults: 

cat /apps/raidprd/raid-prj/scripts/PI_ADSL_STATUS_RECON.sh
#!/bin/bash

file_name=/apps/raidprd/raid-prj/scripts/logs/PI_ADSL_STATUS_RECON.log

$ORACLE_HOME/bin/sqlplus -s >> $file_name <<END
RP7VFEPIDAT/RP7VFEPIDAT@RAIDPRD

INSERT INTO RD_T_ADSL_STATUS_RECON
SELECT DISTINCT RUN_DATE, MSISDN,
CASE WHEN Billing_TYPE IS NULL
THEN '_N'
ELSE Billing_TYPE
END Billing_TYPE,
CASE WHEN BILLING_STATUS IS NULL
THEN '_N'
ELSE BILLING_STATUS
END BILLING_STATUS,
CASE WHEN charging_status IS NULL
THEN '_N'
ELSE charging_status
END charging_status,
CASE WHEN PCRF_STATUS IS NULL
THEN '_N'
ELSE PCRF_STATUS
END PCRF_STATUS,
CASE WHEN RADIUS_STATUS IS NULL
THEN '_N'
ELSE RADIUS_STATUS
END RADIUS_STATUS,
CASE WHEN (BILLING_TYPE IS NULL AND CHARGING_STATUS IS NULL) OR (BILLING_TYPE = 'PRE' AND CHARGING_STATUS IS NULL)
THEN '_N'
ELSE ADSL_STATUS
END ADSL_STATUS, Nopcrfservice_status
,
CASE WHEN (
PCRF_STATUS ='ACTIVE' AND RADIUS_STATUS='ACTIVE' AND (((charging_status in('SUSPENDED','DEACTIVE') or charging_status IS NULL)  AND Billing_TYPE ='PRE')
OR ((BILLING_STATUS in('SUSPENDED','DEACTIVE','NoAutoActivation')) AND Billing_TYPE='POST')
OR (BILLING_STATUS IS NULL AND charging_status IS NULL AND Billing_TYPE IS NULL )))THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='DEACTIVE' AND  (PCRF_STATUS = 'ACTIVE' OR PCRF_STATUS IS NULL) THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND (ADSL_STATUS='SUSPENDED' OR ADSL_STATUS IS NULL ) AND PCRF_STATUS IS NULL THEN 'No Impact'
WHEN RADIUS_STATUS='ACTIVE' AND (ADSL_STATUS='SUSPENDED' OR ADSL_STATUS IS NULL ) AND PCRF_STATUS ='ACTIVE' THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND PCRF_STATUS ='NoPCRFService' AND (Nopcrfservice_status ='ACTIVE' or Nopcrfservice_status ='SUSPENDED') and ADSL_STATUS = 'ACTIVE' THEN 'Customer Experience'
WHEN RADIUS_STATUS='DEACTIVE' AND ADSL_STATUS='ACTIVE'  AND (PCRF_STATUS IN ('SUSPENDED','ON-HOLD') OR PCRF_STATUS IS NULL)  THEN 'Customer Experience'
WHEN RADIUS_STATUS IS NULL AND ADSL_STATUS='ACTIVE' AND  (PCRF_STATUS = 'ON-HOLD' OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='ACTIVE' AND  (PCRF_STATUS = 'SUSPENDED' OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='ACTIVE' AND   (PCRF_STATUS IN ('SUSPENDED','ON-HOLD') OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
ELSE 'No Impact'
END IMPACT_CASE
FROM(
SELECT TO_DATE(COALESCE(Billing.RUN_DATE , TO_NUMBER(Charging.RUN_DATE), PCRF.RUN_DATE, RADIUS.RUN_DATE), 'YYYYMMDD') RUN_DATE, COALESCE(Billing.MSISDN, Charging.MSISDN, PCRF.MSISDN, RADIUS.MSISDN) MSISDN,
Billing.TYPE Billing_TYPE , case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end BILLING_STATUS,
upper(Charging.STATUS) charging_status,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not null then 'NoPCRFService' else UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))end PCRF_STATUS,
UPPER(DECODE(RADIUS.STATUS,'0','DeActive','1','Active','2','Suspended',RADIUS.STATUS)) RADIUS_STATUS,
CASE WHEN Billing.TYPE = 'PRE'
THEN upper(Charging.STATUS)
WHEN Billing.TYPE = 'POST'
THEN (case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end)
END ADSL_STATUS,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not NULL
THEN UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))
END Nopcrfservice_status
from RP7VFEPIDAT.PFM_ADSL_MSISDN ADSL
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG Billing ON ADSL.MSISDN = Billing.MSISDN
left outer join RP7VFEPIDAT.RD_T_IN_ADSL Charging on ADSL.MSISDN =Charging.MSISDN
left outer join (SELECT DISTINCT * FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED) PCRF ON ADSL.MSISDN =PCRF.MSISDN
left outer join RP7VFEPIDAT.PFM_RADIUS_STG RADIUS on ADSL.MSISDN =RADIUS.MSISDN
where (Charging.PACKAGE_TYPE ='DSL Status' or Charging.PACKAGE_TYPE is null)
AND (RADIUS.STATUS IN ('1','2','0','3') OR RADIUS.STATUS IS NULL)
AND (BILLING.TMCODE_DES NOT LIKE '%SDSL%' OR BILLING.TMCODE_DES IS NULL)
AND (BILLING.MSISDN NOT LIKE '%CP%' OR BILLING.msisdn IS NULL)
AND ( BILLING.PRICE_GROUP NOT IN ('TEST PHONES',
                                    'Pre-paid Test SIM',
                                    'Test Individuals',
                                    'Dummy PG') OR BILLING.PRICE_GROUP IS NULL)
)
WHERE msisdn NOT IN (SELECT msisdn FROM (SELECT * FROM
(SELECT TO_DATE(COALESCE(Billing.RUN_DATE , TO_NUMBER(Charging.RUN_DATE), PCRF.RUN_DATE, RADIUS.RUN_DATE), 'YYYYMMDD') RUN_DATE, COALESCE(Billing.MSISDN, Charging.MSISDN, PCRF.MSISDN, RADIUS.MSISDN) MSISDN,
Billing.TYPE Billing_TYPE ,Billing.PRICE_GROUP,
case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end BILLING_STATUS,
upper(Charging.STATUS) charging_status ,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not null then 'NoPCRFService' else UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))end PCRF_STATUS,
UPPER(DECODE(RADIUS.STATUS,'0','DeActive','1','Active','2','Suspended',RADIUS.STATUS)) RADIUS_STATUS,
CASE WHEN Billing.TYPE = 'PRE'
THEN upper(Charging.STATUS)
WHEN Billing.TYPE = 'POST'
THEN (case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end)
END ADSL_STATUS,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not NULL
THEN UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))
END Nopcrfservice_status
from RP7VFEPIDAT.PFM_ADSL_MSISDN ADSL
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG Billing ON ADSL.MSISDN = Billing.MSISDN
left outer join RP7VFEPIDAT.RD_T_IN_ADSL Charging on ADSL.MSISDN =Charging.MSISDN
left outer join (SELECT DISTINCT * FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED) PCRF ON ADSL.MSISDN =PCRF.MSISDN
left outer join RP7VFEPIDAT.PFM_RADIUS_STG RADIUS on ADSL.MSISDN =RADIUS.MSISDN
where (Charging.PACKAGE_TYPE ='DSL Status' or Charging.PACKAGE_TYPE is null)
AND (RADIUS.STATUS IN ('1','2','0','3') OR RADIUS.STATUS IS NULL)
AND (BILLING.TMCODE_DES NOT LIKE '%SDSL%' OR BILLING.TMCODE_DES IS NULL)
AND (BILLING.MSISDN NOT LIKE '%CP%' OR BILLING.msisdn IS NULL)
)
WHERE
( Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status = 'SUSPENDED' AND radius_status IS null )
OR ( Billing_type = 'PRE' AND billing_status= 'ACTIVE'  AND CHARGING_status = 'ACTIVE'  AND pcrf_status = 'ACTIVE'  AND radius_status = 'ACTIVE'  )
OR (Billing_type = 'PRE' AND billing_status= 'SUSPENDED'  AND CHARGING_status = 'SUSPENDED'  AND pcrf_status = 'SUSPENDED' AND radius_status = 'ACTIVE' )
OR (Billing_type = 'PRE' AND billing_status= 'SUSPENDED'  AND CHARGING_status IS null  AND pcrf_status IS null AND radius_status = 'ACTIVE' )
OR (Billing_type = 'PRE' AND billing_status= 'ACTIVE'  AND CHARGING_status IS NULL AND pcrf_status  IS NULL AND radius_status = 'DEACTIVE' )
OR (Billing_type = 'POST' AND billing_status= 'ACTIVE'  AND CHARGING_status IS NULL AND pcrf_status IS NULL AND radius_status = 'DEACTIVE' )
OR (Billing_type = 'POST' AND billing_status= 'ACTIVE' AND CHARGING_status IS NULL AND pcrf_status = 'ACTIVE' AND radius_status = 'ACTIVE')
OR (Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status IS null AND radius_status = 'DEACTIVE')
OR (Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status IS null AND radius_status is null)
));
COMMIT;
Exit;
END
-------------------------------------------------------------------------------------------------

#Run ALL STATUS MISMATCH Vs BSCS BASE Percentage Script
 cd /apps/raidprd/raid-prj/scripts
   ./ADSL_Status_Mismatch_Percentage.sh $1

if [ $? -ne 0 ]; then
    echo "ADSL_Status_Mismatch_Percentage.sh failed"
    exit 1
fi

exit 0

[V25OIbrahim15@DCHQRAIDAPP02-N ~]$ cd /apps/raidprd/raid-prj/scripts
[V25OIbrahim15@DCHQRAIDAPP02-N scripts]$ cat ADSL_Status_Mismatch_Percentage.sh

#!/bin/bash

file_name=/apps/raidprd/raid-prj/scripts/logs/ADSL_Status_Mismatch_Percentage.log
$ORACLE_HOME/bin/sqlplus -s > $file_name <<END
RP7VFEPIDAT/RP7VFEPIDAT@RAIDPRD
INSERT INTO RP7VFEPIDAT.ADSL_ALL_STATUS_MIS
WITH Total_Base AS
(
        SELECT to_date(run_date,'YYYYMMDD') Run_date, COUNT(DISTINCT MSISDN ) Total_Base
        FROM RP7VFEPIDAT.PFM_BSCS_ADSL
        WHERE run_date = $1
        GROUP BY run_date
),
All_Status_Mis AS
(
        SELECT RUN_DATE ,COUNT(DISTINCT MSISDN) Status_Mis_count
        FROM RP7VFEPIDAT.RD_T_ADSL_STATUS_RECON
        WHERE RUN_DATE = to_date($1, 'YYYYMMDD')
        AND IMPACT_CASES <> 'No Impact'
        GROUP BY run_date
)
SELECT B.RUN_DATE ,S.Status_Mis_count, B.Total_Base ,
(B.Total_Base - S.Status_Mis_count) AS VARIANCE,
ROUND((S.Status_Mis_count / B.Total_Base ) *100 ,2 ) AS Percentage
FROM Total_Base B JOIN All_Status_Mis S
        ON B.RUN_DATE = S.Run_Date;
COMMIT;
EXIT;




but there is some difference as some msisdn is repeated as i will give you samples 

tell me why and try to correct it for me :

2026-11-25 00:00:00.000	20132390022	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390022	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390053	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390053	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390056	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390056	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390064	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390064	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390170	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390170	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390176	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390176	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390215	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132390215	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390265	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390265	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390367	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390367	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390421	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390421	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390458	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390458	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390474	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390474	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390813	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390813	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132390825	PRE	ACTIVE	SUSPENDED	NoPCRFService	DEACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132390825	PRE	ACTIVE	_N	NoPCRFService	DEACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132390925	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132390925	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132390973	PRE	NoAutoActivation	ACTIVE	ACTIVE	ACTIVE	ACTIVE		No Impact
2026-11-25 00:00:00.000	20132390973	PRE	NoAutoActivation	_N	ACTIVE	ACTIVE	_N		Revenue Loss
2026-11-25 00:00:00.000	20132390974	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132390974	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391150	PRE	ACTIVE	SUSPENDED	SUSPENDED	DEACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132391150	PRE	ACTIVE	_N	SUSPENDED	DEACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391415	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391415	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132391432	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132391432	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391536	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132391536	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391574	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391574	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132391797	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132391797	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392312	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132392312	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392487	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132392487	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392615	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392615	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132392742	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392742	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132392955	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132392955	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393008	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393008	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393111	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393111	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393137	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393137	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393149	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393149	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393171	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393171	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393234	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393234	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393353	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393353	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393364	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393364	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393388	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393388	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393423	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393423	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393581	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393581	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393691	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393691	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393764	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393764	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393813	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132393813	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132393849	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393849	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132393976	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132393976	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394040	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394040	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394062	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394062	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394064	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132394064	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394169	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394169	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394228	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394228	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394257	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394257	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394284	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394284	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394484	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394484	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394515	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394515	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394580	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394580	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394586	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394586	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394590	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132394590	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132394596	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132394596	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132394691	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394691	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394704	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132394704	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394724	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394724	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394729	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394729	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394779	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394779	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394834	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394834	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394880	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394880	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394914	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394914	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394935	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132394935	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394970	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132394970	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132396086	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132396086	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132396306	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132396306	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397135	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397135	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397146	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397146	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397417	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397417	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397472	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397472	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397494	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397494	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397583	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132397583	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397825	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132397825	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132398127	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398127	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132398317	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132398317	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398338	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398338	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132398458	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398458	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132398599	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398599	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132398611	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132398611	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399027	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399027	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399057	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399057	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399233	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399233	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399301	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399301	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399637	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399637	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132399709	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132399709	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399952	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132399952	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400158	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400158	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400218	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400218	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400383	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400383	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400485	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400485	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400522	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400522	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400544	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400544	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400576	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400576	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400615	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400615	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400646	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400646	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400783	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400783	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132400805	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400805	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400818	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400818	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400840	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132400840	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400927	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132400927	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132401043	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401043	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401046	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401046	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401099	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401099	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401133	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401133	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401139	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401139	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401166	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401166	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401292	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401292	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401349	PRE	NoAutoActivation	_N	ACTIVE	ACTIVE	_N		Revenue Loss
2026-11-25 00:00:00.000	20132401349	PRE	NoAutoActivation	ACTIVE	ACTIVE	ACTIVE	ACTIVE		No Impact
2026-11-25 00:00:00.000	20132401410	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401410	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401602	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401602	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401613	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401613	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401619	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401619	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401623	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401623	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401630	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401630	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401691	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401691	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401704	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401704	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401724	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401724	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401797	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401797	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401813	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401813	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401820	PRE	NoAutoActivation	_N	ACTIVE	ACTIVE	_N		Revenue Loss
2026-11-25 00:00:00.000	20132401820	PRE	NoAutoActivation	ACTIVE	ACTIVE	ACTIVE	ACTIVE		No Impact
2026-11-25 00:00:00.000	20132401834	PRE	NoAutoActivation	_N	ACTIVE	ACTIVE	_N		Revenue Loss
2026-11-25 00:00:00.000	20132401834	PRE	NoAutoActivation	ACTIVE	ACTIVE	ACTIVE	ACTIVE		No Impact
2026-11-25 00:00:00.000	20132401835	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132401835	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401844	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132401844	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132402009	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402009	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402021	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402021	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402043	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402043	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402053	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132402053	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132402055	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402055	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402067	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402067	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402127	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402127	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402168	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402168	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402248	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402248	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402282	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402282	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402316	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402316	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402325	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402325	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402334	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402334	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402346	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402346	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402348	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402348	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402361	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132402361	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132402384	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402384	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402385	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402385	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402478	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402478	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132402514	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402514	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402617	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402617	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402657	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402657	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402745	PRE	NoAutoActivation	ACTIVE	ACTIVE	ACTIVE	ACTIVE		No Impact
2026-11-25 00:00:00.000	20132402745	PRE	NoAutoActivation	_N	ACTIVE	ACTIVE	_N		Revenue Loss
2026-11-25 00:00:00.000	20132402786	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402786	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132402828	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132402828	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403041	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403041	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403056	PRE	ACTIVE	_N	NoPCRFService	ACTIVE	_N	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132403056	PRE	ACTIVE	SUSPENDED	NoPCRFService	ACTIVE	SUSPENDED	SUSPENDED	No Impact
2026-11-25 00:00:00.000	20132403061	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403061	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403326	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403326	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403392	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403392	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403463	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403463	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403480	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403480	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403481	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403481	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403497	PRE	NoAutoActivation	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403497	PRE	NoAutoActivation	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132403515	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403515	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403538	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403538	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403610	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403610	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403704	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403704	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403709	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403709	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403755	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403755	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403786	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403786	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403794	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403794	PRE	ACTIVE	ACTIVE	SUSPENDED	ACTIVE	ACTIVE		Customer Experience
2026-11-25 00:00:00.000	20132403853	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403853	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132403996	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132403996	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404001	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404001	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404011	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404011	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404019	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404019	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404041	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404041	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404090	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404090	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404282	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404282	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404357	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404357	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404389	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404389	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404529	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404529	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404584	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact
2026-11-25 00:00:00.000	20132404584	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404632	PRE	ACTIVE	_N	SUSPENDED	ACTIVE	_N		No Impact
2026-11-25 00:00:00.000	20132404632	PRE	ACTIVE	SUSPENDED	SUSPENDED	ACTIVE	SUSPENDED		No Impact




ok now i will take a sample give you from charging table that oracle select from and from the files that script read from ok 




this is the table : 2026-11-25 00:00:00.000	20132390022	777000000	_N	_N	_N				_N
2026-11-25 00:00:00.000	20132390022	777000002	_N	_N	_N				_N
2026-11-25 00:00:00.000	20132390022	777010042	_N	_N	_N				_N
2026-11-25 00:00:00.000	20132390022	777000007	_N	_N	_N				_N
2026-11-25 00:00:00.000	20132390022	777000081	_N	DSL Main Bundle 30MB200GB Sh/WatchItContentPr	Serv_ADSL_Mc_HS_30M_180G				Main Bundle
2026-11-25 00:00:00.000	20132390022	777001004	SUSPENDED	DSL SUSPENDED Non-Payment	_N				DSL Status







thi is the script 


cat charging_adsl_20261125.csv | grep -i 20132390022
2026-11-25,20132390022,777000081,,DSL Main Bundle 30MB200GB Sh/WatchItContentPr,Serv_ADSL_Mc_HS_30M_180G,Main Bundle
2026-11-25,20132390022,777000007,,,,
2026-11-25,20132390022,777001004,SUSPENDED,DSL SUSPENDED Non-Payment,_N,DSL Status
2026-11-25,20132390022,777000002,,,,
2026-11-25,20132390022,777010042,,,,
2026-11-25,20132390022,777000000,,,,
