i want to create a new kpi that compare the allocated quota main pundle for postpaid 
and compare them with the total consumption to get the finisher people who finishes thier quota
and to get the people who not finish and categorize them by 5 category based on the percentage of consumption
i want to put on mind the double quota as well the people who has double quota. 
i will give you thr inputs i have and the tables with samples and if u have any question or need more info  to understand the request tell me please
and bbefore creating the query tell me what exactly what will u do and what did u understand the reqquested is
i will caculated the consumption based on each client bill-date as each we have 6 bill cycles i will give u the strt and the end of eachone to calculate 
the consumption of each one ok? 

and creae the fill query for me: 





ok if i have the following inputs: 

RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE  table that have the followinf columns 

RUN_DATE               |OFFER_ID  |PCRF_QOUTA_NAME             |PCRF_SERVICE_NAME              |PCRF_TYPE       |PCRF_QOUTA_VALUE_GB|PCRF_QOUTA_VALUE_MB|PCRF_TAXATION_VALUE_MB|PROD_TYPE_NAME|VALIDITY|FREQUENCY         |OFFER_DESCRIPTION                             |
-----------------------+----------+----------------------------+-------------------------------+----------------+-------------------+-------------------+----------------------+--------------+--------+------------------+----------------------------------------------+
2026-09-22 00:00:00.000|777010076 |DSL_Addon_15GB_2Month       |Serv_DSL_Addon_15GB_2Month     |PREPAID         |15                 |15360              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 15 GB 2 months               |
2026-09-22 00:00:00.000|777010077 |DSL_Addon_15GB_3Month       |Serv_DSL_Addon_15GB_3Month     |PREPAID         |15                 |15360              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 15 GB 3 months               |
2026-09-22 00:00:00.000|777010078 |DSL_Addon_25GB_2Month       |Serv_DSL_Addon_25GB_2Month     |PREPAID         |25                 |25600              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 25 GB 2 months               |
2026-09-22 00:00:00.000|777010079 |DSL_Addon_25GB_3Month       |Serv_DSL_Addon_25GB_3Month     |PREPAID         |25                 |25600              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 25 GB 3 months               |
2026-09-22 00:00:00.000|777010080 |DSL_Addon_25GB_6Month       |Serv_DSL_Addon_25GB_6Month     |PREPAID         |25                 |25600              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 25 GB 6 months               |
2026-09-22 00:00:00.000|777010081 |DSL_Addon_50GB_2Month       |Serv_DSL_Addon_50GB_2Month     |PREPAID         |50                 |51200              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 50 GB 2 months               |
2026-09-22 00:00:00.000|777010082 |DSL_Addon_50GB_3Month       |Serv_DSL_Addon_50GB_3Month     |PREPAID         |50                 |51200              |0                     |DSL Promotion |Monthly |Renewable         |DSL Quota Add-on 50 GB 3 months               |
2026-09-22 00:00:00.000|777010083 |DSL_ExtraQuota_Addon_OD     |Serv_DSL_ExtraQuota_Addon_OD   |PREPAID-POSTPAID|50                 |51200              |0                     |Addon         |Monthly |OnDemand          |Router Replacement Quota Add-on 50 GB 1 month |
2026-09-22 00:00:00.000|777010090 |DSL_Addon_5G_OD             |Serv_DSL_Addon_5G_OD           |PREPAID-POSTPAID|5                  |5120               |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 5GB                          |
2026-09-22 00:00:00.000|777010091 |DSL_Addon_15G_OD            |Serv_DSL_Addon_15G_OD          |PREPAID-POSTPAID|15                 |15360              |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 15GB                         |
2026-09-22 00:00:00.000|777010092 |DSL_Addon_25G_OD            |Serv_DSL_Addon_25G_OD          |PREPAID-POSTPAID|25                 |25600              |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 25GB                         |
2026-09-22 00:00:00.000|777010093 |DSL_Addon_50G_OD            |Serv_DSL_Addon_50G_OD          |PREPAID-POSTPAID|50                 |51200              |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 50GB                         |
2026-09-22 00:00:00.000|777010094 |DSL_Addon_100G_OD           |Serv_DSL_Addon_100G_OD         |PREPAID         |100                |102400             |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 100GB OD                     |
2026-09-22 00:00:00.000|777010094 |DSL_Addon_100G_R            |Serv_DSL_Addon_100G_R          |PREPAID         |100                |102400             |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 100GB OD                     |
2026-09-22 00:00:00.000|777010095 |DSL_Addon_100G_OD           |Serv_DSL_Addon_100G_OD         |PREPAID         |100                |102400             |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 100GB Renewable              |
2026-09-22 00:00:00.000|777010095 |DSL_Addon_100G_R            |Serv_DSL_Addon_100G_R          |PREPAID         |100                |102400             |0                     |Addon         |Monthly |Renewable/OnDemand|DSL Quota Add-on 100GB Renewable              |
2026-09-22 00:00:00.000|777010104 |DSL_ExtraQuota_RMDPromo     |Serv_DSL_ExtraQuota_RMDPromo   |PREPAID-POSTPAID|50                 |51200              |1024                  |DSL Promotion |Dynamic |OnDemand          |Extra Quota RMD Promo offer                   |
2026-09-22 00:00:00.000|NULL      |ADSL_120G_Quota_main        |Serv_ADSL_120G                 |POSTPAID        |120                |122880             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_200G_Quota_main        |Serv_ADSL_200G                 |POSTPAID        |200                |204800             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_40G_Quota_main         |Serv_ADSL_40G                  |POSTPAID        |40                 |40960              |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_60G_Quota_main         |Serv_ADSL_60G                  |POSTPAID        |60                 |61440              |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_80G_Quota_main         |Serv_ADSL_80G                  |POSTPAID        |80                 |81920              |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_100M_100G_Main|Serv_ADSL_Compound_100M_100G   |POSTPAID        |100                |102400             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_100M_400G_Main|Serv_ADSL_Compound_100M_400G   |POSTPAID        |400                |409600             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_10G_Addon     |Serv_ADSL_Compound_10G_Addon   |POSTPAID        |10                 |10240              |0                     |Addon         |Monthly |OnDemand          |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_15G_Addon     |Serv_ADSL_Compound_15G_Addon   |POSTPAID        |15                 |15360              |0                     |Addon         |Monthly |OnDemand          |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_16M_50G_Main  |Serv_ADSL_Compound_16M_50G     |POSTPAID        |50                 |51200              |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_20m_100G_Main |Serv_ADSL_Compound_20m_100G    |POSTPAID        |100                |102400             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_20m_150G_Main |Serv_ADSL_Compound_20m_150G    |POSTPAID        |150                |153600             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_20m_200G_Main |Serv_ADSL_Compound_20m_200G    |POSTPAID        |200                |204800             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_20m_250G_Main |Serv_ADSL_Compound_20m_250G    |POSTPAID        |250                |256000             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_25M_125G_Main |Serv_ADSL_Compound_25M_125G    |POSTPAID        |125                |128000             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_40m_100G_Main |Serv_ADSL_Compound_40m_100G    |POSTPAID        |100                |102400             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_40m_200G_Main |Serv_ADSL_Compound_40m_200G    |POSTPAID        |200                |204800             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_40m_350G_Main |Serv_ADSL_Compound_40m_150G    |POSTPAID        |350                |358400             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_40m_500G_Main |Serv_ADSL_Compound_40m_250G    |POSTPAID        |500                |512000             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_Compound_50M_200G_Main |Serv_ADSL_Compound_50M_200G    |POSTPAID        |200                |204800             |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_GC_100M_100G_Main      |Serv_ADSL_GC_100M_100G         |POSTPAID        |100                |102400             |0                     |DSL Package   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_GC_100M_1TB_Main       |Serv_ADSL_GC_100M_1TB          |POSTPAID        |1000               |1024000            |0                     |Main Bundle   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_GC_100M_200G_Main      |Serv_ADSL_GC_100M_200G         |POSTPAID        |200                |204800             |0                     |DSL Package   |Monthly |Renewable         |_N                                            |
2026-09-22 00:00:00.000|NULL      |ADSL_GC_100M_700G_Main      |Serv_ADSL_GC_100M_700G         |POSTPAID        |700                |716800             |0                     |DSL Package   |Monthly |Renewable         |_N                                            |


