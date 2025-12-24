# g3-config - Configuration Management

**Technology**: Rust 2021, TOML, Serde, shellexpand
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate handles all configuration loading, parsing, and validation for G3, including provider settings and runtime options.

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
cargo build -p g3-config
cargo test -p g3-config
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-config -- -D warnings && cargo test -p g3-config
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry, config structs, loading logic
├── tests.rs                  # Unit tests
tests/
├── test_multiple_tool_calls.rs
```

### Configuration Priority

| Priority | Source | Description |
|----------|--------|-------------|
| 5 (Highest) | CLI arguments | Direct overrides |
| 4 | Environment (`G3_*`) | Runtime overrides |
| 3 | `./g3.toml` | Project-local config |
| 2 | `~/.config/g3/config.toml` | User global config |
| 1 (Lowest) | Built-in defaults | Hardcoded fallbacks |

---

## Code Organization Patterns

### Config Loading Pattern

```rust
// Pattern: Load configuration with fallbacks
pub fn load_config() -> Result<Config> {
    let mut config = Config::default();

    // Load from file if exists
    if let Some(path) = find_config_file() {
        let file_config = load_from_file(&path)?;
        config.merge(file_config);
    }

    // Apply environment overrides
    config.apply_env_overrides();

    Ok(config)
}
```

### Named Provider Configuration

```rust
// Pattern: Named provider configs
// Format: providers.<type>.<name>
// Example: providers.anthropic.default, providers.anthropic.planner

pub struct ProvidersConfig {
    pub default_provider: String,  // e.g., "anthropic.default"
    pub planner: Option<String>,   // Optional override for planner
    pub coach: Option<String>,     // Optional override for coach
    pub player: Option<String>,    // Optional override for player
    // Individual provider configs
    pub anthropic: HashMap<String, AnthropicConfig>,
    pub databricks: HashMap<String, DatabricksConfig>,
    // ...
}
```

### Path Expansion Pattern

```rust
// Pattern: Expand shell paths
use shellexpand::tilde;

let expanded = tilde("~/.config/g3/config.toml");
let path = PathBuf::from(expanded.as_ref());
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main config structures
   ```bash
   rg -n "pub struct.*Config" src/lib.rs
   ```

2. **`src/tests.rs`** - Unit tests
   ```bash
   rg -n "#\[test\]" src/tests.rs
   ```

---

## Quick Search Commands

### Find Config Structures

```bash
# Find config structs
rg -n "pub struct.*Config" src/

# Find default implementations
rg -n "impl Default for" src/

# Find loading functions
rg -n "fn load|fn merge" src/
```

### Find Environment Variables

```bash
# Find env var handling
rg -n "G3_|env::var" src/
```

---

## Common Gotchas

### Path Expansion

Always use `shellexpand` for paths that might contain `~`:
- Config paths: `~/.config/g3/`
- Model paths: `~/.cache/g3/models/`

### Config Priority

Remember the priority order (highest to lowest):
1. CLI arguments
2. Environment variables
3. Project config (`./g3.toml`)
4. User config (`~/.config/g3/config.toml`)
5. Built-in defaults

### Named Provider Format

Provider references use dot notation:
- `anthropic.default` - the "default" config under "anthropic"
- `databricks.planner` - the "planner" config under "databricks"

### TOML Parsing Errors

Common issues:
- Missing quotes around strings with special chars
- Incorrect table nesting
- Type mismatches (string vs int)

---

## Testing Guidelines

### Unit Tests

```bash
# Run all config tests
cargo test -p g3-config

# Run with output
cargo test -p g3-config -- --nocapture
```

### Test Patterns

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::tempdir;

    #[test]
    fn test_config_loading() {
        let dir = tempdir().unwrap();
        // Create test config file
        // Load and verify
    }
}
```

---

## Configuration Reference

### Example Config

```toml
[providers]
default_provider = "anthropic.default"
planner = "anthropic.planner"    # Optional
coach = "anthropic.default"      # Optional
player = "anthropic.default"     # Optional

[providers.anthropic.default]
api_key = "sk-ant-..."
model = "claude-sonnet-4-5"
max_tokens = 64000
temperature = 0.1

[providers.anthropic.planner]
api_key = "sk-ant-..."
model = "claude-3-opus-20240229"
max_tokens = 8192

[providers.databricks.default]
host = "https://workspace.cloud.databricks.com"
model = "databricks-claude-sonnet-4"
use_oauth = true

[agent]
max_context_length = 8192
enable_streaming = true
timeout_seconds = 60
max_retry_attempts = 3
autonomous_max_retry_attempts = 6

[computer_control]
enabled = false
require_confirmation = true

[webdriver]
enabled = false
browser = "safari"
```

See `config.example.toml` in the repo root for full reference.
