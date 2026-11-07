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


def export_quota_consum_count(run_date: str, con=None):

    logger = get_logger(
        flow="quota_consum_count",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### quota_consum_count for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        pcrf_dump = f"/project/PLATFORM_INTEGRITY/exports/pcrf_{run_date}*"
        quota_consumption_cases_dump = f"/project/PLATFORM_INTEGRITY/exports/quota_consumption_cases_{run_date}*"

        # PFM_PCRF_FILTERED
        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT RUN_DATE, MSISDN
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)

        # RD_T_PCRF_QUOTA_CONSUM_CASES (output CSV of the quota_consumption_cases job)
        con.execute(f"""
            CREATE OR REPLACE TABLE quota_consumption_cases_src AS
            SELECT RUN_DATE, MSISDN, PCRF_TYPE
            FROM read_csv_auto('{quota_consumption_cases_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute("""
            CREATE OR REPLACE TABLE quota_consum_count AS
            WITH PCRF_Total_Base AS
            (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       COUNT(DISTINCT MSISDN)     AS PCRF_TOTAL_BASE
                FROM pcrf_src
                GROUP BY 1
            ),
            QUOTA_CONSUM AS
            (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       CASE
                           WHEN PCRF_TYPE IN ('PREPAID', 'PREPAID-POSTPAID')
                               THEN 'PREPAID'
                           ELSE PCRF_TYPE
                       END AS PCRF_TYPE_GROUP,
                       COUNT(DISTINCT MSISDN) AS QUOTA_CONSUM_COUNT
                FROM quota_consumption_cases_src
                GROUP BY 1, 2
            )
            SELECT
                B.RUN_DATE,
                SUM(Q.QUOTA_CONSUM_COUNT)                                              AS QUOTA_CONSUM_COUNT,
                B.PCRF_TOTAL_BASE,
                (B.PCRF_TOTAL_BASE - SUM(Q.QUOTA_CONSUM_COUNT))                        AS VARIANCE,
                ROUND((SUM(Q.QUOTA_CONSUM_COUNT) / NULLIF(B.PCRF_TOTAL_BASE, 0)) * 100, 3) AS PERCENTAGE,
                Q.PCRF_TYPE_GROUP                                                      AS PCRF_TYPE
            FROM PCRF_Total_Base B
            JOIN QUOTA_CONSUM Q
                ON B.RUN_DATE = Q.RUN_DATE
            GROUP BY
                B.RUN_DATE,
                B.PCRF_TOTAL_BASE,
                Q.PCRF_TYPE_GROUP
        """)

        count = con.execute("SELECT COUNT(*) FROM quota_consum_count").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"quota_consum_count produced 0 rows for RUN_DATE= {run_date}; check source files and filters")

        logger.info(f"Exporting quota_consum_count_{run_date}.csv ..")
        con.execute(f"""
            COPY (SELECT * FROM quota_consum_count ORDER BY PCRF_TYPE)
            TO '/project/PLATFORM_INTEGRITY/exports/quota_consum_count_{run_date}.csv' (HEADER, DELIMITER ',')
        """)

        logger.info("Inserting into RP7VFEPIDAT.QUOTA_CONSUM_COUNT table ..")
        ctl_file = prepare_ctl("quota_consum_count", run_date)
        run_sqlldr("quota_consum_count", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("quota_consum_count job finished")
        logger.info(f"quota_consum_count total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"quota_consum_count failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_quota_consum_count(run_date=args.run_date)
