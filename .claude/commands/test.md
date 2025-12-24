Run tests for G3. Optionally specify a crate or test name.

## Usage

- `/test` - Run all tests
- `/test g3-core` - Run tests for specific crate
- `/test g3-core test_context_thinning` - Run specific test
- `/test --nocapture` - Run with output visible

## Steps

1. **Run the tests**:
   ```bash
   # If no arguments, run all tests
   cargo test

   # If crate specified
   cargo test -p $ARGUMENTS

   # If specific test
   cargo test -p <crate> <test_name>

   # With output
   cargo test -- --nocapture
   ```

2. **Report results**:
   - Total tests run
   - Passed/Failed counts
   - For failures: file:line and error message

## On Failure

If tests fail:
1. Read the failure output
2. Identify the assertion that failed
3. Check if it's a code bug or test bug
4. Suggest a fix

## Common Test Commands

```bash
# Run all tests in a crate
cargo test -p g3-core

# Run specific test file
cargo test -p g3-core --test test_context_thinning

# Run with verbose output
cargo test -- --nocapture

# Run ignored tests
cargo test -- --ignored
```
