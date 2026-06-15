---
name: handoff
description: >
  Compact the current conversation into a handoff document for another agent to pick up.
  Use when user says /handoff, "handoff", "create a handoff", "write a handoff doc",
  or wants to summarise the session for continuity. Always invoke this skill for these triggers.
---

# /handoff

Compact the current conversation into a handoff document so a fresh agent can continue without context loss.

## Steps

1. **Identify focus** — if the user passed arguments, treat them as the next session's focus and tailor the doc accordingly.

2. **Write the document** to the OS temp directory (`/tmp/handoff-<project>-<date>.md` on Linux/Mac, `%TEMP%\handoff-<project>-<date>.md` on Windows). Never write to the current workspace.

3. **Document structure:**
   - **Context** — what project/task, what problem was being solved, why it matters
   - **Current state** — what works, what's broken, where things stand right now
   - **Decisions made** — key choices and the reasoning behind them (don't duplicate; reference file paths or URLs for artifacts that already exist: plans, PRDs, ADRs, commits, issues)
   - **Open questions** — unresolved decisions, blockers, things that need input
   - **Next steps** — concrete actions the next agent should take, in priority order
   - **Suggested skills** — which skills the next agent should invoke (e.g. `/grill-me`, `/commit`, `/code-review`)

4. **Redact** all sensitive data: API keys, passwords, tokens, PII. Replace with `[REDACTED]`.

5. **Tell the user** the file path so they can paste it into the next session.

## Tone

Terse. Write for an agent, not a human reader. Assume full technical context. No filler.
