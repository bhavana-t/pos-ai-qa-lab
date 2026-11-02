## Summary
After POS app v5.3.0 was deployed to store 0412 on Oct 20, every refund sent to SAP from that store failed with HTTP 401 Unauthorized, so 37 refunds totaling $4,318.55 did not reach SAP. Finance reconciliation detected the gap on Oct 21; the config was fixed and all 37 refunds were reposted and confirmed in SAP by 12:15 that day.

## Impact
- **Affected:** Refund postings from POS to SAP at store 0412 only. Sales from the same store posted normally, and other stores (on v5.2.x) posted refunds normally.
- **Count:** 37 refunds.
- **Money:** $4,318.55 not received by SAP.
- **Duration:** Failed refunds occurred Oct 20 10:12–17:40. They were missing in SAP until confirmed after reposting by Oct 21 12:15.
- **Detection:** Oct 21 08:30, by finance reconciliation rather than by an alert. Why the ERROR log entries did not trigger an alert is Unknown — need: monitoring/alerting configuration for POS "SAP send failed" errors at the time of the incident.
- **Customers:** Received their refunds at the register; no customer complaints.

## Root cause
POS app v5.3.0 moved the SAP refund endpoint to a new client-credential configuration, but store 0412's configuration still held the old client secret. Every refund call to SAP was therefore rejected with HTTP 401 Unauthorized, and after 3 retries each refund was written to the error log instead of being delivered. Why the client secret was not updated as part of the v5.3.0 rollout is Unknown — need: the v5.3.0 deployment checklist/runbook and the change record showing whether a config update for store 0412 was planned and executed.

## Evidence
- "Oct 20 09:05 POS app v5.3.0 deployed to store 0412."
- "Oct 20 10:12 first refund at 0412. POS log: "SAP send failed: HTTP 401 Unauthorized" refundId RF-88120; retried 3 times, then ERROR log entry."
- "Oct 20 10:12–17:40: 37 refunds at store 0412, total $4,318.55, none received by SAP. Sales from the same store posted normally."
- "Other stores (still on v5.2.x) posted refunds normally."
- "Dev note: v5.3.0 moved the SAP refund endpoint to a new client-credential config; store 0412's config still had the old client secret."
- "Oct 21 08:30 finance reconciliation flagged the 37 missing refunds."

## Fix
- Oct 21 11:00: store 0412's client-credential config was corrected.
- The 37 refunds were reposted from the error log.
- All 37 were confirmed in SAP by Oct 21 12:15.

## Prevention
- **Release check:** Before a POS version is deployed to a store, an automated pre-deploy check validates that the store's client-credential config contains every credential the new version requires and that each credential successfully authenticates against SAP. The deploy is blocked on any 401 or missing entry.
- **Automated post-deploy test:** Immediately after deployment, a smoke test posts a test refund and a test sale from the store to SAP (non-production or test-flagged transaction) and asserts both are received. The rollout fails if either is not confirmed.
- **Monitor/alert:** Alert within minutes when the POS logs a "SAP send failed" ERROR for any refund after retries are exhausted, or when the POS refund count for a store diverges from the SAP-received refund count over a rolling window. Verify the alert fires by injecting a 401 in a test environment.