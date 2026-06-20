---
name: setup-flow-audit
description: Configure a repo for the flow-audit skill by writing docs/agents/flow-audit.md — how to serve the app, log in per role, the role→account map, test-only access gates, the chronological dependency order, the data rule, and the test command. Hard-requires the matt-pocock issue-tracker setup and runs setup-matt-pocock-skills first if it is missing. Run before the first flow-audit in a repo, or when flow-audit reports missing browser or issue-tracker config.
disable-model-invocation: true
---

# Setup flow-audit

Scaffold the per-repo configuration that `flow-audit` needs to drive a real browser through a
feature's role-cascade. This is prompt-driven, not a script: explore the repo, present what you
found, confirm with the user, then write. Don't assume — read what exists.

## Prerequisite: the issue-tracker setup (hard requirement)

`flow-audit` files findings through `to-issues` / `triage`, which need the matt-pocock config.
**First** check `docs/agents/issue-tracker.md`:

- **Present** → reuse it. Note which tracker it records (GitHub / GitLab / local markdown).
- **Missing** → invoke `setup-matt-pocock-skills` now and let it write `issue-tracker.md`,
  `triage-labels.md`, and `domain.md`. Do not continue until that file exists. This is the hard
  dependency: flow-audit does not reimplement issue filing — it hands off to skills that read
  this config, so the audit adapts to whatever tracker the repo uses.

## Process

### 1. Explore (probe what you can detect; only ask for the rest)
- **App & framework** — `composer.json` / `package.json` / `Gemfile` (Laravel, Rails, Next, …).
- **Serve URL** — Herd (`herd sites`), `artisan serve`, `next dev`, Docker — resolve the base URL
  an authenticated session loads from.
- **Test command + DB** — how tests run, and any DB/env override the suite needs.
- **Roles** — the auth/roles tables, a role enum, or seeders.
- **Browser tool** — confirm a Playwright (or equivalent) MCP browser is available; flow-audit
  cannot run without one.

### 2. Present findings, then ask one section at a time (short explainer each)
- **Auth / login** — how does a test session start? A local auto-login route, seeded credentials,
  or a bypass. Capture any **test-only access gates** (e.g. an email-verified / password-changed
  flag) — these are env setup, explicitly **not** product bugs.
- **Roles → accounts** — one usable account per role, and how to discover more.
- **Dependency order** — the chronological chain: which role creates the prerequisites the next
  role needs (e.g. admin opens a call → applicant applies → reviewer reviews → admin awards).
- **Data rule** — confirm "create everything through the UI, never the seeder" (or the repo's
  equivalent constraint).

### 3. Write `docs/agents/flow-audit.md`
Use [flow-audit-adapter.md](./flow-audit-adapter.md) as the template. Fill every field; leave a
`TODO` only where the user genuinely doesn't know yet.

### 4. Register
Add an `### Flow audit` line under `## Agent skills` in `CLAUDE.md` (or `AGENTS.md` — edit
whichever already exists; never create the other). Point it at `docs/agents/flow-audit.md`. If an
`## Agent skills` block already exists, add the line in place; don't duplicate the block.

### 5. Done
Tell the user flow-audit is ready and that they can edit `docs/agents/flow-audit.md` directly
later — re-running this skill is only needed to change the login mechanism, roles, or tracker.
