# CLAUDE.md

<!-- Keep this file short. It loads into every session. Put detail in docs/ and link to it. -->

## Project
**Claude Deck**: a free Stream Deck (Neo) plugin that shows the owner's running Claude Code sessions and lets them allow or deny permission requests from the deck, fed by Claude Code hooks.
Full product context: `docs/prd.md`. Current status and next step: `docs/state.md`.

## Commands
```bash
# Build:   npm run build                     # (expected, verify once scaffolded) Rollup bundle into the .sdPlugin folder
# Test:    npm test                          # (expected, verify once scaffolded) Vitest
# Run:     npm run watch                     # (expected, verify once scaffolded) rebuild + restart plugin; first time: npx streamdeck link <id>.sdPlugin
# Lint:    npm run lint                      # (expected, verify once scaffolded) ESLint + tsc --noEmit
# Pack:    npx streamdeck pack <id>.sdPlugin  # (expected, verify once scaffolded) .streamDeckPlugin installer
# Check: ./scripts/check.sh               # format, lint, test, build; the one definition of green
# Board: python3 scripts/dashboard.py --open   # live, read-only view of every feature
```

## Stack
TypeScript on Elgato's `@elgato/streamdeck` SDK (Node, npm, Rollup via `@elgato/cli`), Vitest. Claude Code `http` hooks POST to a localhost server inside the plugin. Why: `docs/adr/0002-typescript-elgato-sdk-http-hooks.md`.

## Conventions
<!-- Top 5 or so rules Claude would otherwise get wrong. Delete what doesn't apply. -->
- Core logic (session state, request queue, hook-settings merging) lives in plain TypeScript modules under `src/core/` with no Stream Deck or HTTP imports, so it unit-tests without the app. `src/actions/` holds thin Stream Deck actions; `src/server/` the localhost hook server.
- Naming: camelCase for code; Claude Code's hook JSON keeps its own snake_case (`session_id`, `tool_name`) at the boundary only, converted when parsed.
- Fail open to Claude: if the plugin can't decide (no key pressed, error, timeout), return no decision so Claude shows its normal prompt. Never auto-allow and never auto-deny.
- Only touch `~/.claude/settings.json` entries the plugin owns (marked by its hook URL); keep everything else byte-for-byte, and back the file up before the first write.
- Bind the hook server to `127.0.0.1` only. Never commit the per-install token; it lives in the plugin's global settings.
- Never commit secrets.

## Temporary files
- Name throwaway files in the repo `<name>.tmp.<ext>` (gitignored by `*.tmp.*`).
- Temp folders, downloads and logs go in the session scratchpad, not the repo.
- Delete files with a plain `rm` on the exact file. Never `rm -rf`.
- Leave `.superpowers/sdd/tasks/` in place.

## Where things live
| Need | Read |
|------|------|
| What we're building and why | `docs/prd.md` |
| How the system fits together | `docs/architecture.md` |
| Why a past decision was made | `docs/adr/` (index in `docs/adr/README.md`) |
| How to test | `docs/testing.md` |
| The app's look, prototypes, reviewing in Lavish | `docs/design.md`; the system itself is `design/` |
| Non-negotiable engineering rules | `.specify/memory/constitution.md` |
| What reviewers check, severity, loop limits | `REVIEW.md` |
| Feature specs, plans, tasks | `specs/NNN-feature-name/` |
| Working in and watching worktrees | `docs/worktrees.md` |
| The whole AI workflow as one diagram | `docs/ai-workflow.html` (open in a browser) |

Read these when the task needs them, not up front.

## Workflow

This project uses **Spec Kit for design** and **Superpowers for implementation**.

**1. Pick the track by size of change**
- **Feature** (new behavior, touches several files, worth remembering why): full Spec Kit track below.
- **Small change** (bug fix, tweak, refactor): skip Spec Kit. Use Superpowers directly (systematic-debugging for bugs, test-driven-development for everything else).
- **Spike / experiment**: Superpowers brainstorming, throwaway branch, no spec. If it survives, promote it to a feature.

