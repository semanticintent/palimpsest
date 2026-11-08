# Roadmap

Palimpsest is a method with one documented run. This page says what is done, what comes next, and the principles that stay fixed.

## Done

- **Concept** (November 2026): reading a module backwards into an intent package, drawn from Project Phoenix's extraction stages. See [`concept/`](concept/).
- **The light version:** four stages, four files, and a Claude Code skill. See [`stages/`](stages/), [`templates/`](templates/), [`skill/`](skill/).
- **Run 1** (2026-11-08): all four stages on one area of a small personal app, compared with Phoenix's earlier extraction. The fresh-session check found eight defects in the app's rebuild. See [`runs/2026-11-08-wealth2track-properties/`](runs/2026-11-08-wealth2track-properties/).
- **Revision 1** (2026-11-08): the method revised from what run 1 showed. See [`CHANGES.md`](CHANGES.md).

## Next

1. **Run 2:** the revised method on a different module, ideally one written by someone else, to test whether the changes help.
2. **A second worked example:** a report pipeline whose bug survived two refactors and was fixed by stating what each document was for, written up as a full intent package.
3. **PX-002, a two-way test:** the same unseen module rebuilt twice, once as a straight port from a minimal spec and once through Palimpsest, comparing defects carried over, findings, time and the record each leaves.
4. **An essay**, *Read backwards, build forwards*, on [semanticintent.dev](https://semanticintent.dev).

## Principles

- **Evidence on every rule.** A rule cites where it was read. If the reason is not in the evidence, the rule stays ASSUMED or UNKNOWN rather than guessed.
- **Conflicts are output.** Where the sources disagree, the method asks a person instead of deciding.
- **Verification is independent.** Stage 4 runs in a session that never saw the build.
- **Keep it small.** One module, one sitting, four files.
- **Examples come from personal projects**, published with their limits.
