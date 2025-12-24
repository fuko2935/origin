# G3 - AI Coding Agent

## Overview

- **Type**: Rust Workspace (Monorepo) with 9 crates
- **Stack**: Rust 2021, Tokio, Clap, Serde, Tracing, Tree-sitter
- **Architecture**: Modular agent system with pluggable LLM providers
- **Purpose**: AI coding agent with autonomous task execution and computer control

This CLAUDE.md is the authoritative source for development guidelines.
Subdirectory CLAUDE.md files extend these rules for specific crates.

---

## Universal Development Rules

### Code Quality (MUST)

- **MUST** write idiomatic Rust following the Rust API Guidelines
- **MUST** use `anyhow::Result` for error propagation in application code
- **MUST** use `thiserror` for library error types with proper error enums
- **MUST** run `cargo fmt` before committing any code
- **MUST** pass `cargo clippy -- -D warnings` with zero warnings
- **MUST** include tests for all new functionality
- **MUST NOT** commit secrets, API keys, or credentials to the repository
- **MUST NOT** use `unwrap()` in production code paths - use `?` or proper error handling

### Best Practices (SHOULD)

- **SHOULD** prefer `async fn` for I/O operations (Tokio runtime)
- **SHOULD** use traits for abstraction and testability
- **SHOULD** keep functions focused and under 50 lines when possible
- **SHOULD** use descriptive variable names (no single letters except iterators)
- **SHOULD** document public APIs with `///` doc comments
- **SHOULD** prefer composition over inheritance patterns

### Anti-Patterns (MUST NOT)

- **MUST NOT** use `.clone()` excessively - consider borrowing or `Arc`
- **MUST NOT** block the async runtime with synchronous operations
- **MUST NOT** ignore `Result` values - always handle or propagate errors
- **MUST NOT** use `unsafe` without explicit justification and review
- **MUST NOT** leave `TODO` or `FIXME` comments without associated issues

---

## Core Commands

### Development

```bash
# Build all workspace crates
cargo build

# Build release version
cargo build --release

# Run the main G3 agent
cargo run

# Run with specific mode
cargo run -- --autonomous         # Autonomous mode (coach-player loop)
cargo run -- --auto              # Accumulative autonomous (default interactive)
cargo run -- --chat              # Traditional chat mode
cargo run -- --planning          # Planning mode
cargo run -- --flock             # Flock mode (parallel multi-agent)
cargo run -- --webdriver         # Enable WebDriver browser automation
cargo run -- --chrome-headless   # Enable Chrome headless mode
cargo run -- --macax             # Enable macOS Accessibility API
cargo run -- --machine           # Machine-readable JSON output
cargo run -- --codebase-fast-start .  # Pre-scan codebase before first turn
cargo run -- --manual-compact    # Disable automatic context compaction
```

### Testing

```bash
# Run all tests
cargo test

# Run tests for specific crate
cargo test -p g3-core
cargo test -p g3-cli
cargo test -p g3-providers

# Run specific test
cargo test -p g3-core test_context_thinning

# Run tests with output
cargo test -- --nocapture
```

### Code Quality

```bash
# Format all code
cargo fmt

# Check formatting without changes
cargo fmt -- --check

# Run Clippy linter
cargo clippy -- -D warnings

# Run all quality checks (pre-commit)
cargo fmt -- --check && cargo clippy -- -D warnings && cargo test
```

### Documentation

```bash
# Generate and open documentation
cargo doc --open

# Generate docs for specific crate
cargo doc -p g3-core --open
```

---

## Project Structure

### Entry Point

- **`src/main.rs`** - Thin wrapper that calls `g3_cli::run()`

### Workspace Crates

| Crate | Purpose | Entry Point |
|-------|---------|-------------|
| [g3-cli](crates/g3-cli/CLAUDE.md) | CLI interface, TUI, interaction modes | `src/lib.rs` → `run()` |
| [g3-core](crates/g3-core/CLAUDE.md) | Agent engine, tools, context management | `src/lib.rs` |
| [g3-providers](crates/g3-providers/CLAUDE.md) | LLM provider abstractions | `src/lib.rs` |
| [g3-config](crates/g3-config/CLAUDE.md) | Configuration management (TOML) | `src/lib.rs` |
| [g3-execution](crates/g3-execution/CLAUDE.md) | Code execution engine | `src/lib.rs` |
| [g3-computer-control](crates/g3-computer-control/CLAUDE.md) | OS automation, WebDriver, OCR | `src/lib.rs` |
| [g3-console](crates/g3-console/CLAUDE.md) | Web-based console UI | `src/main.rs` |
| [g3-ensembles](crates/g3-ensembles/CLAUDE.md) | Multi-agent "Flock" mode | `src/lib.rs` |
| [g3-planner](crates/g3-planner/CLAUDE.md) | Planning/requirements mode | `src/lib.rs` |

### Crate Dependency Graph

