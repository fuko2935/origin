Search the G3 codebase for a pattern.

## Usage

- `/search "pattern"` - Search for pattern in all files
- `/search "pattern" crates/g3-core` - Search in specific directory

## Steps

1. **Search with ripgrep**:
   ```bash
   # Basic search
   rg -n "$ARGUMENTS" crates/

   # With context
   rg -n -C 3 "$ARGUMENTS" crates/

   # Case insensitive
   rg -ni "$ARGUMENTS" crates/
   ```

2. **Report results**:
   - File paths with line numbers
   - Matching lines with context
   - Total match count

## Common Search Patterns

```bash
# Find struct/enum definitions
rg -n "^(pub )?(struct|enum) \w+" crates/

# Find trait definitions
rg -n "^(pub )?trait \w+" crates/

# Find impl blocks
rg -n "impl.*TypeName" crates/

# Find function definitions
rg -n "^(pub )?(async )?fn \w+" crates/

# Find test functions
rg -n "#\[test\]|#\[tokio::test\]" crates/

# Find TODO/FIXME
rg -n "TODO|FIXME" crates/
```

## File Type Filters

```bash
# Only Rust files
rg -n --type rust "$ARGUMENTS"

# Only TOML files
rg -n --type toml "$ARGUMENTS"
```

Results show file path and line number for easy navigation.