the result of the following queryies as shown :

SELECT distinct(PROD_TYPE_NAME ) FROM RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE

PROD_TYPE_NAME|
--------------+
DSL Promotion |
Addon         |
Main Bundle   |
DSL Package   |
DSL Status    |

chooses only main bundle and dsl package 
this previous is the lookup that for pcrf that has all services with its description 

--------------------------------------------------------------------------------------

the billing table: 

SELECT * FROM RP7VFEPIDAT.PFM_BSCS_ADSL

MSISDN      |STATUS   |TYPE|CONTRACT_ID|TM_CODE|RUN_DATE|BILLCYCLE|TMCODE_DES                    |PCRF_SERVICE_NAME_EXPECTED|AUTO_ACTIVATION_FLAG|PRICE_GROUP                   |
------------+---------+----+-----------+-------+--------+---------+------------------------------+--------------------------+--------------------+------------------------------+
20473478259 |ACTIVE   |PRE |  213900078|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20473720246 |ACTIVE   |PRE |  213900079|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20472513936 |ACTIVE   |PRE |  213900083|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20133287864 |ACTIVE   |PRE |  213900110|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20472562673 |ACTIVE   |PRE |  213900147|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20472592773 |ACTIVE   |PRE |  213900153|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20472593435 |ACTIVE   |PRE |  213900155|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20473406527 |ACTIVE   |PRE |  213900160|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20473444228 |ACTIVE   |PRE |  213900162|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20502361017 |ACTIVE   |PRE |  213900217|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20502724435 |ACTIVE   |PRE |  213900220|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20503470572 |ACTIVE   |PRE |  213900222|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |
20504320338 |ACTIVE   |PRE |  213900224|   1352|20260720|03       |ADSL Prepaid Unified Tariff   |_N                        |                   1|PRE PAY                       |

