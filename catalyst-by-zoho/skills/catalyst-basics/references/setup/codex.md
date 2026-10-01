# Using Catalyst Skills with Codex

> Shared installation and project pre-flight steps are in `setup-common.md`. Codex does not require the user to paste an MCP URL: the plugin bundles every supported regional endpoint.

## Skill Activation

Codex loads these skills from the installed Catalyst by Zoho plugin. To verify:

1. Open Codex in your Catalyst project directory.
2. Ask: "What Catalyst skills are available?"
3. Codex should list the Catalyst by Zoho skill index plus the focused Catalyst service skills.

## MCP Setup — Select a Data Center

Catalyst MCP is region-specific for compliance. The plugin bundles disabled definitions for `US`, `EU`, `IN`, `AU`, `CA`, `SA`, `JP`, and `UAE`; it never guesses which region to use. Each `catalyst-<dc-in-lowercase>` key in the bundled `.mcp.json` is a server name, while its `url` is the regional endpoint (including its domain). Connect by server name, not hostname.

1. Ask Codex: **"Switch Catalyst MCP to `<DC>`."**
2. Codex loads `catalyst-switch-dc`, shows the exact regional endpoint, and updates only the plugin policy in `~/.codex/config.toml`.
3. Connect the selected `catalyst-<dc-in-lowercase>` MCP server in Codex; do not use the previous DC's connection.
4. Complete the browser OAuth flow for the selected regional endpoint if prompted.
5. Confirm the selected regional server exposes `CatalystbyZoho_*` tools directly before using Catalyst MCP. No restart is required.

To inspect the current selection, ask Codex: **"Show my Catalyst MCP DC."**

When the selected server exposes `CatalystbyZoho_*` tools, call them directly with their argument schemas. If it exposes only `ZohoMCP_*` meta-tools, follow the dynamic-discovery flow in the MCP guide.

## Common Errors (Codex)

See `setup-common.md` for errors common to all clients. Codex-specific:

| Error | Cause | Fix |
|-------|-------|-----|
| Catalyst skills not appearing | Plugin not installed or the task was opened before installation | Install or update the Catalyst by Zoho plugin, then open a new Codex task |
| No Catalyst MCP DC selected | Every bundled regional server is disabled | Ask Codex to switch Catalyst MCP to the explicit account DC, then connect its regional server |
| Multiple Catalyst MCP DCs enabled | Conflicting plugin policy | Run `catalyst-switch-dc` again; it disables every region except the selected one |
| Duplicate Catalyst MCP tool sets after upgrading | The former app-backed connection is still connected | Disconnect the legacy Catalyst app/connector in Codex settings, then restart |
| MCP tools not appearing | Selected regional server is not connected or OAuth is incomplete | Connect the selected server, complete browser authorization, then verify its `CatalystbyZoho_*` tools |
