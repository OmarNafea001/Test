import os
import sys
 
PROJECT_ROOT = os.path.abspath(
    os.path.join(os.path.dirname(__file__), "..")
)
sys.path.insert(0, PROJECT_ROOT)
 
import argparse
from datetime import datetime, timedelta
from config.duckdb_connection import duckdb_connection
from utils.logger import get_logger
from utils.export_to_oracle import prepare_ctl
from utils.export_to_oracle import run_sqlldr
 
 
def export_quota_consumption_cases(run_date: str, con=None):
 
    logger = get_logger(
        flow="quota_consumption_cases",
        log_dir=None,
        run_date=run_date
    )
 
    try:
        if con is None:
            con = duckdb_connection()
 
        logger.info("###########################################################")
        logger.info(f"###### quota_consumption_cases for run_date= {run_date} ######")
        logger.info("###########################################################")
 
        start_time = datetime.now()
 
        prev_date = (datetime.strptime(run_date, "%Y%m%d") - timedelta(days=1)).strftime("%Y%m%d")
        logger.info(f"Counting subscribers for RUN_DATE= {prev_date}")
 
        pcrf_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_{run_date}*"
        pcrf_consumption_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_consum_{run_date}*"
 
 

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_src AS
            SELECT RUN_DATE,MSISDN,CUSTOMER_STATUS
            FROM read_csv_auto('{pcrf_dump}', all_varchar=true, ignore_errors=True)
        """)
 
        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_consumption_src AS
            SELECT RUN_DATE,MSISDN, SERVICE_NAME , EXPECTED_ORG_QUOTA,EXPECTED_TAX_QUOTA ,QUOTA_SUM
            FROM read_csv_auto('{pcrf_consumption_dump}', all_varchar=true, ignore_errors=True)
        """)
 

        con.execute(f"""
            CREATE OR REPLACE TABLE quota_consumption_cases AS
            SELECT RUN_DATE, SERVICE_NAME, "COUNT"
            FROM (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       PCRF_SERVICE_NAME_EXPECTED AS SERVICE_NAME,
                       COUNT(DISTINCT MSISDN) AS "COUNT"
                FROM bscs_adsl_src
                WHERE TRY_CAST(RUN_DATE AS DATE) = strptime('{prev_date}', '%Y%m%d')
                  AND "TYPE" = 'POST'
                  AND TM_CODE_DESC NOT LIKE 'SDSL%'
                  AND TM_CODE_DESC NOT LIKE 'Cloud%'
                GROUP BY 1, 2
            )
            UNION ALL
            SELECT RUN_DATE, SERVICE_NAME, "COUNT"
            FROM (
                SELECT TRY_CAST(RUN_DATE AS DATE) AS RUN_DATE,
                       PCRF_SERVICE_ID_EXPECTED AS SERVICE_NAME,
                       COUNT(DISTINCT MSISDN) AS "COUNT"
                FROM charging_adsl_src
                WHERE TRY_CAST(RUN_DATE AS DATE) = strptime('{prev_date}', '%Y%m%d')
                  AND UPPER(PACKAGE_TYPE) = 'MAIN BUNDLE'
                GROUP BY 1, 2
            )
        """)
 
        count = con.execute("SELECT COUNT(*) FROM quota_consumption_cases").fetchone()[0]
        logger.info(f"Total processed rows: {count}")
 
        if count == 0:
            raise ValueError(f"quota_consumption_cases produced 0 rows for RUN_DATE= {prev_date}; check source files and filters")
 
        logger.info(f"Exporting quota_consumption_cases_{run_date}.csv ..")
        con.execute(f"""COPY quota_consumption_cases TO '/project/PLATFORM_INTEGRITY/exports/quota_consumption_cases_{run_date}.csv' (HEADER, DELIMITER ',')""")
 
        logger.info("Inserting into RP7VFEPIDAT.ADSL_SUBS table ..")
        ctl_file = prepare_ctl("adsl_subs", run_date)
        run_sqlldr("adsl_subs", run_date, ctl_file=ctl_file)
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











	just write it like that you need to just write the query and consider there is quota_sum 
	and just leave catalouge as it is i will replace it with lookup RD_T_DSL_PRODUCT_CATALOGUE leae it like that
	just write th duckdb query for me ok? 
tell me what you'll do before start execution 




	--------------------------------

	another thing explain to me what this script do?  if i want to create a lookup from a cvs file that i have 
	
#!/usr/bin/env python3

import os
import sys
import duckdb

if len(sys.argv) != 3:
    print("Usage: python csv_to_parquet.py <input_csv> <output_parquet>")
    sys.exit(1)

[raidprd@DCHQRAIDAPP02-N utils]$ cat csv_to_parquet.py
#!/usr/bin/env python3

import os
import sys
import duckdb

if len(sys.argv) != 3:
    print("Usage: python csv_to_parquet.py <input_csv> <output_parquet>")
    sys.exit(1)

input_csv = sys.argv[1]
output_parquet = sys.argv[2]

if not os.path.exists(input_csv):
    raise FileNotFoundError(f"Input file not found: {input_csv}")

con = duckdb.connect()

con.execute(f"""
    COPY (
        SELECT *
        FROM read_csv_auto(
            '{input_csv}',
            ignore_errors=true
        )
    )
    TO '{output_parquet}'
    (FORMAT PARQUET);
""")

print(f"Successfully converted:")
print(f"  CSV     : {input_csv}")
print(f"  Parquet : {output_parquet}")

con.close()


	
