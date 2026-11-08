# Palimpsest

*Working name.* Read an existing module backwards — code, screens, data — into a small **intent package** that any model or tool can build forwards from.

A palimpsest is a manuscript page scraped and written over, where the earlier text can still be read beneath the new one. A legacy module is the same.

Introduction, the side-by-side with Project Phoenix, and run 1's results: **[semanticintent.ai/palimpsest](https://semanticintent.ai/palimpsest)**.

## The light version: four stages, four files

| Stage | Who | Writes |
| --- | --- | --- |
| [1 · Read](stages/1-read.md) | model | `intent/README.md`: scope, workflows, traces, what could not be traced |
| [2 · State](stages/2-state.md) | model | `intent/rules.md`, `intent/conflicts.md`: rules with evidence and status, disagreements as questions |
| [3 · Gate](stages/3-gate.md) | person | decisions in `conflicts.md`, then `intent/verify.md`: the checks |
| [4 · Verify](stages/4-verify.md) | fresh model session | results in `verify.md`: pass, fail or not testable |

Between stages 3 and 4, build forwards with any model or tool.

One module, one sitting. No runtime, no index file, no vendor: plain Markdown instructions and templates any capable model can follow.

## How to run it

1. In the module's repo, start a session with any coding agent and give it `stages/1-read.md`, the templates, and the module path.
2. Review the workflow list it shows you, then give it `stages/2-state.md`.
3. Answer the questions in `intent/conflicts.md` (`stages/3-gate.md`).
4. Build with whatever you use.
5. Open a **new** session, give it only `intent/` and the new code, and `stages/4-verify.md`.

In Claude Code, `skill/SKILL.md` wraps the same steps as a skill (copy the `skill` folder into your skills directory as `palimpsest`).

## What it takes from earlier work

- **Phoenix:** extraction before building, and certification by a separate agent.
- **Governance-Driven Design:** a status on every rule; unresolved questions get an owner.
- **Strata:** transform, preserve or retire.
- **Case PX-001:** decisions written down, skips recorded as skips, verification in a fresh session.

## Related work

Reading code backwards into specs is an active area in 2026. The closest project is [Reversa](https://github.com/sandeco/reversa), a large multi-agent framework with a similar confirmed / inferred / gap marking. Palimpsest aims to stay small: one module, four files, an explicit conflict log with decisions, and verification in a session that never saw the build.

## More

- Concept, draft 1: [`concept/palimpsest-concept.md`](concept/palimpsest-concept.md) (the light version supersedes its 11-file package)
- Where to pick up: [`NEXT.md`](NEXT.md)
- Examples come from personal projects only.
