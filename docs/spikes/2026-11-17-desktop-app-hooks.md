# Spike: do Claude Code hooks work for desktop-app sessions?

**Date:** 2026-11-17 | **Claude Code:** 2.1.325 | **Result:** Yes, with one twist

## Question
Does a `PermissionRequest` HTTP hook fire for sessions in the Claude desktop app's Code tab, and does Claude obey its decision? (PRD open question, the riskiest assumption for capability #1.)

## Setup
A throwaway Node server on `127.0.0.1:47823` logged every hook POST and answered by command text: `spike-allow` → allow, `spike-deny` → deny, `spike-wait` → hold 8s then allow, anything else → no decision. Hooks were added to `~/.claude/settings.json` temporarily: `PermissionRequest` (matcher `Bash`, type `http`), `Notification` (`http`), `SessionStart` (`command`, see below). A fresh desktop session in **Ask permissions** mode ran four `node -e "console.log('spike-…')"` commands. Afterwards the hooks were removed and the settings file restored byte-for-byte.

## Results
| Command | Server answered | In the session |
|---------|-----------------|----------------|
| spike-allow | allow (immediately) | Ran with no prompt, about 1s |
| spike-deny | deny | Refused: "Denied by claude-deck spike" |
| spike-wait | allow after 8s | Ran after 4s, before the server answered; the owner approved it in the app first (to confirm) |
| spike-normal | no decision | Normal prompt; ran when the owner approved it |

## Findings
1. **Desktop-app sessions fire `PermissionRequest` HTTP hooks, and allow/deny are obeyed.** v1 can cover desktop sessions.
2. **The app's own prompt and the hook race; whichever answers first wins.** The deck is an extra place to answer, not a replacement. The plugin must clear a pending key when the request gets answered in the app. Likely signals: the HTTP connection closing, or that session's next `PostToolUse`/`Stop` event. To be designed in capability #1.
3. **`SessionStart` accepts only `command` (and `mcp_tool`) hooks, not `http`.** An HTTP `SessionStart` hook is silently ignored. Use a tiny command hook (for example `curl ... --data-binary @-` to the plugin), or treat the first event from an unknown `session_id` as its start.
4. **No events at all in auto mode for tool calls the classifier allows**: `PermissionRequest` fires only when Claude would actually ask. Status keys can't rely on it to know a session is busy; use `PreToolUse`/`UserPromptSubmit`.
5. A no-decision reply (HTTP 200, empty body) leaves Claude's normal prompt in charge, which confirms the fail-open design.
