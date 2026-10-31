if i want to bulid a logic of the following with duckdb python script i will gove you the xml logic and i will give you two applied pythonduck db to build like them i will give you all the needed input and if u have any question just ask me ok 



this is the index and the name of the fields i want to index 1   rdrCONTRACT_ID
10      rdrMSISDN
11     rdrSN_CODE
2        rdrIMSI
7         rdrTM_CODE
8      rdrSUB_STATUS
9        rdrSUB_TYPE
14     rdrBILL_CYCLE
3      rdrRUNNING_NUMBER
16     TMCODE_DES
19    rdrACTIVE_SERVICE_LI
20     rdrPRICE_GROUP


and this is the files samples : file_path = (
            f"/RAIDdata/PI/BSCS/raw/"
            f"CONTRACT_PROFILE_{run_date}.csv.gz"
        )  

        file: CONTRACT_PROFILE_20261031.csv.gz



zcat   CONTRACT_PROFILE_20261031.csv.gz | head
1|602022018806478|4|30|06/11/1998 11:18:06 am|03/01/1999 12:00:00 am|4|d|POSTPAID|201005999611|1|06/11/1998 11:18:06 am|03/01/1999 06:08:14 pm|01|N|Business Ass. Engineering||||
1|602022014539329|0|55|06/11/1998 11:18:06 am|03/01/1999 12:00:00 am|4|d|POSTPAID|201005999611|1|06/11/1998 11:18:06 am|03/01/1999 06:08:14 pm|01|N|Business Ass. Engineering||||
100|602022099900952|0|1|07/11/1998 04:49:13 pm||5|a|POSTPAID|201005999851|1|07/11/1998 04:49:13 pm||01|N|Business Ass. Roaming Test SIM||100|1,3,4,5,7,8,9,10,11,12,13,14,15,16,17,79,81,141,142,144,145,150,202,270,413,1318|ROAMING
100000002|602022276725424|0|2|02/01/2019 10:36:46 pm||530|a|PREPAID|201067479599|1|11/07/2012 06:47:50 am||03|N|Hakawy Rate Plan||100000002|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667,3229,3589,3689|PRE PAY
100000009|602023359024602|0|2|03/12/2022 11:50:51 am||530|a|PREPAID|201015443950|1|11/07/2012 06:47:47 am||03|N|Hakawy Rate Plan||100000009|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667,3229,3589|PRE PAY
100000033|602022111255139|0|1|11/07/2012 10:48:41 pm||530|a|PREPAID|201016035933|1|11/07/2012 10:48:41 pm||03|N|Hakawy Rate Plan||100000033|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667|PRE PAY
100000050|602023404959954|0|6|27/09/2025 05:45:04 pm||530|a|PREPAID|201061907066|1|11/07/2012 10:48:36 pm||03|N|Hakawy Rate Plan||100000050|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667,3229,3589|PRE PAY
100000057|602023123348146|0|2|30/03/2020 03:04:51 pm||530|a|PREPAID|201061908008|1|11/07/2012 06:48:23 am||03|N|Hakawy Rate Plan||100000057|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667,3229,3589,3689|PRE PAY
10000007|602023420263200|0|7|19/04/2026 06:34:20 pm|19/08/2026 01:05:12 am|530|d|PREPAID|201009196007|1|20/10/2005 08:06:36 pm|19/08/2026 01:05:12 am|03|N|Hakawy Rate Plan||||
100000095|602023396186943|0|5|10/04/2025 09:29:19 pm||530|a|PREPAID|201015446449|1|11/07/2012 10:49:05 pm||03|N|Hakawy Rate Plan||100000095|1,3,4,5,7,8,9,10,11,12,30,73,79,81,126,141,142,143,144,145,270,413,2667,3229,3589|PRE PAY



this is the xml in steps is being used i want to convert it to duck db: 

