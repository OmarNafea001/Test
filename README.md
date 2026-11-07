WITH Total_Base AS
(
        SELECT Run_date, COUNT(*) Total_Base
        FROM RP7VFEPIDAT.PFM_BSCS_ADSL
        WHERE run_date = $1
        GROUP BY run_date
),
All_Status_Mis AS
(
        SELECT TO_NUMBER(TO_CHAR(run_date, 'YYYYMMDD')) AS RUN_DATE ,COUNT(*) Status_Mis_count
        FROM RP7VFEPIDAT.rd_t_pcrf_expiry_date_mismatch
        WHERE TO_CHAR(run_date, 'YYYYMMDD') = '$1'
        GROUP BY run_date
)
SELECT to_date(B.RUN_DATE,'YYYYMMDD') ,S.Status_Mis_count, B.Total_Base ,
(B.Total_Base - S.Status_Mis_count) AS VARIANCE,
ROUND((S.Status_Mis_count / B.Total_Base ) *100 ,2 ) AS Percentage
FROM Total_Base B JOIN All_Status_Mis S
        ON B.RUN_DATE = S.Run_Date;
