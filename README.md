import os
import sys

PROJECT_ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__), ".."))
sys.path.insert(0, PROJECT_ROOT)

import argparse
from datetime import datetime, timedelta
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

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT RUN_DATE,MSISDN,CUSTOMER_STATUS
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_consumption_src AS
            SELECT RUN_DATE,MSISDN, SERVICE_NAME , EXPECTED_ORG_QUOTA,EXPECTED_TAX_QUOTA ,QUOTA_SUM
            FROM read_csv_auto('{quota_consumption_cases_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE quota_consumption_cases AS
            WITH consum AS (
                SELECT TRY_CAST(RUN_DATE AS DATE)            AS run_date,
                       MSISDN                                AS msisdn,
                       SERVICE_NAME                          AS service_name,
                       TRY_CAST(QUOTA_SUM AS DOUBLE)         AS quota_sum,
                       TRY_CAST(EXPECTED_ORG_QUOTA AS DOUBLE) AS expected_org_quota,
                       TRY_CAST(EXPECTED_TAX_QUOTA AS DOUBLE) AS expected_tax_quota
                FROM pcrf_consumption_src
                WHERE TRY_CAST(RUN_DATE AS DATE) = CAST(strptime('{run_date}', '%Y%m%d') AS DATE)
            )
            SELECT *
            FROM (
                SELECT DISTINCT
                       consum.run_date,
                       consum.msisdn,
                       consum.service_name,
                       ROUND(consum.quota_sum / 1024 / 1024, 1) AS quota_sum_gb,
                       consum.expected_org_quota,
                       consum.expected_tax_quota,

                       CASE
                            WHEN ROUND(consum.quota_sum / 1024 / 1024, 1) = consum.expected_org_quota
                            THEN 0

                            WHEN ROUND(consum.quota_sum / 1024 / 1024, 1) =
                                 (consum.expected_org_quota + consum.expected_tax_quota)
                                 AND catalogue.pcrf_type LIKE '%PREPAID%'
                                 AND catalogue.prod_type_name = 'Main Bundle'
                            THEN 0

                            WHEN ROUND(consum.quota_sum / 1024 / 1024, 1) <>
                                 (consum.expected_org_quota + consum.expected_tax_quota)
                                 AND catalogue.pcrf_type LIKE '%PREPAID%'
                                 AND catalogue.prod_type_name = 'Main Bundle'
                            THEN ROUND(consum.quota_sum / 1024 / 1024, 1)
                                 - (consum.expected_org_quota + consum.expected_tax_quota)

                            ELSE ROUND(consum.quota_sum / 1024 / 1024, 1)
                                 - consum.expected_org_quota
                       END AS variance,

                       CASE
                            WHEN catalogue.pcrf_type IS NULL THEN 'NA'
                            ELSE catalogue.pcrf_type
                       END AS pcrf_type,

                       CASE
                            WHEN consum.expected_org_quota = 0
                             AND consum.expected_tax_quota = 0
                            THEN 'EXCLUDED'
                            ELSE 'INCLUDED'
                       END AS lookup_inclusion

                FROM consum

                LEFT OUTER JOIN
                (
                    SELECT pcrf_service_name,
                           pcrf_type,
                           prod_type_name
                    FROM catalouge_test
                ) catalogue
                    ON consum.service_name = catalogue.pcrf_service_name

                WHERE consum.msisdn IN
                (
                    SELECT MSISDN
                    FROM pcrf_src
                    WHERE CUSTOMER_STATUS = '1'
                )
            ) t
            WHERE variance <> 0
              AND service_name NOT LIKE '%Thrott%'
              AND service_name NOT LIKE '%Double%'
              AND service_name NOT LIKE '%Gift%'
              AND service_name NOT LIKE '%Promo%'
              AND quota_sum_gb NOT IN (expected_org_quota * 2,
                                       expected_org_quota * 3,
                                       expected_org_quota * 4,
                                       expected_org_quota * 5)
        """)

        count = con.execute("SELECT COUNT(*) FROM quota_consumption_cases").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"quota_consumption_cases produced 0 rows for RUN_DATE= {prev_date}; check source files and filters")

        logger.info(f"Exporting quota_consumption_cases_{run_date}.csv ..")
        con.execute(f"""COPY quota_consumption_cases TO '/project/PLATFORM_INTEGRITY/exports/quota_consumption_cases_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into RP7VFEPIDAT.QUOTA_CONSUM_CASES table ..")
        ctl_file = prepare_ctl("quota_consumption_cases", run_date)
        run_sqlldr("quota_consumption_cases", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("quota_consumption_cases job finished")
        logger.info(f"quota_consumption_cases total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"quota_consumption_cases failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_quota_consumption_cases(run_date=args.run_date)
