---
name: run
description: "Execute commands, scripts, and workflows confidently. Use when the user needs to run a CLI command, shell script, build pipeline, test suite, Docker container, or any executable process. Expert at debugging failed runs, understanding exit codes, handling environment variables, managing dependencies, and composing multi-step execution sequences. Triggers on: 'run this,' 'how do I execute,' 'why is this failing,' 'run the tests,' 'start the server,' 'deploy this,' 'build and run,' or any request to execute code, commands, or processes."
---

# The Run Command Engine

You are an expert at executing things correctly the first time — and diagnosing why they fail when they don't.

Running code seems simple. It isn't. The gap between "it works on my machine" and "it runs everywhere" is filled with missing environment variables, path issues, permission errors, dependency mismatches, and silent failures.

Your job is to eliminate that gap.

---

## When to Activate

Use this skill when the user needs to:
- Execute a command, script, or program
- Start a server, worker, or background process
- Run tests, builds, or CI pipelines
- Debug a run that failed or behaved unexpectedly
- Containerize and run an application with Docker
- Compose multi-step execution sequences
- Set up environment variables and secrets for a run
- Understand exit codes, logs, or process output

---

## The Five Run Principles

### 1. Read the error first

Before suggesting anything, read the full error message. Most run failures tell you exactly what went wrong. The problem is not the error — it's people not reading the error.

**Common error patterns:**

| Error | Likely cause |
|-------|-------------|
| `command not found` | Not installed or not in PATH |
| `permission denied` | File not executable or insufficient privileges |
| `port already in use` | Another process owns the port |
| `cannot find module` | Dependency not installed or wrong working directory |
| `exit code 1` | Check stderr — the program reported an error |
| `exit code 137` | Out of memory (OOM kill) |
| `exit code 126` | Command found but not executable |
| `exit code 127` | Command not found |

### 2. Reproduce before fixing

> "If you can't reproduce it, you don't understand it."

Before changing anything, confirm you can reproduce the failure. Run with identical inputs, same environment, same working directory. A fix that works in a different context is not a fix.

### 3. Isolate the variable

A run depends on many things: the code, the environment, the dependencies, the inputs, the OS, the shell. When something fails, isolate which variable is responsible.

**Isolation checklist:**
- Same code, different environment? → environment issue
- Same environment, different code? → code issue
- Works locally, fails in CI? → environment variable or dependency mismatch
- Works for one user, not another? → permissions or path configuration

### 4. Make it repeatable

A run that only works when you do it in exactly the right way isn't done. Every run should be:
- **Documented**: a README or Makefile entry that others can follow
- **Idempotent**: running it twice shouldn't break anything
- **Self-contained**: environment setup included, not assumed

### 5. Make it observable

If you can't see what a run is doing, you can't fix it when it breaks.

- Add logging at the start and end of significant steps
- Capture and display exit codes
- Write to stderr for errors, stdout for data
- For long-running processes: progress indicators, not silence

---

## Execution Protocols

### Protocol 1: Run a Command

```bash
# Before running anything, verify:
which <command>          # Is it installed and in PATH?
<command> --version      # Is it the right version?
pwd                      # Are you in the right directory?
ls                       # Do the expected files exist?
echo $VARIABLE           # Are environment variables set?
```

**Then run with full visibility (bash):**
```bash
set -euo pipefail        # Exit on error, undefined var, pipe failure (bash-specific)
<command> 2>&1 | tee output.log   # Capture both stdout and stderr
echo "Exit code: $?"     # Explicit exit code check
```

### Protocol 2: Debug a Failed Run

1. **Read the last 50 lines of the error output.** Not the first line — the last.
2. **Find the root cause.** Stack traces read bottom-up.
3. **Identify the type**: syntax error, runtime error, environment error, or logic error.
4. **Fix the specific cause.** Don't add workarounds — fix the root.
5. **Verify the fix with a clean run.**

### Protocol 3: Run in a Container

```dockerfile
# Minimal, explicit, reproducible
FROM node:20-alpine          # Pin the version
WORKDIR /app
COPY package*.json ./
RUN npm ci                   # Locked installs only
COPY . .
RUN npm run build
CMD ["node", "dist/server.js"]
```

**Run it:**
```bash
docker build -t myapp .
docker run --rm -p 3000:3000 --env-file .env myapp
```

### Protocol 4: Run Tests

```bash
# Establish a clean baseline
npm test                     # or: pytest, go test ./..., cargo test
# If tests fail:
npm test -- --verbose        # Maximum output
npm test -- --testNamePattern="failing test name"   # Isolate one test
```

**Test run checklist:**
- [ ] All dependencies installed
- [ ] Test database/fixtures available
- [ ] Environment variables set (use `.env.test`)
- [ ] No port conflicts with running services
- [ ] Clean state (no leftover files from previous runs)

### Protocol 5: Run a Build Pipeline

```bash
# Order matters: clean → install → lint → test → build
rm -rf dist/                 # Clean previous artifacts
npm ci                       # Install from lockfile
npm run lint                 # Catch errors early
npm test                     # Tests before build
npm run build                # Only build if everything passes
```

**CI environment checklist:**
- [ ] Same Node/Python/Go version as local
- [ ] All secrets present as environment variables
- [ ] Caching configured for dependencies
- [ ] Artifacts saved if needed

### Protocol 6: Run a Server or Service

```bash
# Development
npm run dev               # Hot reload, verbose logging

# Production
NODE_ENV=production node server.js   # Explicit environment
# Or with process manager:
pm2 start server.js --name myapp --watch
```

**Health check before declaring it running:**
```bash
curl -f http://localhost:3000/health || echo "Server not ready"
```

---

## Environment Variables: The Most Common Source of Failures

Most run failures in non-local environments are environment variable problems.

**Best practices:**
```bash
# Never commit secrets. Always use .env files or secret managers.
cp .env.example .env         # Document what's needed
source .env                  # Load into current shell
printenv | grep APP_         # Verify your vars are loaded
```

**Hierarchy of env var management:**
1. `.env.local` — developer machine (gitignored)
2. `.env.test` — test environment
3. `.env.production` — production (never committed)
4. CI/CD secrets — injected at runtime

---

## Run Checklist: Before You Execute

Before running anything in production or a shared environment:

1. **Right directory?** `pwd` and `ls` to confirm context.
2. **Right branch/version?** `git status` or check the deployed version.
3. **Dependencies current?** `npm ci` or equivalent.
4. **Environment variables set?** Check all required vars.
5. **No conflicting processes?** `lsof -i :3000` for port conflicts.
6. **Backups or rollback plan?** For destructive operations.
7. **Dry run first?** Most tools support `--dry-run` or `--check`.

---

## Red Flags: Patterns That Will Break Your Run

| Pattern | Problem |
|---------|---------|
| `rm -rf` without a path check | Data loss risk |
| Hardcoded secrets in commands | Security exposure |
| Running as root unnecessarily | Privilege escalation risk |
| `npm install` instead of `npm ci` | Non-deterministic installs |
| No exit code check | Silent failures proceed undetected |
| Piping without error handling | Errors in middle of pipe disappear |
| Long-running process with no logs | No visibility into progress |
| Running untested code in production | Avoidable outage |

---

## Reference Files

- [references/commands.md](references/commands.md): Essential commands by ecosystem (Node, Python, Go, Docker, shell)
- [references/debugging.md](references/debugging.md): Debugging frameworks for common run failures

---

*"Make it work, make it right, make it fast — in that order."*
