Run full code quality checks on the G3 codebase.

This is the pre-commit validation that should pass before any commit.

## Steps

1. **Check formatting**:
   ```bash
   cargo fmt -- --check
   ```

2. **Run Clippy linter**:
   ```bash
   cargo clippy -- -D warnings
   ```

3. **Run all tests**:
   ```bash
   cargo test
   ```

## Combined Command

```bash
cargo fmt -- --check && cargo clippy -- -D warnings && cargo test
```

## Report Format

Provide a summary:

| Check | Status | Details |
|-------|--------|---------|
| Format | ✅/❌ | Files needing format |
| Clippy | ✅/❌ | Warning count |
| Tests | ✅/❌ | Pass/Fail counts |

## On Failure

If any check fails:
1. For format: Run `cargo fmt` to fix
2. For clippy: Address each warning individually
3. For tests: Debug and fix failing tests

All checks must pass before committing.
