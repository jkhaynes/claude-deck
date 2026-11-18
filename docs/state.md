# Project State

<!-- Session-to-session memory. Claude updates this at the end of each working session. Keep it short: replace old content instead of appending forever. The exception is Escaped bugs, a running log. -->

**Last updated:** 2026-11-17

## Current Focus
Project setup complete; constitution next.

## Recently Done
- Spike: desktop-app sessions fire `PermissionRequest` HTTP hooks and obey allow/deny; app prompt and hook race; `SessionStart` needs a command hook (`docs/spikes/2026-11-17-desktop-app-hooks.md`)
- Stack chosen: TypeScript + Elgato SDK + Claude Code HTTP hooks (ADR 0002)
- PRD drafted (many items marked "(proposed)": the owner said "just dive in")
- Repo created from project-template (public)

## Next Up
1. `/speckit-constitution` with the prepared input below.
2. Design system (`/design-system`): the keys are UI, even if tiny. Skipped during setup.
3. Worktree `001-approve-from-deck` for capability #1, then `/prototype` (key states), then `/speckit-specify`.

## Blockers / Open Questions
- How the plugin learns a pending request was answered in the app instead (connection close? next `PostToolUse`?) needs designing in capability #1.
- How much of a pending request the keys should show is still undecided (see PRD).

## Escaped bugs
<!-- Bugs that reached main. Each gets a regression test (red first) and, if a reviewer should have caught it, a line in REVIEW.md's Learned review rules. -->

## Notes for Next Session
Prepared constitution input (proposed; owner to keep, cut or reword):
1. Fail open to Claude: the plugin never auto-allows or auto-denies; any failure, timeout or ambiguity returns no decision so Claude shows its own prompt. (PRD goal: nothing breaks when the plugin is down.)
2. Local only: the hook server binds to 127.0.0.1, requires the per-install token, and nothing leaves the machine. (PRD non-goal: no remote use.)
3. Leave the owner's Claude settings alone: only plugin-owned hook entries in `~/.claude/settings.json` are ever added or removed, and the file is backed up before the first write. (ADR 0002 consequences.)
4. Free only: no paid services, APIs, tooling or CI. (PRD constraint.)
5. Core logic in `src/core/` has no Stream Deck or network imports and is covered by unit tests. (CLAUDE.md conventions.)

Setup notes: the Elgato CLI isn't installed globally; use `npx @elgato/cli`. The check scripts (`scripts/check.ps1`, `scripts/check.sh`) and CI setup step in `.github/workflows/ci.yml` still have template placeholders; fill them in when the first feature scaffolds the npm project.
