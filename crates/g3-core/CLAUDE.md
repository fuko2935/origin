# g3-core - Core Agent Engine

**Technology**: Rust 2021, Tokio, Tree-sitter, Serde
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This is the largest and most critical crate in the G3 workspace. It contains the main agent orchestration logic, tool system, and context management.

---

## Development Commands

### This Crate

```bash
# From crate directory
cargo build
cargo test
cargo test -- --nocapture
cargo clippy -- -D warnings

# Run specific test
cargo test test_context_thinning
cargo test error_handling
```

### From Root

```bash
cargo build -p g3-core
cargo test -p g3-core
cargo test -p g3-core test_name
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-core -- -D warnings && cargo test -p g3-core
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                          # Main entry - Agent struct, tool execution (LARGE: ~300KB)
├── code_search/                    # Tree-sitter based code search
│   ├── mod.rs
│   └── searcher.rs
├── error_handling.rs               # Error classification (Recoverable/NonRecoverable)
├── feedback_extraction.rs          # Coach feedback extraction for autonomous mode
├── fixed_filter_json.rs            # JSON filtering utilities
├── project.rs                      # Project-level utilities
├── prompts.rs                      # System prompts for native/non-native tool use
├── retry.rs                        # Retry logic with exponential backoff
├── task_result.rs                  # Task completion result types
├── ui_writer.rs                    # UI output writer abstraction
├── *_test.rs                       # Colocated unit tests
tests/
├── test_context_thinning.rs        # Context management tests
├── test_token_counting.rs          # Token counting tests
├── test_todo_*.rs                  # TODO management tests
├── code_search_test.rs             # Code search tests
└── ...
```

### Key Types

| Type | Location | Purpose |
|------|----------|---------|
| `Agent` | `lib.rs` | Main agent orchestration |
| `ToolCall` | `lib.rs` | Tool invocation structure |
| `WebDriverSession` | `lib.rs` | Unified WebDriver (Safari/Chrome) |
| `StreamState` | `lib.rs` | Streaming response state |
| `ErrorContext` | `error_handling.rs` | Rich error context |
| `RetryConfig` | `retry.rs` | Retry configuration |
| `TaskResult` | `task_result.rs` | Task completion result |
| `CodeSearcher` | `code_search/searcher.rs` | Tree-sitter code search |

---

## Code Organization Patterns

### Agent Structure

The `Agent` struct is the core orchestrator. Key responsibilities:
- Managing conversation history
- Context window monitoring and thinning
- Tool execution and result handling
- Streaming response parsing

```rust
// Pattern: Agent creation and execution
let agent = Agent::new(config, provider_registry).await?;
let result = agent.execute_task("Your task here").await?;
```

### Tool Execution Pattern

Tools are defined as functions that take JSON arguments and return results:

```rust
// Tools are executed via match on tool name
match tool_call.tool.as_str() {
    "shell" => execute_shell(args).await,
    "read_file" => read_file(args).await,
    "write_file" => write_file(args).await,
    "str_replace" => apply_diff(args).await,
    _ => Err(anyhow!("Unknown tool: {}", tool_call.tool)),
}
```

### Error Handling Pattern

```rust
// ✅ DO: Use error classification for retry decisions
use crate::error_handling::{classify_error, ErrorClassification};

let classification = classify_error(&error);
match classification {
    ErrorClassification::Recoverable { .. } => {
        // Retry with backoff
    }
    ErrorClassification::NonRecoverable { .. } => {
        // Fail immediately
    }
}
```

### Retry Pattern

```rust
use crate::retry::{RetryConfig, execute_with_retry};

// ✅ DO: Use configured retry for API calls
let config = RetryConfig::default(); // or RetryConfig::planning("player")
let result = execute_with_retry(config, || async {
    provider.complete(request).await
}).await?;
```

---

## Key Files (Read These First)

### Core Understanding

1. **`src/lib.rs`** (Large file - use grep)
   ```bash
   rg -n "pub struct Agent" src/lib.rs
   rg -n "async fn execute_tool" src/lib.rs
   rg -n "fn process_stream" src/lib.rs
   ```

