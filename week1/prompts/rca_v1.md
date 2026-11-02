ROLE
You are a QA lead writing a root cause analysis (RCA) for a retail POS incident.

RULES
1. Use ONLY facts in the <notes>. Do not add causes, numbers or times that are not there.
2. If a section needs a fact that is missing, write: "Unknown — need: <the evidence that would answer it>".
3. Each Evidence bullet quotes the note line it comes from.
4. No blame on individuals.
FORMAT
Use exactly these markdown headings, in this order, with nothing before or after:
## Summary            (2 sentences)
## Impact             (what was affected, counts, money, how long)
## Root cause         (one paragraph: the underlying cause, not the symptom)
## Evidence           (bullets)
## Fix                (what was done)
## Prevention         (3 bullets; each must be testable: an automated test, a monitor or alert, or a release check)

<notes>
INC-POS-2291 (synthetic) — refunds missing in SAP, store 0412
- Oct 20 09:05 POS app v5.3.0 deployed to store 0412.
- Oct 20 10:12 first refund at 0412. POS log: "SAP send failed: HTTP 401 Unauthorized" refundId RF-88120; retried 3 times, then ERROR log entry.
- Oct 20 10:12–17:40: 37 refunds at store 0412, total $4,318.55, none received by SAP. Sales from the same store posted normally.
- Other stores (still on v5.2.x) posted refunds normally.
- Oct 21 08:30 finance reconciliation flagged the 37 missing refunds.
- Dev note: v5.3.0 moved the SAP refund endpoint to a new client-credential config; store 0412's config still had the old client secret.
- Oct 21 11:00 config fixed; the 37 refunds were reposted from the error log; all confirmed in SAP by 12:15.
- Customers received their refunds at the register; no customer complaints.
</notes>