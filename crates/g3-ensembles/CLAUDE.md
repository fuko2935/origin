# g3-ensembles - Multi-Agent "Flock" Mode

**Technology**: Rust 2021, Tokio, UUID
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate implements the "Flock" mode for parallel multi-agent development, allowing multiple AI agents to work on different parts of a project simultaneously.

---

## Development Commands

### This Crate

```bash
# From crate directory
cargo build
cargo test
cargo clippy -- -D warnings
```

### From Root

```bash
cargo build -p g3-ensembles
cargo test -p g3-ensembles
cargo run -- --flock  # Run flock mode
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-ensembles -- -D warnings && cargo test -p g3-ensembles
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry, Flock orchestration
├── flock.rs                  # Flock manager implementation
├── status.rs                 # Status tracking
├── tests.rs                  # Unit tests
tests/
├── integration_tests.rs      # Integration tests
TESTING.md                    # Testing documentation
```

### Flock Mode Concept

Flock mode enables parallel development by:
1. Decomposing a project into modular units
2. Assigning each unit to a separate agent
3. Running agents in parallel
4. Coordinating results and dependencies

```
┌─────────────────┐
│  Flock Manager  │
│  (Orchestrator) │
└───────┬─────────┘
        │
   ┌────┴────┬────────┬────────┐
   ▼         ▼        ▼        ▼
┌─────┐  ┌─────┐  ┌─────┐  ┌─────┐
│Agent│  │Agent│  │Agent│  │Agent│
│  1  │  │  2  │  │  3  │  │  4  │
└─────┘  └─────┘  └─────┘  └─────┘
   │         │        │        │
   ▼         ▼        ▼        ▼
 Module    Module   Module   Module
   A         B        C        D
```

---

## Code Organization Patterns

### Flock Manager

```rust
// Pattern: Flock orchestration
pub struct FlockManager {
    agents: Vec<AgentHandle>,
    shared_context: Arc<SharedContext>,
}

impl FlockManager {
    pub async fn run_flock(&mut self, modules: Vec<Module>) -> Result<Vec<AgentResult>> {
        // Spawn agents for each module
        let handles: Vec<_> = modules.into_iter().map(|module| {
            let agent = self.create_agent(module);
            tokio::spawn(async move {
                agent.execute().await
            })
        }).collect();

        // Wait for all agents
        let results = futures::future::join_all(handles).await;
        self.merge_results(results)
    }
}
```

### Agent Coordination

Agents coordinate via shared context:

```rust
// Pattern: Shared state between agents
pub struct SharedContext {
    completed_modules: RwLock<HashSet<ModuleId>>,
    artifacts: RwLock<HashMap<ModuleId, Artifact>>,
    dependencies: DependencyGraph,
}
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main orchestration
   ```bash
   rg -n "FlockManager|run_flock" src/lib.rs
   ```

2. **`src/flock.rs`** - Flock implementation
   ```bash
   rg -n "pub struct|pub async fn" src/flock.rs
   ```

3. **`TESTING.md`** - Testing documentation
   ```bash
   cat TESTING.md
   ```

---

## Quick Search Commands

### Find Flock Logic

```bash
# Find flock-related types
rg -n "Flock|Agent.*Handle|Module" src/lib.rs

# Find async spawning
rg -n "tokio::spawn|join_all" src/lib.rs

# Find status tracking
rg -n "status|Status" src/status.rs
```

---

## Common Gotchas

### Resource Contention

Multiple agents may try to access the same files:
- Use file locking for shared resources
- Coordinate via shared context
- Avoid overlapping module boundaries

### Token Limits

Each agent has its own context window. With many agents:
- Total token usage = agents × per-agent tokens
- Monitor cumulative costs
- Consider using smaller models for individual agents

### Error Propagation

One failing agent shouldn't crash the flock:
- Errors are collected, not propagated immediately
- Failed modules can be retried
- Results include success/failure status

### Dependency Ordering

Modules may depend on each other:
- Dependency graph determines execution order
- Dependent modules wait for prerequisites
- Circular dependencies are detected and reported

---

## Testing Guidelines

### Unit Tests

```bash
cargo test -p g3-ensembles
```

### Integration Tests

```bash
cargo test -p g3-ensembles --test integration_tests
```

See `TESTING.md` for detailed testing documentation.

---

## Usage

Flock mode is activated via CLI:

```bash
# Basic flock mode
g3 --flock

# With specific workspace
g3 --flock --workspace /path/to/project
```

The flock manager will:
1. Analyze the project structure
2. Identify parallelizable modules
3. Spawn agents for each module
4. Coordinate and merge results

---

## Configuration

Flock mode uses the default provider configuration, but can be customized:

```toml
[providers]
default_provider = "anthropic.default"

[flock]
max_parallel_agents = 4
agent_timeout_seconds = 300
retry_failed_modules = true
```
