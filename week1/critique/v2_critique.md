1. Rules that conflict with each other

“Don’t add anything” vs. “a tester can run it without asking questions.” To make tests runnable, the model needs a store ID, test card numbers, a way to simulate an expired card, a way to make the SAP send fail, and where to check the SAP message and error log. None of that is in the BRD. The model will either invent it, which breaks your scope rule, or leave it vague, which breaks your role. Fix: add a <test_env> block with the fixed test data and tools, such as store ID, test cards, an SAP stub or queue viewer, and a log location. Or allow placeholders like {TEST_MASTERCARD}.

“One behaviour per test” vs. the most important tests. The highest-risk bugs are in interactions between rules:

R3 + R4: a partial refund on a split-tender sale
R5 + R3: a $600 refund across two tenders
R6 + R3: an expired card that was the second tender
R4 across multiple partial refunds, where the cumulative total must not exceed the original

With the current rules, the model will skip these. Fix: add a combination type and let requirement_id be an array.

“Positive + negative for every requirement.” Some requirements have no natural negative. R7, “POS sends a message,” is an example. Forcing one produces filler tests. Fix: “If a negative test doesn’t make sense, explain why in coverage_notes instead of inventing one.”

2. Boundary tests depend on decisions the BRD doesn’t make

Rule 2 asks for boundary tests, but every boundary in this BRD is ambiguous:

R2: Is “within 30 days” inclusive of day 30? Is it calendar days or 30×24 hours? Whose time zone?
R5: Does “over $500” mean $500.00 doesn’t need approval? Is the limit per refund or cumulative per transaction?
R7: Is the 5 minutes measured from refund confirmation or from receipt print?
R8: What’s the retry interval? Is a 4th attempt ever made? Does 3 retries mean 4 total sends?

You’ve told the model not to guess, but you haven’t said what to do with the test in the meantime. Fix: “Write the boundary test anyway, state the assumption in preconditions, and link it to an open question by ID.”

There’s also no reference date. A test like “purchased 30 days ago” isn’t concrete. Fix: “Assume today is 2026-11-01. Use absolute dates.”

3. Output schema gaps
expected as a single string gets messy for R7, where you need to check six fields. Make it an array of checks, one per observable result.
requirement_id as a string can’t hold combination tests. Use an array.
Test data is mixed into steps and preconditions. A separate test_data object makes the tests reusable and easy to review.
Step numbering is redundant. The array is already ordered, and writing “1.” inside each string invites mistakes. Pick one.
open_questions as plain strings can’t be traced. Use objects like {"id": "Q-01", "requirement_id": "R2", "question": "", "impacts": ["TC-005"]}.
There’s no coverage check. Add a coverage summary, for example {"R1": ["TC-001","TC-002"], ...}. It makes gaps visible and pushes the model to check its own work before it finishes.
4. Small fixes
Typo: “Line/Product item,transaction ID” is missing a space, and the “/” is unclear. Say “line item (SKU, qty, price).”
The example omits open_questions and the wrapper, so add one line saying the example is a single test_cases entry.
Missing scenarios the model should consider, worth stating explicitly:
refund without a receipt
refund attempted twice on the same transaction
manager approval rejected (not only granted)
SAP succeeding on retry 2, not just failing all 3 times
Optional: add priority (P1/P2) so testers know what to run first under time pressure.