# g3-planner - Planning/Requirements Mode

**Technology**: Rust 2021, Tokio, Chrono
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate implements the planning mode for requirements-driven development with git integration and structured workflows.

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
cargo build -p g3-planner
cargo test -p g3-planner
cargo run -- --planning --codepath ~/project --workspace ~/g3_workspace
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-planner -- -D warnings && cargo test -p g3-planner
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry, planning workflow
├── planner.rs                # Planner implementation
├── state.rs                  # State management
├── prompts.rs                # Planning prompts
├── llm.rs                    # LLM interactions
├── git.rs                    # Git operations
├── history.rs                # History tracking
├── code_explore.rs           # Code exploration
tests/
├── commit_history_ordering_test.rs
├── logging_test.rs
├── planner_test.rs
├── retry_feedback_test.rs
```

### Planning Mode Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     PLANNING MODE                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. REFINE REQUIREMENTS                                     │
│     ├── Write to: g3-plan/new_requirements.md              │
│     ├── LLM suggests improvements                           │
│     └── User approves or modifies                           │
│                                                             │
│  2. IMPLEMENT                                               │
│     ├── new_requirements.md → current_requirements.md       │
│     ├── Coach-player loop executes                          │
│     └── Creates todo.g3.md for tracking                    │
│                                                             │
│  3. COMPLETE                                                │
│     ├── Archive: completed_requirements_YYYY-MM-DD.md       │
│     ├── Archive: completed_todo_YYYY-MM-DD.md              │
│     └── Git commit with LLM-generated message               │
│                                                             │
│  4. REPEAT → Back to step 1                                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Artifacts Directory

All planning artifacts are stored in `<codepath>/g3-plan/`:

| File | Purpose |
|------|---------|
| `new_requirements.md` | Requirements being refined |
| `current_requirements.md` | Active requirements for implementation |
| `todo.g3.md` | Implementation TODO list |
| `planner_history.txt` | Audit log of planning activities |
| `completed_*.md` | Archived requirements and todos |

---

## Code Organization Patterns

### Planning Session

```rust
// Pattern: Planning session lifecycle
pub struct PlanningSession {
    codepath: PathBuf,
    workspace: PathBuf,
    use_git: bool,
}

impl PlanningSession {
    pub async fn run_planning_cycle(&mut self) -> Result<()> {
        // 1. Refine requirements
        self.refine_requirements().await?;

        // 2. Implement via coach-player
        self.implement_requirements().await?;

        // 3. Complete and archive
        self.complete_cycle().await?;

        Ok(())
    }
}
```

### Requirements Refinement

```rust
// Pattern: LLM-assisted requirement refinement
let raw_requirements = read_file("g3-plan/new_requirements.md")?;
let refined = llm.refine_requirements(&raw_requirements).await?;
// User reviews and approves
```

### Git Integration

```rust
// Pattern: Git commit after implementation
if self.use_git {
    let staged_files = git_status_staged()?;
    let commit_message = llm.generate_commit_message(&staged_files).await?;
    git_commit(&commit_message)?;
}
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main planning logic
   ```bash
   rg -n "PlanningSession|run_planning" src/lib.rs
   ```

2. **`src/planner.rs`** - Planner implementation
   ```bash
   rg -n "pub struct|pub async fn" src/planner.rs
   ```

3. **`src/git.rs`** - Git operations
   ```bash
   rg -n "git_|commit" src/git.rs
   ```

### Test Files

```bash
# Find test coverage
rg -n "#\[test\]|#\[tokio::test\]" tests/
```

---

## Quick Search Commands

### Find Planning Logic

```bash
# Find planning functions
rg -n "async fn.*plan|requirements" src/lib.rs

# Find git operations
rg -n "git_|commit" src/lib.rs
```

### Find Artifacts Handling

```bash
# Find file operations
rg -n "new_requirements|current_requirements|todo.g3" src/lib.rs
```

---

## Common Gotchas

### Git Integration

The `--no-git` flag disables git operations:
- Use when repo isn't initialized
- Use for testing without commits
- Artifacts are still created

### Environment Variables

Planning mode uses environment variables:
- `G3_TODO_PATH` - Custom path for todo.g3.md
- `G3_WORKSPACE_PATH` - Workspace directory for logs

### File Archiving

Completed files are archived with timestamps:
```
completed_requirements_2025-01-15_10-30-00.md
completed_todo_2025-01-15_10-30-00.md
```

### Retry Behavior

Planning mode uses more aggressive retry settings:
- Uses `RetryConfig::planning("player")` or `RetryConfig::planning("coach")`
- Higher retry count for longer operations
- Configured via `autonomous_max_retry_attempts`

---

## Testing Guidelines

### Unit Tests

```bash
cargo test -p g3-planner

# Specific test files
cargo test -p g3-planner --test planner_test
cargo test -p g3-planner --test retry_feedback_test
```

### Test Categories

| Test File | Coverage |
|-----------|----------|
| `planner_test.rs` | Core planning logic |
| `logging_test.rs` | History/audit logging |
| `commit_history_ordering_test.rs` | Git commit ordering |
| `retry_feedback_test.rs` | Retry and feedback loops |

---

## Usage

```bash
# Basic planning mode
g3 --planning --codepath ~/my-project --workspace ~/g3_workspace

# Without git operations
g3 --planning --codepath ~/my-project --no-git --workspace ~/g3_workspace
```

### Workflow Steps

1. Create `g3-plan/new_requirements.md` in your project
2. Run planning mode
3. Review and approve refined requirements
4. Watch implementation via coach-player loop
5. Review git commit
6. Repeat for next iteration

---

## Configuration

Different providers can be used for different roles:

```toml
[providers]
default_provider = "anthropic.default"
planner = "anthropic.planner"    # For planning mode
coach = "anthropic.default"      # For code review
player = "anthropic.default"     # For implementation
```
