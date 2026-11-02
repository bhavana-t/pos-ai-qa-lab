## Summary
After POS app v5.3.0 was deployed to store 0412 on Oct 20, refunds sent to SAP failed with HTTP 401 Unauthorized because the store's new client-credential config still held the old client secret. As a result, 37 refunds did not reach SAP until finance reconciliation flagged them the next day and they were reposted.

## Impact
- **Affected:** Refund postings from POS to SAP at store 0412 only. Sales from the same store posted normally, and other stores still on v5.2.x posted refunds normally.
- **Count and money:** 37 refunds totaling $4,318.55 were not received by SAP.
- **Duration:** The first failed refund was at Oct 20 10:12. Failed refunds continued through 17:40. All 37 were confirmed in SAP by Oct 21 12:15.
- **Customers:** Customers received their refunds at the register, and there were no customer complaints.

## Root cause
POS app v5.3.0 moved the SAP refund endpoint to a new client-credential config. Store 0412's config still had the old client secret, so every refund send from that store was rejected by SAP with HTTP 401 Unauthorized. After three retries, each refund ended as an ERROR log entry and was not delivered. Sales used a path that was not affected, so they kept posting. Why store 0412's config was not updated with the new secret during the v5.3.0 rollout is Unknown — need: the v5.3.0 deployment steps and change record for store 0412's client-credential config. Why the 401 failures were not caught until the next day's reconciliation is Unknown — need: whether any alert or monitor existed on refund "SAP send failed" errors on Oct 20.

## Evidence
- "Oct 20 09:05 POS app v5.3.0 deployed to store 0412."
- "Oct 20 10:12 first refund at 0412. POS log: "SAP send failed: HTTP 401 Unauthorized" refundId RF-88120; retried 3 times, then ERROR log entry."
- "Oct 20 10:12–17:40: 37 refunds at store 0412, total $4,318.55, none received by SAP. Sales from the same store posted normally."
- "Other stores (still on v5.2.x) posted refunds normally."
- "Oct 21 08:30 finance reconciliation flagged the 37 missing refunds."
- "Dev note: v5.3.0 moved the SAP refund endpoint to a new client-credential config; store 0412's config still had the old client secret."
- "Oct 21 11:00 config fixed; the 37 refunds were reposted from the error log; all confirmed in SAP by 12:15."
- "Customers received their refunds at the register; no customer complaints."

## Fix
- Oct 21 11:00: Store 0412's client-credential config was corrected.
- The 37 failed refunds were reposted from the error log.
- All 37 refunds were confirmed in SAP by Oct 21 12:15.

## Prevention
- **Release check:** Before each store is enabled on a version that changes SAP credentials or endpoints, run an automated pre-flight that requests a token with that store's client-credential config for the refund endpoint. Block the rollout for that store if the request is not authorized.
- **Automated test:** Add a post-deploy smoke test that, for each newly deployed store, posts a test refund through the SAP refund path and asserts a 2xx response and receipt in SAP. Run it for sales as well, so a passing sales test cannot hide a failing refund path.
- **Monitor/alert:** Alert on-call when there is any refund "SAP send failed" ERROR entry, with immediate paging on HTTP 401/403. Add a separate alert when SAP-received refunds per store fall below POS refund counts within the same hour, so the gap is found the same day instead of at next-day reconciliation.