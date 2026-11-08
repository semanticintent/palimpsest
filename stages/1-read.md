# Stage 1 · Read

**Goal:** know what the module *does*, before deciding what it is *for*.

**You get:** the path to one module, plus anything the person points you to (screens, docs, data samples, logs).

## Do

1. **Bound the module.** List the files in scope, and the files it calls or is called by. If the boundary is unclear, stop and ask.
2. **List the workflows.** Each thing a user or another system can make the module do, with its entry point: a route, handler, screen action, job or message.
3. **Trace each workflow** from entry to where it ends: a write, an external call, a response, a screen. Follow the code, and the UI path where there is one.
4. **Note what you could not trace:** dynamic dispatch, configuration, external calls, behaviour that depends on data or timing.

## Write

`intent/README.md`, from `templates/README.md`: Scope, Sources read, Workflows, Traces, Not traced.

## Rules

- Describe what the code does, not what it should do. No fixes, no opinions yet.
- Cite every step: `file:line` or `file › function`, or the screen name.
- Keep traces short: a numbered list or a small ASCII flow per workflow.

## Stop

Show the person the workflow list. Ask whether anything is missing, or out of scope, before stage 2.
