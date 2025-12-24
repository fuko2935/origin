Generate and open Rust documentation for G3.

## Usage

- `/docs` - Generate docs for all crates
- `/docs g3-core` - Generate docs for specific crate

## Steps

1. **Generate documentation**:
   ```bash
   # All crates
   cargo doc

   # Specific crate
   cargo doc -p $ARGUMENTS
   ```

2. **Check for documentation warnings**:
   - Missing doc comments on public items
   - Broken intra-doc links
   - Invalid code examples in docs

3. **Report location**:
   ```
   Documentation generated at: target/doc/<crate>/index.html
   ```

## Opening Docs

To open in browser:
```bash
cargo doc --open
# or
cargo doc -p $ARGUMENTS --open
```

## Common Crates

| Crate | Description |
|-------|-------------|
| `g3-core` | Core agent engine |
| `g3-cli` | Command line interface |
| `g3-providers` | LLM provider abstractions |
| `g3-config` | Configuration management |
| `g3-execution` | Code execution engine |
| `g3-computer-control` | OS automation |
| `g3-ensembles` | Multi-agent mode |
| `g3-planner` | Planning mode |
