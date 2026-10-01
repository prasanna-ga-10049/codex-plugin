# Using Catalyst Skills with Claude Code

> Shared installation, personal MCP URL, and pre-flight steps are in `setup-common.md`. This file covers only what's specific to Claude Code.

## Skill Activation

Claude Code picks up skills automatically from the `skills/` directory. To verify:

1. Open Claude Code in your Catalyst project directory
2. Ask: "What Catalyst skills are available?"
3. Claude will list the active skills from `catalyst-by-zoho/SKILL.md`

## MCP Setup — Step 2: Add the MCP server to your project

First complete the **Personal MCP Setup** in `setup-common.md` to get your Zoho MCP URL. Claude Code reads MCP servers from a `.mcp.json` file in your **project root** (this is different from Claude Desktop, which uses `claude_desktop_config.json`). Create or edit `.mcp.json`:

```json
{
  "mcpServers": {
    "catalyst-by-zoho": {
      "type": "streamable-http",
      "url": "https://zcatalyst.zohomcp.com/mcp/<auth-token>/message"
    }
  }
}
```

Alternatively, run `claude mcp add` to register your personal server interactively. Do not copy the bundled multi-region Codex `.mcp.json` as a personal-server config.

After saving, restart Claude Code (or run `/mcp` to reconnect). If the server exposes `ZohoMCP_*` meta-tools, use `ZohoMCP_getSchema` before `ZohoMCP_executeTool`. If it exposes `CatalystbyZoho_*` tools instead, call those directly.

## Common Errors (Claude Code)

See `setup-common.md` for errors common to all IDEs. Claude Code-specific:

| Error | Cause | Fix |
|-------|-------|-----|
| MCP tools not appearing | `.mcp.json` not saved or Claude Code not reconnected | Save `.mcp.json` in the project root, then restart Claude Code or run `/mcp` |
