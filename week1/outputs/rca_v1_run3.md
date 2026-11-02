## Summary
After POS app v5.3.0 was deployed to store 0412 on Oct 20, all 37 refunds processed there that day failed to reach SAP with HTTP 401 Unauthorized, because v5.3.0 used a new client-credential config that still held the old client secret for that store. The gap was flagged by finance reconciliation on Oct 21 at 08:30, the config was fixed at 11:00, and all 37 refunds were reposted and confirmed in SAP by 12:15.

## Impact
- **Affected:** Refund postings from store 0412 to SAP only. Sales from the same store posted normally, and other stores (still on v5.2.x) posted refunds normally.
- **Count and money:** 37 refunds totaling $4,318.55 were not received by SAP.
- **Duration:** Refunds failed from Oct 20 10:12 to 17:40, and remained missing in SAP until all were confirmed on Oct 21 by 12:15.
- **Customers:** No customer impact. Customers received their refunds at the register, and there were no complaints.
- **Detection:** By finance reconciliation on Oct 21 at 08:30, not at the time of failure. Whether any alert fired on the Oct 20 errors is Unknown — need: alerting/monitoring records for SAP send failures on Oct 20.

## Root cause
POS app v5.3.0 moved the SAP refund endpoint to a new client-credential config. When v5.3.0 was deployed to store 0412, that store's config still contained the old client secret, so every refund call to SAP was rejected with HTTP 401 Unauthorized. The calls were retried 3 times and then written to the error log instead of being posted. Sales were unaffected because only the refund endpoint moved to the new credential config. Why the client secret was not updated as part of the v5.3.0 rollout is Unknown — need: the v5.3.0 deployment/config-change procedure for store 0412 and whether a credential update step existed for it.

## Evidence
- "Oct 20 09:05 POS app v5.3.0 deployed to store 0412."
- "Oct 20 10:12 first refund at 0412. POS log: "SAP send failed: HTTP 401 Unauthorized" refundId RF-88120; retried 3 times, then ERROR log entry."
- "Oct 20 10:12–17:40: 37 refunds at store 0412, total $4,318.55, none received by SAP. Sales from the same store posted normally."
- "Other stores (still on v5.2.x) posted refunds normally."
- "Dev note: v5.3.0 moved the SAP refund endpoint to a new client-credential config; store 0412's config still had the old client secret."
- "Oct 21 08:30 finance reconciliation flagged the 37 missing refunds."
- "Oct 21 11:00 config fixed; the 37 refunds were reposted from the error log; all confirmed in SAP by 12:15."
- "Customers received their refunds at the register; no customer complaints."

## Fix
- Oct 21 11:00: Store 0412's client-credential config was corrected.
- The 37 failed refunds were reposted from the error log.
- All 37 refunds were confirmed in SAP by Oct 21 12:15.

## Prevention
- **Automated test (post-deploy smoke):** After any POS release is deployed to a store, run an automated test that sends a test refund through the SAP refund endpoint with that store's config. The test passes only if SAP returns success and the refund is received. Any 401 fails the deployment.
- **Monitor/alert:** Alert per store when SAP refund sends end in an ERROR log entry after retries, including any HTTP 401. The alert should fire on the first occurrence, not at next-day finance reconciliation.
- **Release check:** For releases that change endpoint or credential config (such as v5.3.0), the rollout checklist must verify that each target store's client-credential config contains the current client secret before deployment. Sign-off is blocked until every store on the rollout list passes.