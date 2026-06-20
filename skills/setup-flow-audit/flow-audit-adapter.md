<!--
TEMPLATE for docs/agents/flow-audit.md (the per-repo flow-audit adapter).
setup-flow-audit copies this into the target repo and fills it in. Keep it factual and short;
flow-audit reads it to know how to drive THIS app. Edit it directly whenever the app changes.
-->

# Flow audit — repo adapter

## App & serve
- Framework: <e.g. Laravel 13 + Filament v5>
- Base URL: <e.g. https://app.test (Herd) — resolve with the project's URL tool if it has one>
- Browser tool: <e.g. Playwright MCP>

## Auth / login (how a test session starts)
- Mechanism: <e.g. local-only auto-login route `/panel/auto/{uuid}/{panel}`; or seeded creds; or a bypass>
- **Test-only access gates (env setup, NOT product bugs)**: <e.g. account needs `password_change_status=true`
  + a verified email or it bounces to login>. Apply these before driving; record them as setup, never file them.

## Roles → accounts
| Role | Account (login handle / uuid) | How to discover more |
|------|-------------------------------|----------------------|
| <highest role> | <handle / uuid> | <query / rule> |
| <…>            | <…>             | <…> |
| <lowest role>  | <handle / uuid> | <query / rule> |

## Dependency order (chronological — highest role creates prerequisites first)
1. <role> — <what it must create for the next role> (e.g. admin: create scheme → open call)
2. <role> — <its slice> (e.g. applicant: apply + submit)
3. <role> — <its slice> (e.g. reviewer: screen → shortlist → interview)
4. <role> — <final slice> (e.g. admin: award + issue letter)

## Data rule
- <e.g. Create all data through the UI. NEVER run the scenario/demo seeder. Using an existing
  account to log in is fine; the data under test must be created by hand through the interface.>

## Tests
- Command: <e.g. `DB_DATABASE=app_db php artisan test --compact tests/Feature/<area>`>
- Notes: <e.g. transaction-rollback isolation; any flag a fix agent must pass>

## Issue tracker
- Inherited from `docs/agents/issue-tracker.md` (matt-pocock setup). flow-audit hands findings to
  `to-issues` / `triage`, which read that file. Do not duplicate tracker config here.
