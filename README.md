import os
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
        pcrf_dump = f"{EXPORTS}/pcrf_{run_date}*"                 
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
                   {clean('PROD_TYPE_NAME')}
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
                WHERE (charging.PROD_TYPE_NAME = 'DSL Status' OR charging.PROD_TYPE_NAME IS NULL)
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
