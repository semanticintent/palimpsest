# Stage 2 · State

**Goal:** write down what each workflow is *for*, the rules it must keep, the evidence for each, and every place the sources disagree.

**You get:** `intent/README.md` from stage 1, and the module.

## Write rules as needs, not as the old screens

A rule says what must hold for the user or the data: "a saved value is the value shown back", "deleting a property removes everything that belongs to it". Button labels, layout, navigation order and field names are how the old system did it; they stay in the README's traces. Make them rules only when the person says the UI must be kept. Word every check as behaviour, and put the old structure after it as a hint: *(old: field `limitPercent`)*.

## Do

1. **Purpose.** For each workflow, one sentence from the user's side. Add it to the README's workflow table.
2. **Rules.** Write each in `intent/rules.md`, in the format of `templates/rules.md`. A rule that spans workflows (currency, ownership of data) lists them all.
3. **Status** for every rule. Sources are: the code, its on-screen text, code comments, names, stored data, docs, people. Code and its own on-screen text count as two.
   - **RESOLVED**: two sources agree and the purpose is clear.
   - **ASSUMED**: one source, but the purpose is plain from what the user sees or from naming.
   - **UNKNOWN**: nothing shows why, and the answer would change how it is rebuilt (a magic value, a default, a missing guard).
   - **CONFLICT**: sources disagree, or the code contradicts the evident purpose: a likely bug, or behaviour someone relies on.
4. **Questions.** Every CONFLICT and UNKNOWN gets an entry in `intent/conflicts.md`: one question, a follow-up only if the first answer needs it, and a **harm** level (high: data loss or wrong money; medium: wrong display or a confusing flow; low: cosmetic). Order the file by harm. If existing data is affected, say so in the entry.
5. **Checks.** Given / When / Then for every RESOLVED and ASSUMED rule. For CONFLICT and UNKNOWN rules write `Check: after C-0n`; stage 3 writes it from the decision.
6. **Disposition** per rule:
   - **transform**: the need stays; *how* is free to change.
   - **preserve**: must behave exactly as now, usually because stored data or another system depends on it.
   - **retire**: no longer needed. Only with evidence (unreachable, never read).

## Never

- Invent a reason. If the *why* is not in the evidence, the status is UNKNOWN or ASSUMED, never RESOLVED.
- Smooth over a disagreement. Log it, even when one side looks obviously right.
- Fix the code.

## Write

`intent/rules.md`, `intent/conflicts.md`, and the purpose column in `intent/README.md`.

## Stop

Hand `conflicts.md` to the person, highest harm first. Nothing is built until stage 3 is done.
