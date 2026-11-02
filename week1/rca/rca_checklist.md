# RCA Checklist — Scoring of v1 Runs

Incident: POS v5.3.0 / store 0412 / SAP refund 401s

## Scores

| Check | Run 1 | Run 2 | Run 3 |
|-------|-------|-------|-------|
| All 6 headings, in order, nothing extra | ✅ | ✅ | ✅ |
| Numbers match notes (37 refunds, $4,318.55, store 0412) | ✅ | ✅ | ✅ |
| Root cause = stale client secret after the v5.3.0 config change (not just "401") | ✅ | ✅ | ✅ |
| Nothing invented (no new times, causes, people) | ✅ | ✅ | ❌ |
| Prevention bullets are testable (not "be more careful") | ✅ | ✅ | ✅ |
| **Score /5** | **5** | **5** | **4** |

## Why Run 3 lost a point

Run 3's root cause says sales were unaffected "because only the refund endpoint moved to the new credential config." The notes confirm that v5.3.0 moved the refund endpoint. They say nothing about the sales endpoint staying on the old config, so this line adds a cause that isn't in the notes.

Run 2 makes a similar statement ("Sales used a path that was not affected"). It stays closer to restating the observation, so it passes, but it is borderline.

## Borderline passes

- **Run 1:** "Why the ERROR log entries did not trigger an alert is Unknown" takes for granted that no alert fired. The notes only show that reconciliation found the gap. Run 3 words this better: "Whether any alert fired… is Unknown."
- **All runs:** All three say every refund call got a 401. The notes show the 401 only for RF-88120, plus "none received by SAP." That conclusion is a reasonable inference, but it is still an inference.

## Differences the rubric doesn't catch

- **Run 1:** The Evidence section is missing two quotes, the fix line ("Oct 21 11:00 config fixed…") and the customers line. As a result, its Fix and Customers claims have no supporting quote.
- **Run 2:** The Summary leaves out the $4,318.55 figure, though Impact includes it.
- **Run 2:** It has the most targeted prevention. It tests sales and refunds separately, so a passing sales test cannot hide a failing refund path.
- **Run 1:** It is the only run that verifies its alert actually fires, by injecting a 401 in a test environment.

## Overall

Run 2 is the most reliable: its evidence is complete and its Unknowns are framed neutrally. Run 1 ties it on score but has the evidence gap.

## Suggested rubric addition

| Check | Run 1 | Run 2 | Run 3 |
|-------|-------|-------|-------|
| Every claim in Fix and Impact has a matching Evidence quote | ❌ | ✅ | ✅ |

With this check added, Run 1 drops below Run 2, and unsupported claims are caught more mechanically than "nothing invented" alone can catch them.