<And _comment="Initialize Variables" completeeval="false">
	<RuleBlock _comment="Constants">
		<Assign append="False" constant="20" target="MSISDN_PREFIX"/>
		<Assign append="False" constant="1" target="COMMONCONSTANTS_ACTIVE"/>
		<Assign append="False" constant="0" target="COMMONCONSTANTS_INACTIVE"/>
		<Assign append="False" constant="2" target="COMMONCONSTANTS_SUSPENDED"/>
	</RuleBlock>
	<RuleBlock _comment="Initializations">
		<Iif>
			<IsEmpty field="rdrCONTRACT_ID"/>
			<Assign append="False" constant="" target="rdrCONTRACT_ID"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrMSISDN"/>
			<Assign append="False" constant="" target="rdrMSISDN"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrSUB_STATUS"/>
			<Assign append="False" constant="" target="rdrSUB_STATUS"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrIMSI"/>
			<Assign append="False" constant="" target="rdrIMSI"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrTM_CODE"/>
			<Assign append="False" constant="" target="rdrTM_CODE"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrSUB_TYPE"/>
			<Assign append="False" constant="" target="rdrSUB_TYPE"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrSN_CODE"/>
			<Assign append="False" constant="" target="rdrSN_CODE"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrBILL_CYCLE"/>
			<Assign append="False" constant="" target="rdrBILL_CYCLE"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrRUNNING_NUMBER"/>
			<Assign append="False" constant="" target="rdrRUNNING_NUMBER"/>
		</Iif>
		<Iif>
			<IsEmpty field="TMCODE_DES"/>
			<Assign append="False" constant="" target="TMCODE_DES"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrACTIVE_SERVICE_LIST"/>
			<Assign append="False" constant="" target="rdrACTIVE_SERVICE_LIST"/>
		</Iif>
		<Iif>
			<IsEmpty field="rdrPRICE_GROUP"/>
			<Assign append="False" constant="" target="rdrPRICE_GROUP"/>
		</Iif>
		<Trim source="rdrCONTRACT_ID"/>
		<Trim source="rdrMSISDN"/>
		<Trim source="rdrSUB_STATUS"/>
		<Trim source="rdrIMSI"/>
		<Trim source="rdrTM_CODE"/>
		<Trim source="rdrSN_CODE"/>
		<Trim source="rdrSUB_TYPE"/>
		<Trim source="rdrBILL_CYCLE"/>
		<Trim source="rdrRUNNING_NUMBER"/>
		<Trim source="TMCODE_DES"/>
		<Trim source="rdrACTIVE_SERVICE_LIST"/>
		<Trim source="rdrPRICE_GROUP"/>
		<True/>
	</RuleBlock>
	<Length source="rdrMSISDN" target="LEN_rdrMSISDN"/>
</And>


------------------------------------------------------

