import os
import sys

PROJECT_ROOT = os.path.abspath(
    os.path.join(os.path.dirname(__file__), "..")
)
sys.path.insert(0, PROJECT_ROOT)

import duckdb
import argparse
from datetime import datetime
from config.duckdb_connection import duckdb_connection
from utils.logger import get_logger
from utils.export_to_oracle import prepare_ctl
from utils.export_to_oracle import run_sqlldr



def export_free_traffic(run_date: str, con=None):

    logger = get_logger(
        flow="free_traffic_mismatch",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()

        logger.info("###########################################################")
        logger.info(f"###### FREE TRAFFIC MISMATCH for run_date= {run_date} ######")
        logger.info("###########################################################")

        start_time = datetime.now()
        pcrf_edrs_dump = f"/RAIDdata/PI/PI_PRJ/exports/pcrf_edrs_*.csv"

        con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_edrs_src AS
            SELECT EDR_DATE, QUOTA_USAGE, SERVICE_NAME, QUOTA_NAME
            FROM read_csv_auto('{pcrf_edrs_dump}', ignore_errors=True)
        """)

        con.execute(f"""
            CREATE OR REPLACE TABLE free_traffic_mismatch AS
            WITH Total_base AS (
                SELECT TRY_CAST(EDR_DATE AS DATE) AS EDR_DATE,
                       SUM(QUOTA_USAGE) AS Usage,
                       'TOTAL' AS TYPE
                FROM pcrf_edrs_src
                WHERE TRY_CAST(EDR_DATE AS DATE) = strptime('{run_date}', '%Y%m%d')
                GROUP BY TRY_CAST(EDR_DATE AS DATE)
            ),

            PRE AS (
                SELECT TRY_CAST(EDR_DATE AS DATE) AS EDR_DATE,
                       SUM(QUOTA_USAGE) AS Usage,
                       'PRE' AS TYPE
                FROM pcrf_edrs_src
                WHERE TRY_CAST(EDR_DATE AS DATE) = strptime('{run_date}', '%Y%m%d')
                  AND SERVICE_NAME IN (
                      'Serv_DSL_Generic_Promo',
                      'Serv_DSL_Seasonal_Gift',
                      'Serv_DSL_Double_Quota_Pre',
                      'Serv_DSL_Compensation_Gift'
                  )
                GROUP BY TRY_CAST(EDR_DATE AS DATE)
            ),

            POST AS (
                SELECT TRY_CAST(EDR_DATE AS DATE) AS EDR_DATE,
                       SUM(QUOTA_USAGE) AS Usage,
                       'POST' AS TYPE
                FROM pcrf_edrs_src
                WHERE TRY_CAST(EDR_DATE AS DATE) = strptime('{run_date}', '%Y%m%d')
                  AND (
                      QUOTA_NAME IN ('DSL_Compensation_Gift')
                      OR SERVICE_NAME IN (
                          'Serv_DSL_DoubleQuota_300GB_POST',
                          'Serv_DSL_DoubleQuota_600GB_POST',
                          'Serv_DSL_DoubleQuota_200GB_POST',
                          'Serv_DSL_GC_Gift_3000G'
                      )
                  )
                GROUP BY TRY_CAST(EDR_DATE AS DATE)
            )

            SELECT
                t.EDR_DATE AS "TIME",
                ROUND(t.Usage / 1024 / 1024 / 1024, 2) AS TOTAL_TB,
                ROUND(pre.Usage / 1024 / 1024 / 1024, 2) AS PRE_FREE_TRAFFIC_TB,
                100 - ROUND((t.Usage - pre.Usage) / NULLIF(t.Usage, 0), 4) * 100 AS PRE_PERCENTAGE,
                ROUND(post.Usage / 1024 / 1024 / 1024, 2) AS POST_FREE_TRAFFIC_TB,
                100 - ROUND((t.Usage - post.Usage) / NULLIF(t.Usage, 0), 4) * 100 AS POST_PERCENTAGE
            FROM Total_base t
            JOIN PRE pre ON t.EDR_DATE = pre.EDR_DATE
            JOIN POST post ON t.EDR_DATE = post.EDR_DATE
        """)

        count = con.execute("SELECT COUNT(*) FROM free_traffic_mismatch").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        logger.info(f"Exporting free_traffic_mismatch{run_date}.csv ..")
        con.execute(f"""COPY free_traffic_mismatch TO '/project/PLATFORM_INTEGRITY/exports/free_traffic_mismatch_{run_date}.csv' (HEADER, DELIMITER ',')""")
       
        logger.info(f"Inserting into PI.FREE_TRAFFIC table ..")
        ctl_file = prepare_ctl("free_traffic_mismatch", run_date)
        run_sqlldr("free_traffic_mismatch",run_date, ctl_file=ctl_file)
        logger.info(f"Insertion done")
        
        
        end_time = datetime.now() 
        logger.info(f"free_traffic_mismatch job finished") 
        logger.info(f"free_traffic_mismatch total duration: {end_time - start_time}") 
    except Exception as e: 
        logger.exception(f"free_traffic_mismatch failed for run_date= {run_date} | error= {str(e)}") 
        raise 

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_free_traffic(run_date=args.run_date)







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
 
 
def export_adsl_subs(run_date: str, con=None):
 
    logger = get_logger(
        flow="adsl_subs",
        log_dir=None,
        run_date=run_date
    )
 
    try:
        if con is None:
            con = duckdb_connection()
 
        logger.info("###########################################################")
        logger.info(f"###### ADSL SUBS for run_date= {run_date} ######")
        logger.info("###########################################################")
 
        start_time = datetime.now()
 
        prev_date = (datetime.strptime(run_date, "%Y%m%d") - timedelta(days=1)).strftime("%Y%m%d")
        logger.info(f"Counting subscribers for RUN_DATE= {prev_date}")
 
        bscs_dump = f"/project/PLATFORM_INTEGRITY/exports/bscs_profiles_adsl_{prev_date}*"
        charging_dump = f"/project/PLATFORM_INTEGRITY/exports/charging_adsl_{prev_date}*"
 
 

        con.execute(f"""
            CREATE OR REPLACE TABLE bscs_adsl_src AS
            SELECT RUN_DATE, PCRF_SERVICE_NAME_EXPECTED, MSISDN, TYPE, TM_CODE_DESC
            FROM read_csv_auto('{bscs_dump}', all_varchar=true, ignore_errors=True)
        """)
 
        con.execute(f"""
            CREATE OR REPLACE TABLE charging_adsl_src AS
            SELECT RUN_DATE, PCRF_SERVICE_NAME as PCRF_SERVICE_ID_EXPECTED , MSISDN, PROD_TYPE_NAME as PACKAGE_TYPE
            FROM read_csv_auto('{charging_dump}', all_varchar=true, ignore_errors=True)
        """)
 

        con.execute(f"""
            CREATE OR REPLACE TABLE adsl_subs AS
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
 
        count = con.execute("SELECT COUNT(*) FROM adsl_subs").fetchone()[0]
        logger.info(f"Total processed rows: {count}")
 
        if count == 0:
            raise ValueError(f"adsl_subs produced 0 rows for RUN_DATE= {prev_date}; check source files and filters")
 
        logger.info(f"Exporting adsl_subs_{run_date}.csv ..")
        con.execute(f"""COPY adsl_subs TO '/project/PLATFORM_INTEGRITY/exports/adsl_subs_{run_date}.csv' (HEADER, DELIMITER ',')""")
 
        logger.info("Inserting into RP7VFEPIDAT.ADSL_SUBS table ..")
        ctl_file = prepare_ctl("adsl_subs", run_date)
        run_sqlldr("adsl_subs", run_date, ctl_file=ctl_file)
        logger.info("Insertion done")
 
        end_time = datetime.now()
        logger.info("adsl_subs job finished")
        logger.info(f"adsl_subs total duration: {end_time - start_time}")
    except Exception as e:
        logger.exception(f"adsl_subs failed for run_date= {run_date} | error= {str(e)}")
        raise
 
 
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    export_adsl_subs(run_date=args.run_date)





    based on the two above queries i want to create another one for the following :
SELECT *
FROM
(
    SELECT DISTINCT
           TO_DATE(consum.run_date,'YYYYMMDD') AS run_date,
           consum.msisdn,
           consum.service_name,
           ROUND(consum.quota_sum/1024/1024,1) AS quota_sum_gb,
           expected_org_quota,
           expected_tax_quota,

           CASE
                WHEN ROUND(quota_sum/1024/1024,1) = expected_org_quota
                THEN 0

                WHEN ROUND(quota_sum/1024/1024,1) =
                     (expected_org_quota + expected_tax_quota)
                     AND pcrf_type LIKE '%PREPAID%'
                     AND prod_type_name = 'Main Bundle'
                THEN 0

                WHEN ROUND(quota_sum/1024/1024,1) <>
                     (expected_org_quota + expected_tax_quota)
                     AND pcrf_type LIKE '%PREPAID%'
                     AND prod_type_name = 'Main Bundle'
                THEN ROUND(quota_sum/1024/1024,1)
                     - (expected_org_quota + expected_tax_quota)

                ELSE ROUND(quota_sum/1024/1024,1)
                     - expected_org_quota
           END AS variance,

           CASE
                WHEN pcrf_type IS NULL THEN 'NA'
                ELSE pcrf_type
           END AS pcrf_type,

           CASE
                WHEN expected_org_quota = 0
                 AND expected_tax_quota = 0
                 AND expected_throt_quota = 0
                THEN 'EXCLUDED'
                ELSE 'INCLUDED'
           END AS lookup_inclusion

    FROM RP7VFEPIDAT.PFM_PCRF_STG_CONSUM_FILTERED consum

    LEFT OUTER JOIN
    (
        SELECT pcrf_service_name,
               pcrf_type,
               prod_type_name
        FROM RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE
    ) catalogue
        ON consum.service_name = catalogue.pcrf_service_name

    WHERE consum.msisdn IN
    (
        SELECT msisdn
        FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED
        WHERE customer_status = 1
    )
) t

WHERE variance <> 0
  AND service_name NOT LIKE '%Thrott%'
  AND service_name NOT LIKE '%Double%'
  AND service_name NOT LIKE '%Gift%'
  AND service_name NOT LIKE '%Promo%'
  AND quota_sum_gb NOT IN
      (
          expected_org_quota * 2,
          expected_org_quota * 3,
          expected_org_quota * 4,
          expected_org_quota * 5
      );
	  
	  
	  
	  
	  
tables to select from  : PFM_PCRF_STG_FILTERED , PFM_PCRF_STG_CONSUM_FILTERED , RD_T_DSL_PRODUCT_CATALOGUE
	  
	  



i will you the dumps name we're using? 

PFM_PCRF_STG_CONSUM_FILTERED  : path and columns 
/RAIDdata/PI/PI_PRJ/exports/pcrf_edrs_consum_20261020.csv
cat pcrf_edrs_consum_20261020.csv | head
RUN_DATE,MSISDN,SERVICE_NAME,QUOTA_NAME,QUOTA_STATUS,QUOTA_CONSUMPTION,QUOTA_USAGE,QUOTA_VALUE,ACTUAL_QUOTA_USAGE,SERVICE_EXTENDED_ATTRIBUTE1,EXPECTED_ORG_QUOTA,EXPECTED_TAX_QUOTA
2026-10-20,20862161873,,ADSL_HS_30M_300G_Main,Normal,105263090,2507000,2507000,2507000,4440,,
2026-10-20,20653630716,,ADSL_HS_30M_140G_Main,Normal,25090627,20828,20828,20828,4440,,


PFM_PCRF_STG_FILTERED
/RAIDdata/PI/PI_PRJ/exports/pcrf_20261020.csv
cat pcrf_20261020.csv | head
MSISDN,BILLCYCLE,SERVICE_NAME,SVC_PKG_NAME,VALID_DATE,EXPIRY_DATE,CUSTOMER_STATUS,RUN_DATE,THROT,PROD_TYPE_NAME,RN
20643204513,18,Serv_ADSL_Mc_HS_30M_600G,,2026-10-17 11:13:26,2026-11-16 00:00:00,1,2026-10-20,0,Main Bundle,1
20643204580,18,Serv_DSL_Throttling,,2026-10-14 00:47:36,,1,2026-10-20,0,,1
20643204618,18,Serv_ADSL_Mc_HS_30M_180G,,2026-10-02 18:19:51,2026-11-01 00:00:00,1,2026-10-20,0,Main Bundle,1
20643205110,18,Serv_DSL_Throttling,,2026-09-23 03:51:18,,1,2026-10-20,0,,1



RD_T_DSL_PRODUCT_CATALOGUE  this is lookup i want will give you a query when this lookup is used i want you to explain to me how this lookup work?


con.execute(f"""
            CREATE OR REPLACE TABLE pcrf_edrs_consum AS
            SELECT
            TRY_CAST('{run_date_formatted}' AS DATE) AS RUN_DATE,
            TRY_CAST('2' || column02 AS VARCHAR) AS MSISDN,
            TRY_CAST(column13 AS VARCHAR) AS SERVICE_NAME,
            TRY_CAST(column14 AS VARCHAR) AS QUOTA_NAME,
            TRY_CAST(column15 AS VARCHAR) AS QUOTA_STATUS,
            TRY_CAST(column16 AS DOUBLE) AS QUOTA_CONSUMPTION,
            TRY_CAST(column17 AS DOUBLE) AS QUOTA_USAGE,
            TRY_CAST(column17 AS VARCHAR) AS QUOTA_VALUE,
            TRY_CAST(column30 AS VARCHAR) AS ACTUAL_QUOTA_USAGE,
            TRY_CAST(column35 AS VARCHAR) AS SERVICE_EXTENDED_ATTRIBUTE1,
            TRY_CAST(lkp15.PCRF_QOUTA_VALUE_GB AS VARCHAR) AS EXPECTED_ORG_QUOTA,
            TRY_CAST(lkp16.PCRF_TAXATION_VALUE_GB AS VARCHAR) AS EXPECTED_TAX_QUOTA
        FROM read_csv_auto('{file_pattern}', ignore_errors=True,delim=',',union_by_name = true,auto_detect = true) pcrf_edrs_consum
        LEFT JOIN lkp15
            ON TRY_CAST(pcrf_edrs_consum.column13 AS VARCHAR) = TRY_CAST(lkp15.PCRF_QOUTA_VALUE_GB AS VARCHAR)
        LEFT JOIN lkp16
            ON TRY_CAST(pcrf_edrs_consum.column13 AS VARCHAR) = TRY_CAST(lkp16.PCRF_TAXATION_VALUE_GB AS VARCHAR)
        LEFT JOIN lkp20233
            ON TRY_CAST(pcrf_edrs_consum.column14 AS VARCHAR) = TRY_CAST(lkp20233.PCRF_SERVICE_NAME AS VARCHAR)
""")


result = con.query(f"""
WITH edrs AS (
    SELECT
        RUN_DATE,
        MSISDN,
        SUM(QUOTA_USAGE) AS QUOTA_USAGE_SUM
        FROM pcrf_edrs_consum
    GROUP BY
        RUN_DATE,
        MSISDN
),
procera_agg_2 AS (
    SELECT
        RUN_DATE,
        LANDLINE,
        SUM(TOTAL_BYTES) AS total_bytes
    FROM procera_agg
    GROUP BY
        RUN_DATE,
        LANDLINE
),

comparison AS (
SELECT
        COALESCE(procera_agg_2.RUN_DATE, edrs.RUN_DATE - INTERVAL 1 DAY) AS RUN_DATE,
        edrs.MSISDN AS EDRS_LANDLINE,
        procera_agg_2.LANDLINE AS PROCERA_LANDLINE,
        edrs.QUOTA_USAGE_SUM AS EDRS_CONSUMPTION,
        procera_agg_2.total_bytes / 1024.0 AS PROCERA_CONSUMPTION,
        COALESCE(procera_agg_2.total_bytes / 1024.0, 0) - COALESCE(edrs.QUOTA_USAGE_SUM, 0) AS VARIANCE
  lkp15   this is the catlouge lookup i want to understand how it works?

  import duckdb
from pathlib import Path

def load_lookups(con, directory="/project/PLATFORM_INTEGRITY/lookups"):
    lookup_tables = {}
    for parquet_file in Path(directory).glob("*.parquet"):
        table_name = parquet_file.stem  # filename without .parquet
        rel = con.query(f"SELECT * FROM read_parquet('{parquet_file}')")
        con.register(table_name, rel)
        lookup_tables[table_name] = rel
    return lookup_tables

#create duckdb connection with below config
def duckdb_connection():
    con = duckdb.connect()
    con.execute("SET memory_limit='120GB'")
    con.execute("SET threads=16")
    con.execute("SET preserve_insertion_order=false")
    con.execute("PRAGMA temp_directory='/project/PLATFORM_INTEGRITY/duckdb_tmp'")
    con.execute("PRAGMA max_temp_directory_size='800GB'")
    con.execute("LOAD excel")
    load_lookups(con, "/project/PLATFORM_INTEGRITY/lookups")
    return con
this is the connection it imports all the lookup i wan to understand i will give you samples of the lookup 

 lkp1019.parquet
-rw-rw-r-- 1 raidprd raidprd  11K OCT 19 2026 13:29 export_rateplans.parquet
-rw-rw-r-- 1 raidprd raidprd 3.6K OCT 19 2026 15:23 lkp16.parquet
-rw-rw-r-- 1 raidprd raidprd 3.7K OCT 19 2026 15:24 lkp15.parquet
-rw-rw-r-- 1 raidprd raidprd 1.9K OCT 21 2026 13:05 LK_TMCODE_VS_SERVICE_NAME.parquet
drwxrwxr-x 2 raidprd raidprd 4.0K OCT 21 2026 13:05 csv_lookups
-rw-rw-r-- 1 raidprd raidprd 5.0K NOV 07 2026 11:30 lkp20233.parquet
-rw-r--r-- 1 raidprd raidprd  11K NOV 07 2026 11:30 rateplan_mapping.parquet
-rw-r--r-- 1 raidprd raidprd  15K NOV 07 2026 11:30 services_mapping.parquet
-rw-r--r-- 1 raidprd raidprd 9.6K NOV 07 2026 11:30 PC_DSL.parquet
-rw-r--r-- 1 raidprd raidprd  15K NOV 07 2026 11:30 LK_SERVICE_CLASSES.parquet
drwxrwxr-x 2 raidprd raidprd  20K NOV 07 2026 11:30 old_lookups

ARROW:schema▒/////4gCAAAQAAAAAAAKAA4ABgAFAAgACgAAAAABBAAQAAAAAAAKAAwAAAAEAAgACgAAANABAAAEAAAAAQAAAAwAAAAIAAwABAAIAAgAAACoAQAABAAAAJsBAAB7ImluZGV4X2NvbHVtbnMiOiBbXSwgImNvbHVtbl9pbmRleGVzIjogW10sICJjb2x1bW5zIjogW3sibmFtZSI6ICJQQ1JGX1NFUlZJQ0VfTkFNRSIsICJmaWVsZF9uYW1lIjogIlBDUkZfU0VSVklDRV9OQU1FIiwgInBhbmRhc190eXBlIjogInVuaWNvZGUiLCAibnVtcHlfdHlwZSI6ICJvYmplY3QiLCAibWV0YWRhdGEiOiBudWxsfSwgeyJuYW1lIjogIlBDUkZfUU9VVEFfVkFMVUVfR0IiLCAiZmllbGRfbmFtZSI6ICJQQ1JGX1FPVVRBX1ZBTFVFX0dCIiwgInBhbmRhc190eXBlIjogInVuaWNvZGUiLCAibnVtcHlfdHlwZSI6ICJvYmplY3QiLCAibWV0YWRhdGEiOiBudWxsfV0sICJjcmVhdG9yIjogeyJsaWJyYXJ5IjogInB5YXJyb3ciLCAidmVyc2lvbiI6ICIyMC4wLjAifSwgInBhbmRhc192ZXJzaW9uIjogIjIuMy4yIn0ABgAAAHBhbmRhcwAAAgAAAEwAAAAEAAAAzP///wAAAQUQAAAAJAAAAAQAAAAAAAAAEwAAAFBDUkZfUU9VVEFfVkFMVUVfR0IAvP///xAAFAAIAAYABwAMAAAAEAAQAAAAAAABBRAAAAAoAAAABAAAAAAAAAARAAAAUENSRl9TRVJWSUNFX05BTUUAAAAEAAQABAAAAAAAAAA= parquet-cpp-arrow version 20.0.0,▒PAR1[V25OIbrahim15@DCHQRAIDAPP02-N lookups]$ xterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256cm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256colorxterm-256color


  this is the lookup when i can 




  











    
