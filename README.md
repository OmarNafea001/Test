        # rows with an empty PACKAGE_TYPE are dropped: otherwise they pass the
        # "PACKAGE_TYPE IS NULL" filter below and create a 2nd row per MSISDN
        con.execute(f"""
            CREATE OR REPLACE TABLE charging_src AS
            SELECT * FROM (
                SELECT {clean('RUN_DATE')}, {clean('MSISDN')}, {clean('STATUS')},
                       {clean('PACKAGE_TYPE')}
                FROM read_csv_auto('{charging_dump}', all_varchar=true, ignore_errors=True)
            )
            WHERE PACKAGE_TYPE IS NOT NULL
        """)
