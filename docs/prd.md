# Claude Deck Product Requirements

<!-- Product-level PRD: the whole thing, not one feature. Individual features get their own spec via /speckit-specify in specs/. Keep this to what stays true across features. -->

**Status:** Draft | **Last updated:** 2026-11-17

<!-- Drafted on 2026-11-17 from the setup conversation. The owner asked to "just dive in", so several items below are Claude's proposals rather than the owner's stated choices; they are marked "(proposed)" and are open to change. -->

## Problem
When several Claude Code sessions run at once, the owner has to keep switching windows to see which session is working, which is done, and which is stuck waiting for a permission prompt. A paid Stream Deck plugin (Claude Code Approver) does this, but the owner doesn't want to pay for it. Claude Code already exposes everything needed through hooks, so a small free plugin can do the same job.

## Users
| User | What they need | Notes |
|------|----------------|-------|
| The owner | Glance at a Stream Deck Neo to see every running Claude Code session's state, and approve or deny permission requests with one tap | Windows 11, Stream Deck app 7.x, Claude Code native (not WSL), mostly in the Claude desktop app's Code tab |
| Other Stream Deck owners (proposed) | The same, installed from the public repo | Repo is public; supporting others is a nice-to-have, not a v1 requirement |

## Goals
- A permission request never sits unnoticed: within a second of a session asking, the deck shows it, and one tap answers it.
- The owner can tell at a glance which sessions are working, waiting on them, or finished, without switching windows.
- Nothing breaks when the plugin isn't running: Claude Code falls back to its normal prompt.
- It costs nothing to build, run, or distribute.

## Non-Goals
<!-- The most useful section for keeping AI agents from over-building. Be explicit. -->
- No Elgato Marketplace listing in v1; install by sideloading the built plugin.
- No plan or usage-limit tracking (it relies on an undocumented Anthropic endpoint and a long-lived token).
- No sending prompts or typing into Claude from the deck; it answers permission requests only.
- No remote or networked use: the plugin listens on localhost only and never talks to other machines.
- No macOS or WSL support work in v1 (the code shouldn't block macOS, but it isn't tested there).
- No changes to the owner's global permission rules; "Always allow" (if built) is scoped to the current session.

## Core Capabilities
<!-- High level. Each one usually becomes one or more Spec Kit features. Release (v1 / v2 / later) keeps future ideas in the list without building them yet. -->
| # | Capability | Release | Priority | Spec |
|---|------------|---------|----------|------|
| 1 | Approve or deny a pending permission request from the deck (Allow / Deny keys; the pending request shows which session and tool) | v1 | Must | not started |
| 2 | Hook setup: the plugin adds its hooks to `~/.claude/settings.json` on first run and removes them cleanly, leaving the owner's other hooks untouched | v1 | Must | not started |
| 3 | Session status keys: one key per running session, labeled with its project folder, colored by state (working, needs approval, waiting for input, done, idle) | v1 | Must | not started |
| 4 | A bundled "Claude" profile for the Neo, and switching to it automatically when an approval is pending (proposed) | v1 | Should | not started |
| 5 | "Always allow" for the rest of the session | v1 | Could | not started |
| 6 | Context-window fill bar on each status key | v2 | Could | not started |
| 7 | Packaged `.streamDeckPlugin` release on GitHub for others to install | later | Could | not started |

Capability #1 is first: it is the smallest piece that delivers the core value and proves the riskiest assumption (see Open Questions).

## Success Measures
- The owner answers permission requests from the deck in daily use instead of from the app.
- With the plugin stopped, Claude Code sessions behave exactly as before (no hangs, no denied requests).
- Zero cost: no paid services, tooling, or CI.

## Constraints and Assumptions
- Hardware: Stream Deck Neo (8 LCD keys; the info bar and touch points can't be drawn on by plugins). Stream Deck app 7.6.
- Must be free: Elgato SDK and CLI, Node.js, and GitHub Actions on a public repo are all free.
- Assumption: Claude Code's `PermissionRequest` hook can allow or deny a request, and an unreachable HTTP hook is a non-blocking error, so Claude shows its normal prompt. (Confirmed from the hooks docs on 2026-11-17.)
- Assumption (unverified): `PermissionRequest` hooks run for sessions in the Claude desktop app's Code tab, not just the terminal CLI. If not, v1 is terminal-only.
- Claude Code must be installed natively on Windows; WSL keeps its settings in a different home folder.

## Open Questions
- [ ] Does the `PermissionRequest` hook fire, and is its decision honored, for sessions in the Claude desktop app? (Spike this before building capability #1 in full.)
- [ ] How much of the request should the deck show before you tap: just session and tool name, or enough of the command to decide without looking at the screen? (Asked during setup; not yet answered.)
- [ ] When the Neo has more sessions than free keys, which sessions get a key?
- [ ] Should a request time out to "ask in the app" after a while, or wait as long as Claude allows?
