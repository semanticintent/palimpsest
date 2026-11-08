# Stage 3 · Gate

**Goal:** a person settles each conflict and unknown, or names who will. This is the only stage where a person decides.

**You get:** `intent/conflicts.md` and `intent/rules.md`.

## Do (the person, with the model's help)

1. For each entry in `conflicts.md`, record either:
   - **a decision**: what holds from now on, who decided, the date, and a one-line reason; or
   - **an owner**: who will decide, and that the affected rules stay blocked until then.
2. The model then updates `rules.md`: a decided CONFLICT or UNKNOWN becomes RESOLVED, with `Decision: C-0n` and a check that matches the decision.
3. The model writes `intent/verify.md` from `templates/verify.md`: every check from `rules.md`, plus a **Not testable from code** list (from the README's *Not traced* list and any rule whose check needs live data, timing or an external system).

## Rules

- Nothing is built against a rule that is still CONFLICT or UNKNOWN without an owner.
- A decision with no note is not a decision. Write the reason, even if it is "kept as the old system did".
- A skipped question is recorded as **skipped**, never as approved.

## Write

Decisions in `intent/conflicts.md`, updated statuses in `intent/rules.md`, and `intent/verify.md`.

## Then

Build forwards with any model or tool. Point it at `intent/README.md` first.