<RuleBlock _comment="Enrich Variables">
	<RuleBlock _comment="Assign to final variables">
		<Assign append="False" source="rdrCONTRACT_ID" target="CONTRACT_ID"/>
		<RightTrim append="False" char="`" ignorecase="True" source="rdrMSISDN" target="rdrMSISDN"/>
		<Assign append="False" source="rdrMSISDN" target="MSISDN"/>
		<Assign append="False" source="rdrSUB_STATUS" target="SUB_STATUS"/>
		<Assign append="False" source="rdrIMSI" target="IMSI"/>
		<Assign append="False" source="rdrTM_CODE" target="TM_CODE"/>
		<Assign append="False" source="rdrSN_CODE" target="SN_CODE"/>
		<Assign append="False" source="rdrBILL_CYCLE" target="BILL_CYCLE"/>
		<Assign append="False" source="rdrRUNNING_NUMBER" target="RUNNING_NUMBER"/>
		<Assign append="False" source="rdrACTIVE_SERVICE_LIST" target="ACTIVE_SERVICE_LIST"/>
		<Assign append="False" constant="0" target="AUTO_ACTIVATION_FLAG"/>
		<Assign append="False" source="rdrPRICE_GROUP" target="PRICE_GROUP"/>
		<Length source="MSISDN" target="LEN_MSISDN"/>
		<RuleBlock _comment="Common RegEx">
			<Assign append="False" source="MSISDN_PREFIX" target="REGEX_MSISDN_PREFIX"/>
			<Assign append="True" constant="*" target="REGEX_MSISDN_PREFIX"/>
		</RuleBlock>
		<Assign append="False" constant="_N" target="PCRF_SERVICE_NAME"/>
	</RuleBlock>
	<RuleBlock _comment="STATUS">
		<Iif>
			<Equals constant="A" ignorecase="True" target="SUB_STATUS"/>
			<Assign append="False" constant="ACTIVE" target="STATUS"/>
			<Iif>
				<Equals constant="S" ignorecase="True" target="SUB_STATUS"/>
				<Assign append="False" constant="SUSPENDED" target="STATUS"/>
				<Assign append="False" constant="DEACTIVE" target="STATUS"/>
			</Iif>
		</Iif>
	</RuleBlock>
	<Case _comment="Subscriber Type">
		<Equals constant="PREPAID" ignorecase="False" target="rdrSUB_TYPE"/>
		<Assign append="False" constant="PRE" target="TYPE"/>
		<Equals constant="POSTPAID" ignorecase="False" target="rdrSUB_TYPE"/>
		<Assign append="False" constant="POST" target="TYPE"/>
		<Equals constant="C2BCONTROL" ignorecase="False" target="rdrSUB_TYPE"/>
		<Assign append="False" constant="CTRL" target="TYPE"/>
		<Equals constant="CONTROL" ignorecase="False" target="rdrSUB_TYPE"/>
		<Assign append="False" constant="CTRL" target="TYPE"/>
		<Assign append="False" source="rdrSUB_TYPE" target="TYPE"/>
	</Case>
	<RuleBlock _comment="Extract Date">
		<RegExpPart field="FILENAME" pattern=".*_.*_([0-9]*).*" target="EXTRACT_DATE"/>
		<Assign append="False" source="EXTRACT_DATE" target="RUN_DATE"/>
		<Assign append="True" constant="000000" target="EXTRACT_DATE"/>
	</RuleBlock>
	<RuleBlock _comment="ADSL_TYPE">
		<Iif>
			<Equals constant="03" ignorecase="True" target="BILL_CYCLE"/>
			<Assign append="False" constant="PRE" target="ADSL_TYPE"/>
			<Assign append="False" constant="POST" target="ADSL_TYPE"/>
		</Iif>
	</RuleBlock>
	<RuleBlock>
		<Iif>
			<LookupTable insert="false" tableid="17" update="false">
				<where keyvalue="TM_CODE"/>
				<get column="PCRF_SERVICE_NAME" into="PCRF_SERVICE_NAME"/>
			</LookupTable>
			<Assign append="False" source="PCRF_SERVICE_NAME" target="PCRF_SERVICE_NAME_EXPECTED"/>
			<Assign append="False" constant="_N" target="PCRF_SERVICE_NAME_EXPECTED"/>
		</Iif>
	</RuleBlock>
	<RuleBlock>
		<Trim source="ACTIVE_SERVICE_LIST" target="ACTIVE_SERVICE_LIST"/>
		<Assign append="False" constant="," target="AUX_rdrACTIVE_SERVICE_LIST"/>
		<Assign append="TRUE" source="ACTIVE_SERVICE_LIST" target="AUX_rdrACTIVE_SERVICE_LIST"/>
		<Assign append="TRUE" constant="," target="AUX_rdrACTIVE_SERVICE_LIST"/>
		<Assign append="False" source="AUX_rdrACTIVE_SERVICE_LIST" target="ACTIVE_SERVICE_LIST"/>
		<Assign append="FALSE" constant=".*,505,.*" target="PATTERN"/>
		<Assign append="FALSE" constant=".*,507,.*" target="PATTERN2"/>
		<Iif>
			<Or>
				<StrMatchRE field="ACTIVE_SERVICE_LIST" ignorecase="true" patternfield="PATTERN"/>
				<StrMatchRE field="ACTIVE_SERVICE_LIST" ignorecase="true" patternfield="PATTERN2"/>
			</Or>
			<Assign append="False" constant="1" target="AUTO_ACTIVATION_FLAG"/>
			<Assign append="False" constant="0" target="AUTO_ACTIVATION_FLAG"/>
		</Iif>
	</RuleBlock>
</RuleBlock>


