> **Zoho MCP** lets AI assistants (Codex, Claude, GitHub Copilot, Cursor, etc.) manage Catalyst infrastructure
> by invoking visible `CatalystbyZoho_*` tools directly on upfront-discovery servers. No console
> clicks or REST API calls needed. Servers using dynamic discovery expose `ZohoMCP_*` meta-tools
> instead; see "How to Call Tools Correctly" below.

---

## Setup — Regional MCP Server

**Step 1 — Choose your Data Center (DC):**

The Catalyst regional MCP endpoint changes by data center. Select the DC that matches your Zoho account. In the bundled plugin's `.mcp.json`, each key (for example, `catalyst-in`) is a **server name** used in MCP controls and Codex policy; its `url` is the **endpoint** whose hostname identifies the regional domain. `CatalystbyZoho_*` refers to **tools**, not servers. Do not copy an endpoint from another region.

Supported DCs: US, EU, IN, AU, CA, SA, JP, UAE.

**Step 2 — Add your DC-specific URL to Codex or your AI client:**

For a manual client setup, copy the full URL for your selected regional server from the bundled `.mcp.json` (or use your own personal MCP server URL). Replace `<selected-mcp-url>` below with that full URL; do not append another path.

**For Codex** — install or enable the Catalyst by Zoho plugin, then load `catalyst-switch-dc` and explicitly choose the account's DC. The plugin already bundles each literal regional URL, disabled by default. The switch skill enables exactly one through `~/.codex/config.toml` plugin policy. Connect the selected regional MCP server and complete OAuth if prompted; no restart is required.

Do not edit the installed plugin's `.mcp.json`; managed plugin files can be replaced during upgrades or cache reconciliation.

