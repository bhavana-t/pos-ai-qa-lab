# BRD-POS-017 Refund to Original Tender: Questions for the BA

**Purpose:** These questions came up while writing test cases for BRD-POS-017. Until each one is answered, the affected tests run on the stated assumption. If your answer differs from the assumption, those tests change.

**Test reference:** `brd-pos-017-test-cases.json` (question IDs Q-01 to Q-14 match the JSON)

**Priority:** 🔴 = blocks money-movement, approval or SAP tests; 🟡 = affects edge cases only

---

## R1: Refund to original tender

### Q-08 🟡 Can the cashier choose a different refund tender?
Does the refund screen let the cashier pick a tender at all? If it does, what should happen when they pick a tender other than the original one?

- **Current assumption:** the POS offers no way to send the refund to a different tender.
- **Affects:** TC-002

---

## R2: 30-day refund window

### Q-01 🔴 Is day 30 inside the window, and how are days counted?
- Is a refund on exactly day 30 allowed?
- Are days counted as calendar dates in store local time, or as 24-hour periods from the purchase timestamp?

  *Example:* a sale at 9:00 PM on 2026-10-02 and a refund at 10:00 AM on 2026-11-01 is 30 calendar days but less than 30 × 24 hours.
- **Current assumption:** day 30 is allowed; days are calendar dates in store local time.
- **Affects:** TC-003, TC-004, TC-005

### Q-02 🔴 What happens after 30 days?
Is the refund blocked outright, can a manager override it, or is it issued as store credit?

- **Current assumption:** blocked outright, with no override and no store credit.
- **Affects:** TC-005

---

## R3: Split tender, reverse order

### Q-06 🔴 What does "reverse order" mean for a partial refund?
Is the last tender refunded up to its full paid amount before any money goes to the earlier tender?

*Example:* a sale paid VISA $70.00 first, then MC $30.00. A partial refund of $50.00 would go:

| Interpretation | MC | VISA |
|---|---|---|
| Fill last tender first (assumed) | $30.00 | $20.00 |
| Pro-rata by amount paid | $15.00 | $35.00 |

- **Current assumption:** fill the last tender first.
- **Affects:** TC-007

---

## R4: Partial refunds and the original-amount cap

### Q-07 🔴 Can the same line item be refunded twice?
R4 only caps the *total* refunded. On a $100.00 sale with a $20.00 item (qty 1), can that item be refunded a second time, since the running total ($40.00) is still under $100.00?

- **Current assumption:** no. A line item cannot be refunded beyond its purchased quantity.
- **Affects:** TC-017

---

## R5: Manager approval over $500

### Q-03 🔴 Does a refund of exactly $500.00 need approval?
"Over $500" reads as strictly greater than $500.00. Please confirm.

- **Current assumption:** $500.00 needs no approval; $500.01 does.
- **Affects:** TC-019, TC-020

### Q-04 🔴 What amount is checked against the $500 limit?
Which of these is it?
1. The total of one refund transaction
2. Each tender's share of the refund
3. The running total of all refunds on the same sale

*Examples:*
- A $600.00 split-tender refund of $300.00 + $300.00 needs approval under option 1, but not under option 2.
- Two separate $300.00 partial refunds on one sale need approval under option 3 only.

- **Current assumption:** option 1, the total of one refund transaction.
- **Affects:** TC-009, TC-018, TC-019, TC-020, TC-022

### Q-05 🔴 What happens when a manager rejects the approval?
- How does the manager reject it at the register? Is there a reject button, or do they just not enter the PIN?
- Is the refund cancelled? Can it be retried with another manager?
- Is a failed or wrong PIN treated the same as a rejection?

- **Current assumption:** the refund is cancelled, and no money or store credit is issued.
- **Affects:** TC-021

---

## R6: Expired or closed card → store credit

