import os
import sys

PROJECT_ROOT = os.path.abspath(
    os.path.join(os.path.dirname(__file__), "..")
)
sys.path.insert(0, PROJECT_ROOT)

import duckdb
import argparse
import glob
from datetime import datetime
from config.duckdb_connection import duckdb_connection
from utils.logger import get_logger
from utils.send_mail import send_mail
from utils.export_to_oracle import prepare_ctl
from utils.export_to_oracle import run_sqlldr
from utils.dump_vs_database import validate_loaded_data



def load_radius(run_date: str, con=None):
     
    start_time = datetime.now() 
    logger = get_logger(
        flow="radius",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()
            

        run_date_formatted = f"{run_date[:4]}-{run_date[4:6]}-{run_date[6:]}"
        csv_path = f"/project/PLATFORM_INTEGRITY/exports/radius_{run_date}.csv"
        file_path = (f"/RAIDdata/PI/RADIUS/out/RadiusReport{run_date}*.gz*")

        con.execute(f"""
CREATE OR REPLACE TABLE radius_data AS
 SELECT
    '2' || TRIM(BOTH '"' FROM TRY_CAST(ADSL_USER_NAME AS VARCHAR)) AS MSISDN,

    CASE
        WHEN TRIM(BOTH '"' FROM TRY_CAST(ADSL_STATUS AS VARCHAR)) = 'Active'
            THEN '1'
        WHEN TRIM(BOTH '"' FROM TRY_CAST(ADSL_STATUS AS VARCHAR)) = 'Deactive'
            THEN '0'
        ELSE TRIM(BOTH '"' FROM TRY_CAST(ADSL_STATUS AS VARCHAR))
    END AS STATUS,

    CASE
        WHEN CREATION_DATE IS NULL
             OR TRIM(BOTH '"' FROM TRY_CAST(CREATION_DATE AS VARCHAR)) = ''
        THEN ''
        ELSE TRIM(BOTH '"' FROM TRY_CAST(CREATION_DATE AS VARCHAR))
    END AS CREATION_DATE,

    TRY_CAST('{run_date_formatted}' AS DATE) AS RUN_DATE

FROM read_csv_auto(
    '{file_path}',
    delim=',',
    header=TRUE,
    ignore_errors=TRUE,
    quote='"'
);
""")

        count = con.execute("SELECT COUNT(*) FROM radius_data").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            logger.info("No radius data processed!")
            raise
        
        con.execute(f"""
        COPY (
            SELECT RUN_DATE, MSISDN , STATUS , CREATION_DATE
            FROM radius_data 
        )
        TO '{csv_path}'
        (DELIMITER ',', HEADER TRUE, QUOTE '"', ESCAPE '"');
    """)
    
        # logger.info(f"Inserting into PI.radius table ..")
        # ctl_file = prepare_ctl("radius", run_date)
        # run_sqlldr("radius",run_date, ctl_file=ctl_file)
        # logger.info(f"Insertion done")
        # logger.info(f"Validating inserted data ..")

        # validate_loaded_data(
        # flow_name="radius",
        # run_date=run_date,
        # expected_rows=count,
        # logger=logger)

        # logger.info("Inserted data validated successfully.")

        # end_time = datetime.now()
        # logger.info("radius finished")
        # logger.info(f"Total duration: {end_time - start_time}")
        # send_mail(
        #             subject="RADIUS Flow Finished",
        #             content=f"""
        #         Run Date : {run_date}
        #         Rows      : {count}
        #         DB Validation : Passed
        #         Duration  : {end_time - start_time}""")

        return con.table("radius_data")

    except Exception as e:
        logger.exception( f"RADIUS failed for run_date= {run_date} | error= {str(e)}")
        # send_mail(
        # subject="RADIUS Flow Failed",
        # content=f"""
        #     Run Date : {run_date}
        #     Error:
        #     {str(e)}
        #     """)
        raise

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    load_radius(run_date=args.run_date)


    remove the comments #  and then add adjust the indentation of the script 
