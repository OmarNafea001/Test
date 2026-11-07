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


def export_postpaid_quota_consumption_cases(run_date: str, con=None):

    logger = get_logger(
        flow="postpaid_quota_consumption_cases",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### postpaid_quota_consumption_cases for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        # output CSV of the quota_consumption_cases job
        quota_consumption_cases_dump = f"/project/PLATFORM_INTEGRITY/exports/quota_consumption_cases_{run_date}*"

        con.execute(f"""
            CREATE OR REPLACE TABLE quota_consumption_cases_src AS
            SELECT RUN_DATE, MSISDN, SERVICE_NAME, QUOTA_SUM_GB,
                   EXPECTED_ORG_QUOTA, EXPECTED_TAX_QUOTA, VARIANCE,
                   PCRF_TYPE, LOOKUP_INCLUSION
            FROM read_csv_auto('{quota_consumption_cases_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute("""
            CREATE OR REPLACE TABLE postpaid_quota_consumption_cases AS
            SELECT TRY_CAST(RUN_DATE AS DATE)            AS RUN_DATE,
                   MSISDN,
                   SERVICE_NAME,
                   TRY_CAST(QUOTA_SUM_GB AS DOUBLE)       AS QUOTA_SUM_GB,
                   TRY_CAST(EXPECTED_ORG_QUOTA AS DOUBLE) AS EXPECTED_ORG_QUOTA,
                   TRY_CAST(EXPECTED_TAX_QUOTA AS DOUBLE) AS EXPECTED_TAX_QUOTA,
                   TRY_CAST(VARIANCE AS DOUBLE)           AS VARIANCE,
                   PCRF_TYPE,
                   LOOKUP_INCLUSION
            FROM quota_consumption_cases_src
            WHERE PCRF_TYPE = 'POSTPAID'
        """)

        count = con.execute("SELECT COUNT(*) FROM postpaid_quota_consumption_cases").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"postpaid_quota_consumption_cases produced 0 rows for RUN_DATE= {run_date}; check source files and filters")

        logger.info(f"Exporting postpaid_quota_consumption_cases_{run_date}.csv ..")
        con.execute(f"""COPY postpaid_quota_consumption_cases TO '/project/PLATFORM_INTEGRITY/exports/postpaid_quota_consumption_cases_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into RP7VFEPIDAT.POSTPAID_QUOTA_CONSUMPTION_CASES table ..")
        ctl_file = prepare_ctl("postpaid_quota_consumption_cases", run_date)
        run_sqlldr("postpaid_quota_consumption_cases", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("postpaid_quota_consumption_cases job finished")
        logger.info(f"postpaid_quota_consumption_cases total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"postpaid_quota_consumption_cases failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_postpaid_quota_consumption_cases(run_date=args.run_date)
