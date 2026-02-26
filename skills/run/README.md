# run

Know exactly how to execute things. For the ones who ship.

You know that moment when a command that worked yesterday fails today with a cryptic error? Or when CI is red and you're staring at 400 lines of log output not knowing where to start? You need someone who reads errors instead of guessing, isolates the variable instead of changing everything, and makes it reproducible instead of "just works on my machine."

That's this skill.

## Install

```bash
npx skills add openclaw-rocks/skills --skill run
```

Works with Claude Code, Cursor, GitHub Copilot, Gemini CLI, and [27+ other agents](https://agentskills.io).

## What it does

Turns your AI agent into an expert executor and debugger. Six battle-tested protocols for everything from running a single command to managing a full CI pipeline.

| Protocol | What it does |
|----------|-------------|
| **Run a Command** | Verify first, execute with full visibility, check exit codes. |
| **Debug a Failure** | Read errors bottom-up, isolate the variable, fix the root cause. |
| **Run in a Container** | Minimal Dockerfiles, pinned versions, clean builds. |
| **Run Tests** | Clean baseline, isolated failures, maximum output when things break. |
| **Run a Build Pipeline** | Clean → install → lint → test → build, in that order. |
| **Run a Server** | Health checks, environment separation, process management. |

Plus a pre-run checklist that catches the common failures before they happen.

## When to use it

- A command is failing and you don't know why
- CI is red and you need to read the logs correctly
- You need to set up environment variables for a new environment
- You're Dockerizing an application for the first time
- You want to make a run script repeatable and documented
- You're debugging a "works on my machine" situation
- You need to run tests in isolation to find which one is failing
- You're starting a service and need to verify it's actually up

## Example prompts

```
Run the tests and tell me what's failing.

Debug this error: "cannot find module './config'"

Write a Dockerfile for this Node.js app.

My CI pipeline is failing. Here are the logs. What's wrong?

Set up environment variables for this service.

How do I run this in the background and see its logs?

Make this run script work on both Mac and Linux.
```

## What's inside

```
run/
├── SKILL.md                          # The run command engine
└── references/
    ├── commands.md                   # Commands by ecosystem: Node, Python, Go, Docker, shell
    └── debugging.md                  # Debugging frameworks for common run failures
```

## The Five Run Principles

1. **Read the error first.** Most failures tell you exactly what went wrong. Read the full message — not just the first line.
2. **Reproduce before fixing.** If you can't reproduce it, you don't understand it.
3. **Isolate the variable.** Code, environment, dependencies, inputs — find which one changed.
4. **Make it repeatable.** A run that only works when done in exactly the right way isn't done.
5. **Make it observable.** If you can't see what it's doing, you can't fix it when it breaks.

## Built by

[OpenClaw.rocks](https://openclaw.rocks) builds the simplest way to host AI agents. One-click deployment to EU infrastructure, zero platform fees.

---

*Make it work, make it right, make it fast — in that order.*
