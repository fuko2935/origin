# MCP Server Setup for G3

This guide covers setting up Model Context Protocol (MCP) servers for enhanced Claude Code integration with the G3 project.

---

## Quick Start

```bash
# Install the most useful MCP server for this project
claude mcp add --scope user github -- npx -y @modelcontextprotocol/server-github

# Set your GitHub token
export GITHUB_TOKEN="ghp_your_token_here"

# Verify
claude mcp list
```

---

## Recommended MCP Servers

### 1. GitHub Integration (Highly Recommended)

For issues, PRs, and repository operations:

```bash
# Install globally (user scope)
claude mcp add --scope user github -- npx -y @modelcontextprotocol/server-github

# Or use bunx (faster)
claude mcp add --scope user github -- bunx @modelcontextprotocol/server-github
```

**Required Environment Variable:**
```bash
export GITHUB_TOKEN="your-github-personal-access-token"
```

**Capabilities:**
- Create and manage issues
- Create and review pull requests
- Read repository contents
- Search code and issues
- Manage branches

### 2. Sequential Thinking (For Complex Decisions)

For multi-step reasoning and architectural decisions:

```bash
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

**Use Cases:**
- Complex refactoring decisions
- Architecture planning
- Debugging multi-step issues
- Trade-off analysis

### 3. Filesystem (For Extended File Operations)

If you need advanced file operations beyond Claude Code's built-in tools:

```bash
claude mcp add --scope user filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed/directory
```

### 4. Web Search/Documentation (Optional)

For looking up Rust documentation and crate information:

```bash
# Brave Search
claude mcp add --scope user brave-search -- npx -y @anthropics/mcp-server-brave-search

# Requires: export BRAVE_API_KEY="your-key"
```

---

## Project-Specific Configuration

Create a `.mcp.json` file in the project root for project-specific MCP servers:

```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

This file can be committed to git so all team members get the same MCP configuration.

---

## Setup Steps

### 1. Install Node.js (Required)

MCP servers require Node.js for npx/npm:

```bash
# macOS
brew install node

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Verify
node --version
npm --version
```

### 2. Create GitHub Personal Access Token

1. Go to GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Click "Generate new token (classic)"
3. Select scopes:
   - `repo` (Full control of private repositories)
   - `read:org` (Read org membership)
   - `read:user` (Read user profile data)
4. Copy the token (you won't see it again)

### 3. Configure Environment

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, etc.):

```bash
# GitHub token for MCP
export GITHUB_TOKEN="ghp_your_token_here"
```

Reload your shell:
```bash
source ~/.zshrc  # or ~/.bashrc
```

### 4. Install MCP Servers

```bash
# GitHub (most useful for this project)
claude mcp add --scope user github -- npx -y @modelcontextprotocol/server-github

# Optional: Sequential thinking for complex decisions
claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

### 5. Verify Installation

```bash
# List installed MCP servers
claude mcp list

# Test GitHub integration
claude mcp test github
```

---

## Usage Examples

Once configured, you can use MCP tools directly in Claude Code:

### GitHub Operations

```
# Create an issue
"Use the GitHub MCP to create an issue titled 'Add retry logic to provider' with label 'enhancement'"

# Find issues
"Use GitHub MCP to find open issues labeled 'bug'"

# Create a PR
"Use GitHub MCP to create a pull request from this branch to main"

# Review PR
"Use GitHub MCP to get the diff for PR #123"
```

### Sequential Thinking

```
# Complex architecture decision
"Use sequential thinking to analyze the trade-offs between using async channels vs shared state for the flock manager"

# Debugging
"Use sequential thinking to trace through why the context window is overflowing"
```

---

## MCP Server Reference

| Server | Purpose | Install Command |
|--------|---------|-----------------|
| GitHub | Issues, PRs, Repo ops | `claude mcp add --scope user github -- npx -y @modelcontextprotocol/server-github` |
| Sequential Thinking | Complex reasoning | `claude mcp add --scope user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking` |
| Filesystem | Extended file ops | `claude mcp add --scope user filesystem -- npx -y @modelcontextprotocol/server-filesystem /path` |
| Brave Search | Web search | `claude mcp add --scope user brave-search -- npx -y @anthropics/mcp-server-brave-search` |

---

## Troubleshooting

### MCP Server Not Found

```bash
# Check if npx works
npx -y @modelcontextprotocol/server-github --help

# Try with bunx instead (if you have bun installed)
claude mcp add --scope user github -- bunx @modelcontextprotocol/server-github

# Clear npm cache
npm cache clean --force
```

### GitHub Token Issues

```bash
# Verify token is set
echo $GITHUB_TOKEN

# Test token with GitHub API
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/user

# Check token permissions
curl -H "Authorization: token $GITHUB_TOKEN" https://api.github.com/rate_limit
```

### Permission Denied

```bash
# Ensure npm packages can be executed globally
npm config set prefix ~/.npm-global
export PATH=~/.npm-global/bin:$PATH

# Add to shell profile
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
```

### MCP Connection Issues

```bash
# Remove and re-add the server
claude mcp remove github
claude mcp add --scope user github -- npx -y @modelcontextprotocol/server-github

# Check Claude Code logs
# Look for MCP-related errors in the output
```

---

## Security Notes

- **Never commit tokens**: Your `GITHUB_TOKEN` should never appear in the repository
- **Use environment variables**: The `.mcp.json` uses `${GITHUB_TOKEN}` reference, not the actual token
- **Token scopes**: Only grant the minimum required permissions
- **Token rotation**: Rotate tokens periodically (every 90 days recommended)
- **Team sharing**: Each team member needs their own token

---

## G3-Specific Recommendations

For this Rust workspace project, the most valuable MCP servers are:

1. **GitHub** - Essential for issue tracking and PR workflows
2. **Sequential Thinking** - Useful for complex refactoring decisions

The built-in Claude Code tools already handle:
- File reading/writing
- Grep/search
- Git operations
- Shell commands

So additional filesystem MCP servers are usually not needed.
