# g3-cli - Command Line Interface

**Technology**: Rust 2021, Clap, Ratatui, Rustyline, Crossterm
**Entry Point**: `src/lib.rs` → `pub async fn run()`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate provides the command-line interface for G3, including interactive modes, TUI, and autonomous execution.

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
cargo build -p g3-cli
cargo test -p g3-cli
cargo run  # Runs the CLI via the main binary
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-cli -- -D warnings && cargo test -p g3-cli
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry point with run() and mode dispatching
├── machine_ui_writer.rs      # Machine-readable JSON output
├── retro_tui.rs              # Full-screen TUI interface
├── simple_output.rs          # Simple text output
├── theme.rs                  # Terminal color themes
├── tui.rs                    # TUI utilities
├── ui_writer_impl.rs         # UI writer implementation
tests/
├── coach_feedback_extraction_test.rs  # Coach feedback parsing tests
```

### Execution Modes

| Mode | Flag | Description |
|------|------|-------------|
| **Accumulative Autonomous** | (default) | Interactive + auto-triggers autonomous mode |
| **Single-shot** | `g3 "task"` | One task, then exit |
| **Autonomous** | `--autonomous` | Coach-player loop with requirements.md |
| **Chat** | `--chat` | Traditional interactive chat |
| **Planning** | `--planning` | Requirements-driven development |
| **Flock** | `--flock` | Multi-agent parallel development |
| **Retro TUI** | `--retro` | Full-screen terminal interface |
| **Console** | `--console` | Web-based monitoring UI |

### Additional CLI Flags

| Flag | Description |
|------|-------------|
| `--machine` | Machine-readable JSON output |
| `--codebase-fast-start <PATH>` | Pre-scan codebase using LLM before first turn |
| `--manual-compact` | Disable automatic context compaction |
| `--webdriver` | Enable WebDriver browser automation (Safari) |
| `--chrome-headless` | Enable Chrome in headless mode |
| `--macax` | Enable macOS Accessibility API |
| `--max-turns <N>` | Limit autonomous mode turns |
| `--codepath <PATH>` | Set project path for planning mode |
| `--workspace <PATH>` | Set workspace for logs/artifacts |

---

## Code Organization Patterns

### Main Entry Point

```rust
// src/lib.rs
pub async fn run() -> Result<()> {
    // 1. Parse CLI arguments with Clap
    // 2. Initialize configuration
    // 3. Dispatch to appropriate mode
    match mode {
        Mode::Autonomous => run_autonomous(...).await,
        Mode::Interactive => run_interactive(...).await,
        Mode::Planning => run_planning_mode(...).await,
        Mode::Flock => run_flock_mode(...).await,
        // ...
    }
}
```

### CLI Argument Parsing

Uses Clap derive macros:

```rust
#[derive(Parser)]
#[command(name = "g3")]
pub struct Args {
    /// Task to execute (single-shot mode)
    #[arg(default_value = None)]
    pub task: Option<String>,

    /// Run in autonomous mode
    #[arg(long)]
    pub autonomous: bool,

    /// Run in chat mode
    #[arg(long)]
    pub chat: bool,

    // ... more flags
}
```

### Interactive Mode Pattern

Uses `rustyline` for REPL with history:

```rust
let mut rl = DefaultEditor::new()?;
loop {
    let readline = rl.readline("g3> ");
    match readline {
        Ok(line) => {
            rl.add_history_entry(&line)?;
            // Process input
        }
        Err(ReadlineError::Interrupted) => break,
        Err(ReadlineError::Eof) => break,
        Err(err) => return Err(err.into()),
    }
}
```

### Control Commands

Interactive mode supports control commands:

| Command | Description |
|---------|-------------|
| `/compact` | Manually trigger summarization |
| `/thinnify` | Trigger context thinning |
| `/skinnify` | Full context thinning |
| `/readme` | Reload README.md and AGENTS.md |
| `/stats` | Show context and performance stats |
| `/help` | Display available commands |

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main entry point
   ```bash
   # Find the run function
   rg -n "pub async fn run" src/lib.rs

   # Find mode dispatching
   rg -n "run_autonomous|run_interactive|run_planning" src/lib.rs
   ```

2. **`src/machine_ui_writer.rs`** - JSON output mode
   ```bash
   rg -n "MachineUIWriter|write_json" src/machine_ui_writer.rs
   ```

3. **`src/retro_tui.rs`** - Full-screen TUI
   ```bash
   rg -n "RetroTUI|render|draw" src/retro_tui.rs
   ```

---

## Quick Search Commands

### Find Mode Implementations

```bash
# Find async mode functions
rg -n "async fn run_" src/lib.rs

# Find mode enum
rg -n "enum Mode|Mode::" src/lib.rs
```

### Find Control Commands

```bash
# Find control command handling
rg -n '"/compact"|"/thinnify"|"/stats"' src/lib.rs
```

### Find UI Components

```bash
# Find color/styling
rg -n "Color::|SetForegroundColor" src/

# Find TUI widgets
rg -n "Paragraph|Block|Borders" src/
```

---

## Common Gotchas

### Terminal Handling

- Uses `crossterm` for cross-platform terminal control
- Ratatui for TUI mode (retro interface)
- Always restore terminal state on exit

### Async Runtime

The CLI initializes the Tokio runtime. The `run()` function is async:

```rust
// Main binary (src/main.rs in root)
#[tokio::main]
async fn main() -> Result<()> {
    g3_cli::run().await
}
```

### History Persistence

Command history is saved to `~/.g3_history` via rustyline.

### Cancellation

Ctrl+C handling is implemented with `CancellationToken`:

```rust
let cancel_token = CancellationToken::new();
// Handle Ctrl+C to cancel ongoing operations
```

---

## Testing Guidelines

### Unit Tests

Location: `tests/` directory

```bash
# Run all CLI tests
cargo test -p g3-cli

# Run specific test
cargo test -p g3-cli coach_feedback
```

### Integration Testing

The CLI is best tested by running the actual binary:

```bash
# Single-shot test
cargo run -- "print hello world"

# Verify modes work
cargo run -- --help
cargo run -- --chat
```

---

## Dependencies

This crate depends on:
- `g3-core` - Core agent engine
- `g3-config` - Configuration management
- `g3-providers` - LLM providers
- `g3-planner` - Planning mode
- `g3-ensembles` - Flock mode

---

## Adding New CLI Flags

To add a new CLI flag:

1. Add field to `Args` struct with Clap attributes
2. Handle the flag in the mode dispatching logic
3. Update help text
4. Add tests if behavior is complex

```rust
/// Enable new feature
#[arg(long)]
pub new_feature: bool,
```
