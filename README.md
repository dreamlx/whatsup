# whatsup

An agent skill that rehydrates the current session into a compact, fixed-width **recall panel** — what you were doing and where you stopped, the next move, open loops, decisions already settled or dropped, and the workspace state. It's for the moment you come back to a long-running session after hours away and would otherwise spend minutes re-reading the chat and output history. Inspired by the [Codey](https://github.com/its-ahoh/codey/) Mac app's Task HUD.

```
┌─ WHATSUP ──────────────────────────────────────────────┐
│ NOW     [waiting] wiring auth token refresh            │
│         stopped at: interceptor + backoff written,     │
│         awaiting 401 retry-policy call                 │
├────────────────────────────────────────────────────────┤
│ NEXT    confirm exponential vs fixed retry interval    │
├────────────────────────────────────────────────────────┤
│ OPEN    ? 401 retry policy — asked you, no answer      │
│         □ refresh-failure fallback not written         │
├────────────────────────────────────────────────────────┤
│ DECIDED ✓ tokens in keychain, not localStorage         │
│         ✗ dropped silent-iframe refresh (CORS)         │
├────────────────────────────────────────────────────────┤
│ STATE   auth-refresh · 3 files dirty · 1 test ✗        │
├────────────────────────────────────────────────────────┤
│ RECENT  • interceptor + backoff written     — 2m ago   │
│         • token store wired                 — 8m ago   │
└────────────────────────────────────────────────────────┘
```

It's a recall primer, not a progress dashboard: it leads with **where the cursor is** (`NOW` / `stopped at:`), the **open loops** that are usually why you paused (`OPEN`), and the **decisions already made or dropped** so you don't relitigate or retry them (`DECIDED`). There's deliberately no progress percentage — coming back, you need the workspace state, not an abstract number.

## Install

Clone into your agent's skills directory. For Claude Code:

```bash
git clone https://github.com/its-ahoh/whatsup.git ~/.claude/skills/whatsup
```

For other agents, clone into wherever that agent loads skills from.

## Usage

Invoke it when you return to a session to catch back up fast:

- `/whatsup`
- or just ask: "whatsup", "where are we", "where did I stop", "what was I doing", "catch me up"

The panel is a fixed **58-column** outlined box that wraps long lines so it fits a narrow terminal, with right-aligned timestamps that form a clean column. See [`SKILL.md`](SKILL.md) for the full format spec.