--------------------------------------------------

<Equals constant="504" ignorecase="True" target="SN_CODE"/>

-------------------------------------------------

now here you can find sample of the target table i want to create : 

MSISDN     |STATUS   |TYPE|CONTRACT_ID|TM_CODE|RUN_DATE|BILLCYCLE|TMCODE_DES                    |PCRF_SERVICE_NAME_EXPECTED|AUTO_ACTIVATION_FLAG|PRICE_GROUP   |
-----------+---------+----+-----------+-------+--------+---------+------------------------------+--------------------------+--------------------+--------------+
2035439751 |ACTIVE   |POST|  101030789|   1323|20261030|02       |High Speed 30MB 750GB Postpaid|Serv_ADSL_HS_30M_600G     |                   1|EMPLOYEES     |
20233837714|ACTIVE   |PRE |  101050734|    376|20261030|03       |1 MB Prepaid                  |_N                        |                   1|PRE PAY       |
20553505960|ACTIVE   |PRE |  101064308|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20663242387|ACTIVE   |POST|  101064316|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
20226349445|ACTIVE   |POST|  101075758|   1365|20261030|02       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   1|INDIVIDUALS   |
20237487185|SUSPENDED|POST|  101076549|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
20233049967|SUSPENDED|POST|  101076549|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
20225751433|ACTIVE   |POST|  101076551|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
2034869643 |ACTIVE   |POST|  101076650|   1365|20261030|09       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   1|SOHO          |
20225881565|ACTIVE   |PRE |  101077017|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20222661116|SUSPENDED|POST|  101223593|   1365|20261030|09       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   0|SOHO          |
2035488360 |ACTIVE   |PRE |  101223929|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20133212518|ACTIVE   |PRE |  101223983|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20226569546|SUSPENDED|POST|  101224222|   1334|20261030|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   0|INDIVIDUALS   |
20242198889|ACTIVE   |POST|  101224547|   1364|20261030|02       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS   |
20403512206|SUSPENDED|POST|  101224622|   1334|20261030|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   0|INDIVIDUALS   |
20226506438|ACTIVE   |POST|  101224868|   1321|20261030|01       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
20222346509|ACTIVE   |POST|  101411287|   1334|20261030|02       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS   |
20235428916|ACTIVE   |POST|  101412555|   1365|20261030|01       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   1|INDIVIDUALS   |
20242189291|ACTIVE   |POST|  101418242|   1364|20261030|01       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS   |
20222568754|ACTIVE   |POST|  101506596|   1321|20261030|01       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
20222907511|SUSPENDED|POST|  101584074|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
20225900380|ACTIVE   |POST|  101585559|   1323|20261030|01       |High Speed 30MB 750GB Postpaid|Serv_ADSL_HS_30M_600G     |                   1|INDIVIDUALS   |
2035485241 |ACTIVE   |POST|  101586134|   1334|20261030|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS   |
20224019144|ACTIVE   |PRE |  101589262|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20224151869|ACTIVE   |POST|  101594495|   1334|20261030|02       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS   |
20237490540|ACTIVE   |POST|  101596752|   1365|20261030|02       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   1|INDIVIDUALS   |
20663220418|SUSPENDED|POST|  101597335|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
2033923540 |ACTIVE   |PRE |  101598062|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
2033927662 |ACTIVE   |PRE |  101598063|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20222472153|ACTIVE   |POST|  101598357|   1364|20261030|02       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS   |
20133224495|ACTIVE   |POST|  101598440|   1364|20261030|01       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS   |
20244796413|ACTIVE   |POST|  101598651|   1321|20261030|01       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
20226633649|SUSPENDED|POST|  101598708|    934|20261030|02       |ADSL Light 4 Mbps Premium     |Serv_ADSL_Light_4M_45G    |                   0|INDIVIDUALS   |
20226928341|ACTIVE   |POST|  101599038|   1321|20261030|02       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS   |
20224713158|ACTIVE   |POST|  101599053|   1365|20261030|01       |30MB200GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_180G     |                   1|INDIVIDUALS   |
20233389467|ACTIVE   |POST|  101599424|   1364|20261030|01       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS   |
2035536810 |ACTIVE   |POST|  101600184|   1334|20261030|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS   |
20226901469|ACTIVE   |PRE |  101719327|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20403340582|ACTIVE   |PRE |  101719681|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20882313948|SUSPENDED|POST|  101720656|   1321|20261030|01       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
20225907537|SUSPENDED|POST|  101823675|   1321|20261030|01       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   0|INDIVIDUALS   |
20235835104|ACTIVE   |PRE |  101823823|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |
20472565483|ACTIVE   |PRE |  101926669|   1352|20261030|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY       |