i guess u can use thoses conditions: 

MSISDN     |STATUS|TYPE|CONTRACT_ID|TM_CODE|RUN_DATE|BILLCYCLE|TMCODE_DES                    |PCRF_SERVICE_NAME_EXPECTED|AUTO_ACTIVATION_FLAG|PRICE_GROUP                   |
-----------+------+----+-----------+-------+--------+---------+------------------------------+--------------------------+--------------------+------------------------------+
20233164018|ACTIVE|POST|  213903155|   1364|20260720|11       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|GOVERNMENT                    |
20222409919|ACTIVE|POST|  213903696|   1321|20260720|04       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS                   |
20552354507|ACTIVE|POST|  213904109|   1334|20260720|11       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|SOHO                          |
20226353430|ACTIVE|POST|  213905084|   1364|20260720|04       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |
20222438316|ACTIVE|POST|  213906467|   1321|20260720|04       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS                   |
20227296370|ACTIVE|POST|  213907062|   1364|20260720|07       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|SME                           |
20224718159|ACTIVE|POST|  213907559|   1334|20260720|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |
20222828418|ACTIVE|POST|  213917847|   1364|20260720|04       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |
20238837268|ACTIVE|POST|  213918081|   1326|20260720|02       |High Speed 70MB 300GB Postpaid|Serv_ADSL_HS_70M_300G     |                   1|INDIVIDUALS                   |
20403598797|ACTIVE|POST|  213918430|   1364|20260720|02       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |
20453683389|ACTIVE|POST|  213919001|   1364|20260720|07       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|SME20                         |
20222752884|ACTIVE|POST|  213919387|   1334|20260720|11       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|SOHO                          |
20553504358|ACTIVE|POST|  213919746|   1334|20260720|04       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |
20552356981|ACTIVE|POST|  213919747|   1334|20260720|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |

