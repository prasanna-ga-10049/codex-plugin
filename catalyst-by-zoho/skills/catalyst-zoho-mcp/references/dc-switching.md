## DC Switching — Catalyst Zoho MCP

Use this reference when the user wants to switch the Catalyst MCP server to a different data center.

---

## DC URL Map

| DC | Region | MCP URL |
|----|--------|---------|
| US | United States | `https://catalyst.zohomcp.com/mcp/message` |
| EU | Europe | `https://catalyst.zohomcp.eu/mcp/message` |
| IN | India | `https://catalyst.zohomcp.in/mcp/message` |
| AU | Australia | `https://catalyst.zohomcp.com.au/mcp/message` |
| CA | Canada | `https://catalyst.zohomcp.ca/mcp/message` |
| SA | Saudi Arabia | `https://catalyst.zohomcp.sa/mcp/message` |
| JP | Japan | `https://catalyst.zohomcp.jp/mcp/message` |
| UAE | United Arab Emirates | `https://catalyst.zohomcp.ae/mcp/message` |

---

## Codex

Load the dedicated `catalyst-switch-dc` skill. The Codex plugin bundles one immutable OAuth MCP definition per supported region; the skill enables exactly one through plugin-scoped policy in `~/.codex/config.toml`.

Do **not** modify the installed plugin's `.mcp.json`. Codex manages that file and plugin upgrades or cache reconciliation can replace local edits.

After switching, connect the selected `catalyst-<dc-in-lowercase>` MCP server in Codex and complete browser authorization if prompted. Do not use the old DC connection. Confirm the selected server's `ZohoMCP_*` meta-tools are available before further Catalyst MCP operations; a restart is not required. Credentials and sessions on the old DC are not affected.

---

## Claude Code

Claude Code loads the MCP server URL from the catalyst-by-zoho plugin. The plugin config is cached in multiple locations — **all of them must be updated** or the switch has no effect.

**Files to update:**

1. **Plugin source** — find the source `.mcp.json` by checking `~/.claude/settings.json` under `extraKnownMarketplaces.catalyst-by-zoho.source.path`, then read the `.mcp.json` at that path.

2. **Marketplace cache:**
   `~/.claude/plugins/marketplaces/catalyst-by-zoho/.mcp.json`

3. **Version cache** — find it with:
   ```
   find ~/.claude/plugins/cache/catalyst-by-zoho -name ".mcp.json"
   ```

In each file, update the `url` field to the new DC's MCP URL. Do not change any other fields.

**After updating:** Instruct the user to restart Claude Code. The new DC server will require re-authentication — a browser login prompt appears on the next connection. Credentials on the old DC are not affected.

---

## Claude Desktop

Edit `claude_desktop_config.json`:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Update the `url` field under `mcpServers.catalyst-by-zoho` to the new DC's MCP URL. Restart Claude Desktop.

---

## Cursor

Edit `.cursor/mcp.json` in the project root. Update the `url` field under `mcpServers.catalyst-by-zoho`. Restart Cursor.

---

## GitHub Copilot (VS Code)

Edit `.vscode/mcp.json` in the workspace root. Update the `url` field under `servers.catalyst-by-zoho`. Reload the VS Code window.

---

## What to Tell the User After Switching

- Config files have been updated to `<new-dc>` DC.
- In Codex, connect the selected regional MCP server without restarting. For other clients, follow their restart instructions above.
- When connecting, you may be prompted to log in to your Zoho account for the new DC — this is expected.
- Your session on the old DC is not affected.
- In Codex, do not perform Catalyst MCP operations through the old DC connection; verify the selected server before continuing.
- Until authentication completes, only `authenticate` and `complete_authentication` tools will be visible.

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| DC switch has no effect | Client did not connect the selected endpoint | In Codex, run the `catalyst-switch-dc` status check and connect the selected regional server. In Claude Code, find and update all `.mcp.json` files under `~/.claude/plugins/` referencing `zohomcp` |
| Only `authenticate` tool visible after switch | Not yet authorized on the new DC | Complete the browser login flow for the selected server |
| Org data from wrong DC appears | More than one regional server is enabled or the old connection is still in use | In Codex, select exactly one DC with `catalyst-switch-dc` and connect that server. In Claude Code, verify all three cache paths have the new URL |
