---
"@allanphilipbarku/skills": patch
---

Harden and clarify flow-audit / setup-flow-audit:

- **Fail fast on missing handoff skills** — the guard now checks `to-issues` / `triage` are
  installed before the walk, not after a full browser cascade has already run.
- **Test-data disposability** — the audit creates real records and never tears them down, so setup
  now captures which DB it writes to and how it resets, the adapter template gains a `Disposability`
  field, and the orchestrator refuses to run against staging/prod.
- **Filament-first honesty** — the description, REFERENCE, and README now state that the cascade /
  verification / adapter machinery is stack-agnostic while the concrete browser tactics are richest
  for Laravel + Filament/Livewire, with other stacks falling back to the general tactics plus the
  per-repo adapter.