then join with catalouge to get allocated quota and do not forget to get the double quota from the 

SELECT * FROM RP7VFEPIDAT.PFM_BSCS_ADSL billing
JOIN RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE cata
ON cata.PCRF_SERVICE_NAME =billing.PCRF_SERVICE_NAME_EXPECTED
WHERE billing.TYPE ='POST'
AND billing.STATUS ='ACTIVE'
AND billing.PRICE_GROUP NOT IN ('TEST PHONES',
                                                                'Pre-paid Test SIM',
                                                                'Test Individuals',
                                                                'Dummy PG'
                                                           )
										
									
MSISDN     |STATUS|TYPE|CONTRACT_ID|TM_CODE|RUN_DATE|BILLCYCLE|TMCODE_DES                    |PCRF_SERVICE_NAME_EXPECTED|AUTO_ACTIVATION_FLAG|PRICE_GROUP                   |RUN_DATE               |OFFER_ID|PCRF_QOUTA_NAME       |PCRF_SERVICE_NAME     |PCRF_TYPE|PCRF_QOUTA_VALUE_GB|PCRF_QOUTA_VALUE_MB|PCRF_TAXATION_VALUE_MB|PROD_TYPE_NAME|VALIDITY|FREQUENCY|OFFER_DESCRIPTION|
-----------+------+----+-----------+-------+--------+---------+------------------------------+--------------------------+--------------------+------------------------------+-----------------------+--------+----------------------+----------------------+---------+-------------------+-------------------+----------------------+--------------+--------+---------+-----------------+
20233164018|ACTIVE|POST|  213903155|   1364|20260720|11       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|GOVERNMENT                    |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20222409919|ACTIVE|POST|  213903696|   1321|20260720|04       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_175G_Main |Serv_ADSL_HS_30M_175G |POSTPAID |175                |179200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20552354507|ACTIVE|POST|  213904109|   1334|20260720|11       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|SOHO                          |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20226353430|ACTIVE|POST|  213905084|   1364|20260720|04       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20222438316|ACTIVE|POST|  213906467|   1321|20260720|04       |High Speed 30MB 175GB Postpaid|Serv_ADSL_HS_30M_175G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_175G_Main |Serv_ADSL_HS_30M_175G |POSTPAID |175                |179200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20227296370|ACTIVE|POST|  213907062|   1364|20260720|07       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|SME                           |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20224718159|ACTIVE|POST|  213907559|   1334|20260720|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20222828418|ACTIVE|POST|  213917847|   1364|20260720|04       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20238837268|ACTIVE|POST|  213918081|   1326|20260720|02       |High Speed 70MB 300GB Postpaid|Serv_ADSL_HS_70M_300G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_70M_300G_Main |Serv_ADSL_HS_70M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20403598797|ACTIVE|POST|  213918430|   1364|20260720|02       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20453683389|ACTIVE|POST|  213919001|   1364|20260720|07       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|SME20                         |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20222752884|ACTIVE|POST|  213919387|   1334|20260720|11       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|SOHO                          |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20553504358|ACTIVE|POST|  213919746|   1334|20260720|04       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20552356981|ACTIVE|POST|  213919747|   1334|20260720|01       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20663667123|ACTIVE|POST|  213919977|   1323|20260720|02       |High Speed 30MB 750GB Postpaid|Serv_ADSL_HS_30M_600G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_600G_Main |Serv_ADSL_HS_30M_600G |POSTPAID |750                |768000             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20244736945|ACTIVE|POST|  213920376|   1364|20260720|04       |30MB300GB Sh/WatchItContentPo |Serv_ADSL_HS_30M_300G     |                   1|INDIVIDUALS                   |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_300G_Main |Serv_ADSL_HS_30M_300G |POSTPAID |300                |307200             |0                     |Main Bundle   |Monthly |Renewable|_N               |
20453906431|ACTIVE|POST|  213920441|   1334|20260720|07       |High Speed 30MB 150GB Postpaid|Serv_ADSL_HS_30M_140G     |                   1|SME                           |2026-09-22 00:00:00.000|NULL    |ADSL_HS_30M_140G_Main |Serv_ADSL_HS_30M_140G |POSTPAID |150                |153600             |0                     |Main Bundle   |Monthly |Renewable|_N               |