### Q-09 🔴 How is an expired or closed card detected, and is the fallback automatic?
- Does the POS check the card's expiry date at refund time, or rely on a decline from the card network? A closed account can only be found by a decline.
- Does the POS switch to store credit automatically, or does the cashier choose it?
- Does the customer need to be present (for example, to receive a store credit card)?

- **Current assumption:** the POS detects the card status and switches to store credit automatically.
- **Affects:** TC-010, TC-011, TC-023, TC-024, TC-027

---

## R7: SAP refund message

### Q-10 🔴 One SAP message per refund, or one per tender?
For a split-tender refund, does the POS send one message per tender, or one for the whole refund? If it sends one, how are multiple tender types and amounts represented, given that `tenderType` and `amount` are single fields?

- **Current assumption:** one message per tender.
- **Affects:** TC-006, TC-028

### Q-11 🔴 What are the field formats and allowed values?
- What are the allowed values for `tenderType`, including the value for store credit?
- What is the format of `amount`: decimal places, currency, and sign?
- What is the format and time zone of `timestamp`?
- What is the format of `refundId`?

- **Current assumption:** none. Tests only check that each field is present and identifies the right tender.
- **Affects:** TC-025, TC-027, TC-028

### Q-12 🔴 What starts the 5-minute clock, and do retries count?
- Is the clock started by refund confirmation at the register, or by something else?
- Must retries (R8) also finish within the 5 minutes?
- What is the interval between retries?

- **Current assumption:** the clock starts at refund confirmation, and retries must finish within the 5 minutes.
- **Affects:** TC-025, TC-028, TC-029

---

## R8: SAP retry and error log

### Q-13 🔴 How many send attempts in total?
- Does "retries 3 times" mean 4 attempts in total (1 first send + 3 retries), or 3 in total?
- Is each attempt recorded in the integration log?
- After the error log entry, is the message ever re-sent: queued, picked up by a batch job, or resent manually? Or is it lost?

- **Current assumption:** 4 attempts in total, each one visible in the integration log, and no re-send after the error log entry.
- **Affects:** TC-029, TC-030, TC-031, TC-032, TC-033

### Q-14 🟡 What must the error log entry contain?
For example: refundId, originalTxnId, failure reason, attempt count, timestamp.

- **Current assumption:** none. Tests only check that exactly one entry exists.
- **Affects:** TC-032, TC-033

---

## Summary

| ID | Req | Priority | Question in short | Tests affected |
|---|---|---|---|---|
| Q-01 | R2 | 🔴 | Is day 30 inside the window; calendar days or 24-hour periods? | 3 |
| Q-02 | R2 | 🔴 | After 30 days: block, override, or store credit? | 1 |
| Q-03 | R5 | 🔴 | Does exactly $500.00 need approval? | 2 |
| Q-04 | R5 | 🔴 | Is the $500 limit per refund, per tender, or cumulative? | 5 |
| Q-05 | R5 | 🔴 | What happens on manager rejection? | 1 |
| Q-06 | R3 | 🔴 | Reverse order on partial refunds: fill last tender first or pro-rata? | 1 |
| Q-07 | R4 | 🔴 | Can the same line item be refunded twice? | 1 |
| Q-08 | R1 | 🟡 | Can the cashier choose a different refund tender? | 1 |
| Q-09 | R6 | 🔴 | How is an expired or closed card detected; is the fallback automatic? | 5 |
| Q-10 | R7 | 🔴 | One SAP message per refund or per tender? | 2 |
| Q-11 | R7 | 🔴 | What are the SAP field formats and values? | 3 |
| Q-12 | R7 | 🔴 | What starts the 5-minute clock; do retries count? | 3 |
| Q-13 | R8 | 🔴 | 3 or 4 attempts in total; is a failed message ever re-sent? | 5 |
| Q-14 | R8 | 🟡 | What must the error log entry contain? | 2 |

**Suggested order:** Q-04, Q-09 and Q-13 each affect 5 tests, so answering them first unblocks the most tests.
