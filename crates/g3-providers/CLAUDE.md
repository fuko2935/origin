# g3-providers - LLM Provider Abstractions

**Technology**: Rust 2021, Reqwest, llama_cpp, Axum (OAuth)
**Entry Point**: `src/lib.rs`
**Parent Context**: Extends [../../CLAUDE.md](../../CLAUDE.md)

This crate provides a unified interface for multiple LLM providers, including streaming support and OAuth authentication.

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
cargo build -p g3-providers
cargo test -p g3-providers
```

### Pre-PR Checklist

```bash
cargo fmt -- --check && cargo clippy -p g3-providers -- -D warnings && cargo test -p g3-providers
```

---

## Architecture

### Supported Providers

| Provider | API | Features |
|----------|-----|----------|
| **Anthropic** | Claude API | Native tool calling, streaming |
| **Databricks** | Foundation Model API | OAuth/token auth, streaming |
| **OpenAI Compatible** | OpenAI API format | Works with OpenRouter, Groq, local servers |
| **Embedded** | llama.cpp | Local models, Metal/CUDA acceleration |

### Directory Structure

```
src/
├── lib.rs                    # Main entry, ProviderRegistry, traits
├── anthropic.rs              # Anthropic Claude provider
├── databricks.rs             # Databricks provider with OAuth
├── openai.rs                 # OpenAI-compatible providers
├── embedded.rs               # Local llama.cpp provider
├── oauth.rs                  # OAuth flow implementation
tests/
├── cache_control_*.rs        # Cache control tests
```

### Key Traits

```rust
#[async_trait]
pub trait LLMProvider: Send + Sync {
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionResponse>;
    async fn complete_stream(&self, request: CompletionRequest)
        -> Result<impl Stream<Item = Result<StreamChunk>>>;
    fn supports_tools(&self) -> bool;
    fn max_context_length(&self) -> usize;
}
```

---

## Code Organization Patterns

### Provider Registration

```rust
// Pattern: Register providers at startup
let mut registry = ProviderRegistry::new();
registry.register("anthropic", AnthropicProvider::new(config)?);
registry.register("databricks", DatabricksProvider::new(config)?);

// Get provider by name
let provider = registry.get("anthropic")?;
```

### Completion Request

```rust
// Pattern: Create and send completion request
let request = CompletionRequest {
    messages: vec![
        Message::system("You are a helpful assistant"),
        Message::user("Hello!"),
    ],
    tools: Some(available_tools),
    max_tokens: 8192,
    temperature: 0.1,
    stream: true,
};

let response = provider.complete(request).await?;
```

### Streaming Pattern

```rust
// Pattern: Handle streaming responses
let stream = provider.complete_stream(request).await?;
tokio::pin!(stream);

while let Some(chunk) = stream.next().await {
    match chunk? {
        StreamChunk::Text(text) => print!("{}", text),
        StreamChunk::ToolCall(call) => execute_tool(call).await?,
        StreamChunk::Done => break,
    }
}
```

### OAuth Flow (Databricks)

```rust
// Pattern: OAuth authentication
let oauth = OAuthFlow::new(client_id, redirect_uri);
let auth_url = oauth.authorization_url()?;
// User visits auth_url, gets redirected back
let token = oauth.exchange_code(code).await?;
```

---

## Key Files

### Core Understanding

1. **`src/lib.rs`** - Registry and trait definitions
   ```bash
   rg -n "trait LLMProvider|ProviderRegistry" src/lib.rs
   ```

2. **`src/anthropic.rs`** - Anthropic implementation
   ```bash
   rg -n "impl.*LLMProvider.*Anthropic" src/anthropic.rs
   ```

3. **`src/oauth.rs`** - OAuth implementation
   ```bash
   rg -n "OAuthFlow|authorization_url|exchange_code" src/oauth.rs
   ```

4. **`src/embedded.rs`** - Local models
   ```bash
   rg -n "llama_cpp|LlamaModel" src/embedded.rs
   ```

---

## Quick Search Commands

### Find Provider Implementations

```bash
# Find all provider implementations
rg -n "impl.*LLMProvider" src/

# Find provider structs
rg -n "pub struct.*Provider" src/
```

### Find API Endpoints

```bash
# Find API URLs
rg -n "https://|api_url|endpoint" src/
```

### Find Streaming Logic

```bash
rg -n "Stream|stream|chunk" src/
```

---

## Common Gotchas

### API Key Security

- API keys are read from config, not hardcoded
- Never log API keys
- Use environment variables in CI/CD

### Streaming vs Non-Streaming

Some providers handle streaming differently:
- Anthropic: Native streaming with tool calls
- OpenAI-compatible: SSE format
- Embedded: Token-by-token

### OAuth Token Refresh

Databricks OAuth tokens expire. The provider handles refresh automatically.

### llama.cpp GPU Acceleration

The embedded provider uses the `metal` feature for macOS:

```toml
llama_cpp = { version = "0.3.2", features = ["metal"] }
```

For Linux, use CUDA features.

### Context Length Differences

Each provider has different max context:
- Anthropic Claude: 200k tokens
- Databricks: Varies by model
- Embedded: 4k-32k depending on model

---

## Testing Guidelines

### Unit Tests

```bash
# Run provider tests
cargo test -p g3-providers

# Test specific provider
cargo test -p g3-providers anthropic
cargo test -p g3-providers cache_control
```

### Integration Tests

Integration tests require API keys. They're skipped by default:

```bash
# Run with API key
ANTHROPIC_API_KEY=sk-... cargo test -p g3-providers --test integration
```

---

## Adding a New Provider

To add a new LLM provider:

1. Create `src/new_provider.rs`
2. Implement the `LLMProvider` trait
3. Add to `ProviderRegistry` in `lib.rs`
4. Add configuration in `g3-config`
5. Add tests

```rust
// src/new_provider.rs
pub struct NewProvider {
    api_key: String,
    model: String,
}

#[async_trait]
impl LLMProvider for NewProvider {
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionResponse> {
        // Implementation
    }
    // ... other trait methods
}
```

---

## Configuration Reference

Providers are configured in `~/.config/g3/config.toml`:

```toml
[providers]
default_provider = "anthropic.default"

[providers.anthropic.default]
api_key = "sk-ant-..."
model = "claude-sonnet-4-5"
max_tokens = 64000

[providers.databricks.default]
host = "https://workspace.cloud.databricks.com"
model = "databricks-claude-sonnet-4"
use_oauth = true
```

See `config.example.toml` in the repo root for full reference.