SELECT * FROM RP7VFEPIDAT.PFM_BSCS_ADSL
WHERE lower(PCRF_SERVICE_NAME_EXPECTED) LIKE '%double%' 

is empty so the double quota doesnt appear in the billing table then i should get it from other table which is pcrf_consum_filtered
so from the billing table i want only to get the msisdn with thier corresponding bill cycles to get the period and caculated the consumption.

with the consitions of the post

SELECT * FROM RP7VFEPIDAT.PFM_BSCS_ADSL billing
JOIN RP7VFEPIDAT.RD_T_DSL_PRODUCT_CATALOGUE cata
ON cata.PCRF_SERVICE_NAME =billing.PCRF_SERVICE_NAME_EXPECTED
WHERE billing.TYPE ='POST'
AND billing.STATUS ='ACTIVE'
AND billing.PRICE_GROUP NOT IN ('TEST PHONES',
                                                                'Pre-paid Test SIM',
                                                                'Test Individuals',
                                                                'Dummy PG'
                                                         )
or u can ignore the join we will see.

the bill cycles from the billing table:

this is the time of each bill cycles when it start and when it ends to caculated the consumption based on this period 

BC01: 1 : 31 
BC02: 15:16
BC04: 4:3
BC07:7:6
BC09:20 : 21
BC11: 11-10
BC25: 25:24

---------------------------------------------------------------------------------------------------------------
no i want first to calculated the allocated quota for each msisdn though out the month i want to consider 
only main quota and if there is a double quota i want to consider it as well and i may calculate it or get the allocated quota from the 
following table pcrf_consumption_filtered which suppose to have the allocaed quota for each msisdn at the end i will

SELECT * FROM RP7VFEPIDAT.PFM_PCRF_CONSUM_FILTERED

