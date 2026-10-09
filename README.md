# @outreachbox/mcp

Model Context Protocol server for [OutreachBox](https://outreachbox.com). Run email outreach, prospecting, inbox triage and content campaigns from Claude, Cursor, ChatGPT, or any MCP-compatible client.

## Quick start

Create an API key at [outreachbox.com/apiEndpoints](https://outreachbox.com/apiEndpoints), then:

```bash
npx -y @outreachbox/mcp init --client claude --api-key pb_your_key
```

Restart your client. That is it.

Supported `--client` values: `claude` (Claude Desktop), `claude-code`, `cursor`, `vscode`.

### Manual configuration

```json
{
  "mcpServers": {
    "outreachbox": {
      "command": "npx",
      "args": ["-y", "@outreachbox/mcp"],
      "env": { "OUTREACHBOX_API_KEY": "pb_your_key" }
    }
  }
}
```

## Skills

The package bundles a library of Agent Skills — step-by-step playbooks that teach the model how to combine the tools into real workflows, including when to ask before sending anything.

```bash
npx -y @outreachbox/mcp skills install --client claude
npx -y @outreachbox/mcp skills list
```

| Skill | Use it for |
| --- | --- |
| `outreachbox-getting-started` | Connecting, scopes, troubleshooting |
| `outreachbox-cold-email-campaign` | Project, contacts, sequence, launch, monitor |
| `outreachbox-prospect-research` | Finding and qualifying prospects |
| `outreachbox-inbox-triage` | Classifying replies and drafting responses |
| `outreachbox-content-engine` | Topic research through to publishing |
| `outreachbox-deliverability` | Account health, verification, warmup |
| `outreachbox-analytics-report` | Campaign and team reporting |

## Commands

```
outreachbox-mcp                  Run the MCP server over stdio (default)
outreachbox-mcp init             Add OutreachBox to a client's MCP config
outreachbox-mcp skills install   Copy the skills into a client
outreachbox-mcp skills list      List the bundled skills
outreachbox-mcp doctor           Check connectivity and credentials
```

`doctor` is the first thing to run when something is wrong — it validates the key, names the organization it is bound to, and reports how many tools the key's scopes allow.

## Configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `OUTREACHBOX_API_KEY` | — | API key. Required. |
| `OUTREACHBOX_BASE_URL` | `https://api.outreachbox.com` | API base URL |
| `OUTREACHBOX_APP_URL` | `https://outreachbox.com` | Used to build deeplinks in tool results |
| `OUTREACHBOX_SOURCE` | `mcp` | Attribution tag sent with each request |
| `OUTREACHBOX_TIMEOUT_MS` | `30000` | Per-request timeout |

## Scopes

The server only advertises tools the key is allowed to use, so a read-only key never exposes a tool that sends mail. Keys minted before scopes existed are treated as unrestricted. Check what a key can do with `outreachbox-mcp doctor`.

## Programmatic use

The tool table is exported for hosts that want to serve it over their own transport:

```ts
import { TOOLS_LIST, executeTool, configure, authContext } from '@outreachbox/mcp';

configure({ baseUrl: 'https://api.outreachbox.com' });

const result = await authContext.run({ token: userToken }, () =>
  executeTool('contacts_list', { limit: 10 }),
);
```

`@outreachbox/mcp/server` exports `createServer()` if you want a preconfigured MCP `Server` instance.

## License

MIT
