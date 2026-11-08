# Resume here

Last session: 2026-11-08. Light version built, run once end to end on wealth2track (found 8 live-app defects in the PX-001 rebuild), and revised from that run.

## Where things stand

- Concept, draft 1, with two worked examples: PX-001 (wealth2track) and the StratIQX report pipeline.
- The working copy is the Claude Docs version; `concept/palimpsest-concept.md` is a snapshot. Re-export after big edits.
- Name is a working name; a name check is still to do.
- The light version (README, `stages/`, `templates/`) supersedes the concept's 11-file package. Related work: Reversa is the closest project; staying small is the intended difference.

## Next steps (from the concept doc)

1. ~~Write the stage instructions~~ done 2026-11-08 as the light version (4 stages, 4 files).
2. ~~First real run~~ done 2026-11-08: `runs/2026-11-08-wealth2track-properties/` (all 4 stages, see COMPARISON.md). Lessons folded into the method the same day (CHANGES.md).
2b. **Run 2:** the improved method on a different module, ideally one not built by Michael, to test the changes.
3. Write the StratIQX case up as a full intent package, beside PX-001.
4. PX-002: a two-way test on a module neither run has seen (straight port from a minimal spec vs Palimpsest then build).
5. Essay for semanticintent.dev: *Read backwards, build forwards*.

## Ground rules

- Personal-project examples only; nothing from work, not even with names changed.
- Every rule cites its evidence; unresolved rules stay ASSUMED or UNKNOWN rather than guessed.
- Keep it small: one module, one sitting.