Columns
AUTO_ACTIVATION_FLAG
BILLCYCLE
CONTRACT_ID
MSISDN
PCRF_SERVICE_NAME_EXPE
PRICE_GROUP
RUN_DATE
STATUS
TM_CODE
TMCODE_DES
TYPE



ok and here is the path i want to export into 
export_path = (
            f"/project/PLATFORM_INTEGRITY/exports/"
            f"bscs_profiles_adsl_{run_date}.csv"
        )


here is the table i created in the db : bscs_profiles_adsl
RUN_DATE               |CONTRACT_ID|MSISDN     |TM_CODE|STATUS   |TYPE|BILL_CYCLE|TM_CODE_DESC                  |PRICE_GROUP                |PCRF_SERVICE_NAME_EXPECTED|AUTO_ACTIVATION_FLAG

here is the ctl and sqlldr :  ctl_file = prepare_ctl(
     "bscs_profiles_adsl",
     run_date
 )
 run_sqlldr(
     "bscs_profiles_adsl",
     run_date,
     ctl_file=ctl_file
 )
 logger.info("Insertion done")



 ok i want it the exported file to be delimted , 

 and now i will give you examples to think like it and take the shape we are uding from it : 

 radius:
 "import os
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
    ignore_errors=TRUE
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
    
        logger.info(f"Inserting into PI.radius table ..")
        ctl_file = prepare_ctl("radius", run_date)
        run_sqlldr("radius",run_date, ctl_file=ctl_file)
        logger.info(f"Insertion done")
        logger.info(f"Validating inserted data ..")

        validate_loaded_data(
        flow_name="radius",
        run_date=run_date,
        expected_rows=count,
        logger=logger)

        logger.info("Inserted data validated successfully.")

        end_time = datetime.now()
        logger.info("radius finished")
        logger.info(f"Total duration: {end_time - start_time}")
        send_mail(
                    subject="RADIUS Flow Finished",
                    content=f"""
                Run Date : {run_date}
                Rows      : {count}
                DB Validation : Passed
                Duration  : {end_time - start_time}""")

        return con.table("radius_data")

    except Exception as e:
        logger.exception( f"RADIUS failed for run_date= {run_date} | error= {str(e)}")
        send_mail(
        subject="RADIUS Flow Failed",
        content=f"""
            Run Date : {run_date}
            Error:
            {str(e)}
            """)
        raise

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    load_radius(run_date=args.run_date)
"


solarwinds "import os
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



