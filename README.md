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


def export_expiry_date_mismatch_count(run_date: str, con=None):

    logger = get_logger(
        flow="expiry_date_mismatch_count",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### expiry_date_mismatch_count for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        # PFM_BSCS_ADSL
        bscs_dump = f"/RAIDdata/PI/PI_PRJ/exports/bscs_profiles_adsl_{run_date}*"
        # RD_T_PCRF_EXPIRY_DATE_MISMATCH (output CSV of the expiry_date_mismatch job)
        expiry_date_mismatch_dump = f"/RAIDdata/PI/PI_PRJ/exports/expiry_date_mismatch_{run_date}*"

        con.execute(f"""
            CREATE OR REPLACE TABLE bscs_src AS
            SELECT RUN_DATE, MSISDN
            FROM read_csv_auto('{bscs_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE expiry_date_mismatch_src AS
            SELECT RUN_DATE, MSISDN
            FROM read_csv_auto('{expiry_date_mismatch_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute("""
            CREATE OR REPLACE TABLE expiry_date_mismatch_count AS
            WITH Total_Base AS
            (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       COUNT(*)                   AS TOTAL_BASE
                FROM bscs_src
                GROUP BY 1
            ),
            All_Status_Mis AS
            (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       COUNT(*)                   AS STATUS_MIS_COUNT
                FROM expiry_date_mismatch_src
                GROUP BY 1
            )
            SELECT B.RUN_DATE,
                   S.STATUS_MIS_COUNT,
                   B.TOTAL_BASE,
                   (B.TOTAL_BASE - S.STATUS_MIS_COUNT)                          AS VARIANCE,
                   ROUND((S.STATUS_MIS_COUNT / NULLIF(B.TOTAL_BASE, 0)) * 100, 2) AS PERCENTAGE
            FROM Total_Base B
            JOIN All_Status_Mis S
                ON B.RUN_DATE = S.RUN_DATE
        """)

        count = con.execute("SELECT COUNT(*) FROM expiry_date_mismatch_count").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"expiry_date_mismatch_count produced 0 rows for RUN_DATE= {run_date}; check source files and filters")

        logger.info(f"Exporting expiry_date_mismatch_count_{run_date}.csv ..")
        con.execute(f"""COPY expiry_date_mismatch_count TO '/project/PLATFORM_INTEGRITY/exports/expiry_date_mismatch_count_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into EXPIRY_DATE_MISMATCH_COUNT table ..")
        ctl_file = prepare_ctl("expiry_date_mismatch_count", run_date)
        run_sqlldr("expiry_date_mismatch_count", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("expiry_date_mismatch_count job finished")
        logger.info(f"expiry_date_mismatch_count total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"expiry_date_mismatch_count failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_expiry_date_mismatch_count(run_date=args.run_date)
