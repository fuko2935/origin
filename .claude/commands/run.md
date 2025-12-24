Run the G3 agent. Optionally specify a mode or task.

## Usage

- `/run` - Run in default interactive mode
- `/run --autonomous` - Run in autonomous mode
- `/run --chat` - Run in chat mode
- `/run --planning` - Run in planning mode
- `/run --flock` - Run in flock mode
- `/run "task"` - Run single-shot with task

## Steps

1. **Build if needed**:
   ```bash
   cargo build
   ```

2. **Run with specified arguments**:
   ```bash
   cargo run -- $ARGUMENTS
   ```

## Available Flags

| Flag | Description |
|------|-------------|
| `--autonomous` | Coach-player loop with requirements.md |
| `--auto` | Accumulative autonomous (default) |
| `--chat` | Traditional interactive chat |
| `--planning` | Requirements-driven development |
| `--flock` | Multi-agent parallel development |
| `--webdriver` | Enable WebDriver browser automation |
| `--chrome-headless` | Enable Chrome headless mode |
| `--macax` | Enable macOS Accessibility API |
| `--machine` | Machine-readable JSON output |
| `--codebase-fast-start .` | Pre-scan codebase |
| `--manual-compact` | Disable auto context compaction |

## For Release Build

```bash
cargo run --release -- $ARGUMENTS
```