MSISDN        |SERVICE_NAME             |RUN_DATE               |QUOTA_CONSUMPTION|QUOTA_INITIAL|QUOTA_BALANCE|QUOTA_SUM |EXPECTED_ORG_QUOTA|EXPECTED_THROT_QUOTA|EXPECTED_TAX_QUOTA|
--------------+-------------------------+-----------------------+-----------------+-------------+-------------+----------+------------------+--------------------+------------------+
20472712563   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|7992             |524288000    |524280008    | 524288000|                 0|                   0|                 0|
20472712563   |Serv_ADSL_Mc_HS_30M_140G |2026-07-21 00:00:00.000|111908468        |157286400    |45377932     | 157286400|               150|                   0|      0.0068359375|
20242170251   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|17505372         |524288000    |506782628    | 524288000|                 0|                   0|                 0|
20962342314   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|2322658          |524288000    |521965342    | 524288000|                 0|                   0|                 0|
20962342314   |Serv_ADSL_Mc_HS_30M_300G |2026-07-21 00:00:00.000|34810381         |314572800    |279762419    | 314572800|               300|                   0|         0.0390625|
20483883082   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20228870529   |Serv_ADSL_Mc_HS_30M_140G |2026-07-21 00:00:00.000|33426291         |157286400    |123860109    | 157286400|               150|                   0|      0.0068359375|
20842240274   |Serv_ADSL_HS_30M_180G    |2026-07-21 00:00:00.000|175299076        |209715200    |34416124     | 209715200|               200|                   0|                 0|
20824407173   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|9844053          |524288000    |514443947    | 524288000|                 0|                   0|                 0|
20402736655   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|468949           |524288000    |523819051    | 524288000|                 0|                   0|                 0|
20402736655   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|131458431        |209715200    |78256769     | 209715200|               200|                   0|        0.02734375|
20223394017   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|16007208         |524288000    |508280792    | 524288000|                 0|                   0|                 0|
20223394017   |Serv_ADSL_Mc_HS_30M_140G |2026-07-21 00:00:00.000|157291051        |157286400    |0            | 157291051|               150|                   0|      0.0068359375|
20404633211   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|14564829         |524288000    |509723171    | 524288000|                 0|                   0|                 0|
20404633211   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|209718381        |209715200    |0            | 209718381|               200|                   0|        0.02734375|
20235464224   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|372969           |524288000    |523915031    | 524288000|                 0|                   0|                 0|
20235464224   |Serv_ADSL_Mc_HS_30M_300G |2026-07-21 00:00:00.000|314581047        |314572800    |0            | 314581047|               300|                   0|         0.0390625|
20482331947   |Serv_ADSL_HS_30M_180G    |2026-07-21 00:00:00.000|16205507         |209715200    |193509693    | 209715200|               200|                   0|                 0|
20553691770   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|577622           |524288000    |523710378    | 524288000|                 0|                   0|                 0|
20553691770   |Serv_ADSL_Mc_HS_30M_300G |2026-07-21 00:00:00.000|146383607        |314572800    |168189193    | 314572800|               300|                   0|         0.0390625|
20237729126   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|19368            |524288000    |524268632    | 524288000|                 0|                   0|                 0|
20237729126   |Serv_ADSL_Mc_HS_70M_300G |2026-07-21 00:00:00.000|283102357        |314572800    |31470443     | 314572800|               300|                   0|      0.0537109375|
20228902893   |Serv_ADSL_HS_30M_180G    |2026-07-21 00:00:00.000|32608853         |209715200    |177106347    | 209715200|               200|                   0|                 0|
20227151434   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20227151434   |Serv_ADSL_Mc_HS_30M_140G |2026-07-21 00:00:00.000|142502501        |157286400    |14783899     | 157286400|               150|                   0|      0.0068359375|
20236821205   |Serv_ADSL_HS_30M_140G    |2026-07-21 00:00:00.000|24369700         |157286400    |132916700    | 157286400|               150|                   0|                 0|
20504111125   |Serv_ADSL_HS_30M_140G    |2026-07-21 00:00:00.000|109463895        |157286400    |47822505     | 157286400|               150|                   0|                 0|
20842574441   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|1044572          |524288000    |523243428    | 524288000|                 0|                   0|                 0|
20237807388   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20237807388   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|71642402         |209715200    |138072798    | 209715200|               200|                   0|        0.02734375|
20482574345   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20482574345   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|81232397         |209715200    |128482803    | 209715200|               200|                   0|        0.02734375|
20452923521   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20452923521   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|27251235         |209715200    |182463965    | 209715200|               200|                   0|        0.02734375|
20452173895   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|0                |524288000    |524288000    | 524288000|                 0|                   0|                 0|
20482749081   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|252181           |524288000    |524035819    | 524288000|                 0|                   0|                 0|
20482749081   |Serv_ADSL_Mc_HS_30M_180G |2026-07-21 00:00:00.000|80707523         |209715200    |129007677    | 209715200|               200|                   0|        0.02734375|
20932910407   |Serv_ADSL_HS_30M_180G    |2026-07-21 00:00:00.000|0                |209715200    |209715200    | 209715200|               200|                   0|                 0|
20452155478   |Serv_DSL_Throttling      |2026-07-21 00:00:00.000|2195624          |524288000    |522092376    | 524288000|                 0|                   0|                 0|



so the quota consumption is the total consumption quota initial is the quota this msisdn have quota sum is the sum of balances and consumption 
expected org quota is calculated from the catalouge lookup so we can use it from here 

let us test the 

