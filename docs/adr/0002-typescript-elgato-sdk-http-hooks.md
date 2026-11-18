# 0002. TypeScript plugin on the Elgato SDK, fed by Claude Code HTTP hooks

**Status:** Accepted
**Date:** 2026-11-17

<!-- Chosen by Claude after the owner said "just dive in"; revisit if any of it doesn't fit. -->

## Context
Claude Deck has to run inside the Stream Deck app and react to Claude Code events from several sessions at once. It must be free (no paid tools, services, or CI), run on Windows 11 with the Stream Deck app 7.x, and fall back safely to Claude's own prompt when the plugin isn't running. Claude Code hooks can be `command` hooks (a script) or `http` hooks (Claude POSTs the event JSON to a URL). A `PermissionRequest` hook can return an allow/deny decision. If an HTTP hook can't connect, Claude treats it as a non-blocking error and shows its normal prompt.

## Options Considered
1. **TypeScript + `@elgato/streamdeck` SDK, Claude Code `http` hooks to a localhost server inside the plugin**: Elgato's official, free SDK and CLI (`@elgato/cli`) handle scaffolding, linking, and packaging. HTTP hooks need no script on disk, and Claude's own fallback covers the plugin being down. Cons: the plugin has to run a small HTTP server and pick a port.
2. **TypeScript plugin + `command` hooks running a Node script that talks to the plugin**: works even where HTTP hooks are restricted. Cons: an extra script to install and keep in sync, and a Node process per event, which is slower.
3. **C# / .NET plugin (community SDKs)**: familiar language. Cons: unofficial SDKs, Elgato's tooling and docs target Node, and packaging is harder.

## Decision
Build the plugin in TypeScript on Elgato's `@elgato/streamdeck` SDK (Node runtime bundled with the Stream Deck app). The plugin runs an HTTP server on `127.0.0.1` with a fixed port, and registers Claude Code `http` hooks that POST to it (`PermissionRequest` for approvals; `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `Notification`, `Stop`, `SessionEnd` for status). Tests use Vitest. Build and packaging use the Elgato CLI and its default Rollup setup.

## Consequences
- Easier: one process holds all state; no helper scripts on disk; Claude's own fallback covers the plugin being down.
- Harder: the port can clash with another app, so it must be configurable and the hooks must match it. Requests should carry a per-install token so other local programs can't fake approvals.
- Editing `~/.claude/settings.json` is risky (the owner has other hooks there), so the hook installer needs tests and must only touch entries it owns.
- Follow-up: confirm HTTP `PermissionRequest` hooks work in the Claude desktop app's Code tab (PRD Open Questions).
