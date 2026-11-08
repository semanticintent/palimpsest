---
name: palimpsest
description: Read an existing module backwards into a small intent package (intent/README.md, rules.md, conflicts.md, verify.md) before rebuilding, porting or changing it. Use when the user wants to understand a legacy or unfamiliar module, extract its business rules, prepare a rewrite or migration, or verify a rebuilt module against the original intent. Runs one stage at a time and stops for the person between stages.
---

# Palimpsest

Read one module backwards: what it does, what it is for, which rules it must keep, and where the sources disagree. Then build forwards with anything, and verify in a fresh session.

The stage instructions and templates live in the Palimpsest repo, `stages/` and `templates/` (locally `~/workspace/palimpsest`, or github.com/semanticintent/palimpsest). Read the stage file before each stage. Copy the templates into the target repo's `intent/` folder as you go.

## Pick the stage

- No `intent/` folder yet → **Stage 1 · Read**: follow `stages/1-read.md`. Stop after showing the workflow list.
- `intent/README.md` exists, no `rules.md` → **Stage 2 · State**: follow `stages/2-state.md`. Stop after handing over `conflicts.md`.
- `conflicts.md` has open entries → **Stage 3 · Gate**: help the person record decisions or owners, then write `verify.md` (`stages/3-gate.md`).
- The person says the new code is built and this session has **not** seen the build → **Stage 4 · Verify** (`stages/4-verify.md`). If this session did see the build, say so and ask for a fresh session instead.

## Always

- One stage per turn; stop where the stage says to stop.
- Cite evidence for every step and rule. If the reason is not in the evidence, the status is UNKNOWN.
- Never fix code during stages 1, 2 or 4.
