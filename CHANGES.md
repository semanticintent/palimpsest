# Changes to the method

## 2026-11-08 · after run 1 (wealth2track, properties)

Each change answers something run 1 showed. Details in `runs/2026-11-08-wealth2track-properties/COMPARISON.md`.

| Change | Why |
| --- | --- |
| Rules describe needs, not the old screens; checks are worded as behaviour with old names as a hint | 6 of the 14 stage-4 failures were the redesign showing through rules that recorded old UI |
| "Workflow" defined: one action that ends in a different write, call or screen | Stage 1 had to invent its own rule for splitting and merging |
| Stop step works without a person: decide, record under *Calls to confirm*, continue | "Stop and ask" stalled a non-interactive run |
| README gains *Data touched*; *Not traced* split into four kinds | Several conflicts lived in stored field names; unreachable code had no home |
| Plain code-fact defects go into traces without a verdict | The line between describing and judging was unclear |
| Sources defined; code and its on-screen text count as two | RESOLVED required "other sources" that a code-only run did not have |
| ASSUMED and UNKNOWN separated by whether the answer changes the rebuild | The two overlapped; taken literally, UNKNOWN would swamp the questions |
| Conflicts carry a harm level and an *Existing data* line, ordered by harm | The run ordered by harm on its own; data migration had no place |
| One question, plus a follow-up only when needed; an explicit *Open* state | Some questions needed two parts; there was no form for "not yet asked" |
| CONFLICT and UNKNOWN rules get `Check: after C-0n` | Writing checks for each possible answer meant guessing the answers |
| Confidence field dropped; rules may span several workflows | Confidence mostly repeated the status; currency and ownership rules span workflows |
| Disposition definitions sharpened (preserve = something depends on exact behaviour) | Preserve versus transform was a judgement call |
| Stage 4: judge the rule, not only its scenario; formats don't count; unreachable starting states handled; tests go in `intent/verify-tests/` or reasoning is stated | Two results depended on the verifier's choice; writing tests into the target was not allowed |
| verify.md gains a Note column and an evidence count (tests run vs code read) | Cells were cramped; it was unclear how much was actually run |
| Template examples now come from an invented invoicing module | The old examples came from the same app the run read, which could hint the answers |
