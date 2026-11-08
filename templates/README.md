# <Module name> — intent

> Read backwards with Palimpsest on <date>. Status: stage <n> of 4.

**For agents:** before changing anything in this module, read `rules.md`. Keep every RESOLVED rule; anything not covered by a rule is free to change. If a change would break a rule, or a rule seems wrong, add an entry to `conflicts.md` instead of guessing.

## Scope

- In scope: <files / folders>
- Calls out to: <files, services>
- Reached from: <callers, routes, navigation into its screens, wiring>

## Sources read

- <code paths>
- <screens, docs, data samples, logs, people asked>

## Workflows

| ID | Workflow | Entry point | Purpose (stage 2) |
| --- | --- | --- | --- |
| W-01 | <name> | `<file › function>` or <screen action> | <one sentence, user's side> |

## Data touched

| Store / table | Fields read or written | By |
| --- | --- | --- |
| <name> | `<field>`, `<field>` | W-01, W-03 |

## Traces

### W-01 <name>

1. <step> — `file:line`
2. <step> — `file:line`
3. Ends: <write / call / response / screen>

## Not traced

- **Out of scope:** <areas left out, and why>
- **Platform and external services:** <framework behaviour, outside calls>
- **Timing and shared state:** <unawaited calls, delays, app-wide state>
- **Unreachable code:** <files or functions nothing calls>

## Calls to confirm

- <a scope or workflow call made without the person, and why>
