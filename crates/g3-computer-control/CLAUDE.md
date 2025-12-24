# g3-computer-control - OS Automation

**Technology**: Rust 2021, Core Graphics (macOS), X11 (Linux), Win32 (Windows)
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate provides cross-platform computer control capabilities including mouse/keyboard automation, screenshots, OCR, and WebDriver browser control.

---

## Development Commands

### This Crate

```bash
# From crate directory
cargo build
cargo test
cargo clippy -- -D warnings

# Run examples (require OS permissions)
cargo run --example test_screenshot_fix
cargo run --example test_vision
```

### From Root

```bash
cargo build -p g3-computer-control
cargo test -p g3-computer-control
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-computer-control -- -D warnings && cargo test -p g3-computer-control
```

---

## Architecture

### Directory Structure

```
src/
├── lib.rs                    # Main entry, trait definitions
├── types.rs                  # Shared types (Rect, Image, etc.)
├── platform/                 # Platform-specific implementations
│   ├── mod.rs
│   ├── macos.rs              # macOS Core Graphics implementation
│   ├── linux.rs              # X11 implementation
│   └── windows.rs            # Win32 implementation
├── webdriver/                # WebDriver implementations
│   ├── mod.rs                # WebDriver trait and factory
│   ├── safari.rs             # Safari WebDriver
│   └── chrome.rs             # Chrome/ChromeDriver
├── macax/                    # macOS Accessibility API
│   ├── mod.rs                # Controller and types
│   ├── controller.rs         # AX API implementation
│   └── tests.rs              # Unit tests
├── ocr/                      # OCR implementations
│   ├── mod.rs
│   ├── tesseract.rs          # Tesseract OCR
│   └── vision.rs             # macOS Vision framework
examples/
├── test_screenshot_fix.rs
├── test_vision.rs
├── test_window_capture.rs
├── macax_demo.rs             # macOS Accessibility demo
├── safari_demo.rs            # Safari WebDriver demo
└── ...
tests/
├── integration_test.rs
```

### Key Features

| Feature | Description |
|---------|-------------|
| **Mouse Control** | Click, move, drag operations |
| **Keyboard Control** | Type text, key presses |
| **Screenshots** | Full screen, region, window capture |
| **OCR** | Text extraction via Tesseract |
| **Window Management** | List, activate, capture windows |
| **WebDriver** | Safari and Chrome browser automation |
| **macOS Accessibility** | Native app control via Accessibility API |

---

## Code Organization Patterns

### Platform Abstraction

```rust
// Pattern: Platform-specific trait implementation
#[cfg(target_os = "macos")]
mod macos;
#[cfg(target_os = "linux")]
mod linux;
#[cfg(target_os = "windows")]
mod windows;

pub trait ComputerControl: Send + Sync {
    fn mouse_click(&self, x: i32, y: i32) -> Result<()>;
    fn type_text(&self, text: &str) -> Result<()>;
    fn take_screenshot(&self, region: Option<Rect>) -> Result<Image>;
    // ...
}
```

### WebDriver Pattern

```rust
// Pattern: WebDriver abstraction
#[async_trait]
pub trait WebDriverController: Send + Sync {
    async fn navigate(&mut self, url: &str) -> Result<()>;
    async fn current_url(&self) -> Result<String>;
    async fn find_element(&mut self, selector: &str) -> Result<WebElement>;
    async fn execute_script(&mut self, script: &str, args: Vec<Value>) -> Result<Value>;
    async fn screenshot(&mut self, path: &str) -> Result<()>;
    async fn quit(self) -> Result<()>;
}
```

### OCR Pattern

