---
name: whatsup
description: Render a compact recall panel that rehydrates an agent session after you've been away — what you were doing and where you stopped, the immediate next action, open loops (unanswered questions, unfinished work), decisions already settled or dropped, and the workspace state. Use when the user asks "whatsup", "what's up", "where are we", "where did I stop", "what's left", "what was I doing", "catch me up", "resume", "status", or types /whatsup or /status, especially when returning to a long-running session.
---

# Recall Panel

Rehydrate the **current session** into a compact panel for someone returning after time away — they should be able to resume in seconds without re-reading the chat and output history. The job is **recall, not reporting**: optimize for "where was I, what's settled, what's the next move," not for a progress dashboard. Everything you need is already in the conversation — do not investigate the codebase. Just read the session and render the panel.

## Output format

Render the panel inside a fenced ``` block as a **fully outlined box** of **fixed width** (`┌─┐ │ ├─┤ └─┘` on all four sides, sections split by `├─┤`).

**Fixed width — never grow the box; wrap instead.** The box is a constant **58 columns** wide (total, including both `│`) so it fits a small/narrow terminal. Lines never extend it — long values wrap onto new lines inside the box.

```
┌─ WHATSUP ──────────────────────────────────────────────┐
│ NOW     [<status>] <what you're doing, ≤ 8 words>      │
│         stopped at: <last action + its result>         │
├────────────────────────────────────────────────────────┤
│ NEXT    <immediate resume action, ≤ 10 words>          │
├────────────────────────────────────────────────────────┤
│ OPEN    ? <unanswered question you raised>             │
│         □ <started-but-unfinished thread>              │
├────────────────────────────────────────────────────────┤
│ DECIDED ✓ <decision already settled>                   │
│         ✗ <approach tried and dropped, + why>          │
├────────────────────────────────────────────────────────┤
│ STATE   <branch> · <N files dirty> · <test/build sig>  │
├────────────────────────────────────────────────────────┤
│ RECENT  • <newest event>                    — <when>   │
│         • <a longer event that needs to wrap onto      │
│           a second line>                    — <when>   │
└────────────────────────────────────────────────────────┘
```

**How to lay it out (do this exactly):**
1. Each row is `│ ` + an 8-char label column (`NOW`/`NEXT`/`OPEN`/`DECIDED`/`STATE`/`RECENT`, or 8 spaces for continuation/wrapped lines) + the value, then pad with trailing spaces and close ` │`. The inner area is **54 columns**; the value area after the label is **46**.
2. **Wrap, don't widen.** If a value exceeds the value area (46 cols), break it at word boundaries onto continuation lines that reuse the 8-space blank label. For a wrapped timeline bullet, indent the continuation 2 spaces so text lines up under the `•`. For wrapped `OPEN`/`DECIDED` items, indent the continuation 2 spaces so text lines up under the marker.
3. The top rule (`┌─ WHATSUP ─…─┐`), every divider (`├─…─┤`), and the bottom rule (`└─…─┘`) all span the same **58** columns. (Treat each `─ │ • ✓ ✗ □ ?` as one column.)

**Section dividers:** a `├─┤` rule between every section, plus the `└─┘` close after RECENT.

**Timeline timestamps:** **right-align** each `— <when>` to the box's right inner edge — because the box is fixed-width, they form a clean vertical column with no manual padding. If an entry's last line leaves no room for the timestamp, put `— <when>` on its own continuation line, still right-aligned.

The label appears once per section; continuation, wrapped, and extra lines use the 8-space blank label. Show the timeline newest-first.

**Omit empty sections.** `OPEN` with nothing open, `DECIDED` with nothing settled, `NEXT` when `status` is `done` — drop the whole section (and its divider) rather than printing a placeholder. Always render `NOW`; render `STATE` whenever you can ground any of branch / dirty files / test or build signal.

## Fields

- **status** — one of, shown as `[status]` next to NOW:
  - `waiting` — blocked on a user decision/answer (you asked something and are waiting)
  - `blocked` — stuck on an external problem (failing dep, missing access, broken env)
  - `done` — task finished and verified
  - `working` — anything else (default)
- **now** — what you're currently doing, in one line (≤ 8 words), plus a `stopped at:` line naming **the last concrete action and its result** — this is the cursor the reader resumes from. If unclear, infer the current focus from the most recent activity.
- **next** — the single most useful resume action (≤ 10 words). If `waiting`, this is usually the decision needed.
- **open** — open loops, the highest-value recall content. Two markers:
  - `?` — a question you raised that hasn't been answered (often *why* the session was paused).
  - `□` — a thread started but not finished (TODO, half-written code path, deferred edge case).
- **decided** — settled context, the anti-rework section. Two markers:
  - `✓` — a decision already made (so it isn't relitigated).
  - `✗` — an approach tried and dropped, with the reason in a few words (so it isn't retried).
- **state** — the **physical workspace** state to restore into: branch, count of dirty/modified files, and the test/build signal (`1 test ✗`, `build ✓`, `lint clean`). Ground these in what actually happened this session; omit any part you can't ground. No progress percentage — recall is about the workspace, not an abstract number.
- **recent** — concrete things that happened (an `action`, `progress`, or notable observation). Newest first. Keep it short (≤ 4 entries) — `OPEN` and `DECIDED` carry the load; `recent` is just the trail. `when` = relative time (`just now`, `Nm ago`, `Nh ago`, `Nd ago`); omit it for an entry you can't ground.

## Rules

- **Terse.** Hard caps: now ≤ 8 words, next ≤ 10 words, each OPEN/DECIDED/RECENT item ≤ 14 words. Items may use the room to name files, symbols, or the reason — still fragments, not full sentences. No trailing punctuation.
- **Optimize for resume time.** The reader was here before and forgot. Surface what they'd waste minutes rediscovering: where the cursor is, what's already decided, what's still open. Skip what they can re-derive at a glance.
- **Match the session's language.** Render values, now, next, open, decided, and recent in the same language the conversation is being conducted in. (Keep the structural labels `WHATSUP`/`NOW`/`NEXT`/`OPEN`/`DECIDED`/`STATE`/`RECENT` and status tokens `working`/`waiting`/`blocked`/`done` as-is.)
- **Ground it** in what actually happened this session — never invent events, decisions, or timestamps you can't justify.
- **Be honest about status.** If you just asked the user something and haven't acted, that's `waiting`. If tests are failing and unresolved, that's `working` (or `blocked` if external), not `done`.
- Render only the panel — no preamble or explanation around it unless the user asks.

## Example

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
