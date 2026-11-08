# Stage 4 · Verify

**Run this in a fresh session** that has not seen the build or its conversation. Give it only the `intent/` folder and the new code.

**Goal:** an independent check of the new code against the intent.

## Do

1. For each check in `intent/verify.md`, find the code that implements it.
2. Run the existing tests. Where a check needs a new test, write it **outside the target code**, in `intent/verify-tests/`, or reason from the code; both are acceptable evidence if you say which.
3. Record a result for every check:
   - **PASS**, with the evidence: a test name, or `file:line`.
   - **FAIL**, with what happens instead and where.
   - **NOT TESTABLE**, with why: needs live data, an outside system or timing that neither code nor tests can establish.

## How to judge

- **Judge the rule, not only its scenario.** If the check's own scenario passes but the rule is broken elsewhere, it is a FAIL; say "scenario passes; rule broken at …".
- **Behaviour, not structure.** The stack may have changed. Field names, labels and layout in a check are hints; judge what the user would see or what the data would hold.
- **Formats don't count** (date style, currency symbols, decimals) unless the rule is about the format.
- **Unreachable starting states.** If the new app cannot produce the check's starting state through its own screens, judge whether the rule holds for every state it *can* reach and for imported data; note the limit.
- A rule with no implementation at all, or new behaviour that contradicts a rule, is a FAIL.
- Retired rules pass if the thing is absent or harmless.

## Never

- Fix the code in this session. Report only; fixes happen in the build session, then verify runs again, fresh.
- Read the build conversation, its notes, or anyone else's conclusions.

## Write

The results in `intent/verify.md`: result, evidence and any note per check, then the Run section, dated, with the model and tool that ran it and how much of the evidence was run tests versus reasoning from code.
