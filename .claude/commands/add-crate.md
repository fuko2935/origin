Scaffold a new workspace crate.

## Usage

`/add-crate <crate-name>`

Example: `/add-crate g3-analytics`

## Steps

1. **Create crate directory structure**:
   ```bash
   mkdir -p crates/$ARGUMENTS/src
   mkdir -p crates/$ARGUMENTS/tests
   ```

2. **Create Cargo.toml**:
   ```toml
   [package]
   name = "$ARGUMENTS"
   version = "0.1.0"
   edition = "2021"

   [dependencies]
   anyhow = { workspace = true }
   tokio = { workspace = true }
   tracing = { workspace = true }
   serde = { workspace = true }

   [dev-dependencies]
   tokio-test = { workspace = true }
   ```

3. **Create src/lib.rs**:
   ```rust
   //! $ARGUMENTS - [Brief description]

   use anyhow::Result;

   pub fn hello() -> Result<String> {
       Ok("Hello from $ARGUMENTS".to_string())
   }

   #[cfg(test)]
   mod tests {
       use super::*;

       #[test]
       fn test_hello() {
           assert!(hello().is_ok());
       }
   }
   ```

4. **Create CLAUDE.md**:
   Follow the template from other crates.

5. **Add to workspace Cargo.toml**:
   ```toml
   members = [
       # ... existing members
       "crates/$ARGUMENTS",
   ]
   ```

6. **Verify build**:
   ```bash
   cargo build -p $ARGUMENTS
   cargo test -p $ARGUMENTS
   ```

## After Creation

1. Update the crate's CLAUDE.md with specific guidance
2. Add as dependency to crates that need it
3. Implement actual functionality
