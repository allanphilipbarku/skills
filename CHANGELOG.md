# @allanphilipbarku/skills

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
