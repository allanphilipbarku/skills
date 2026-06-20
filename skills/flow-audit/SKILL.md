---
name: flow-audit
description: End-to-end audit of a feature flow by driving the real UI from the highest role down to the lowest, creating data through the interface (never seeders), discovering bugs / UX / polish gaps, verifying them, and filing them as issues. Optionally dispatches AFK agents to fix, test, and commit. Use when the user wants an e2e flow check, a role-by-role walkthrough, to find issues by actually using a feature, or a chronological audit from the highest role to the lowest. The cascade structure is stack-agnostic, but the concrete browser-driving tactics are richest for Laravel + Filament/Livewire; other stacks fall back to the general tactics plus the per-repo adapter. Requires a Playwright (browser) MCP plus the setup-flow-audit and matt-pocock config.
---

# Flow audit

Drive a feature's whole lifecycle through the browser — highest role first, using whichever role
owns each step — to discover *real* issues, verify them, and file them where the repo tracks work.
[REFERENCE.md](REFERENCE.md) holds the browser-driving tactics, the discrimination checklist, the
subagent contract, and a worked example. Read it before step 3.

## Guard (setup rule, then run)
- **No browser MCP connected** (no Playwright `browser_*` tools available) → stop and tell the user
  to connect one, with the exact command: `/plugin install playwright@claude-plugins-official` (plugin)
  or `claude mcp add playwright -- npx @playwright/mcp@latest` (standalone), then `npx playwright install
  chromium`. The skill is just instructions — it can't install the MCP. In a headless / background /
  cron run, confirm the MCP is loaded in *that* runtime, not just your interactive session.
- Missing `docs/agents/flow-audit.md` → tell the user to run **`setup-flow-audit`**. Stop.
- Missing `docs/agents/issue-tracker.md` → run **`setup-matt-pocock-skills`** (or `setup-flow-audit`,
  which triggers it). Stop.
- **`to-issues` / `triage` not installed** (the handoff skills from `mattpocock/skills`, separate
  from their config above) → step 5 has nothing to hand off to. Confirm both are available *before*
  the walk, not after. If absent: `npx skills@latest add mattpocock/skills`. Stop. (Fail here, cheaply
  — not after a full browser cascade has already run.)

Read both adapters before doing anything.

## Process

### 1. Map the flow
From `docs/agents/flow-audit.md` build: roles → the steps each can perform → the chronological
dependency chain (the highest role creates the prerequisites the next role needs). Confirm the
feature / scope with the user.

### 2. Prepare access
Apply the adapter's **test-only access gates** (env setup — record them as such, never as bugs).
Resolve the base URL and the per-role accounts. Confirm the target is the adapter's **disposable
DB** — the cascade creates real records and never tears them down; refuse to run against staging/prod.

### 3. Walk the cascade — one role-phase at a time, in dependency order
Dispatch **one subagent per role-phase** (keeps the orchestrator's context lean). Each subagent
logs in as its role, performs its slice **through the UI**, creating real data — **never the
seeder** — and returns `{ findings[], createdData{ids} }`. Thread `createdData` into the next
phase so dependent steps use real, hand-made data. See REFERENCE.md for the contract and the
per-framework browser tactics (readonly pickers, targeting the right Livewire/component, file
uploads, lazy-load waits, modal/overlay interception, DB/console verification).

### 4. Discriminate — verify before filing (the quality bar)
For every candidate issue, **re-test it a second way** before believing it: human-like input,
wait for hydration, check the DB / console / network. Drop anything that turns out to be a
test-harness artifact (rapid-fill races, pre-hydration snapshots, your own automation mistakes).
A skill that files false issues is worse than one that files none. See the REFERENCE checklist.

### 5. File via to-issues + triage
Hand each confirmed finding to **`to-issues`** (it writes to whatever tracker
`docs/agents/issue-tracker.md` records — GitHub / GitLab / local `.scratch/`). Then **`triage`**:
crisp + fully specified → `ready-for-agent`; design-level or ambiguous → `needs-info`. Group
coupled findings (shared files / one root cause) into a single issue, with acceptance criteria
and the adapter's test command.

### 6. (Optional) AFK fix + commit
Only if the user asked to fix-and-commit. For `ready-for-agent` issues, dispatch fix agents
**grounded in the root cause + acceptance criteria**, fixing **with a test** (use `tdd` /
`diagnose`). Run the adapter's test command, mark the issue done, commit per coherent slice.
Run agents **sequentially when issues touch the same files** (avoid working-tree / DB collisions);
parallel worktrees only when the issues are file-disjoint. Leave `needs-info` issues for the human.
