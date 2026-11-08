# Stage 4 · Verify

**Run this in a fresh session** that has not seen the build or its conversation. Give it only the `intent/` folder and the new code.

**Goal:** an independent check of the new code against the intent.

## Do

1. For each check in `intent/verify.md`, find the code that implements it.
2. Where you can, run an existing test or write one, and run it.
3. Record a result for every check:
   - **PASS**, with the evidence: a test name, or `file:line`.
   - **FAIL**, with what happens instead.
   - **NOT TESTABLE**, with why (needs live data, an external system, timing).
4. Also mark **FAIL** for a rule with no implementation at all, and for any behaviour in the new code that contradicts a rule.

## Never

- Fix the code in this session. Report only; fixes happen in the build session, then verify runs again, fresh.
- Read the build conversation or its notes.

## Write

The results table in `intent/verify.md`, dated, with the model and tool that ran it.
