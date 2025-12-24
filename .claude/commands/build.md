Build the G3 workspace. Optionally specify a crate name.

## Usage

- `/build` - Build all crates
- `/build g3-core` - Build specific crate
- `/build --release` - Build in release mode

## Steps

1. **Build the project**:
   ```bash
   # If no arguments, build all
   cargo build

   # If crate specified
   cargo build -p $ARGUMENTS

   # If --release specified
   cargo build --release
   ```

2. **Report results**:
   - Number of warnings (should be 0)
   - Build time
   - Any errors with file:line references

## On Failure

If build fails:
1. Read the error message carefully
2. Identify the file and line number
3. Suggest a fix based on the error type

## Full Quality Build

For a complete quality-checked build:
```bash
cargo fmt -- --check && \
cargo clippy -- -D warnings && \
cargo test && \
cargo build --release
```
