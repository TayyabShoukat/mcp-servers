# GitHub MCP Server Setup Guide for Claude Code

## Overview

This guide provides step-by-step instructions for integrating the GitHub Model Context Protocol (MCP) server with Claude Code. Once configured, Claude Code will be able to interact with your GitHub repositories, pull requests, issues, and workflows directly.

---

## Prerequisites

Before you begin, ensure you have:

- **Claude Code** installed and running
- **Node.js** (v16 or higher) with `npm` installed
- A **GitHub Personal Access Token** (PAT)
- Internet connectivity

### Create a GitHub Personal Access Token

1. Visit [GitHub Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens)
2. Click **"Generate new token (classic)"**
3. Give your token a descriptive name (e.g., `Claude Code MCP`)
4. Select the scopes you need:
   - For read-only access: Leave all scopes unchecked (recommended for initial setup)
   - For full access: Select `repo`, `workflow`, `admin:org_hook`
5. Click **"Generate token"**
6. **Copy and save your token securely** (you won't see it again)

---

## Step-by-Step Installation

### Step 1: Remove Existing GitHub MCP Configurations

If you have previously configured GitHub MCP servers, remove them from all scopes:

```bash
claude mcp remove "github" -s project
claude mcp remove "github" -s user
```

**Expected Output:**
```
MCP server "github" removed successfully
```

### Step 2: Verify Removal

Check that the GitHub server has been removed:

```bash
claude mcp list
```

**Expected Output:**
```
No MCP servers configured.
```

### Step 3: Add GitHub MCP Server with Your Token

Run the following command, replacing `YOUR_GITHUB_TOKEN` with your actual GitHub Personal Access Token:

```bash
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_GITHUB_TOKEN -- npx -y @modelcontextprotocol/server-github
```

**Example with a sample token:**
```bash
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=ghp_1234567890abcdefghijklmnopqrstuvwxyz -- npx -y @modelcontextprotocol/server-github
```

**Expected Output:**
```
MCP server "github" added successfully
```

### Step 4: Verify Installation

List all configured MCP servers:

```bash
claude mcp list
```

**Expected Output:**
```
github: (stdio) npx -y @modelcontextprotocol/server-github
```

---

## Verifying with Claude Code

### Step 1: Restart Claude Code

Close and completely reopen Claude Code to load the new MCP configuration.

### Step 2: Check MCP Server Status

In the Claude Code terminal, run:

```bash
/mcp
```

**Expected Output:**
```
Connected MCP Servers:
├─ github
│  ├─ Tools: browse_repository, list_issues, create_issue, etc.
│  └─ Status: Ready
```

### Step 3: Test GitHub Integration

Try one of these test commands in Claude Code:

#### Test 1: Browse a Repository
```
> Can you help me understand the structure of the https://github.com/anthropics/anthropic-sdk-python repository?
```

#### Test 2: List GitHub Issues
```
> Show me recent issues from my GitHub repositories
```

#### Test 3: Check Pull Requests
```
> List open pull requests assigned to me
```

### Step 4: Monitor MCP Connection

If you encounter any issues, check the MCP server status:

```bash
claude mcp get github
```

---

## Troubleshooting

### Issue 1: "MCP server github already exists"

**Solution:**
```bash
# Remove from both scopes
claude mcp remove "github" -s project
claude mcp remove "github" -s user

# Then re-add with the correct command
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_TOKEN -- npx -y @modelcontextprotocol/server-github
```

### Issue 2: "Failed to connect" or "Connection timeout"

**Possible causes:**
- Invalid GitHub token
- Network connectivity issues
- npm not installed or not in PATH

**Solutions:**
```bash
# Verify npm is installed
npm --version

# Verify your token is valid by checking GitHub CLI
# First, install GitHub CLI if you haven't
npm install -g @github/cli

# Test your token
gh auth status

# If token is invalid, generate a new one and try again
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=NEW_TOKEN -- npx -y @modelcontextprotocol/server-github
```

### Issue 3: "Connection closed" Error (Windows)

**Solution:** On Windows, you may need to use the `cmd /c` wrapper:

```bash
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_TOKEN -- cmd /c npx -y @modelcontextprotocol/server-github
```

### Issue 4: Token Not Being Recognized

**Solution:** Make sure your token is properly quoted and has no spaces:

```bash
# Verify token format
echo %GITHUB_PERSONAL_ACCESS_TOKEN%

# Try adding with proper escaping
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN="ghp_yourtoken" -- npx -y @modelcontextprotocol/server-github
```

---

## Configuration Details

### What This Command Does

```bash
claude mcp add --transport stdio github --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_TOKEN -- npx -y @modelcontextprotocol/server-github
```

**Breakdown:**
| Parameter | Meaning |
|-----------|---------|
| `--transport stdio` | Uses local process communication (vs. HTTP/SSE) |
| `github` | Server name for reference |
| `--scope user` | Available globally across all your projects |
| `-e GITHUB_PERSONAL_ACCESS_TOKEN=...` | Sets authentication token as environment variable |
| `--` | Separator between Claude CLI flags and server command |
| `npx -y @modelcontextprotocol/server-github` | Downloads and runs the official GitHub MCP server |

### Configuration File Location

After installation, your configuration is stored at:

**Windows:**
```
%USERPROFILE%\.claude.json
```

**macOS/Linux:**
```
~/.claude.json
```

**View your configuration:**
```bash
cat ~/.claude.json
```

**Example configuration:**
```json
{
  "mcpServers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token_here"
      }
    }
  }
}
```

---

## What You Can Do Now

With GitHub MCP server configured, Claude Code can help you with:

### Repository Management
- Browse repository structure and files
- Search code across repositories
- Analyze commits and commit history
- View branches and tags

### Issue Management
- Create new issues
- List and filter issues
- Update issue status and assignments
- Add comments to issues

### Pull Request Workflow
- List open pull requests
- Review PR changes
- Create new pull requests
- Merge pull requests
- Check PR status and reviews

### Code Analysis
- Analyze security findings
- Review Dependabot alerts
- Check code patterns
- Understand project structure

### CI/CD Integration
- Monitor GitHub Actions workflows
- Check build status
- View deployment history
- Analyze workflow failures

---

## Best Practices

### Security
- ✅ Use personal access tokens instead of passwords
- ✅ Grant minimum required permissions
- ✅ Rotate tokens regularly
- ✅ Never commit tokens to version control
- ❌ Don't share your token with others

### Token Scope Recommendations

**For read-only access:**
- No scopes selected (default - works for public repos)

**For moderate access:**
- `repo` (for private repositories)

**For full GitHub integration:**
- `repo` (repository access)
- `workflow` (GitHub Actions)
- `admin:org_hook` (organization webhooks)

### Usage Tips
1. Start with read-only access and expand permissions as needed
2. Use project-specific tokens if managing multiple GitHub accounts
3. Regenerate tokens periodically for security
4. Monitor token usage in GitHub settings

---

## Advanced Configuration

### Using Environment Variables

Instead of hardcoding your token, use environment variables:

**PowerShell (Windows):**
```powershell
$env:GITHUB_PERSONAL_ACCESS_TOKEN = "your_token_here"
claude mcp add --transport stdio github --scope user -- npx -y @modelcontextprotocol/server-github
```

**Bash (macOS/Linux):**
```bash
export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
claude mcp add --transport stdio github --scope user -- npx -y @modelcontextprotocol/server-github
```

### Project-Specific Configuration

To configure GitHub MCP for a single project only:

```bash
# In your project directory
claude mcp add --transport stdio github --scope project -e GITHUB_PERSONAL_ACCESS_TOKEN=YOUR_TOKEN -- npx -y @modelcontextprotocol/server-github
```

This creates a `.mcp.json` file in your project root that can be shared with your team.

### Multiple GitHub Accounts

Create separate servers for different accounts:

```bash
# Personal account
claude mcp add --transport stdio github-personal --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=PERSONAL_TOKEN -- npx -y @modelcontextprotocol/server-github

# Work account
claude mcp add --transport stdio github-work --scope user -e GITHUB_PERSONAL_ACCESS_TOKEN=WORK_TOKEN -- npx -y @modelcontextprotocol/server-github
```

---

## Additional Resources

- **Claude Code MCP Documentation:** https://code.claude.com/docs/claude-code/mcp
- **GitHub MCP Server Repository:** https://github.com/modelcontextprotocol/servers
- **Create GitHub Token:** https://github.com/settings/tokens
- **Model Context Protocol:** https://modelcontextprotocol.io/

---

## Support and Help

### Getting Help
- Check `/mcp` status in Claude Code
- Review error messages carefully
- Verify token validity and permissions
- Test network connectivity

### Common Commands Reference

```bash
# List all MCP servers
claude mcp list

# Get details about GitHub server
claude mcp get github

# Remove GitHub server
claude mcp remove github

# Reset MCP configuration
claude mcp reset-project-choices

# Check server health
claude mcp health github

# View MCP server status in Claude Code
/mcp
```

---

## Version Information

- **Last Updated:** December 2025
- **Claude Code Version:** 2.0.60+
- **GitHub MCP Server:** @modelcontextprotocol/server-github (latest)
- **Node.js Requirement:** v16 or higher

---

**Created for:** Claude Code Users  
**Status:** Complete and Ready to Use
