# Stage 2 · State

**Goal:** write down what each workflow is *for*, the rules it must keep, the evidence for each, and every place the sources disagree.

**You get:** `intent/README.md` from stage 1, and the module.

## Do

1. **Purpose.** For each workflow, one sentence from the user's side: what it is for. Add it under the workflow in `README.md`.
2. **Rules.** Extract the statements that must hold. Write each in `intent/rules.md`, in the format in `templates/rules.md`.
3. **Status** for every rule:
   - **RESOLVED**: the code and the other sources agree, and the purpose is clear.
   - **ASSUMED**: inferred from a single source; plausible, not confirmed.
   - **UNKNOWN**: the rule is there, but nothing says why.
   - **CONFLICT**: sources disagree, or the code contradicts the evident purpose. A likely bug, or behaviour someone relies on.
4. **Questions.** For every CONFLICT and UNKNOWN, add an entry to `intent/conflicts.md` with the one question a person must answer.
5. **Checks.** Give every rule a Given / When / Then check. For CONFLICT rules, write one check per possible answer, or leave it to stage 3.
6. **Disposition** per rule: **transform** (rebuild it, free to change *how*), **preserve** (must behave exactly as now), **retire** (no longer needed; only with evidence).

## Never

- Invent a reason. If the *why* is not in the evidence, the status is UNKNOWN.
- Smooth over a disagreement. Log it as a CONFLICT, even when one side looks obviously right.
- Fix the code.

## Write

`intent/rules.md`, `intent/conflicts.md`, and the purpose lines in `intent/README.md`.

## Stop

Hand `conflicts.md` to the person. Nothing is built until stage 3 is done.