**For Claude Desktop** — edit `claude_desktop_config.json`
(macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
Windows: `%APPDATA%\Claude\claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "catalyst-by-zoho": {
      "type": "streamable-http",
      "url": "<selected-mcp-url>"
    }
  }
}
```

**For Cursor** — create or edit `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "catalyst-by-zoho": {
      "type": "streamable-http",
      "url": "<selected-mcp-url>"
    }
  }
}
```

**For GitHub Copilot (VS Code)** — create `.vscode/mcp.json` in your workspace root:

```json
{
  "servers": {
    "catalyst-by-zoho": {
      "type": "http",
      "url": "<selected-mcp-url>"
    }
  }
}
```

> **Using Codex?** Run `catalyst-switch-dc`, select one explicit region, and connect that regional server. Never use the old DC connection after switching.
> **Using Claude Code?** Select and connect the chosen regional server in the client's MCP controls; do not edit installed plugin caches.

**Step 3 — Authorize:**
In Codex, connect the selected server without restarting. In other clients, restart as instructed by their setup guides. The client may open a browser window and prompt you to log in to your Zoho account and grant access. The token is stored automatically by the client.

**Step 4 — Verify:**
Look for `CatalystbyZoho_*` tools from the selected regional server in your client's tool list.
Their presence means the upfront-discovery server is available. If you configured a different,
dynamic-discovery server, look for `ZohoMCP_*` meta-tools instead.

---

## Pre-flight Sequence

Before your first MCP tool call, complete the canonical **workspace readiness gate** once per session: `../../catalyst-basics/references/preflight.md`. It establishes org/project (from `.catalystrc` + a single `CatalystbyZoho_Get_Project_By_Id` reconciliation, or the `List_All_Organizations` → `List_All_Projects` fallback when `.catalystrc` is absent) and confirms access with `List_All_Tables` before DataStore work. Once it passes, trust it and operate freely — do not re-run it before every call.

---

## How to Call Tools Correctly

**When `CatalystbyZoho_*` tools are visible (upfront discovery):** Call the selected server's tool directly with the arguments described by its exposed schema. Do not wait for `ZohoMCP_*` meta-tools or wrap the call in `ZohoMCP_executeTool`. If an operation is not exposed, do not assume it exists; check the selected server's available tools.

**For a dynamic-discovery server only:** Call `ZohoMCP_getSchema` before `ZohoMCP_executeTool` for any operation you haven't called before. The following steps apply only to that server type.

### Step 1 — Get the schema

`ZohoMCP_getSchema` takes `query_params`, **not** `body`:

```
ZohoMCP_getSchema({
  query_params: { tool_name: "CatalystbyZoho_List_All_Functions" }
})
```

> ⚠️ Passing `body: { tool_name: "..." }` instead of `query_params` returns "tool_name is required" — this is the wrong parameter location.

### Step 2 — Call the tool

`ZohoMCP_executeTool` always takes a `body` with this shape:

```
ZohoMCP_executeTool({
  body: {
    tool_name: "CatalystbyZoho_List_All_Functions",
    arguments: {
      path_variables: { project_id: "31594000000127002" },
      headers: {},
      body: {}
    }
  }
})
```

- `path_variables` — URL path segments the tool requires (get names from the schema)
- `headers` — extra HTTP headers (usually empty `{}`)
- `body` — request payload for POST/PUT tools (empty `{}` for GET-style tools)

Tools with no required path variables (e.g. `List_All_Organizations`, `List_All_Projects`) can be called with `arguments: {}`.

---

## Available Tools

The operations available depend on the connected Zoho MCP server. When listed in the client, the names below are directly callable tools. On dynamic-discovery servers, they are `tool_name` values passed to `ZohoMCP_executeTool`. Example operation names:

| Tool | Description |
|------|-------------|
| `CatalystbyZoho_List_All_Organizations` | List all Zoho organizations the account has access to |
| `CatalystbyZoho_List_All_Projects` | List all Catalyst projects in the organization |
| `CatalystbyZoho_List_All_Tables` | List all Data Store tables in the project |
| `CatalystbyZoho_List_All_Segments` | List all Cache segments in the project |
| `CatalystbyZoho_List_All_Jobpools` | List all Job Scheduling pools in the project |
| `CatalystbyZoho_Create_Job_Pool` | Create a new Job Scheduling pool |

For upfront discovery, inspect the selected server's visible `CatalystbyZoho_*` tools. For dynamic discovery only, call `ZohoMCP_listTools` (or `ZohoMCP_getFeatures`) to enumerate operation names.

---

## MCP-First Workflow

### Golden Rule: "MCP First, Console Fallback"

When an AI agent needs to create or manage Catalyst infrastructure (tables, cache segments, buckets, job pools), **always try MCP tools first**. Only fall back to the Catalyst Console UI if MCP is unavailable or fails.

| Approach | Time | Repeatable | Auditable |
|----------|------|-----------|-----------|
| ✅ MCP tools | ~30 seconds | Yes | Yes (in conversation) |
| ❌ Console UI | 5+ minutes | No | No |

### Decision Tree

```
Need to create Catalyst infrastructure?
        │
        ▼
Are CatalystbyZoho_* tools (or dynamic-server ZohoMCP_* meta-tools) present?
        │
   YES──┘──NO
   │          │
   ▼          ▼
Use MCP    Guide user to set up Zoho MCP first
tools      (see Setup section above)
  ✅        Then retry with MCP tools
```

**Only instruct manual Console steps when:**
- MCP config is not set up AND user cannot set it up right now
- MCP tools fail with an unresolvable error
- User explicitly requests a manual UI walkthrough

### Example: Table Creation

❌ **Manual Console (5+ minutes)**
```
1. Open https://console.catalyst.zoho.com
2. Navigate to project → Data Store
3. Click Create Table, enter name
4. Add each column manually via the UI
5. Click Create
```

✅ **MCP (two calls — table, then its columns)**

`Create_Table` takes only `table_name` + `table_scope`; columns are a **separate** `Create_Column` call (a batch array) against the new table's ID. Use the exposed schema for the direct tools. The following wrapper examples apply **only to dynamic-discovery servers**; they include `projectId` + `Catalyst-org` + `Environment`.

```javascript
// 1) Create the table (no inline columns)
ZohoMCP_executeTool({ body: {
  tool_name: "CatalystbyZoho_Create_Table",
  arguments: {
    path_variables: { projectId: "<projectId>" },
    headers: { "Catalyst-org": <orgId>, "Environment": "Development" },
    body: { table_name: "Todos", table_scope: "GLOBAL" }
  }
}})

// 2) Add columns in one batch, using the table id returned above
ZohoMCP_executeTool({ body: {
  tool_name: "CatalystbyZoho_Create_Column",
  arguments: {
    path_variables: { projectId: "<projectId>", id: "<tableId>" },
    headers: { "Catalyst-org": <orgId>, "Environment": "Development" },
    body: [
      { column_name: "title", data_type: "text", is_mandatory: "true", audit_consent: "false" },
      { column_name: "completed", data_type: "boolean", is_mandatory: "false", search_index_enabled: "false", default_value: "false", audit_consent: "false" }
    ]
  }
}})
```

See `mcp-datastore.md` for the full column-type field rules.

---

## Common Patterns

### Create a table from schema description

> "Create a Tasks table with columns: Title (text, required), DueDate (date), Status (text), Priority (integer)"

The AI calls `CatalystbyZoho_Create_Table` (name + scope), then `CatalystbyZoho_Create_Column` with the batch of column specs against the new table ID.

### Query data

> "Show me all rows in the Tasks table where Status is 'In Progress'"

The AI calls the query tool with a ZCQL query against the correct table ID.

### Schema exploration

> "What tables do I have and what are their columns?"

The AI calls `CatalystbyZoho_List_All_Tables` then describes the schema.

### Submit an immediate job

> "Run ProcessOrderFunction now as a job"

**Required pre-flight — always do this before calling `CatalystbyZoho_Create_Immediate_Job`:**
1. Call `CatalystbyZoho_List_All_Jobpools` to get existing pools and their IDs.
2. If no pools exist, call `CatalystbyZoho_Create_Job_Pool` first (type `"Function"`, memory e.g. `"256"`).
3. Pass the `jobpool_id` from step 1 or 2 to `CatalystbyZoho_Create_Immediate_Job`.

`jobpool_id` is a required field — there is no default or fallback. Job submission fails immediately if it is omitted.

---

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Expected `CatalystbyZoho_*` tools not showing | Regional server not connected or URL wrong | Verify the selected DC's URL and connection; complete OAuth if prompted. Dynamic-discovery servers show `ZohoMCP_*` instead. |
| `PERMISSION_NEEDED` on table operations | Project context not set | Run `CatalystbyZoho_List_All_Organizations` → `List_All_Projects` first |
| Operations applying to wrong project | Skipped pre-flight | Always run the org → project → verify sequence before any operation |
| MCP server shows red/error *(Option B)* | Token expired or URL invalid | Regenerate the authenticated URL at mcp.zoho.com |
| Browser auth loop not completing *(Option A)* | AI client doesn't support OAuth 2.0 browser flow | Check client version supports MCP 2025-03; try a different supported client |
| MCP targets wrong environment | Zoho MCP defaults to Development | Switch environment explicitly in the Zoho MCP console if production is needed (use caution) |
| `INVALID_INPUT: job_name must contain only alphanumeric and underscore` on `CatalystbyZoho_Create_Immediate_Job` | `job_name` contains hyphens or spaces | Use underscores only — `doc_audit_run_1` not `doc-audit-run-1` |
| Job submission fails with missing field error | `jobpool_id` not provided to `CatalystbyZoho_Create_Immediate_Job` | Call `CatalystbyZoho_List_All_Jobpools` first; if none exist, call `CatalystbyZoho_Create_Job_Pool` then use the returned ID |
