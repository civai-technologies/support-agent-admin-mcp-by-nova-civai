# Support Agent Admin by Nova (CIVAI)

I manage your Support Agent CRM, pipeline analytics, follow-ups, Leads Discovery, and workspace output (Sheets, Docs, Calendar, research)

**Keywords:** mcp-server, model-context-protocol, nova-by-civai, remote-mcp, streamable-http, civai, cursor, claude-desktop, crm, support-agent, outreach, whatsapp-crm, lead-management

Remote **Model Context Protocol (MCP)** server from **Nova by CIVAI**. Connect from **Cursor**, **Claude Desktop**, **Claude Code**, or any MCP client that supports **Streamable HTTP**.

## Links

| Resource | URL |
|----------|-----|
| MCP endpoint | https://nova.civai.co/mcp/agents/support-agent-admin |
| Agent page | https://nova.civai.co/agent/support-agent-admin |
| API keys | https://nova.civai.co/api-keys |
| Official Registry | `co.civai.nova/support-agent-admin` |

## Prerequisites

1. Sign in at [nova.civai.co/api-keys](https://nova.civai.co/api-keys)
2. Create a **programmatic Nova API key** (JWT)
3. Add the server to your MCP client with `Authorization: Bearer <your_nova_jwt>`

## Cursor Marketplace plugin

This repository ships a **Cursor Plugin** manifest (`.cursor-plugin/plugin.json` + root `mcp.json`) for one-click install after marketplace review. Users configure `NOVA_API_KEY` in Cursor **Plugins → Configure** (never commit secrets).

Local test: copy this folder to `~/.cursor/plugins/local/support-agent-admin-by-nova`, reload the window, set `NOVA_API_KEY`, verify MCP tools load.

## Cursor (manual MCP)

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "support-agent-admin-by-nova": {
      "url": "https://nova.civai.co/mcp/agents/support-agent-admin",
      "headers": {
        "Authorization": "Bearer YOUR_NOVA_API_KEY"
      }
    }
  }
}
```

## Claude Code

```bash
claude mcp add --transport http support-agent-admin-by-nova \
  https://nova.civai.co/mcp/agents/support-agent-admin \
  --header "Authorization: Bearer $NOVA_API_KEY"
```

## Transport

- **Type:** Streamable HTTP (stateless)
- **Auth:** Bearer JWT or `X-API-Key` header
- **Tools:** 83 (bundled catalog)

## About Nova by CIVAI

**Nova by CIVAI** is a platform for running, building, and deploying AI agents. This MCP server exposes **Support Agent Admin** tools to external MCP hosts without sharing upstream credentials.

| | |
|---|---|
| **Company** | CIVAI TECHNOLOGIES |
| **Creator** | [Kayode Femi Amoo](https://x.com/usecodenaija) |
| **Product** | [Nova by CIVAI](https://nova.civai.co) |

## License

MIT. See [LICENSE](LICENSE).