SELECT * FROM RP7VFEPIDAT.PFM_PCRF_CONSUM_FILTERED
WHERE RUN_DATE > date'2026-10-01'
AND MSISDN ='20225610908'

MSISDN     |SERVICE_NAME         |RUN_DATE               |QUOTA_CONSUMPTION|QUOTA_INITIAL|QUOTA_BALANCE|QUOTA_SUM|EXPECTED_ORG_QUOTA|EXPECTED_THROT_QUOTA|EXPECTED_TAX_QUOTA|
-----------+---------------------+-----------------------+-----------------+-------------+-------------+---------+------------------+--------------------+------------------+
20225610908|Serv_ADSL_HS_30M_180G|2026-10-02 00:00:00.000|60639682         |209715200    |149075518    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-03 00:00:00.000|64841972         |209715200    |144873228    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-04 00:00:00.000|67525297         |209715200    |142189903    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-05 00:00:00.000|69728415         |209715200    |139986785    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-06 00:00:00.000|69728415         |209715200    |139986785    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-07 00:00:00.000|69728415         |209715200    |139986785    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-08 00:00:00.000|74993460         |209715200    |134721740    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-09 00:00:00.000|79188487         |209715200    |130526713    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-10 00:00:00.000|83387317         |209715200    |126327883    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-11 00:00:00.000|83387317         |209715200    |126327883    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-12 00:00:00.000|87592082         |209715200    |122123118    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-13 00:00:00.000|91787744         |209715200    |117927456    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-14 00:00:00.000|92939595         |209715200    |116775605    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-15 00:00:00.000|97137100         |209715200    |112578100    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-16 00:00:00.000|0                |209715200    |209715200    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-17 00:00:00.000|3523586          |209715200    |206191614    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-18 00:00:00.000|7718020          |209715200    |201997180    |209715200|               200|                   0|                 0|
20225610908|Serv_ADSL_HS_30M_180G|2026-10-19 00:00:00.000|11915356         |209715200    |197799844    |209715200|               200|                   0|                 0|

now i want to test a double quota case : 

MSISDN     |SERVICE_NAME                   |RUN_DATE               |QUOTA_CONSUMPTION|QUOTA_INITIAL|QUOTA_BALANCE|QUOTA_SUM|EXPECTED_ORG_QUOTA|EXPECTED_THROT_QUOTA|EXPECTED_TAX_QUOTA|
-----------+-------------------------------+-----------------------+-----------------+-------------+-------------+---------+------------------+--------------------+------------------+
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-01 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-01 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-02 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-02 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-03 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-03 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-04 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-04 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-05 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-05 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-06 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-06 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-07 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-07 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-08 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-08 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-09 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-09 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-10 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-10 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-11 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-11 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-12 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-12 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-13 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-13 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-14 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-14 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-15 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-15 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-16 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-16 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|
20220398489|Serv_ADSL_HS_30M_50G           |2026-10-17 00:00:00.000|0                |52428800     |52428800     | 52428800|                50|                   0|                 0|
20220398489|Serv_DSL_DoubleQuota_300GB_POST|2026-10-17 00:00:00.000|0                |314572800    |314572800    |314572800|               300|                   0|                 0|

this is a sample of example of double quota i guess it will have two records i dont know if we shoul look at each day of each bill cycle to see the org allocated data
and sum them to get the allocated quota

then compare it from edrs let us test more samples for sep 