```
src/main.rs
    └── g3-cli (entry point)
            ├── g3-core ─────────┬── g3-providers
            │                    ├── g3-config
            │                    ├── g3-execution
            │                    └── g3-computer-control
            ├── g3-planner
            ├── g3-ensembles
            └── g3-console
```

### Other Directories

- **`logs/`** - Session logs (auto-created, gitignored)
- **`g3-plan/`** - Planning artifacts storage
- **`scripts/`** - Utility scripts (e.g., Chrome setup)
- **`examples/`** - Example usage scripts

---

## Quick Find Commands

### Code Navigation

```bash
# Find a struct or enum definition
rg -n "^(pub )?(struct|enum) \w+" crates/

# Find trait definitions
rg -n "^(pub )?trait \w+" crates/

# Find impl blocks for a type
rg -n "impl.*Agent" crates/g3-core/src/

# Find function definitions in a file
rg -n "^(pub )?(async )?fn \w+" crates/g3-core/src/lib.rs

# Find test functions
rg -n "#\[test\]" crates/
rg -n "#\[tokio::test\]" crates/
```

### Dependency Analysis

```bash
# Check crate dependencies
cargo tree -p g3-core

# Find unused dependencies
cargo +nightly udeps

# Check for outdated dependencies
cargo outdated
```

### Error Patterns

```bash
# Find error handling patterns
rg -n "anyhow::(Result|Error)" crates/

# Find error classification
rg -n "Recoverable|NonRecoverable" crates/g3-core/

# Find retry logic
rg -n "retry_with_backoff|execute_with_retry" crates/
```

### Tool System

```bash
# Find available tools
rg -n '"shell"|"read_file"|"write_file"|"str_replace"' crates/g3-core/src/lib.rs

# Find tool execution
rg -n "execute_tool|ToolCall" crates/g3-core/src/
```

---

## Architecture Patterns

### Error Handling

The project uses a two-tier error handling approach:

1. **Application Errors** (`anyhow::Result`): For propagating errors up the call stack
   - Example: `crates/g3-core/src/lib.rs`

2. **Error Classification**: Errors are classified as Recoverable or NonRecoverable
   - Location: `crates/g3-core/src/error_handling.rs`
   - Recoverable: Rate limits, network issues, 5xx errors, timeouts
   - NonRecoverable: Auth failures, invalid requests

```rust
// ✅ DO: Use error classification for retry decisions
use crate::error_handling::{classify_error, ErrorClassification};

match classify_error(&error) {
    ErrorClassification::Recoverable { .. } => { /* retry */ }
    ErrorClassification::NonRecoverable { .. } => { /* fail */ }
}
```

### Retry Logic

Centralized retry logic with exponential backoff:
- Location: `crates/g3-core/src/retry.rs`
- Default mode: 3 retries
- Autonomous mode: 6 retries (configurable)
- Uses jitter to avoid thundering herd

```rust
// ✅ DO: Use configured retry for API calls
use crate::retry::{RetryConfig, execute_with_retry};

let config = RetryConfig::default();
let result = execute_with_retry(config, || async {
    provider.complete(request).await
}).await?;
```

### Provider Abstraction

```rust
// Provider trait defined in g3-providers
#[async_trait]
pub trait LLMProvider: Send + Sync {
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionResponse>;
    async fn complete_stream(&self, request: CompletionRequest) -> Result<impl Stream<...>>;
    fn supports_tools(&self) -> bool;
    fn max_context_length(&self) -> usize;
}
```

### Context Management

Automatic context window management:
- Monitors token usage with percentage-based thresholds
- Thinning at 50%, 60%, 70%, 80% capacity
- Auto-summarization at 80% to prevent overflow
- Location: `crates/g3-core/src/lib.rs`

---

## Security Guidelines

### Secrets Management

- **NEVER** commit API keys, tokens, or credentials
- Use `~/.config/g3/config.toml` for local credentials (not in repo)
- Use environment variables for CI/CD secrets
- Config example file: `config.example.toml` (safe to commit)

### Safe Operations

- Confirm before destructive operations (`rm -rf`, `git push --force`)
- Review generated shell commands before execution
- Use staging environment for risky operations
- WebDriver and computer control require explicit enablement

### Files to Protect

| File | Contains | Action |
|------|----------|--------|
| `~/.config/g3/config.toml` | API keys | Never read/edit |
| `.env*` files | Environment secrets | Warn before edit |
| `config.example.toml` | Template (safe) | Ask before edit |
| `logs/` directory | Session data | Read-only |

---

## Git Workflow

### Branch Naming

- Features: `feature/short-description`
- Fixes: `fix/issue-description`
- Refactors: `refactor/what-changed`

### Commit Messages

Use Conventional Commits format:

```
feat: add new provider support for OpenAI
fix: resolve context overflow in long sessions
docs: update README with new commands
refactor: simplify error handling in g3-core
test: add integration tests for planner
chore: update dependencies
```

### Pre-Commit Checklist

```bash
# Run before every commit
cargo fmt -- --check && cargo clippy -- -D warnings && cargo test
```

---

## Testing Requirements

