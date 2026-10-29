# Reference

| Page | Covers |
|---|---|
| [MCP tools](mcp-tools.md) | the `kb_*` tools and every action they take, and the Git Repos and workspace tools |
| [HTTP API](http-api.md) | the `/api/knowledgebase` endpoints and the workspace and branch headers |
| [Configuration](configuration.md) | the `knowledgebase` section: sources, personas, node types, flows, orchestrations |
| [Store schema](store.md) | the SQLite tables behind each workspace's knowledge base |

## Connecting an AI

Roadmap's MCP server is served by the Roadmap backend. A client such as Claude
Code connects with an entry like this in its `.mcp.json`, with an
[API token](../get-started/account.md#api-tokens) of a user with the editor role or
higher:

```json
{
  "mcpServers": {
    "roadmap": {
      "type": "sse",
      "url": "http://127.0.0.1:8090/mcp/sse",
      "headers": { "Authorization": "Bearer <token>" }
    }
  }
}
```

`/mcp/sse` runs in the `default` workspace and `/mcp/<workspace>/sse` in another.
Every tool also takes `workspace="<name>"`, which wins over the connection's.
