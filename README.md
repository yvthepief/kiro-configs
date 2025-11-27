# Kiro Configuration Setup

Quick setup repository for Kiro AI assistant with pre-configured MCP servers, steering rules, and automation hooks.

## Quick Start

1. Clone this repository into your project's root:
   ```bash
   git clone <repo-url> .kiro
   ```

2. Install required dependencies:
   ```bash
   # Install uv for Python MCP servers
   brew install uv  # macOS
   # or: pip install uv
   ```

3. Restart Kiro or reconnect MCP servers from the MCP Server view

## What's Included

### MCP Servers

- **Context7**: Library documentation and compatibility checking
- **AWS Docs**: Official AWS documentation access

### Steering Rules (15)

Development best practices automatically applied to all Kiro interactions:

- `aws-cli-best-practices.md` - AWS CLI usage patterns
- `cdk-best-practices.md` - AWS CDK development standards
- `development-standards.md` - General development guidelines
- `docker-best-practices.md` - Container best practices
- `documentation-style.md` - Documentation writing style
- `git-best-practices.md` - Version control workflows
- `mcp-best-practices.md` - MCP server configuration
- `product.md` - Product context
- `python-best-practices.md` - Python coding standards
- `react-best-practices.md` - React development patterns
- `security-best-practices.md` - Security guidelines
- `structure.md` - Project structure conventions
- `tech.md` - Technology stack documentation
- `testing-best-practices.md` - Testing strategies
- `typescript-best-practices.md` - TypeScript coding standards

### Automation Hooks (17)

Pre-configured hooks for common development workflows:

- `auto-test-on-save.kiro.hook` - Run tests when files change
- `lint-and-format-on-save.kiro.hook` - Auto-format code
- `cdk-synth-on-change.kiro.hook` - Validate CDK changes
- `security-scan-on-dependency-change.kiro.hook` - Security checks
- `update-documentation.kiro.hook` - Keep docs in sync
- And 12 more...

## Customization

### Adding MCP Servers

Edit `.kiro/settings/mcp.json`:

```json
{
  "mcpServers": {
    "your-server": {
      "command": "uvx",
      "args": ["package-name@latest"],
      "env": {
        "FASTMCP_LOG_LEVEL": "ERROR"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

### Modifying Steering Rules

Edit files in `.kiro/steering/` or add new ones. All `.md` files are automatically included.

### Managing Hooks

Use the Kiro Hook UI (Command Palette → "Open Kiro Hook UI") or edit `.kiro/hooks/*.kiro.hook` files directly.

## Project-Specific Setup

For project-specific configurations:

1. Copy this `.kiro` directory to your project root
2. Update `product.md` with your project context
3. Update `structure.md` with your project structure
4. Update `tech.md` with your technology stack
5. Disable irrelevant hooks or steering rules

## Requirements

- Kiro AI Assistant
- Node.js (for npx-based MCP servers)
- Python 3.8+ with uv (for uvx-based MCP servers)

## Troubleshooting

**MCP servers not connecting:**
- Check MCP Server view in Kiro feature panel
- Verify `uv` is installed: `uv --version`
- Check server logs in MCP Server view

**Hooks not triggering:**
- Verify hook is enabled in Agent Hooks view
- Check hook trigger conditions match your workflow

**Steering rules not applying:**
- Ensure files are in `.kiro/steering/` directory
- Check file has `.md` extension
- Restart Kiro if rules were just added