def load_solarwinds(run_date: str, con=None):
     
    start_time = datetime.now() 
    logger = get_logger(
        flow="solarwinds",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()
            

        run_date_formatted = f"{run_date[:4]}-{run_date[4:6]}-{run_date[6:]}"
        solarwinds_date = f"{run_date[:4]}_{run_date[4:6]}_{run_date[6:]}"
        csv_path = f"/project/PLATFORM_INTEGRITY/exports/solarwinds_{run_date}.csv"
        file_path = (f"/RAIDdata/PI/SOLARWINDS/out/{solarwinds_date}_*mod*")

        con.execute(f"""
CREATE OR REPLACE TABLE solarwinds_data AS
SELECT

    CASE
        WHEN TRY_CAST(CAST("Timestamp" AS VARCHAR) AS DATE) IS NOT NULL
        THEN CAST(TRY_CAST(CAST("Timestamp" AS VARCHAR) AS DATE) AS VARCHAR)
        ELSE ''
    END AS RUN_DATE,

    COALESCE(
        TRY_CAST(TRIM(CAST("BNG Name" AS VARCHAR)) AS VARCHAR),
        ''
    ) AS BNG_NAME,

    '_N' AS LAG,

    COALESCE(
        TRY_CAST(regexp_extract(filename, '[^/]+$') AS VARCHAR),
        ''
    ) AS FILE_NAME,

    TRY_CAST(
        regexp_extract(
            TRIM(CAST("Total Bytes Received" AS VARCHAR)),
            '^([^.]*)'
        ) AS BIGINT
    ) AS RECEIVED_VOLUME,

    TRY_CAST(
        regexp_extract(
            TRIM(CAST("Total Bytes Transmitted" AS VARCHAR)),
            '^([^.]*)'
        ) AS BIGINT
    ) AS TRANSMITTED_VOLUME,

    TRY_CAST(
        regexp_extract(
            TRIM(CAST("Total Bytes (Transmitted + Received)" AS VARCHAR)),
            '^([^.]*)'
        ) AS BIGINT
    ) AS TOTAL_VOLUME

FROM read_csv_auto(
    '{file_path}',
    delim=',',
    header=TRUE,
    filename=TRUE,
    ignore_errors=TRUE
);
""")

        count = con.execute("SELECT COUNT(*) FROM solarwinds_data").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            logger.info("No solarwinds data processed!")
            raise
        
        con.execute(f"""
        COPY (
            SELECT RUN_DATE, BNG_NAME , LAG , FILE_NAME , RECEIVED_VOLUME , TRANSMITTED_VOLUME , TOTAL_VOLUME
            FROM solarwinds_data 
        )
        TO '{csv_path}'
        (DELIMITER ',', HEADER TRUE, QUOTE '"', ESCAPE '"');
    """)

        logger.info(f"Inserting into PI.SOLARWINDS table ..")
        ctl_file = prepare_ctl("solarwinds", run_date)
        run_sqlldr("solarwinds",run_date, ctl_file=ctl_file)
        logger.info(f"Insertion done")
        logger.info(f"Validating inserted data ..")

        validate_loaded_data(
        flow_name="solarwinds",
        run_date=run_date,
        expected_rows=count,
        logger=logger)

        logger.info("Inserted data validated successfully.")

        end_time = datetime.now()
        logger.info("solarwinds finished")
        logger.info(f"Total duration: {end_time - start_time}")
        send_mail(
                    subject="SOLARWINDS Flow Finished",
                    content=f"""
                Run Date : {run_date}
                Rows      : {count}
                DB Validation : Passed
                Duration  : {end_time - start_time}""")

        return con.table("solarwinds_data")

    except Exception as e:
        logger.exception( f"solarwinds failed for run_date= {run_date} | error= {str(e)}")
        send_mail(
        subject="SOLARWINDS Flow Failed",
        content=f"""
            Run Date : {run_date}
            Error:
            {str(e)}
            """)
        raise

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    load_solarwinds(run_date=args.run_date)
"

new_solar:
"import os
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



def load_solarwinds_new(run_date: str, con=None):
     
    start_time = datetime.now() 
    logger = get_logger(
        flow="solarwinds_new",
        log_dir=None,
        run_date=run_date
    )

    try:
        if con is None:
            con = duckdb_connection()
            

        run_date_formatted = f"{run_date[:4]}-{run_date[4:6]}-{run_date[6:]}"
        solarwinds_date = f"{run_date[:4]}_{run_date[4:6]}_{run_date[6:]}"
        csv_path = f"/project/PLATFORM_INTEGRITY/exports/solarwinds_new_{run_date}.csv"
        file_path = (f"/RAIDdata/PI/SOLARWINDS/out_2/*{solarwinds_date}*updated*")

        con.execute(f"""
CREATE OR REPLACE TABLE solarwinds_new_data AS
SELECT
    CASE
        WHEN TRY_CAST(TRIM(TRY_CAST("Timestamp" AS VARCHAR)) AS DATE) IS NOT NULL
        THEN TRY_CAST(
            TRY_CAST(TRIM(TRY_CAST("Timestamp" AS VARCHAR)) AS DATE)
            AS VARCHAR
        )
        ELSE ''
    END AS RUN_DATE,

    COALESCE(
        TRIM(TRY_CAST("BNG Name" AS VARCHAR)),
        ''
    ) AS BNG_NAME,

    COALESCE(
        TRIM(TRY_CAST("Interface Name" AS VARCHAR)),
        ''
    ) AS INTERFACE_NAME,

    COALESCE(
        TRIM(TRY_CAST("Interface Caption" AS VARCHAR)),
        ''
    ) AS INTERFACE_CAPTION,

     COALESCE(
        TRY_CAST(regexp_extract(filename, '[^/]+$') AS VARCHAR),
        ''
    ) AS FILE_NAME,

    TRY_CAST(
        REGEXP_REPLACE(
            TRIM(TRY_CAST("Total Bytes Received" AS VARCHAR)),
            '\.[0-9]+[[:space:]]*B$',
            ''
        ) AS BIGINT
    ) AS RECEIVED_VOLUME,

    TRY_CAST(
        REGEXP_REPLACE(
            TRIM(TRY_CAST("Total Bytes Transmitted" AS VARCHAR)),
            '\.[0-9]+[[:space:]]*B$',
            ''
        ) AS BIGINT
    ) AS TRANSMITTED_VOLUME,

    TRY_CAST(
        REGEXP_REPLACE(
            TRIM(TRY_CAST("Total Bytes (Transmitted + Received)" AS VARCHAR)),
            '\.[0-9]+[[:space:]]*B$',
            ''
        ) AS BIGINT
    ) AS TOTAL_VOLUME,

    CASE
        WHEN UPPER(
            COALESCE(
                TRIM(TRY_CAST("Interface Caption" AS VARCHAR)),
                ''
            )
        ) LIKE '%HSUBBT%'
        THEN 'HSUBBT'
        ELSE 'UBBT'
    END AS SPEED

FROM read_csv_auto(
    '{file_path}',
    delim=',',
    header=TRUE,
    filename=TRUE,
    ignore_errors=TRUE
);
""")

        count = con.execute("SELECT COUNT(*) FROM solarwinds_new_data").fetchone()[0]
        logger.info(f"Total processed rows: {count}")

        if count == 0:
            logger.info("No solarwinds_new data processed!")
            raise
        
        con.execute(f"""
        COPY (
            SELECT RUN_DATE, BNG_NAME ,INTERFACE_NAME , INTERFACE_CAPTION , FILE_NAME , 
            RECEIVED_VOLUME , TRANSMITTED_VOLUME , TOTAL_VOLUME , SPEED
            FROM solarwinds_new_data 
        )
        TO '{csv_path}'
        (DELIMITER ',', HEADER TRUE, QUOTE '"', ESCAPE '"');
    """)

        logger.info(f"Inserting into PI.SOLARWINDS_NEW table ..")
        ctl_file = prepare_ctl("solarwinds_new", run_date)
        run_sqlldr("solarwinds_new",run_date, ctl_file=ctl_file)
        logger.info(f"Insertion done")
        logger.info(f"Validating inserted data ..")

        validate_loaded_data(
        flow_name="solarwinds_new",
        run_date=run_date,
        expected_rows=count,
        logger=logger)

        logger.info("Inserted data validated successfully.")

        end_time = datetime.now()
        logger.info("solarwinds_new finished")
        logger.info(f"Total duration: {end_time - start_time}")
        send_mail(
                    subject="SOLARWINDS_NEW Flow Finished",
                    content=f"""
                Run Date : {run_date}
                Rows      : {count}
                DB Validation : Passed
                Duration  : {end_time - start_time}""")

        return con.table("solarwinds_new_data")

    except Exception as e:
        logger.exception( f"SOLARWINDS_NEW failed for run_date= {run_date} | error= {str(e)}")
        send_mail(
        subject="SOLARWINDS_NEW Flow Failed",
        content=f"""
            Run Date : {run_date}
            Error:
            {str(e)}
            """)
        raise

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--run_date", required=True)
    args = parser.parse_args()
    load_solarwinds_new(run_date=args.run_date)
"

















  