### Unit Tests

- Location: Colocated with source (`*_test.rs` or `tests.rs` modules)
- Example: `crates/g3-core/src/error_handling_test.rs`
- All business logic must have unit tests

### Integration Tests

- Location: `crates/*/tests/` directories
- Example: `crates/g3-core/tests/test_context_thinning.rs`
- Test cross-module interactions

### Test Patterns

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_functionality() {
        // Arrange
        // Act
        // Assert
    }

    #[tokio::test]
    async fn test_async_functionality() {
        // Use tokio::test for async tests
    }
}
```

### Running Tests

```bash
# All tests
cargo test

# Specific crate
cargo test -p g3-core

# Specific test
cargo test -p g3-core test_context_thinning

# With output
cargo test -- --nocapture

# Watch mode (requires cargo-watch)
cargo watch -x test
```

---

## Available Tools

### Standard Tools

- `cargo` - Rust build system and package manager
- `rustfmt` - Code formatting (via `cargo fmt`)
- `clippy` - Linter (via `cargo clippy`)
- `git` - Version control
- `rg` (ripgrep) - Fast code search
- `gh` - GitHub CLI (optional)

### Tool Permissions

| Permission | Tools/Commands |
|------------|----------------|
| ✅ **Allowed** | `cargo build/test/fmt/clippy/doc/run/check/tree` |
| ✅ **Allowed** | `git status/diff/log/branch/add/commit` |
| ✅ **Allowed** | `rg`, `ls`, `cat`, `head`, `tail`, `wc`, `find` |
| ✅ **Allowed** | Read any file in repository |
| ✅ **Allowed** | Write/Edit files in `crates/` and `src/` |
| ⚠️ **Ask First** | `git push`, `git checkout`, `cargo publish` |
| ⚠️ **Ask First** | Editing `config.example.toml` |
| ⚠️ **Ask First** | Editing workspace `Cargo.toml` dependencies |
| ❌ **Blocked** | `rm -rf`, `git push --force`, `git reset --hard` |
| ❌ **Blocked** | `cargo clean` |

---

## Common Gotchas

### Rust-Specific

- **Async in Tests**: Use `#[tokio::test]` not `#[test]` for async tests
- **Workspace Dependencies**: Add new deps to root `Cargo.toml` workspace section first
- **Feature Flags**: llama_cpp uses `metal` feature for macOS GPU acceleration
- **Platform-Specific**: `g3-computer-control` has different impls per OS

### Project-Specific

- **Large Files**: `g3-core/src/lib.rs` is ~300KB - use grep for navigation
- **No CI/CD**: Tests must be run locally before committing
- **Config Path**: Default config at `~/.config/g3/config.toml`
- **Tree-sitter Versions**: Some parsers pinned to specific versions for compatibility

### Common Mistakes

```rust
// ❌ DON'T: Use unwrap in production code
let value = some_option.unwrap();

// ✅ DO: Use ? or proper error handling
let value = some_option.ok_or_else(|| anyhow!("Missing value"))?;

// ❌ DON'T: Block the async runtime
std::thread::sleep(Duration::from_secs(1));

// ✅ DO: Use async sleep
tokio::time::sleep(Duration::from_secs(1)).await;

// ❌ DON'T: Clone excessively
let data = expensive_data.clone();
process(data.clone());

// ✅ DO: Use references or Arc
let data = Arc::new(expensive_data);
process(Arc::clone(&data));
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Build | `cargo build` |
| Test | `cargo test` |
| Format | `cargo fmt` |
| Lint | `cargo clippy -- -D warnings` |
| Run | `cargo run` |
| Run Release | `cargo run --release` |
| Docs | `cargo doc --open` |
| Pre-commit | `cargo fmt -- --check && cargo clippy -- -D warnings && cargo test` |

---

## Specialized Context

When working in specific directories, refer to their CLAUDE.md:

| Directory | Purpose |
|-----------|---------|
| [crates/g3-cli/CLAUDE.md](crates/g3-cli/CLAUDE.md) | CLI Development |
| [crates/g3-core/CLAUDE.md](crates/g3-core/CLAUDE.md) | Core Agent Engine |
| [crates/g3-providers/CLAUDE.md](crates/g3-providers/CLAUDE.md) | LLM Providers |
| [crates/g3-config/CLAUDE.md](crates/g3-config/CLAUDE.md) | Configuration |
| [crates/g3-execution/CLAUDE.md](crates/g3-execution/CLAUDE.md) | Code Execution |
| [crates/g3-computer-control/CLAUDE.md](crates/g3-computer-control/CLAUDE.md) | Computer Control |
| [crates/g3-console/CLAUDE.md](crates/g3-console/CLAUDE.md) | Web Console |
| [crates/g3-ensembles/CLAUDE.md](crates/g3-ensembles/CLAUDE.md) | Flock Mode |
| [crates/g3-planner/CLAUDE.md](crates/g3-planner/CLAUDE.md) | Planning Mode |

These files provide detailed, context-specific guidance for each crate.