2. **`src/error_handling.rs`** - Error classification system
   ```bash
   rg -n "ErrorClassification|classify_error" src/error_handling.rs
   ```

3. **`src/retry.rs`** - Retry logic with backoff
   ```bash
   rg -n "RetryConfig|execute_with_retry" src/retry.rs
   ```

4. **`src/prompts.rs`** - System prompts
   ```bash
   rg -n "SYSTEM_PROMPT" src/prompts.rs
   ```

### Context Management

```bash
# Find context thinning logic
rg -n "thin|context|capacity" src/lib.rs | head -50

# Find summarization logic
rg -n "summarize|auto_summarize" src/lib.rs
```

### Tool System

```bash
# Find available tools
rg -n '"shell"|"read_file"|"write_file"|"str_replace"' src/lib.rs

# Find tool execution
rg -n "execute_tool|ToolCall" src/lib.rs
```

---

## Quick Search Commands

### Find Structs and Types

```bash
# Find struct definitions
rg -n "^pub struct \w+" src/

# Find enum definitions
rg -n "^pub enum \w+" src/

# Find trait implementations
rg -n "^impl.*for.*Agent" src/lib.rs
```

### Find Functions

```bash
# Find public async functions
rg -n "^pub async fn \w+" src/

# Find private functions
rg -n "^(async )?fn \w+" src/lib.rs | head -30
```

### Find Tests

```bash
# Find test modules
rg -n "#\[cfg\(test\)\]" src/

# Find test functions
rg -n "#\[test\]|#\[tokio::test\]" src/ tests/
```

---

## Common Gotchas

### Large File Navigation

`lib.rs` is ~300KB. Never try to read it entirely. Use these patterns:

```bash
# Find specific sections
rg -n "pub struct Agent" src/lib.rs
rg -n "impl Agent" src/lib.rs
rg -n "async fn.*execute" src/lib.rs
```

### Async Context

All I/O operations are async. Remember:
- Use `#[tokio::test]` for async tests
- Don't block with `.blocking_*()` calls
- Use `tokio::spawn` for concurrent tasks

### Tree-sitter Versions

Tree-sitter parsers are pinned to specific versions for compatibility:
- Most are 0.23.x
- tree-sitter core is 0.24
- Some (Kotlin) are disabled due to version conflicts

### Context Window

The agent monitors context usage. Key thresholds:
- 50% - First thinning pass
- 60-70% - Progressive thinning
- 80% - Auto-summarization triggered

---

## Testing Guidelines

### Unit Tests

Location: `src/*_test.rs` files

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_error_classification() {
        // Test error handling patterns
    }

    #[tokio::test]
    async fn test_retry_logic() {
        // Test async retry behavior
    }
}
```

### Integration Tests

Location: `tests/*.rs`

Key test files:
- `tests/test_context_thinning.rs` - Context management
- `tests/test_token_counting.rs` - Token counting accuracy
- `tests/test_todo_*.rs` - TODO management

### Running Tests

```bash
# All tests
cargo test -p g3-core

# Specific test file
cargo test -p g3-core --test test_context_thinning

# Specific test function
cargo test -p g3-core test_thinning_at_50_percent

# With output
cargo test -p g3-core -- --nocapture
```

---

## Dependencies on This Crate

This crate is used by:
- `g3-cli` - Main CLI interface
- `g3-planner` - Planning mode
- `g3-ensembles` - Multi-agent mode

Changes here affect the entire system. Test thoroughly.

---

## Adding New Tools

To add a new tool:

1. Define the tool schema (JSON schema for arguments)
2. Add execution logic in the tool match block
3. Add tool to the available tools list in prompts
4. Add tests for the new tool

```rust
// Example: Adding a new tool
"new_tool" => {
    let arg1 = args["arg1"].as_str().ok_or_else(|| anyhow!("Missing arg1"))?;
    // Tool implementation
    Ok(json!({ "result": "success" }))
}
```
