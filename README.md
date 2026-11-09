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


def export_pfm_adsl_msisdn(run_date: str, con=None):

    logger = get_logger(
        flow="pfm_adsl_msisdn",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### pfm_adsl_msisdn for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()

        bscs_dump = f"/project/PLATFORM_INTEGRITY/exports/bscs_profiles_adsl_{run_date}*"   
        charging_dump = f"/project/PLATFORM_INTEGRITY/exports/charging_adsl_{run_date}*"    
        radius_dump = f"/project/PLATFORM_INTEGRITY/exports/radius_{run_date}*"             
        pcrf_dump = f"/project/PLATFORM_INTEGRITY/exports/pcrf_{run_date}*"                 

        con.execute(f"""
            CREATE OR REPLACE TABLE bscs_src AS
            SELECT MSISDN, RUN_DATE
            FROM read_csv_auto('{bscs_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE charging_src AS
            SELECT MSISDN, RUN_DATE
            FROM read_csv_auto('{charging_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE radius_src AS
            SELECT MSISDN, RUN_DATE
            FROM read_csv_auto('{radius_dump}', all_varchar=true, ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT MSISDN, RUN_DATE
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)

        for t in ("bscs_src", "charging_src", "radius_src", "pcrf_src"):
            n = con.execute(f"SELECT COUNT(*) FROM {t}").fetchone()[0]
            logger.info(f"{t}: {n} rows")

        con.execute("""
            CREATE OR REPLACE TABLE pfm_adsl_msisdn AS
            SELECT DISTINCT MSISDN, RUN_DATE
            FROM (
                SELECT MSISDN, RUN_DATE FROM bscs_src
                UNION ALL
                SELECT MSISDN, RUN_DATE FROM charging_src
                UNION ALL
                SELECT MSISDN, RUN_DATE FROM radius_src
                UNION ALL
                SELECT MSISDN, RUN_DATE FROM pcrf_src
            )
        """)

        count = con.execute("SELECT COUNT(*) FROM pfm_adsl_msisdn").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            raise ValueError(f"pfm_adsl_msisdn produced 0 rows for RUN_DATE= {run_date}; check source files")

        logger.info(f"Exporting pfm_adsl_msisdn_{run_date}.csv ..")
        con.execute(f"""COPY pfm_adsl_msisdn TO '/project/PLATFORM_INTEGRITY/exports/pfm_adsl_msisdn_{run_date}.csv' (HEADER, DELIMITER ',')""")

        logger.info("Inserting into PFM_ADSL_MSISDN table ..")
        ctl_file = prepare_ctl("pfm_adsl_msisdn", run_date)
        run_sqlldr("pfm_adsl_msisdn", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")

        end_time = datetime.now()
        logger.info("pfm_adsl_msisdn job finished")
        logger.info(f"pfm_adsl_msisdn total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"pfm_adsl_msisdn failed for run_date= {run_date} | error= {str(e)}")
        raise


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_pfm_adsl_msisdn(run_date=args.run_date)
