---
name: background
description: Run long tasks in the background and keep working while they run, then read their results when they're needed. Use whenever a step takes more than a few seconds (a whole suite's coverage, concolic explore, forge test on a big project, a build), so reading and planning don't wait on it, and the person can keep talking to you.
---

# Long tasks in the background

A whole suite's coverage or an explore run takes minutes. Waiting on it idle wastes the time you'd spend reading the
code it's about. Start it, do something useful, read it when you need it.

## The pattern

1. **Start it in the background** and note what it is for:
   - In Claude Code: run the command with the Bash tool's `run_in_background`, and redirect its output to a file
     (`concolity coverage . --json > /tmp/cov.json`).
   - In Formal Mind (the Bug Hunter's tools): `run_in_background {"tool": "coverage" | "explore" | "run", "input": {…}}`
     returns a task number; a hunt starts the suite's coverage as task 1 for you.
2. **Say so**: "Coverage of the whole suite is running in the background (about a minute); meanwhile I'm reading the
   matching engine."
3. **Do the work that doesn't need it**: read code, write hypotheses, write the PoC skeleton.
4. **Read the result when you need it**:
   - Claude Code: you're told when a background command finishes; read its output file then. Don't poll it in a loop.
   - Formal Mind: `background_results` lists tasks with their status and time; `background_results {"id": 1}` returns
     the result, and `"wait": true` waits for it.
5. **Use it, and say what it showed.**

## Rules

- Start the long task first, then read: the order matters for wall-clock time.
- One run per question. Don't start a second identical run while the first is going; wait for it (Formal Mind's
  `coverage` waits for a run already in progress, then reads the cache).
- Prefer narrow questions that don't need a long task at all: `coverage --at file:line` on cached coverage,
  `explore --target file:line`.
- Never report a result you haven't read. If a task failed or is still running when you finish, say so.
