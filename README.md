Postpaid ADSL Quota Finisher KPI

Objective: Measure whether postpaid ADSL customers consume their full allocated quota within their last complete bill cycle.

Conditions:

Active postpaid ADSL customers only, based on their latest billing record.
Test lines and SDSL customers are excluded.
Measured per customer over their last completed bill cycle (BC01, 02, 04, 07, 09, 11, 25).
Allocated quota is the main bundle quota plus any double quota, in GB.
Package upgrades or downgrades within the cycle are added together.
Add-ons, promos, gifts, ADN and throttling services are excluded from both quota and usage.
Only services listed as a Main Bundle in the product catalogue are counted as quota.

Indicators:

Finisher: consumed 100% or more of allocated quota.
Non-Finisher: consumed less than 100%, grouped as follows:
CAT1: below 25%
CAT2: 25% to under 50%
CAT3: 50% to under 75%
CAT4: 75% to under 100%
No PCRF data: no quota record found for the cycle.
Additional flags: package change during the cycle, double quota, number of quota renewals, consumption %, and unused quota (variance).

Special Cases:

Customer has more than one service (package change): The final quota of each main service in the cycle is added together to make the main quota. The customer is flagged “Service Changed = Yes”, and the services are listed in the order they were used.
Customer has double quota: The double quota is added on top of the main quota, so total allocated quota = main quota + double quota. The customer is flagged “Double Quota = Yes”.
Customer has more than one double quota: The final quota of each double-quota service is added together, then added to the main quota. The customer is still flagged “Double Quota = Yes”.