**2. Feature track: design (Spec Kit owns this)**
0. Worktree first: from the main checkout on `master`, `git worktree add .worktrees/NNN-feature-name -b NNN-feature-name master` (NNN = next number unused in `specs/` and in `git worktree list`; use `git worktree add`, not Claude's own worktree tool, so names match). Then start a new Claude session with that folder as its working directory (in the Claude desktop app, start it yourself; elsewhere, tell the owner to run `claude` there) and continue the feature in that session. Every later step runs in that folder. Guide: `docs/worktrees.md`.
1. `/prototype` : any feature with new or changed UI gets a clickable prototype (`specs/NNN-*/mockup/prototype.html`) styled only with the design system in `design/`, tried by `reviewer-ux`, then reviewed by the owner in Lavish. Planning waits for the owner's explicit approval, recorded in `specs/NNN-*/mockup.md` (`Approved: YYYY-MM-DD`, `Design system: vN`, the screens and states approved). No design system yet? `/design-system` first. Guide: `docs/design.md`.
2. `/speckit-specify` : what and why, no tech choices. Give it the worktree's NNN, and `mockup.md` if there is one: every screen and state it lists gets an acceptance scenario.
3. `/speckit-clarify` : resolve ambiguity before planning
4. `/speckit-plan` : technical approach
5. `/speckit-tasks` : ordered task list
6. `/speckit-analyze` : consistency check across spec, plan, tasks

During these steps Spec Kit is the source of truth. Do not run Superpowers brainstorming or writing-plans on top of a Spec Kit feature; that creates a second, competing plan.

**3. Feature track: implementation (Superpowers owns this)**
Do NOT use `/speckit-implement`. Instead, for `specs/NNN-*/tasks.md`:
1. using-git-worktrees : confirms you're in the feature's worktree from design step 0, installs dependencies and checks the baseline is green
2. subagent-driven-development : one fresh subagent per task from tasks.md, test-first (red, green, refactor), spec-compliance review then code-quality review. When a task passes its review, check it off in tasks.md (`- [x]`) in that task's commit; Superpowers doesn't, and the dashboard and the approver count those boxes.
3. requesting-code-review, then `/review-panel` : every `.claude/agents/reviewer-*.md` in parallel, fixes for at most 3 rounds, then the approver writes `decision.md` (`REVIEW.md`, Review panel). On ESCALATE, stop and ask the owner. Findings left unfixed go in the PR's "Review findings not fixed".
4. Open a PR, wait for `gh pr checks --watch` to be green, then merge. The merge-gate hook (`.claude/hooks/merge-gate.py`) blocks merging on red, pushing to `main`/`master`, and local merges of a branch whose PR checks aren't green.

If implementation reveals the spec was wrong, stop and update spec.md/plan.md first, then continue.

**4. Before ending a session**
- Update `docs/state.md` (what changed, what's next, any blockers).
- Made a real architectural decision (library, data store, pattern, trade-off)? Add an ADR in `docs/adr/`.
- Changed how components connect? Update `docs/architecture.md`.
- Changed the workflow itself (this Workflow section, the review panel, the design or prototype skills, hooks, `scripts/check`, the dashboard, `ci.yml`)? Update `docs/ai-workflow.html` to match.
- Did a bug reach main? Log it under Escaped bugs in `docs/state.md` and close the loop.

## Learned rules

<!-- Short rules from owner corrections, dated. -->

- Stop only the processes you started, by PID. Never kill by image name (`taskkill /IM node.exe`, `chrome.exe`, `pkill node`): it ends the owner's browser and other sessions' servers. Run at most one dev server per worktree; agents that need the app at the same time serve their own copy of the build. (From back-room 007's review panel, 2026-10-20.)

