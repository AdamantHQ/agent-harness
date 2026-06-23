# Agent Harness

A universal CLAUDE.md harness for coding agents. Drop it into any project and get production-grade behavior from day one.

Built for Claude Code. Works with any agent that reads a CLAUDE.md or system-level instruction file.

---

## What This Is

Most people prompt coding agents the same way they prompt a chatbot. That works for throwaway scripts. It fails for real software.

This harness is a behavioral contract — a set of operating rules that shapes how an AI agent thinks, plans, executes, and recovers. It doesn't make the model smarter. It gives the model a structured environment to work inside. The difference in output quality is significant.

The core insight: an agent without a harness is an intern with no onboarding. Technically capable. Structurally lost.

---

## How To Use It

### New project

```bash
cp CLAUDE.md /your/project/root/CLAUDE.md
```

Then add a project-specific block at the bottom:

```md
---
## Project Context

**What this is:** [one sentence]
**Stack:** [e.g. Node.js, PostgreSQL, Redis, deployed on Railway]
**Folder structure:** [e.g. /src /tests /migrations /scripts]
**Staging:** [url or deploy command]
**Production:** [url — agent should never deploy here directly]
**Repo:** [github link]
**Main engineer contact:** [name / handle]
```

Project-specific rules override the generic ones when they conflict.

### Existing project

Same process. Read the harness first, note any rules that conflict with how your project already works, and override them in the project context block. Don't modify the harness core — keep it clean so you can pull updates.

### With Claude Code specifically

Place `CLAUDE.md` in your project root. Claude Code reads it automatically at session start. No additional configuration needed.

For other agents, pass the file contents as a system prompt or prepend it to your first message.

---

## What's In The Harness

### Section 0 — Session Start Protocol
Forces the agent to orient before acting. Reads git history, reads the progress file, runs tests, states its understanding, states its plan. Prevents the most common failure mode: starting blind.

### Section 1 — Think Before You Code
Surfaces uncertainty before it becomes wrong implementation. Requires explicit assumption-stating and a written plan for any task over ~30 minutes of work. Plan gets confirmed before execution.

### Section 2 — Minimum Viable Code
Hard constraint against scope creep. No speculative features, no premature abstraction, no over-engineering. The agent writes the least code that fully solves the problem.

### Section 3 — Surgical Changes
When editing existing code, touch only what the task requires. Don't improve adjacent code. Don't silently refactor. Every changed line must trace directly to the request.

### Section 4 — Mid-Task Uncertainty
What to do when blocked. Stop after two failed attempts. Report what was tried and what's suspected. Propose a revised plan before pivoting. Never finish a task the wrong way just to finish it.

### Section 5 — Code Quality Standards
Naming, structure, error handling, security, comments. Concrete rules, not vague principles. Non-negotiables called out explicitly (parameterized queries, no hardcoded secrets, no swallowed errors).

### Section 6 — Testing
Tests are written before fixes, not after. Every new function gets a test. Unhappy paths are tested. Structure for how to handle bugs: reproduce first, then fix.

### Section 7 — Database Rules
Migrations only, no direct schema edits, rollbacks required, no DROP without confirmation, writes to primary only.

### Section 8 — Git Discipline
Commit message format, branch rules, one concern per commit, never commit secrets.

### Section 9 — Hard Stops
Explicit list of actions that require human confirmation before proceeding. File deletion, migrations, new dependencies, architectural changes, CI config modifications.

### Section 10 — When Something Feels Wrong
Permission and expectation for the agent to push back. If a design decision will cause problems, flag it before building it. The agent is a collaborator, not just an executor.

### Section 11 — Session Close Protocol
Structured format for `claude-progress.txt` — the file that bridges context windows. What was completed, current state, next steps, blockers, things noticed but not touched. This file is the only memory that survives a session reset.

### Section 12 — Architecture Changes
Any change touching more than 3 files or introducing a new pattern requires a written proposal and explicit confirmation before code is written.

---

## Guiding Principles

**Speed comes from not having to redo things.**

Every rule in this harness exists because the alternative creates more work, not less. A clarifying question before starting costs 30 seconds. Building the wrong thing costs hours. A test costs 10 minutes. An untested bug in production costs everything.

**The harness doesn't constrain the agent. It orients it.**

Without structure, agents default to optimistic execution — they assume they understood correctly, assume the current state matches what they expect, assume their change won't have downstream effects. Most of that is wrong. The harness replaces assumption with verification.

**Irreversibility is the real risk.**

Most rules in the harness are soft — judgment calls the agent can reason about. The hard stops (Section 9) are different. They exist specifically for actions that are expensive or impossible to undo. The asymmetry is intentional: a false positive (asking when you didn't need to) costs 30 seconds. A false negative (not asking when you should have) can cost hours of recovery.

**Session continuity is an engineering problem.**

Context windows reset. Memory doesn't persist natively. Most multi-session failures aren't model failures — they're context failures. The session start and close protocols treat continuity as a first-class concern, not an afterthought.

**The agent is a collaborator, not an executor.**

The harness explicitly gives the agent permission to push back. A good engineer doesn't build the wrong thing because they were told to. They flag the problem, propose an alternative, and wait for a decision. The same standard applies here.

---

## The `claude-progress.txt` File

This file is the memory layer for multi-session work. The harness instructs the agent to write it at the end of every session and read it at the start of the next.

Format defined in Section 11. Keep it in your project root. Commit it to git — it's part of the project state.

When you start a new session and the agent reads this file first, session 2 picks up where session 1 ended. Without it, every session starts blind.

---

## What This Doesn't Do

This harness shapes behavior. It doesn't enforce it at the system level.

Rules in a markdown file are advisory — the model can reason around them in edge cases. For hard enforcement (blocking specific tool calls, preventing certain bash commands from running), you need hooks in `settings.json` and OS-level environment controls. That's a separate layer.

Think of this harness as the engineer's handbook. The environment controls are the locked server room. Both matter. This file handles the handbook layer.

---

## Customization

The harness is designed to be merged, not replaced. Add project context at the bottom. Override specific rules where your project demands it. Keep the core intact.

Don't modify the harness to be less cautious just because it slows you down on a simple task. The opening note covers this: "For trivial tasks, use judgment." The harness is optimized for real software, not scripts.

---

## Contributing

If you find a failure mode this harness doesn't cover, open an issue with a concrete example. Rules that get added should solve real problems that came up in real sessions — not theoretical edge cases.
