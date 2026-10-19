the if non finisher i want to caterize them into 4 catergy 


THIS IS THE EDR TABLE: SELECT *
FROM RP7VFEPIDAT.PFM_PCRF_EDRS_CONSUM ppec

MSISDN    |SERVICE_NAME             |RUN_DATE|QUOTA_CONSUMPTION|EXPECTED_ORG_QUOTA|EXPECTED_TAX_QUOTA|TIME                   |TRIGGER_TYPE             |QUOTA_NAME              |QUOTA_STATUS|QUOTA_USAGE|QUOTA_VALUE|ACTUAL_QUOTA_USAGE|SERVICE_EXTENDED_ATTRIBUTE1|
----------+-------------------------+--------+-----------------+------------------+------------------+-----------------------+-------------------------+------------------------+------------+-----------+-----------+------------------+---------------------------+
0223933513|Serv_DSL_Double_Quota_Pre|20250901|63050568         |                 0|                 0|2025-09-01 00:00:00.000|GX_CC_REQUEST_UPDATE     |DSL_Double_Quota_Pre    |Normal      |6299850    |3145728    |6299850           |44601                      |
0237217783|Serv_ADSL_Mc_HS_30M_180G |20250901|146767813        |               200|        0.02734375|2025-09-01 00:00:00.000|GX_CC_REQUEST_UPDATE     |ADSL_Mc_HS_30M_180G_Main|Level1      |4196988    |209715200  |4196988           |4440                       |
0553297025|Serv_ADSL_HS_30M_140G    |20250901|293614474        |               140|                 0|2025-09-01 00:00:00.000|GX_CC_REQUEST_UPDATE     |ADSL_HS_30M_140G_Main   |Exhaust     |15260289   |146800640  |15260292          |4440                       |
035426002 |Serv_ADSL_Mc_HS_30M_300G |20250901|53257157         |               300|         0.0390625|2025-09-01 00:00:00.000|GX_CC_REQUEST_UPDATE     |ADSL_Mc_HS_30M_300G_Main|Normal      |6301377    |314572800  |6301377           |4440                       |
0225934022|Serv_ADSL_HS_30M_140G    |20250901|2939638          |               140|                 0|2025-09-01 00:00:00.000|GX_CC_REQUEST_UPDATE     |ADSL_HS_30M_140G_Main   |Normal      |2939638    |146800640  |2939639           |4440                       |

YOU CAN USE THE TIME NOT THE RUNDATE ALSO USE THE QUOTA USAGE TO SUM ACROSS ALL bill cycle period then get the from the quta_usage column to get the actuly consumption 
across the month ? 
