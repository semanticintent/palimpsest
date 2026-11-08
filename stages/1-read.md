# Stage 1 · Read

**Goal:** know what the module *does*, before deciding what it is *for*.

**You get:** the path to one module, plus anything the person points you to (screens, docs, data samples, logs).

## Do

1. **Bound the module.** List the files in scope, what it calls out to, and what reaches it: callers, routes, navigation into its screens, wiring.
2. **List the workflows.** A workflow is one action a user or another system can take that ends in a different write, external call or screen. Two actions with the same code path and the same ending are one workflow. Give each its entry point.
3. **Trace each workflow** from entry to where it ends. Follow the code, and the UI path where there is one. Keep it short, except where the arithmetic or the data shape *is* the behaviour; then spell it out.
4. **Record the data touched:** the stores, tables or collections, and the stored field names each workflow reads or writes.
5. **Record what you could not trace**, sorted into: out of scope; platform and external services; timing and shared state; unreachable code.

## Write

`intent/README.md`, from `templates/README.md`.

## Rules

- Describe what the code does, not what it should do.
- Cite every step: `file:line` or `file › function`, or the screen name.
- A defect that is a plain code fact (a wrong field name, a missing branch, an unawaited call) goes into the trace as a fact, with its citation and no verdict. Stage 2 decides what it means.

## Stop

If the person is here, show the workflow list and ask whether anything is missing or out of scope.

If no one can answer now, make the call yourself, write it under **Calls to confirm** in the README with the reason, and go on.
