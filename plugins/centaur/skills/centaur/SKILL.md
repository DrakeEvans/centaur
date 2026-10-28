---
name: centaur
description: Use an authenticated Centaur MCP deployment to discover and call team-approved tools. Use when a task requires Centaur, its tool catalog, or the user's Centaur identity and permissions.
---

# Centaur

Use Centaur's MCP tools for actions and context exposed by the user's deployment.

## Workflow

1. When identity or authorization matters, call `centaur_whoami` before other Centaur tools.
2. If `centaur_catalog_search` is listed, search the catalog with `{"query": "<service, URL, or action>"}` when the correct tool is unclear.
3. Load the chosen tool with `centaur_catalog_load` and `{"tool": "<tool>"}`. It returns the tool's description and top-level CLI help. Treat that help as the current contract.
4. Run the tool with `centaur_tool_call` and `{"tool": "<tool>", "argv": ["<command>", "..."]}`. Pass every CLI argument as a separate array element and do not guess command names or options. To inspect a subcommand, call it with its command segments followed by `--help`.
5. If the catalog tools are not listed, use the legacy per-service MCP tools. Each accepts a `method` and `arguments` object. Call it with `method: "help"` when its methods or arguments are unclear.
6. Summarize consequential writes and return relevant identifiers or links.

Centaur authorizes calls using the signed-in principal's live roles and grants. Never request, paste, print, or store Centaur OAuth tokens.

## Connection recovery

If no Centaur tools are available, explain that the client still needs the deployment-specific MCP endpoint.

- Codex: register it with `codex mcp add centaur --url <CENTAUR_MCP_URL>`, then run `codex mcp login centaur`. The `--url` flag is required for Streamable HTTP and OAuth.
- Claude Code: configure the plugin's `mcp_url`, open `/mcp`, and authenticate the `centaur` server.
- Other MCP clients: configure a remote HTTP server named `centaur` with the deployment's `/mcp` URL and complete its OAuth flow.

Do not substitute a guessed hostname. Ask for the deployment URL when it is not already configured or supplied.
