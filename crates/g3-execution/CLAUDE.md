# g3-execution - Code Execution Engine

**Technology**: Rust 2021, Tokio, Futures
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate provides safe code execution capabilities for G3, including shell command execution with streaming output.

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
cargo build -p g3-execution
cargo test -p g3-execution
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-execution -- -D warnings && cargo test -p g3-execution
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry, execution engine
examples/
├── setup_coverage_tools.rs   # Coverage tool setup
```

### Execution Modes

| Mode | Description |
|------|-------------|
| **Shell** | Direct command execution with streaming output (primary) |
| **Python** | Script execution via temporary files (legacy) |
| **JavaScript** | Node.js-based execution (legacy) |

---

## Code Organization Patterns

### Command Execution Pattern

```rust
// Pattern: Execute shell command with streaming output
pub async fn execute_shell(command: &str) -> Result<ExecutionResult> {
    let mut child = Command::new("sh")
        .arg("-c")
        .arg(command)
        .stdout(Stdio::piped())
        .stderr(Stdio::piped())
        .spawn()?;

    // Stream output as it arrives
    let stdout = child.stdout.take().unwrap();
    let reader = BufReader::new(stdout);

    let mut output = String::new();
    let mut lines = reader.lines();
    while let Some(line) = lines.next_line().await? {
        output.push_str(&line);
        output.push('\n');
        // Optionally yield line for real-time display
    }

    let status = child.wait().await?;
    Ok(ExecutionResult {
        stdout: output,
        exit_code: status.code().unwrap_or(-1),
    })
}
```

### Script Execution Pattern

```rust
// Pattern: Execute script via temporary file
pub async fn execute_python(script: &str) -> Result<ExecutionResult> {
    let dir = tempfile::tempdir()?;
    let script_path = dir.path().join("script.py");
    tokio::fs::write(&script_path, script).await?;

    execute_shell(&format!("python3 {}", script_path.display())).await
}
```

### Streaming Output Pattern

```rust
// Pattern: Stream execution output
pub struct StreamingOutput {
    pub stdout_rx: mpsc::Receiver<String>,
    pub stderr_rx: mpsc::Receiver<String>,
    pub exit_code: oneshot::Receiver<i32>,
}

impl StreamingOutput {
    pub async fn collect(self) -> Result<ExecutionResult> {
        // Collect all output and wait for completion
    }
}
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main execution engine
   ```bash
   rg -n "pub async fn execute|pub fn run" src/lib.rs
   ```

---

## Quick Search Commands

### Find Execution Logic

```bash
# Find command execution
rg -n "Command::new|spawn|execute" src/lib.rs

# Find streaming logic
rg -n "BufReader|lines|stream" src/lib.rs

# Find error handling
rg -n "Result|Error|exit_code" src/lib.rs
```

---

## Common Gotchas

### Async Execution

All I/O operations are async using Tokio:
- Use `tokio::process::Command` not `std::process::Command`
- Use `async_trait` for trait implementations
- Handle timeouts properly for long-running commands

### Output Buffering

Output is line-buffered by default. For immediate output:
- Consider using PTY for interactive commands
- Some commands may not flush output immediately

### Exit Codes

Exit codes may vary by platform:
- Unix: 0-255
- Windows: Different conventions
- Signals: Negative values for signal termination

### Temporary Files

When executing scripts:
- Use `tempfile` crate for proper cleanup
- Ensure temp directory persists for script duration
- Clean up explicitly on error paths

---

## Testing Guidelines

### Unit Tests

```bash
# Run all tests
cargo test -p g3-execution

# Run with output
cargo test -p g3-execution -- --nocapture
```

### Test Patterns

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_shell_execution() {
        let result = execute_shell("echo hello").await.unwrap();
        assert_eq!(result.exit_code, 0);
        assert!(result.stdout.contains("hello"));
    }

    #[tokio::test]
    async fn test_exit_code() {
        let result = execute_shell("exit 42").await.unwrap();
        assert_eq!(result.exit_code, 42);
    }
}
```

---

## Security Considerations

### Command Injection

Be careful with user input in commands:
- Never interpolate untrusted input directly
- Use argument arrays when possible
- Sanitize and validate command strings

### Resource Limits

Consider implementing:
- Execution timeouts
- Memory limits
- CPU limits (cgroups on Linux)

### Sandboxing

Currently, commands run with the agent's permissions. Future improvements:
- Container-based isolation
- chroot/jail environments
- Reduced privilege execution
