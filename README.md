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


def export_expiry_date_mismatch(run_date: str, con=None):

    logger = get_logger(
        flow="expiry_date_mismatch",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### expiry_date_mismatch for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        pcrf_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_{run_date}*"
        bscs_dump = f"/RAIDdata/PI/PI_PRJ/exports/bscs_profiles_adsl_{run_date}*"

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT RUN_DATE, MSISDN, SERVICE_NAME, VALID_DATE, EXPIRY_DATE, CUSTOMER_STATUS
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE bscs_src AS
            SELECT RUN_DATE, MSISDN, "TYPE", PRICE_GROUP
            FROM read_csv_auto('{bscs_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute("""
            CREATE OR REPLACE TABLE expiry_date_mismatch AS
            SELECT DISTINCT
                   TRY_CAST(pcrf.RUN_DATE AS DATE)                                     AS RUN_DATE,
                   pcrf.MSISDN,
                   STRING_AGG(pcrf.SERVICE_NAME, ',' ORDER BY pcrf.SERVICE_NAME)       AS SERVICE_NAME,
                   CAST(TRY_CAST(pcrf.VALID_DATE  AS TIMESTAMP) AS DATE)               AS VALID_DATE,
                   CAST(TRY_CAST(pcrf.EXPIRY_DATE AS TIMESTAMP) AS DATE)               AS EXPIRY_DATE
            FROM pcrf_src pcrf
            LEFT OUTER JOIN bscs_src billing
                ON pcrf.MSISDN = billing.MSISDN
            WHERE (
                    (billing."TYPE" LIKE '%PRE%'
                     AND pcrf.SERVICE_NAME NOT LIKE '%Th%'
                     AND pcrf.SERVICE_NAME NOT LIKE '%OnHold%')
                 OR (billing."TYPE" LIKE '%POST%'
                     AND pcrf.SERVICE_NAME IN (
                         SELECT DISTINCT PCRF_SERVICE_NAME
                         FROM RD_T_DSL_PRODUCT_CATALOGUE
                         WHERE FREQUENCY LIKE '%OnDemand%'
                           AND (PROD_TYPE_NAME = 'Addon' OR PROD_TYPE_NAME = 'DSL Promotion')
                           AND PCRF_SERVICE_NAME NOT LIKE '%_R%'
                           AND PCRF_TYPE LIKE '%POSTPAID%'
                     ))
                  )
              AND (NULLIF(TRIM(pcrf.EXPIRY_DATE), '') IS NULL
                   OR TRY_CAST(pcrf.EXPIRY_DATE AS TIMESTAMP) < CURRENT_DATE - 1)
              AND pcrf.CUSTOMER_STATUS = '1'
              AND pcrf.SERVICE_NAME NOT LIKE '%Th%'
              AND pcrf.SERVICE_NAME NOT LIKE '%OnHold%'
              AND billing.PRICE_GROUP NOT IN ('TEST PHONES',
                                              'Pre-paid Test SIM',
                                              'Test Individuals',
                                              'Dummy PG')
            GROUP BY pcrf.RUN_DATE, pcrf.MSISDN, pcrf.VALID_DATE, pcrf.EXPIRY_DATE
        """)

        count = con.execute("SELECT COUNT(*) FROM expiry_date_mismatch").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"expiry_date_mismatch produced 0 rows for RUN_DATE= {run_date}; check source files and filters")

        logger.info(f"Exporting expiry_date_mismatch_{run_date}.csv ..")
        con.execute(f"""COPY expiry_date_mismatch TO '/project/PLATFORM_INTEGRITY/exports/expiry_date_mismatch_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into EXPIRY_DATE_MISMATCH table ..")
        ctl_file = prepare_ctl("expiry_date_mismatch", run_date)
        run_sqlldr("expiry_date_mismatch", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("expiry_date_mismatch job finished")
        logger.info(f"expiry_date_mismatch total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"expiry_date_mismatch failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_expiry_date_mismatch(run_date=args.run_date)
