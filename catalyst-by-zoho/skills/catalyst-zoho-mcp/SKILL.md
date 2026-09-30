---
name: catalyst-zoho-mcp
description: "Catalyst Zoho MCP — manage Catalyst infrastructure (tables, buckets, cache) via CatalystbyZoho_* MCP tools using natural language. Trigger on 'Zoho MCP', 'MCP tools', 'catalyst MCP', 'CatalystbyZoho', 'create table with AI', 'MCP setup', 'MCP config', 'global MCP server', 'infrastructure as conversation', 'MCP first', or 'avoid Catalyst console'. For Codex DC selection or switching, load catalyst-switch-dc."
metadata:
  version: "2.4.0"
  compatibility: "Requires an MCP-capable client. The Codex plugin bundles regional OAuth MCP definitions; the user must explicitly select one DC with catalyst-switch-dc and connect its regional server."
---

## How It Works

1. **Check if MCP is connected** — Look for the `ZohoMCP_*` **meta-tools** (`ZohoMCP_getSchema`, `ZohoMCP_executeTool`, `ZohoMCP_listTools`, `ZohoMCP_getFeatures`) in your tool list. Their presence is the connectivity signal. The `CatalystbyZoho_*` names are **not** shown as tools — they are `tool_name` values passed to `ZohoMCP_executeTool`.

2. **⛔ MCP NOT connected — HARD STOP.**
   Do NOT write any code, create any files, or call any SDK.
  In Codex, load `catalyst-switch-dc`, require an explicit regional DC, apply the supported plugin-policy configuration, and connect the selected regional server. In other clients, load `references/zoho-mcp.md` and follow that client's setup path.
   Do not proceed to step 3 until the `ZohoMCP_*` meta-tools are present in the tool list (this is the "MCP connected" signal — the `CatalystbyZoho_*` names never appear as tools).

3. **DC switch request?** — If the user asks to switch data centers in Codex (e.g. "connect to IN DC", "switch to EU"), load `catalyst-switch-dc`. It updates only supported user-level plugin policy, never the installed `.mcp.json`. Do not use the old connection; connect and verify the selected regional server before further MCP operations. For other clients, load `references/dc-switching.md`.

4. **Pre-flight sequence** — Once per session, before your first MCP tool call, follow the single canonical pre-flight in `../catalyst-basics/references/preflight.md`: read org (`projects[].env[].id`) and project (`projects[].id`) from `.catalystrc` and confirm parity with MCP via `CatalystbyZoho_Get_Project_By_Id`, or resolve via `List_All_Organizations` → `List_All_Projects` when `.catalystrc` is absent. Once it passes, trust the context for the rest of the session — do not re-verify before every call.

5. **Load `references/zoho-mcp.md`** — for the full tool catalog, execution flow, and common error fixes.

6. **If the query involves DataStore** (create table, add columns, query data) — also load `references/mcp-datastore.md`.

7. **Answer** — Invoke the appropriate `CatalystbyZoho_*` operation via `ZohoMCP_executeTool` (passing its name as the `tool_name` argument; fetch its schema first with `ZohoMCP_getSchema`). Show the user which operation was called and what it returned.

## Triggers

Use this skill for: "Zoho MCP", "MCP tools", "catalyst MCP", "create table with AI", "MCP DataStore", "MCP Cache", "MCP Stratus", `zoho-mcp-server`, "MCP setup", "MCP config", "global MCP server", "infrastructure as conversation", "Codex Catalyst MCP", "MCP tool error", "MCP not connecting", "use AI to create database table", or a `CatalystbyZoho_` tool. Route Codex DC selection and switching to `catalyst-switch-dc`.

## References

| Reference | Load when the query is about… |
|-----------|-------------------------------|
| `../catalyst-basics/references/preflight.md` | **The workspace readiness gate** (canonical pre-flight, lives in `catalyst-basics`) — establishing + verifying org/project (via `.catalystrc` + `Get_Project_By_Id` reconciliation, or `List_All_*` fallback) and environment awareness. The single source every skill links to. |
| `references/zoho-mcp.md` | Global MCP server setup (all 3 clients), all available CatalystbyZoho_* tools, execution flow, common MCP errors |
| `references/dc-switching.md` | Switching data centers on Codex or another MCP-capable client |
| `references/mcp-datastore.md` | Creating tables/columns via MCP, DataStore column types, batch column creation, data type constraints |
