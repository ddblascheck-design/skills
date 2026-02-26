# Debugging Reference: Frameworks for Common Run Failures

When something fails to run, there is always a reason. This document provides structured frameworks for diagnosing the most common categories of run failure.

---

## The Debugging Stack: Read This Order

Most developers read errors top-to-bottom. **Read them bottom-up.** The root cause is almost always near the bottom of a stack trace. The top lines are usually consequences of a deeper failure.

```
Error: Cannot read properties of undefined (reading 'id')    ← consequence
    at getUserById (users.js:14)                              ← consequence
    at processRequest (handler.js:42)                         ← consequence
    at Layer.handle (express/lib/router/layer.js:95)          ← framework
Error: Database connection failed                             ← root cause
    at connect (db.js:28)
```

Always find the root cause before fixing anything.

---

## Category 1: Environment Failures

**Symptoms:**
- Works locally, fails in CI or production
- "Cannot find module" or "command not found"
- Unexpected values or missing configuration

**Diagnosis:**
```bash
# Compare environments
printenv | sort > local.env
# In CI: printenv | sort > ci.env
# diff local.env ci.env

# Check specific variables
echo "NODE_ENV=$NODE_ENV"
echo "DATABASE_URL=$DATABASE_URL"
echo "PORT=$PORT"

# Check PATH
echo $PATH
which node
which npm
```

**Common causes and fixes:**

| Symptom | Root cause | Fix |
|---------|-----------|-----|
| Works locally, fails in CI | Missing environment variable | Add to CI secrets |
| `command not found` in CI | Not in PATH or not installed | Add install step to CI |
| Different behavior by user | User-specific config file | Standardize config |
| "Wrong version" errors | Version mismatch | Pin version in `.nvmrc`, `.python-version`, `go.mod` |

---

## Category 2: Dependency Failures

**Symptoms:**
- "Cannot find module"
- "No module named X"
- "Package not found"
- Import/require errors

**Diagnosis:**
```bash
# Node.js
ls node_modules/<package>          # Does the package exist?
cat package-lock.json | grep <pkg> # Is it in the lockfile?
npm ls <package>                   # Check installed version

# Python
pip list | grep <package>          # Is it installed?
python -c "import <package>"       # Can Python import it?

# Go
go list -m all | grep <module>     # Is the module in go.sum?
```

**Fixes:**
```bash
# Node.js: always prefer ci over install for reproducibility
rm -rf node_modules/
npm ci

# Python: use exact versions in requirements.txt
pip install -r requirements.txt

# Go: ensure all deps are downloaded
go mod download
go mod tidy
```

---

## Category 3: Permission Failures

**Symptoms:**
- "Permission denied"
- "Operation not permitted"
- "Access is denied"
- 403 errors when accessing files

**Diagnosis:**
```bash
# Check file permissions
ls -la <file>

# Check who you are
whoami
id

# Check what's blocking
stat <file>
```

**Fixes:**
```bash
# Make a script executable
chmod +x script.sh

# Fix directory permissions
chmod 755 /path/to/directory

# Run with elevated privileges (use sparingly)
sudo <command>
```

---

## Category 4: Port and Network Failures

**Symptoms:**
- "Address already in use"
- "EADDRINUSE"
- "Port is already allocated"
- Connection refused

**Diagnosis:**
```bash
# Find what's using a port
lsof -i :3000
ss -tlnp | grep 3000      # Alternative on Linux

# Check if a service is reachable
curl -v http://localhost:3000/health
nc -zv localhost 3000     # Netcat port check
```

**Fixes:**
```bash
# Kill whatever is using the port
kill $(lsof -t -i :3000)

# Or use a different port
PORT=3001 npm start

# Wait for a service to be ready before connecting (times out after 30 seconds)
for i in $(seq 1 30); do curl -sf http://localhost:3000/health && break || sleep 1; done
```

---

## Category 5: Process Failures

**Symptoms:**
- Process exits immediately
- Exit code 1, 137, or other non-zero
- No error message (silent failure)
- "Killed" in output

**Diagnosis:**
```bash
# Run with immediate exit code capture
command; echo "Exit: $?"

# Trace execution
bash -x script.sh          # Trace each command

# Check for OOM (exit code 137)
dmesg | grep -i oom        # Linux
# If exit 137: increase memory or reduce usage

# Check if process is still running
ps aux | grep <process>
```

**Common exit codes and their meaning:**
- **Exit 1**: General failure — read stderr
- **Exit 2**: Bad arguments or usage error
- **Exit 137**: OOM kill — increase memory limits
- **Exit 126**: File not executable — `chmod +x`
- **Exit 127**: Not found — check PATH and install

---

## Category 6: Docker Failures

**Symptoms:**
- Build fails mid-way
- Container exits immediately
- "No such container"
- Volume mount issues

**Diagnosis:**
```bash
# Build with verbose output
docker build --progress=plain -t myapp . 2>&1 | tee build.log

# Check container logs after failure
docker logs <container_id>

# Shell into a failed build layer
docker run --rm -it <image_from_failed_step> sh

# Check what's actually in the image
docker run --rm myapp ls -la /app
```

**Fixes:**
```bash
# Always tag your builds
docker build -t myapp:debug .

# For "container exits immediately": check CMD/ENTRYPOINT
docker run --rm -it --entrypoint sh myapp  # Override entrypoint

# For volume issues: check path mapping
docker run --rm -v $(pwd):/app myapp ls /app  # Verify mount
```

---

## Category 7: CI/CD Failures

**Symptoms:**
- Fails in CI but works locally
- Intermittent failures
- "Resource not available"
- Cache-related failures

**Diagnosis framework:**
1. **Read the full log**, not just the summary
2. **Find the first failure** — CI logs often continue after failure
3. **Compare CI environment to local**: Node version, env vars, file system
4. **Check for race conditions**: parallel jobs, port conflicts
5. **Check cache**: stale cache can cause phantom failures

**Common CI failure patterns:**

| Symptom | Cause | Fix |
|---------|-------|-----|
| Works locally, fails CI | Missing env var | Add to CI secrets |
| Fails first run, works on retry | Race condition or network | Add retry or wait logic |
| Worked yesterday, fails today | Dependency update | Pin all dependency versions |
| "Port already in use" | Parallel test jobs | Use random ports or test isolation |
| "File not found" | Different working directory | Use absolute paths |

---

## The Debug Loop

When nothing else works, use this loop:

1. **Minimize**: Strip the command to its simplest form. Can you reproduce with just one file?
2. **Verify assumptions**: Don't assume the env var is set — print it. Don't assume the file exists — list it.
3. **Add output**: Run with `--verbose`, `--debug`, `-x`, or equivalent. Turn off all output suppression.
4. **Compare**: What changed since it last worked? Git diff, dependency update, environment change?
5. **Search the exact error**: Copy the exact error string into a search. Someone has seen this before.
6. **Ask for help with context**: Provide the full error, the command you ran, and what you already tried.

---

*The most dangerous assumption in debugging is that you already know what's wrong.*
