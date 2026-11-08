# Stage 3 · Gate

**Goal:** a person settles each conflict and unknown, or names who will. This is the only stage where a person decides.

**You get:** `intent/conflicts.md` and `intent/rules.md`.

## Do (the person, with the model's help)

1. Go through `conflicts.md` highest harm first. For each entry record one of:
   - **Decision:** what holds from now on, who decided, the date, and a one-line reason;
   - **Owner:** who will decide, by when; the affected rules stay blocked;
   - **Skipped:** who, the date, and why; the affected rules stay blocked.
   Not every question has to be answered in one sitting. Answer the high-harm ones; the rest can be owned or skipped.
2. The model updates `rules.md`: a decided entry turns its rules RESOLVED, with `Decision: C-0n`, and writes their checks from the decision.
3. The model writes `intent/verify.md` from `templates/verify.md`:
   - every check for a RESOLVED or ASSUMED rule, worded as behaviour;
   - retired rules, to be checked for absence;
   - **Blocked**: rules whose questions were owned or skipped;
   - **Not testable from code**: from the README's *Not traced* lists, and any check that needs live data, timing or an outside system.

## Rules

- Nothing is built against a blocked rule.
- A decision with no reason is not a decision. "Kept as the old system did" is a reason.
- A skipped question is recorded as skipped, never as approved.

## Then

Build forwards with any model or tool. Point it at `intent/README.md` first.
