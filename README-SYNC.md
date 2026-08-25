# Agent Deck Fork: Upgrade & Sync Workflow

This repository (`ttunguz/agent-deck`) is a fork of upstream [`asheshgoplani/agent-deck`](https://github.com/asheshgoplani/agent-deck) customized to use **Pi** as the conductor orchestrator runtime.

---

## 1. Fork Architecture & Philosophy

- **`main`**: Pure upstream mirror. Never contains custom commits. Tracked strictly via `--ff-only` merges from `upstream/main`.
- **`pi-orchestrator`**: Integration branch containing the minimal, additive patch set for Pi conductor support:
  - `internal/session/conductor.go`: Register `ConductorAgentPi = "pi"` in `conductorAgentSpecs` map with safe stale-file cleanup.
  - `internal/session/conductor_templates.go`: Add `conductorPerNamePiMDTemplate` tailored for Pi (`AGENTS.md` instructions).
  - `cmd/agent-deck/conductor_cmd.go`: Update `--agent` setup flag and help text.
  - `internal/session/conductor_test.go`: Tests for Pi conductor setup and spec lookup.
  - `FORK.md`: Local usage and conductor guide.

Because all changes are localized behind the existing `ConductorAgentSpec` abstraction and data structures, rebasing onto new upstream releases is clean and virtually conflict-free.

---

## 2. Standard Upgrade Sync Ritual

When upstream `asheshgoplani/agent-deck` releases updates or new commits, run the following steps:

```bash
cd ~/Documents/coding/pi-agent-deck

# Step 1: Fetch the latest commits from upstream
git fetch upstream

# Step 2: Fast-forward the local main mirror
git checkout main
git merge --ff-only upstream/main
git push origin main

# Step 3: Rebase the pi-orchestrator patch set onto main
git checkout pi-orchestrator
git rebase main

# Step 4: Run unit tests to verify the rebase
go test ./internal/session/ -run 'TestSetupConductorWithAgent_Pi|TestGetConductorAgentSpec_Pi' -v
go test ./cmd/agent-deck/ -run 'TestConductor.*' -v

# Step 5: Force-push updated branch to fork
git push --force-with-lease origin pi-orchestrator

# Step 6: Recompile local binary (if installed locally)
go build -o ~/.local/bin/agent-deck ./cmd/agent-deck
```

---

## 3. Conflict Resolution Protocol

In the unlikely event of an upstream conflict during `git rebase main`:

1. **`internal/session/conductor.go`**:
   - Upstream may add new conductor runtimes or rename fields in `ConductorAgentSpec`.
   - Ensure `ConductorAgentPi: { ... }` remains present in `conductorAgentSpecs`.
   - Ensure the stale-file cleanup loop preserves `otherSpec.InstructionsFileName == spec.InstructionsFileName` skip check.
2. **`cmd/agent-deck/conductor_cmd.go`**:
   - Ensure `-agent` flag description lists `pi`.
3. Complete the rebase:
   ```bash
   git add <resolved-files>
   git rebase --continue
   ```

---

## 4. Verification Checklist

After any sync / rebase:
- [ ] `go build ./...` compiles cleanly.
- [ ] `go test ./internal/session/ -run 'TestSetupConductor.*|TestGetConductor.*'` passes.
- [ ] Test isolated setup:
  ```bash
  export HOME=$(mktemp -d)
  agent-deck conductor setup test-orch --agent pi --no-heartbeat
  cat $HOME/.local/share/agent-deck/conductor/test-orch/AGENTS.md
  ```
