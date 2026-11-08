# Run 1 · wealth2track, properties and their money · 2026-11-08

First real run of the light version (stages 1 and 2), compared with what Phoenix's full extraction (A-00 to A-03, April 2026) recorded for the same area.

## How the run was set up

- A separate agent ran stages 1 and 2 with only the Palimpsest stage files, the templates and the original Flutter code (`lib/`, `pubspec.yaml`).
- It was told not to open anything Phoenix produced (signals, traces, screens, specs, certifications, the rebuild) or the git history.
- Scope: create, edit and delete a property; its expenses and additional expenses; the market-value and loan-to-value view; the portfolio overview totals. Borrowing create/edit/delete, auth, onboarding, funds and settings were out of scope.
- Effort: about 17 tool calls for stage 1 and 7 for stage 2. Output: 16 workflows, 59 rules (30 RESOLVED, 10 ASSUMED, 3 UNKNOWN, 16 CONFLICT) and 17 conflict questions, in three files.

## What each found in this area

| Finding | Palimpsest run 1 | Phoenix (April 2026) |
| --- | --- | --- |
| Property delete leaves additional expenses behind (`additionalProperties` vs `additionalExpenses`) | C-05 | Found, "confirmed bug" |
| Loan-to-value with market value 0 gives NaN or Infinity | C-16 | Found |
| Default exchange rate 1.42, unexplained | C-12 (UNKNOWN) | Found (direction questioned) |
| Address lookup restricted to Canada while the form allows USA and Other | C-04 (conflict) | Noted as a fact, not as a conflict |
| Property delete is not all-or-nothing | C-07 (question) | Noted, then set aside as "an implementation concern" |
| Rent increase limit saved as `rentIncreaseLimit`, read as `rentIncreaseLimitPercent` | **C-11** | Missed: recorded the field as existing |
| Market-value edits from the market-value list dropped unless the last details tab was Main | **C-17** | Not found |
| USD additional expenses not converted in the per-property total | **C-13** | Not found (only "selectedCurrency is local state") |
| Money with no frequency stored with factor 0, while the model treats a missing factor as 1 | **C-10** | Not found |
| Success messages shown whether or not anything was saved | **C-01** | Not found |
| Expenses already orphaned by past deletes need a migration decision | **C-06** | Not raised |
| Metrics service created twice, with a race between init and calculate | Recorded in stage 1 under Not traced | Found as a bug |
| Expense totals not updating until you navigate away (W2T-44) | Not raised | Found |
| Firestore reads per property in the overview | Not raised | Found (performance) |

## Reading

- **Light found what mattered, and more.** Both headline bugs Phoenix found here were found again, independently. Six findings in scope were new, including two data-loss or data-integrity problems (C-17, C-11).
- **Conflicts as questions changed the outcome twice.** Phoenix noticed the Canada restriction and the non-atomic delete, and resolved them itself. The light run turned both into questions for a person.
- **It missed things Phoenix caught.** The metrics-service race was seen but not raised as a rule, and the stale expense totals were not found.
- **Effort was small:** about 24 tool calls and three files for this area, against Phoenix's 57 files for the whole app (not a like-for-like count).

## Limits

- One run, one area, one model; Phoenix covered the whole app.
- The comparison above was made by the session that set up the run, which knew Phoenix's findings. Only the run itself was blind.
- The templates' examples came from this same app (LTV at zero, the 1.42 rate). The agent reports finding both in stage 1 from the code, with citations, but the examples should come from an unrelated module.
- C-10 (factor 0) is worth a closer look: it may be the real root of the "frequencyFactor: 0" story on PX-001's first page, which the audit could not source. Not yet checked.
