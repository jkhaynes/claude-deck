# Testing Strategy

<!-- The constitution says tests come first. This file says what kind of tests, where they go, and how to run them. Delete sections that don't apply. -->

## Run
```bash
# All tests:         npm test                       (expected, verify once scaffolded)
# One file:          npx vitest run tests/unit/sessions.test.ts
# With coverage:     npx vitest run --coverage
# Watch:             npx vitest
```

## Levels
| Level | What it covers | Location | Framework |
|-------|----------------|----------|-----------|
| Unit | `src/core/`: session state machine, pending-request queue, settings.json hook merge/removal | `tests/unit` | Vitest |
| Integration | The localhost hook server: real HTTP requests shaped like Claude Code hook payloads, with the Stream Deck side faked | `tests/integration` | Vitest + Node `fetch` |
| Manual | On the real Neo: one live Claude Code session asks for permission and the key answers it; plugin stopped falls back to Claude's prompt | `tests/manual/checklist.md` | Owner, by hand |

## Rules
- Each acceptance scenario in a feature's spec.md maps to at least one test.
- Tests describe behavior, not implementation (name them after the scenario).
- No mocking what you own at the integration level; fake only external services.
- External APIs are never called from tests. Use recorded fixtures or fakes.
- A bug fix starts with a failing test that reproduces the bug.
- Tests never read or write the real `~/.claude/settings.json`; point the installer at a temp file.

## Test Data
Hook payload fixtures in `tests/fixtures/hooks/` (one JSON file per event type: `PermissionRequest`, `SessionStart`, `Notification`, and so on), captured from a real Claude Code session and with personal paths scrubbed. Sample `settings.json` files in `tests/fixtures/settings/` cover: no hooks, the owner's existing hooks, plugin hooks already installed, and malformed JSON.