MSISDN     |SERVICE_NAME                   |RUN_DATE               |QUOTA_CONSUMPTION|QUOTA_INITIAL|QUOTA_BALANCE|QUOTA_SUM|EXPECTED_ORG_QUOTA|EXPECTED_THROT_QUOTA|EXPECTED_TAX_QUOTA|
-----------+-------------------------------+-----------------------+-----------------+-------------+-------------+---------+------------------+--------------------+------------------+
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-25 00:00:00.000|0                |209715200    |209715200    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-25 00:00:00.000|0                |209715200    |209715200    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-26 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-26 00:00:00.000|4761709          |209715200    |204953491    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-27 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-27 00:00:00.000|10586554         |209715200    |199128646    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-28 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-28 00:00:00.000|22714985         |209715200    |187000215    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-29 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-29 00:00:00.000|35681269         |209715200    |174033931    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-30 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-30 00:00:00.000|46086447         |209715200    |163628753    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-08-31 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-08-31 00:00:00.000|50285113         |209715200    |159430087    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-01 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-01 00:00:00.000|58684954         |209715200    |151030246    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-02 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-02 00:00:00.000|65090736         |209715200    |144624464    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-03 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-03 00:00:00.000|69292246         |209715200    |140422954    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-04 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-04 00:00:00.000|77687843         |209715200    |132027357    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-05 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-05 00:00:00.000|95592949         |209715200    |114122251    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-06 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-06 00:00:00.000|104860293        |209715200    |104854907    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-07 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-07 00:00:00.000|112256200        |209715200    |97459000     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-08 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-08 00:00:00.000|122028457        |209715200    |87686743     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-09 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-09 00:00:00.000|126291158        |209715200    |83424042     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-10 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-10 00:00:00.000|131702856        |209715200    |78012344     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-11 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-11 00:00:00.000|137963389        |209715200    |71751811     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-12 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-12 00:00:00.000|150263193        |209715200    |59452007     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-13 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-13 00:00:00.000|161493180        |209715200    |48222020     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-14 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-14 00:00:00.000|169900858        |209715200    |39814342     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-15 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-15 00:00:00.000|178297914        |209715200    |31417286     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-16 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-16 00:00:00.000|186698045        |209715200    |23017155     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-17 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-17 00:00:00.000|191209967        |209715200    |18505233     |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-18 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-18 00:00:00.000|201920267        |209715200    |7794933      |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-19 00:00:00.000|29314            |209715200    |209685886    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-19 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-20 00:00:00.000|4227698          |209715200    |205487502    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-20 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-21 00:00:00.000|16825594         |209715200    |192889606    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-21 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-22 00:00:00.000|28020973         |209715200    |181694227    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-22 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-23 00:00:00.000|34158162         |209715200    |175557038    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-23 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-24 00:00:00.000|41781957         |209715200    |167933243    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-24 00:00:00.000|209722213        |209715200    |0            |209722213|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-25 00:00:00.000|0                |209715200    |209715200    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-25 00:00:00.000|0                |209715200    |209715200    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-26 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-26 00:00:00.000|14803261         |209715200    |194911939    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-27 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-27 00:00:00.000|25577656         |209715200    |184137544    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-28 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-28 00:00:00.000|34115044         |209715200    |175600156    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-29 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-29 00:00:00.000|35875251         |209715200    |173839949    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-09-30 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-09-30 00:00:00.000|42689345         |209715200    |167025855    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-10-01 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|
20643441742|Serv_ADSL_HS_30M_180G          |2026-10-01 00:00:00.000|45093398         |209715200    |164621802    |209715200|               200|                   0|                 0|
20643441742|Serv_DSL_DoubleQuota_200GB_POST|2026-10-02 00:00:00.000|932051           |209715200    |208783149    |209715200|               200|                   0|                 0|


for bill cycles 25 which apearing here that we can calculated get the calculated allocated org quota as the sum of the both 200 and 200 allocated will be 400
or an easier way if he has double quota in thei cycle period just get the original quota from catalouge then ,ultiple it by two to be the allocated quota.



now ther is the consumption there are two ways to test 1- from the pcrf_con_filtered to get the quota consumption on the last day of the billcycle 
and sum them to get the total and convert them to gb then compare it with allocted.


the second way is to use another table which called pcrf_edrs which has wuota usage colum that is have all the sessions saved u can jusm sum them
acorss the bil cycle period. 









after comparision i want to get a variance column that is have the variance between both across all bill cycle 
and a double quota or not table as well yes or no
then finisher or non finisher 
the if non finisher i want to caterize them into 4 catergy 



































