INSERT INTO RD_T_ADSL_STATUS_RECON
SELECT DISTINCT RUN_DATE, MSISDN,
CASE WHEN Billing_TYPE IS NULL
THEN '_N'
ELSE Billing_TYPE
END Billing_TYPE,
CASE WHEN BILLING_STATUS IS NULL
THEN '_N'
ELSE BILLING_STATUS
END BILLING_STATUS,
CASE WHEN charging_status IS NULL
THEN '_N'
ELSE charging_status
END charging_status,
CASE WHEN PCRF_STATUS IS NULL
THEN '_N'
ELSE PCRF_STATUS
END PCRF_STATUS,
CASE WHEN RADIUS_STATUS IS NULL
THEN '_N'
ELSE RADIUS_STATUS
END RADIUS_STATUS,
CASE WHEN (BILLING_TYPE IS NULL AND CHARGING_STATUS IS NULL) OR (BILLING_TYPE = 'PRE' AND CHARGING_STATUS IS NULL)
THEN '_N'
ELSE ADSL_STATUS
END ADSL_STATUS, Nopcrfservice_status
,
CASE WHEN (
PCRF_STATUS ='ACTIVE' AND RADIUS_STATUS='ACTIVE' AND (((charging_status in('SUSPENDED','DEACTIVE') or charging_status IS NULL)  AND Billing_TYPE ='PRE')
OR ((BILLING_STATUS in('SUSPENDED','DEACTIVE','NoAutoActivation')) AND Billing_TYPE='POST')
OR (BILLING_STATUS IS NULL AND charging_status IS NULL AND Billing_TYPE IS NULL )))THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='DEACTIVE' AND  (PCRF_STATUS = 'ACTIVE' OR PCRF_STATUS IS NULL) THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND (ADSL_STATUS='SUSPENDED' OR ADSL_STATUS IS NULL ) AND PCRF_STATUS IS NULL THEN 'No Impact'
WHEN RADIUS_STATUS='ACTIVE' AND (ADSL_STATUS='SUSPENDED' OR ADSL_STATUS IS NULL ) AND PCRF_STATUS ='ACTIVE' THEN 'Revenue Loss'
WHEN RADIUS_STATUS='ACTIVE' AND PCRF_STATUS ='NoPCRFService' AND (Nopcrfservice_status ='ACTIVE' or Nopcrfservice_status ='SUSPENDED') and ADSL_STATUS = 'ACTIVE' THEN 'Customer Experience'
WHEN RADIUS_STATUS='DEACTIVE' AND ADSL_STATUS='ACTIVE'  AND (PCRF_STATUS IN ('SUSPENDED','ON-HOLD') OR PCRF_STATUS IS NULL)  THEN 'Customer Experience'
WHEN RADIUS_STATUS IS NULL AND ADSL_STATUS='ACTIVE' AND  (PCRF_STATUS = 'ON-HOLD' OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='ACTIVE' AND  (PCRF_STATUS = 'SUSPENDED' OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
WHEN RADIUS_STATUS='ACTIVE' AND ADSL_STATUS='ACTIVE' AND   (PCRF_STATUS IN ('SUSPENDED','ON-HOLD') OR PCRF_STATUS IS NULL) THEN 'Customer Experience'
ELSE 'No Impact'
END IMPACT_CASE
FROM(
SELECT TO_DATE(COALESCE(Billing.RUN_DATE , TO_NUMBER(Charging.RUN_DATE), PCRF.RUN_DATE, RADIUS.RUN_DATE), 'YYYYMMDD') RUN_DATE, COALESCE(Billing.MSISDN, Charging.MSISDN, PCRF.MSISDN, RADIUS.MSISDN) MSISDN,
Billing.TYPE Billing_TYPE , case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end BILLING_STATUS,
upper(Charging.STATUS) charging_status,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not null then 'NoPCRFService' else UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))end PCRF_STATUS,
UPPER(DECODE(RADIUS.STATUS,'0','DeActive','1','Active','2','Suspended',RADIUS.STATUS)) RADIUS_STATUS,
CASE WHEN Billing.TYPE = 'PRE'
THEN upper(Charging.STATUS)
WHEN Billing.TYPE = 'POST'
THEN (case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end)
END ADSL_STATUS,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not NULL
THEN UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))
END Nopcrfservice_status
from RP7VFEPIDAT.PFM_ADSL_MSISDN ADSL
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG Billing ON ADSL.MSISDN = Billing.MSISDN
left outer join RP7VFEPIDAT.RD_T_IN_ADSL Charging on ADSL.MSISDN =Charging.MSISDN
left outer join (SELECT DISTINCT * FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED) PCRF ON ADSL.MSISDN =PCRF.MSISDN
left outer join RP7VFEPIDAT.PFM_RADIUS_STG RADIUS on ADSL.MSISDN =RADIUS.MSISDN
where (Charging.PACKAGE_TYPE ='DSL Status' or Charging.PACKAGE_TYPE is null)
AND (RADIUS.STATUS IN ('1','2','0','3') OR RADIUS.STATUS IS NULL)
AND (BILLING.TMCODE_DES NOT LIKE '%SDSL%' OR BILLING.TMCODE_DES IS NULL)
AND (BILLING.MSISDN NOT LIKE '%CP%' OR BILLING.msisdn IS NULL)
AND ( BILLING.PRICE_GROUP NOT IN ('TEST PHONES',
                                    'Pre-paid Test SIM',
                                    'Test Individuals',
                                    'Dummy PG') OR BILLING.PRICE_GROUP IS NULL)
)
WHERE msisdn NOT IN (SELECT msisdn FROM (SELECT * FROM
(SELECT TO_DATE(COALESCE(Billing.RUN_DATE , TO_NUMBER(Charging.RUN_DATE), PCRF.RUN_DATE, RADIUS.RUN_DATE), 'YYYYMMDD') RUN_DATE, COALESCE(Billing.MSISDN, Charging.MSISDN, PCRF.MSISDN, RADIUS.MSISDN) MSISDN,
Billing.TYPE Billing_TYPE ,Billing.PRICE_GROUP,
case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end BILLING_STATUS,
upper(Charging.STATUS) charging_status ,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not null then 'NoPCRFService' else UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))end PCRF_STATUS,
UPPER(DECODE(RADIUS.STATUS,'0','DeActive','1','Active','2','Suspended',RADIUS.STATUS)) RADIUS_STATUS,
CASE WHEN Billing.TYPE = 'PRE'
THEN upper(Charging.STATUS)
WHEN Billing.TYPE = 'POST'
THEN (case when upper(Billing.STATUS)='ACTIVE' and AUTO_ACTIVATION_FLAG =0 then 'NoAutoActivation' else upper(Billing.STATUS) end)
END ADSL_STATUS,
Case when PCRF.SERVICE_NAME is Null and Customer_STATUS is not NULL
THEN UPPER(DECODE(Customer_STATUS,0,'DeActive',1,'Active',2,'Suspended',3,'On-Hold', Customer_STATUS))
END Nopcrfservice_status
from RP7VFEPIDAT.PFM_ADSL_MSISDN ADSL
LEFT OUTER JOIN RP7VFEPIDAT.PFM_BSCS_ADSL_STG Billing ON ADSL.MSISDN = Billing.MSISDN
left outer join RP7VFEPIDAT.RD_T_IN_ADSL Charging on ADSL.MSISDN =Charging.MSISDN
left outer join (SELECT DISTINCT * FROM RP7VFEPIDAT.PFM_PCRF_STG_FILTERED) PCRF ON ADSL.MSISDN =PCRF.MSISDN
left outer join RP7VFEPIDAT.PFM_RADIUS_STG RADIUS on ADSL.MSISDN =RADIUS.MSISDN
where (Charging.PACKAGE_TYPE ='DSL Status' or Charging.PACKAGE_TYPE is null)
AND (RADIUS.STATUS IN ('1','2','0','3') OR RADIUS.STATUS IS NULL)
AND (BILLING.TMCODE_DES NOT LIKE '%SDSL%' OR BILLING.TMCODE_DES IS NULL)
AND (BILLING.MSISDN NOT LIKE '%CP%' OR BILLING.msisdn IS NULL)
)
WHERE
( Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status = 'SUSPENDED' AND radius_status IS null )
OR ( Billing_type = 'PRE' AND billing_status= 'ACTIVE'  AND CHARGING_status = 'ACTIVE'  AND pcrf_status = 'ACTIVE'  AND radius_status = 'ACTIVE'  )
OR (Billing_type = 'PRE' AND billing_status= 'SUSPENDED'  AND CHARGING_status = 'SUSPENDED'  AND pcrf_status = 'SUSPENDED' AND radius_status = 'ACTIVE' )
OR (Billing_type = 'PRE' AND billing_status= 'SUSPENDED'  AND CHARGING_status IS null  AND pcrf_status IS null AND radius_status = 'ACTIVE' )
OR (Billing_type = 'PRE' AND billing_status= 'ACTIVE'  AND CHARGING_status IS NULL AND pcrf_status  IS NULL AND radius_status = 'DEACTIVE' )
OR (Billing_type = 'POST' AND billing_status= 'ACTIVE'  AND CHARGING_status IS NULL AND pcrf_status IS NULL AND radius_status = 'DEACTIVE' )
OR (Billing_type = 'POST' AND billing_status= 'ACTIVE' AND CHARGING_status IS NULL AND pcrf_status = 'ACTIVE' AND radius_status = 'ACTIVE')
OR (Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status IS null AND radius_status = 'DEACTIVE')
OR (Billing_type IS null AND billing_status IS null AND CHARGING_status IS null AND pcrf_status IS null AND radius_status is null)
));
COMMIT;



for the above table write it with the exact same logic but you can make it more readbable and put it in script using the same path for each bscs pcrf charging pcrf  as we worked before ok and tell me exactly what you you understood and anything needed from my side? 
Exit;
END
