# Pi Agent Deck Fork

This repository is a lightweight fork of [asheshgoplani/agent-deck](https://github.com/asheshgoplani/agent-deck) that adds **Pi** as a supported conductor (orchestrator) runtime.

## What's Changed

1. **Pi Conductor Runtime (`--agent pi`)**:
   - `internal/session/conductor.go`: Added `ConductorAgentPi = "pi"` to `conductorAgentSpecs` map.
   - `internal/session/conductor_templates.go`: Added Pi-tailored `AGENTS.md` per-conductor template.
   - `cmd/agent-deck/conductor_cmd.go`: Updated conductor setup CLI flags & help text.
   - Stale instruction file cleanup safely skips agents sharing `AGENTS.md` (Pi & Codex).
2. **Upstream Drift Detection**:
   - `.github/workflows/upstream-sync.yml`: Automates daily dry-run rebase checks against upstream.

---

## Setup & Usage

### 1. Build & Install Locally

```bash
# Build binary
go build -o ~/.local/bin/agent-deck ./cmd/agent-deck

# Verify Pi is supported
agent-deck conductor setup --help
```

### 2. Set Up a Pi Conductor

```bash
# Create a named conductor running Pi
agent-deck conductor setup pi-orch --agent pi --heartbeat --description "Primary Pi Orchestrator"

# Inspect the created configuration
cat ~/.local/share/agent-deck/conductor/pi-orch/AGENTS.md

# Start the conductor session in TUI or terminal
agent-deck session start conductor-pi-orch
```

---

## Upstream Upgrade & Sync Workflow

To keep this fork up-to-date with upstream releases while preserving our minimal patch set:

```bash
# 1. Fetch latest upstream commits
git fetch upstream

# 2. Fast-forward the local main mirror
git checkout main
git merge --ff-only upstream/main

# 3. Rebase the pi-orchestrator patch set onto main
git checkout pi-orchestrator
git rebase main

# 4. Push updated branch to your fork
git push --force-with-lease origin pi-orchestrator
```
