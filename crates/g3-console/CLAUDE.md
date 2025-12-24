# g3-console - Web-Based Monitoring Console

**Technology**: Rust 2021, Axum, Tokio, React
**Entry Point**: `src/lib.rs` (library) / `src/main.rs` (binary)
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate provides a web-based UI for monitoring and managing G3 agent instances, with real-time log streaming and process control.

---

## Development Commands

### This Crate

```bash
# From crate directory
cargo build
cargo test
cargo clippy -- -D warnings

# Run the console server
cargo run
```

### From Root

```bash
cargo build -p g3-console
cargo test -p g3-console
cargo run -p g3-console
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-console -- -D warnings && cargo test -p g3-console
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Library exports
├── main.rs                   # Binary entry point
├── api/                      # REST API handlers
│   ├── mod.rs
│   ├── control.rs            # Process control endpoints
│   ├── instances.rs          # Instance management
│   ├── logs.rs               # Log streaming
│   └── state.rs              # Shared state
├── models/                   # Data models
│   ├── mod.rs
│   ├── instance.rs           # Instance model
│   └── message.rs            # Message model
├── process/                  # Process management
│   ├── mod.rs
│   ├── controller.rs         # Process controller
│   └── detector.rs           # G3 process detection
├── logs.rs                   # Log file handling
└── launch.rs                 # G3 instance launching
web/                          # React frontend
├── index.html
├── js/
├── css/
└── src/                      # React components
    ├── App.jsx
    ├── main.jsx
    ├── components/
    └── pages/
examples/
├── debug_detector.rs
├── test_api.rs
└── test_detector.rs
```

### Key Components

| Component | Description |
|-----------|-------------|
| **API Server** | Axum-based REST API (port 9090) |
| **Instance Manager** | Tracks running G3 instances |
| **Log Streamer** | Real-time log file watching |
| **Process Detector** | Finds G3 processes on the system |
| **Web UI** | React-based frontend |

---

## Code Organization Patterns

### Axum Server Pattern

```rust
// Pattern: Set up Axum server
use axum::{Router, routing::{get, post}};

pub async fn start_server(port: u16) -> Result<()> {
    let state = AppState::new();

    let app = Router::new()
        .route("/api/instances", get(list_instances))
        .route("/api/instances/:id/logs", get(stream_logs))
        .route("/api/instances/:id/stop", post(stop_instance))
        .nest_service("/", ServeDir::new("web"))
        .with_state(state);

    let addr = SocketAddr::from(([0, 0, 0, 0], port));
    axum::Server::bind(&addr).serve(app.into_make_service()).await?;
    Ok(())
}
```

### Shared State Pattern

```rust
// Pattern: Shared application state
use std::sync::Arc;
use tokio::sync::RwLock;

#[derive(Clone)]
pub struct AppState {
    pub instances: Arc<RwLock<HashMap<Uuid, Instance>>>,
    pub log_watchers: Arc<RwLock<HashMap<Uuid, LogWatcher>>>,
}
```

### Process Detection Pattern

```rust
// Pattern: Detect running G3 processes
use sysinfo::{System, ProcessExt};

pub fn detect_g3_processes() -> Vec<ProcessInfo> {
    let mut system = System::new_all();
    system.refresh_all();

    system.processes()
        .values()
        .filter(|p| p.name().contains("g3"))
        .map(|p| ProcessInfo::from(p))
        .collect()
}
```

### Log Streaming Pattern

```rust
// Pattern: Stream log file changes
use notify::{Watcher, RecursiveMode};

pub async fn stream_logs(path: &Path) -> impl Stream<Item = String> {
    let (tx, rx) = mpsc::channel(100);

    let mut watcher = notify::recommended_watcher(move |res| {
        if let Ok(event) = res {
            // Read new log lines and send
        }
    })?;

    watcher.watch(path, RecursiveMode::NonRecursive)?;

    ReceiverStream::new(rx)
}
```

---

## Key Files

### Core Understanding

1. **`src/main.rs`** - Binary entry point
   ```bash
   rg -n "async fn main" src/main.rs
   ```

2. **`src/api/mod.rs`** - API routes
   ```bash
   rg -n "Router|route" src/api/
   ```

3. **`src/process/detector.rs`** - Process detection
   ```bash
   rg -n "detect|find.*process" src/process/detector.rs
   ```

---

## Quick Search Commands

### Find API Endpoints

```bash
# Find route definitions
rg -n "\.route\(|get\(|post\(" src/api/

# Find handler functions
rg -n "async fn.*handler|pub async fn" src/api/
```

### Find React Components

```bash
# Find React components
rg -n "function.*\(\)|const.*=" web/src/components/
```

---

## Common Gotchas

### Port Conflicts

Default port is 9090. If it's in use:
- Check for existing processes: `lsof -i :9090`
- Use a different port via CLI flag

### CORS Configuration

The server includes CORS middleware for development:
- Allows requests from different origins
- May need adjustment for production

### File Watching

Log file watching uses the `notify` crate:
- Cross-platform but behavior differs
- macOS uses FSEvents
- Linux uses inotify

### Static File Serving

The web UI is served from the `web/` directory:
- In dev: relative path from crate root
- In production: may need to bundle or adjust paths

---

## Testing Guidelines

### Unit Tests

```bash
# Run all tests
cargo test -p g3-console

# Run with output
cargo test -p g3-console -- --nocapture
```

### Example Scripts

```bash
# Test process detection
cargo run -p g3-console --example test_detector

# Test API
cargo run -p g3-console --example test_api
```

### Integration Testing

The console is best tested by running it:

```bash
# Start the console
cargo run -p g3-console

# Open in browser
open http://localhost:9090
```

---

## Web UI Development

### Frontend Structure

The React frontend is in `web/`:
- `web/src/` - React source files
- `web/js/` - Bundled JavaScript
- `web/css/` - Stylesheets

### Key React Components

| Component | Purpose |
|-----------|---------|
| `App.jsx` | Main application |
| `ChatView.jsx` | Conversation display |
| `InstancePanel.jsx` | Instance list |
| `ToolCall.jsx` | Tool call display |
| `ProgressBar.jsx` | Progress indicators |

### Styling

Uses a dark theme with custom CSS:
- `web/styles/app.css` - Main styles
- `web/css/highlight-dark.min.css` - Code highlighting

---

## Configuration

The console can be configured via CLI:

```bash
# Custom port
g3-console --port 8080

# Custom log directory
g3-console --log-dir /path/to/logs
```

Or via environment variables:
- `G3_CONSOLE_PORT` - Server port
- `G3_LOG_DIR` - Log directory path
