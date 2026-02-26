# Commands Reference: Essential Commands by Ecosystem

Quick reference for running, testing, and building in common ecosystems.

---

## Shell / Bash

```bash
# Process management
ps aux | grep <process>       # Find a running process
kill <PID>                    # Stop a process by PID
lsof -i :<port>               # Find what's using a port
kill $(lsof -t -i :<port>)    # Kill what's using a port

# File operations
chmod +x script.sh            # Make a script executable
./script.sh                   # Run a script in current directory
bash -x script.sh             # Run with trace output (debugging)
bash -n script.sh             # Syntax check only (no execution)

# Environment
printenv                      # List all environment variables
export VAR=value              # Set for current session
source .env                   # Load from .env file
env VAR=value ./script.sh     # Set for one command only

# Output and logging
command 2>&1 | tee output.log      # Display stdout + stderr and also save to file
command > /dev/null 2>&1           # Suppress all output
command &                          # Run in background
nohup command &                    # Run in background, immune to hangup
tail -f logfile.log                # Follow a log file in real time
```

---

## Node.js / npm

```bash
# Install dependencies
npm ci                        # Clean install from lockfile (use in CI)
npm install                   # Install and update lockfile (use locally)
npm install <pkg>             # Add a dependency
npm install --save-dev <pkg>  # Add a dev dependency

# Run scripts
npm run <script>              # Run a script from package.json
npm start                     # Run the "start" script
npm test                      # Run the "test" script
npm run build                 # Run the "build" script

# Debug
NODE_ENV=development npm start          # Set environment
DEBUG=* npm start                       # Enable all debug output
node --inspect server.js                # Start with debugger
node --trace-warnings server.js        # Show stack traces for warnings

# Check versions
node --version
npm --version
npx --version
```

---

## Python / pip

```bash
# Virtual environment
python -m venv .venv                   # Create virtual environment
source .venv/bin/activate              # Activate (Unix/Mac)
.venv\Scripts\activate                 # Activate (Windows)
deactivate                             # Deactivate

# Install dependencies
pip install -r requirements.txt        # Install from requirements file
pip install -e .                       # Install package in editable mode
pip freeze > requirements.txt         # Export current environment

# Run
python script.py                       # Run a script
python -m module_name                  # Run a module
python -c "import sys; print(sys.version)"   # One-liner

# Test
pytest                                 # Run all tests
pytest -v                              # Verbose output
pytest tests/test_specific.py         # Run specific file
pytest -k "test_name"                 # Run tests matching name
pytest -x                             # Stop on first failure
pytest --pdb                          # Drop into debugger on failure
```

---

## Go

```bash
# Run
go run main.go                         # Run directly
go run .                               # Run package in current directory

# Build
go build -o myapp .                    # Build binary
go build ./...                         # Build all packages

# Test
go test ./...                          # Test all packages
go test -v ./...                       # Verbose output
go test -run TestName ./...            # Run specific test
go test -race ./...                    # Race condition detection
go test -cover ./...                   # Coverage report

# Deps
go mod tidy                            # Add missing, remove unused deps
go mod download                        # Download deps to local cache
```

---

## Docker

```bash
# Build
docker build -t myapp .                         # Build image
docker build -t myapp:v1.0 .                   # Build with specific tag
docker build --no-cache -t myapp .             # Force fresh build

# Run
docker run myapp                               # Run and remove when done
docker run --rm myapp                          # Auto-remove after stop
docker run -d --name myapp myapp               # Run as daemon
docker run -p 3000:3000 myapp                  # Map port
docker run -v $(pwd):/app myapp                # Mount volume
docker run --env-file .env myapp               # Load env file

# Debug
docker logs myapp                              # View logs
docker logs -f myapp                           # Follow logs
docker exec -it myapp sh                       # Shell into container
docker inspect myapp                           # Full container details
docker stats                                   # Live resource usage

# Cleanup
docker stop myapp                              # Graceful stop
docker rm myapp                               # Remove stopped container
docker rmi myapp                              # Remove image
docker system prune                           # Remove all unused resources
```

---

## Docker Compose

```bash
docker compose up                      # Start all services
docker compose up -d                   # Start in background
docker compose up --build              # Rebuild before starting
docker compose down                    # Stop and remove containers
docker compose down -v                 # Also remove volumes
docker compose logs -f                 # Follow all logs
docker compose logs -f service         # Follow one service
docker compose exec service sh         # Shell into service
docker compose ps                      # Status of all services
```

---

## Make / Makefile

A Makefile is the standard way to document and run project commands:

```makefile
.PHONY: install test build run clean

install:
	npm ci

test:
	npm test

build:
	npm run build

run:
	npm start

clean:
	rm -rf dist/ node_modules/
```

Usage:
```bash
make install
make test
make build
```

---

## Exit Codes Reference

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error (check stderr) |
| 2 | Misuse of shell command |
| 126 | Command found, not executable |
| 127 | Command not found |
| 128+n | Fatal signal n (e.g., 130 = Ctrl+C, 137 = SIGKILL/OOM) |
| 137 | Out of memory (OOM kill) |
| 143 | SIGTERM (graceful shutdown requested) |

Check exit codes:
```bash
command && echo "Success" || echo "Failed with $?"
echo $?    # Exit code of last command
```
