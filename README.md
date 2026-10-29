# Week 1 – Prompt engineering: BRD to test cases

## What it does
Uses an LLM to turn a synthetic retail POS requirement (BRD-POS-017: Refund to Original Tender, rules R1–R8) into JSON test cases.
Compares a simple prompt (v1) with a structured prompt (v2), 3 runs each, in fresh chats.

## Why it matters for testing
LLM output is non-deterministic: the same prompt gives different test cases on every run.
Before trusting AI-generated tests, you need a baseline and a repeatable way to measure coverage, boundaries and invented details. This is the same idea as a traceability matrix.

## Folder layout
| Path | Contents |
|------|----------|
| `brd/BRD-POS-017.md` | Synthetic BRD (8 refund rules incl. split tender, $500 manager approval, SAP message, retries) |
| `prompts/v1.md` | Baseline prompt (one instruction + output fields) |
| `prompts/v2.md` | Structured prompt: role, scope, rules, JSON format, one example |
| `outputs/v1_run1-3.json`, `outputs/v2_run1-3.json` | Raw model outputs, saved as returned |
| `comparison.md` | Run-by-run comparison and findings |

## How to run
1. Open a new chat in <AI tool you used, e.g. Claude / ChatGPT>.
2. Paste `prompts/v2.md` with the BRD text inside the `<brd>` tags.
3. Save only the JSON to `outputs/`, then validate it: `python -m json.tool outputs/v2_run1.json`
4. Repeat in a new chat for each run (never reuse a chat).

## Results (3 runs per prompt)
| | v1 | v2 |
|---|---|---|
| Test cases per run | 33–40 | 28–34 |
| Rules covered (R1–R8) | 8/8 | 8/8 |
| Invented details (IDs, card numbers, store IDs) | Many | Far fewer |
| Open questions for the BA | – | 12–15 per run |
| Day-29 boundary tested | 0/3 runs | 2/3 runs |
| $499.99 boundary tested | 1/3 runs | 2/3 runs |

**Missed by every v2 run:** split tender where the card part is expired, STORE_CREDIT in the SAP message, a non-manager trying to approve, and a refund of original + $0.01.

## What I learned
- More test cases ≠ better coverage. v2 wrote fewer cases but invented less and covered boundaries more often.
- Telling the model "don't guess, ask" turned hallucinated rules into 12–15 useful questions for the BA.
- v2 still varies between runs (case count, which boundaries, day-30 behaviour), so one run is never enough. This is why I'll add automated checks in Week 3.

## Next
Day 3: AI critique of v2 → prompt v3 with a coverage map, plus a BRD ambiguity list.

