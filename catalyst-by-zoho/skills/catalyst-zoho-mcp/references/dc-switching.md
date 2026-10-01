## DC Switching — Catalyst Zoho MCP

Use this reference when the user wants to switch the Catalyst MCP server to a different data center.

---

## DC Selection

Supported regions: US, EU, IN, AU, CA, SA, JP, UAE. Read the exact URL for the selected `catalyst-<dc-in-lowercase>` server from the bundled `.mcp.json`; do not maintain another endpoint table in skills. Personal-server users should obtain the URL for the desired DC from their server provider.

---

## Codex

Load the dedicated `catalyst-switch-dc` skill. The Codex plugin bundles one immutable OAuth MCP definition per supported region; the skill enables exactly one through plugin-scoped policy in `~/.codex/config.toml`.

Do **not** modify the installed plugin's `.mcp.json`. Codex manages that file and plugin upgrades or cache reconciliation can replace local edits.

After switching, connect the selected `catalyst-<dc-in-lowercase>` MCP server in Codex and complete browser authorization if prompted. Do not use the old DC connection. Confirm the selected server's `CatalystbyZoho_*` tools are available before further Catalyst MCP operations; a restart is not required. Credentials and sessions on the old DC are not affected.

---

## Claude Code

For a personally configured server, update the URL in the project's `.mcp.json` to the desired DC's endpoint, then reconnect with `/mcp` or restart Claude Code and complete OAuth. For a plugin installation with bundled regional servers, select and connect only the requested regional server through the client's MCP controls; do not edit installed plugin files or caches. Confirm the new server's tools before continuing: call visible `CatalystbyZoho_*` tools directly, or use `ZohoMCP_*` meta-tools if only those are exposed.

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

- The selected MCP server now targets `<new-dc>` DC.
- In Codex, connect the selected regional MCP server without restarting. For other clients, follow their restart instructions above.
- When connecting, you may be prompted to log in to your Zoho account for the new DC — this is expected.
- Your session on the old DC is not affected.
- In Codex, do not perform Catalyst MCP operations through the old DC connection; verify the selected server before continuing.
- Until authentication completes, the Catalyst resource tools may not be available; complete the client OAuth flow before retrying.

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| DC switch has no effect | Client did not connect the selected endpoint | In Codex, run the `catalyst-switch-dc` status check and connect the selected regional server. In Claude Code, reconnect the selected server in the MCP controls; do not edit installed plugin caches. |
| Only `authenticate` tool visible after switch | Not yet authorized on the new DC | Complete the browser login flow for the selected server |
| Org data from wrong DC appears | More than one regional server is enabled or the old connection is still in use | In Codex, select exactly one DC with `catalyst-switch-dc` and connect that server. In Claude Code, disconnect the old regional server and reconnect the selected one. |
