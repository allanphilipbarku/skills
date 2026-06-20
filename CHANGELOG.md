# @allanphilipbarku/skills

## 0.1.1

### Patch Changes

- 76639b8: Harden and clarify flow-audit / setup-flow-audit:

  - **Fail fast on missing handoff skills** — the guard now checks `to-issues` / `triage` are
    installed before the walk, not after a full browser cascade has already run.
  - **Test-data disposability** — the audit creates real records and never tears them down, so setup
    now captures which DB it writes to and how it resets, the adapter template gains a `Disposability`
    field, and the orchestrator refuses to run against staging/prod.
  - **Filament-first honesty** — the description, REFERENCE, and README now state that the cascade /
    verification / adapter machinery is stack-agnostic while the concrete browser tactics are richest
    for Laravel + Filament/Livewire, with other stacks falling back to the general tactics plus the
    per-repo adapter.

## 0.1.0

### Minor Changes

- Initial release.
  - **flow-audit** — end-to-end audit of a feature flow by driving the real UI from the highest
    role down to the lowest, creating data through the interface (never seeders), verifying
    findings before filing, and handing them to `to-issues` / `triage`. Optionally dispatches AFK
    agents to fix, test, and commit.
  - **setup-flow-audit** — one-time per-repo setup that writes `docs/agents/flow-audit.md` (serve
    URL, per-role login, test-only access gates, dependency order, data rule, test command) and
    runs `setup-matt-pocock-skills` if its issue-tracker config is missing.
