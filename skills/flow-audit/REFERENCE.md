# Flow audit — reference

Browser-driving tactics, the discrimination checklist, the subagent contract, and a worked
example. Loaded by `flow-audit` for the walkthrough and verification steps.

## The role-phase subagent contract

The orchestrator runs the cascade as a sequence of subagents — one per role-phase, in dependency
order — so the heavy, stateful browser work never bloats the orchestrator's context.

Give each subagent:
- the **adapter** (`docs/agents/flow-audit.md`): base URL, login mechanism, access gates, its role's
  account, the data rule;
- the **`createdData`** from earlier phases (IDs of records the previous roles made through the UI);
- its **slice**: the exact steps this role owns.

Each subagent returns strict JSON-ish:
```
{ "createdData": { "<entity>": <id>, ... },   // real data it made via the UI, for the next phase
  "findings": [ { "title", "where" (url/screen), "severity", "kind" (bug|ux|polish|data),
                  "evidence" (DB row / console / screenshot), "verified" (how re-checked),
                  "false_positive_ruled_out" (what artifact you excluded) } ] }
```
A finding with no `verified` field is not ready to file — send it back through discrimination.

## Discrimination checklist (verify before filing)

Re-test every candidate a second way. Common false positives this catches:
- **Rapid-fill / morph races** — programmatic `fill()` of several reactive fields in a row can drop
  a value a *human typing + tabbing* would keep. Re-test by typing one field, blurring, waiting,
  then submitting. (Seen: a "budget didn't save" that saved fine with normal input.)
- **Pre-hydration snapshots** — a page captured before lazy widgets/JS hydrate looks "empty/broken".
  Wait for the content, or read the DOM/console, before judging. (Seen: an "empty dashboard" that
  was just the lazy-load skeleton.)
- **Your own automation mistakes** — e.g. calling a framework JS API on the wrong component. Confirm
  the error is the *app's*, not your driving. (Seen: a Livewire `.set()` aimed at the Sidebar.)
- **Data vs code** — a missing avatar / 404 on a seeded asset is usually data, not a code bug. Note
  it, don't file it as a defect.
- **Env setup vs product bug** — access gates you had to flip (verified email, password-changed) are
  setup, not findings.

If a candidate survives a genuine second check (DB row, server log, console error, repro with
human-like input), file it. Otherwise drop it and say why.

## Browser-driving tactics (Playwright MCP)

General:
- Prefer the **accessibility snapshot** over screenshots for finding elements; screenshot only to
  *judge* rendering. After a lazy page, **wait for hydration** (poll for expected text) before
  asserting "empty/broken".
- **Verify state in the database / server log**, not just the UI, after each mutating action.
- Read **console errors** and **network requests** — a recurring error often unmasks a real bug
  (or reveals it's just a benign 404). Distinguish a 200 Livewire/XHR from one returning HTML.

Framework-specific (extend per adapter):
- **Filament / Livewire**: layout components live in `Filament\Schemas\Components\`; row actions are
  `->recordActions()`; date-time pickers render a **readonly** display input (you can't type — drive
  the calendar, or set state via the page's Livewire component). To set form state directly, find the
  component by scanning `[wire:id]` for one whose `data` contains the field — **not** the first match
  (that's usually the Sidebar; `.set('data...')` on it throws). `FileUpload` consumes the temp file on
  ingest — re-saving a stale temp ref throws `UnableToRetrieveMetadata`; this is a real product-bug
  pattern worth probing, not a driving error.
- **Overlays** — a floating "feedback"/help button or sticky footer can intercept clicks on modal
  buttons (z-index). If a click is intercepted, that's both a thing to note and a reason to click the
  control via its accessible role/JS instead.
- **Auth** — apply the adapter's access gates first; a bounce to the login page usually means a gate
  (verified email / password-changed / role) rather than a broken page.

## Sizing the walk

- Smoke ("does the flow work?") → one pass per role, happy path, light verification.
- Audit ("find issues / polish") → exercise edge inputs, resume/draft paths, required-field timing,
  empty vs populated states, and read console/network at each step.
- Always **report what you did NOT cover** so the audit isn't mistaken for exhaustive.

## Worked example (STUFSO scholarship lifecycle, Laravel + Filament)

One real run, top role → bottom, all data made through the UI:
- **Admin** created a Scheme + opened a Call → **Student** completed the apply wizard and submitted
  → **Officer** verified documents, shortlisted, scheduled the interview, recorded a pass →
  **Admin** granted the award + issued the letter (PDF).

Confirmed findings filed (after discrimination): a submit-blocking `UnableToRetrieveMetadata` from
re-hydrating a consumed `FileUpload` temp ref; required documents enforced only at final submit;
no review summary on the final step; a generic "Create" submit label; a misleading call-name
auto-suggest; an "unknown" academic-year option; a feedback FAB intercepting modal buttons.

Dropped as false positives (did **not** file): a "budget didn't save" (rapid-fill race — saved
fine with human-like input), and an "empty dashboard" (lazy-load skeleton captured too early).

Crisp issues went `ready-for-agent` and were fixed AFK with tests, sequentially where they shared
files; design-level ones (review summary, interview scheduling) were filed `needs-info` for the
maintainer. That discipline — verify-before-file, group-by-root-cause, gate design work — is the
whole point of the skill.