```rust
// Pattern: OCR text extraction
use crate::ocr::extract_text;

let text = extract_text(&image_path)?;
// or
let text = find_text_on_screen("Button Label")?;
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Main traits and types
   ```bash
   rg -n "trait ComputerControl|trait WebDriverController" src/lib.rs
   ```

2. **`src/platform/macos.rs`** - macOS implementation
   ```bash
   rg -n "CGEvent|NSScreen|screencapture" src/platform/macos.rs
   ```

3. **`src/webdriver/safari.rs`** - Safari WebDriver
   ```bash
   rg -n "SafariDriver|safaridriver" src/webdriver/safari.rs
   ```

4. **`src/webdriver/chrome.rs`** - Chrome WebDriver
   ```bash
   rg -n "ChromeDriver|chromedriver" src/webdriver/chrome.rs
   ```

---

## Quick Search Commands

### Find Platform Code

```bash
# Find macOS-specific code
rg -n "target_os.*macos" src/

# Find platform implementations
rg -n "impl.*ComputerControl" src/platform/
```

### Find WebDriver Code

```bash
# Find WebDriver implementations
rg -n "impl.*WebDriverController" src/webdriver/

# Find browser-specific code
rg -n "safari|chrome" src/webdriver/
```

### Find OCR Code

```bash
rg -n "tesseract|extract_text|ocr" src/
```

---

## Common Gotchas

### OS Permissions Required

Computer control requires explicit OS permissions:

**macOS**:
- System Preferences → Security & Privacy → Privacy → Accessibility
- Add your terminal app (Terminal.app, iTerm, etc.)
- For screenshots: Screen Recording permission

**Linux**:
- X11/Xtest access
- May need `xdotool` or similar

**Windows**:
- Run as administrator (first time)

### WebDriver Setup

**Safari**:
```bash
# Enable Safari Remote Automation (one time)
safaridriver --enable
# Or run: ./scripts/enable-safari-automation.sh
```

**Chrome**:
```bash
# Option 1: Chrome for Testing (recommended)
./scripts/setup-chrome-for-testing.sh

# Option 2: System Chrome + matching ChromeDriver
brew install chromedriver  # macOS
```

### ChromeDriver Version Mismatch

If you see "ChromeDriver version doesn't match Chrome version":
1. Use Chrome for Testing (guarantees matching versions)
2. Or manually download matching ChromeDriver

### Tesseract for OCR

OCR requires Tesseract to be installed:

```bash
# macOS
brew install tesseract

# Linux
apt install tesseract-ocr

# Windows
# Download from GitHub releases
```

---

## Testing Guidelines

### Unit Tests

```bash
# Run all tests
cargo test -p g3-computer-control

# Run specific test
cargo test -p g3-computer-control window_matching
```

### Integration Tests

Integration tests require OS permissions and may open windows:

```bash
# Run with visible output
cargo test -p g3-computer-control --test integration_test -- --nocapture
```

### Example Scripts

Examples demonstrate specific functionality:

```bash
# Test screenshot
cargo run -p g3-computer-control --example test_screenshot_fix

# Test OCR
cargo run -p g3-computer-control --example test_vision

# Test window capture
cargo run -p g3-computer-control --example test_window_capture
```

---

## Configuration

Enabled via config or CLI flags:

```toml
# ~/.config/g3/config.toml

[computer_control]
enabled = false  # Set true to enable
require_confirmation = true
max_actions_per_second = 5

[webdriver]
enabled = false
browser = "safari"  # or "chrome-headless"
safari_port = 4444
chrome_port = 9515
# chrome_binary = "/path/to/chrome"  # Optional

[macax]
enabled = false  # macOS Accessibility API
```

CLI flags:
```bash
g3 --webdriver        # Enable WebDriver (Safari default)
g3 --chrome-headless  # Enable Chrome headless
g3 --macax            # Enable macOS Accessibility
```

---

## Adding Platform Support

To add a new platform:

1. Create `src/platform/new_platform.rs`
2. Implement `ComputerControl` trait
3. Add conditional compilation in `src/platform/mod.rs`
4. Add tests

```rust
// src/platform/new_platform.rs
pub struct NewPlatformController { ... }

impl ComputerControl for NewPlatformController {
    fn mouse_click(&self, x: i32, y: i32) -> Result<()> {
        // Platform-specific implementation
    }
    // ... other methods
}
```
