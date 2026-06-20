# Allan's Claude skills

A personal, growing collection of [Claude Code](https://claude.com/claude-code) skills, packaged
the same way as [`mattpocock/skills`](https://github.com/mattpocock/skills) so it installs with the
`skills` CLI and coexists with other skill collections.

## Install

```bash
npx skills@latest add allanphilipbarku/skills
```

Pick the skills you want; they're copied into `~/.claude/skills/`. Re-run to update.

(Also works as a native Claude plugin: `/plugin marketplace add allanphilipbarku/skills` then
`/plugin install`. Or just copy a folder from `skills/` into `~/.claude/skills/`.)

## Skills

### flow-audit
End-to-end audit of a feature flow by **driving the real UI from the highest role down to the
lowest**, creating data through the interface (never seeders), discovering bugs / UX / polish gaps,
**verifying them before filing**, and filing them as issues — then optionally dispatching AFK agents
to fix, test, and commit. Chronological: the highest role creates the prerequisites the next role
needs.

### setup-flow-audit
One-time per-repo setup for `flow-audit`. Writes `docs/agents/flow-audit.md` (how to serve the app,
log in per role, the role→account map, test-only access gates, the dependency order, the data rule,
the test command) and registers it in `CLAUDE.md`. **Run this once per repo before `flow-audit`.**

## How it fits together

`flow-audit` deliberately does **not** reimplement issue tracking. It files findings through the
[`mattpocock/skills`](https://github.com/mattpocock/skills) `to-issues` / `triage` skills, which read
the per-repo config that `setup-matt-pocock-skills` writes (`docs/agents/issue-tracker.md` etc.). So
those skills are a **hard prerequisite** — `setup-flow-audit` runs `setup-matt-pocock-skills` for you
if its config is missing.

```
setup-matt-pocock-skills   →  docs/agents/issue-tracker.md   (where issues go: GitHub / GitLab / .scratch)
setup-flow-audit           →  docs/agents/flow-audit.md      (how to drive THIS app per role)
flow-audit                 →  walk roles → verify → to-issues → triage → (optional) AFK fix
```

The **skill files are generic and live here**; the **per-repo config lives in each target repo**.
Drop this collection into any project, run the two setups once, and `flow-audit` adapts.

### Requirements

These are **separate from the skill** — installing the skill (markdown) does not install them.

1. **Playwright (browser) MCP** connected to your agent. Either:
   - plugin: `/plugin install playwright@claude-plugins-official`, or
   - standalone: `claude mcp add playwright -- npx @playwright/mcp@latest`

   then the browser binary once: `npx playwright install chromium`.
   For headless / background / cron runs, make sure the MCP is loaded in *that* runtime too.
2. **`mattpocock/skills`** installed: `npx skills@latest add mattpocock/skills` (provides
   `to-issues` / `triage` and the `setup-matt-pocock-skills` config flow-audit files through).

## License

MIT © Allan Philip Barku
