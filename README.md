# Microsoft Learn in M365 Copilot Chat

| Agent store listing | Agent in action |
|---|---|
| ![Agent store listing showing Learn Docs agent with description and permissions](splash.png) | ![Copilot Chat showing a grounded response with citations from Microsoft Learn](chat.png) |

## Context

The [Microsoft Learn MCP Server](https://learn.microsoft.com/api/mcp) gives AI assistants access to official Microsoft documentation through 3 tools: search docs, search code samples, and fetch full articles. It's public, read-only, and requires no authentication.

Today, these tools are available to developers — in VS Code, CLI agents, and as a [certified Power Platform connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) for Copilot Studio. But none of these reach **end users** directly in M365 Copilot Chat. There's a discoverability gap: the capability exists, but it's only accessible to people who build agents, not to people who use them.

## What changed

The [MCP Apps announcement](https://devblogs.microsoft.com/microsoft365dev/mcp-apps-now-available-in-copilot-chat/) (March 2026) was the trigger — agents can now bring interactive UI into M365 Copilot Chat. But for Learn MCP Server, the interactive UI path (MCP Apps) has unclear added value over just browsing learn.microsoft.com. Rendering docs in a chat widget is a worse browser.

What's more interesting is the tools-only path that came before it: **MCP plugins** let you create a declarative agent backed by an MCP server — no UI, just tool calls with text responses and citations — and publish it to the M365 marketplace.

| When | What | Significance |
|---|---|---|
| Dec 2025 | [Declarative agents with MCP plugins](https://devblogs.microsoft.com/microsoft365dev/build-declarative-agents-for-microsoft-365-copilot-with-mcp/) | Developers can package MCP servers as installable Copilot agents |
| Mar 2026 | [ATK v6.6.0 — MCP plugin GA](https://www.voitanos.io/blog/microsoft-365-agents-toolkit-v6-6-0-release-review/) | Production-ready tooling for building and submitting MCP agents |
| Mar 2026 | [MCP Apps in Copilot Chat](https://devblogs.microsoft.com/microsoft365dev/mcp-apps-now-available-in-copilot-chat/) | Agents can also render interactive UI widgets (HTML) inline |

## Integration layers

Each integration serves a different audience and sits at a different level of the stack:

| Layer | Integration | Who it reaches | Discovery |
|---|---|---|---|
| **Protocol** | MCP Server (`learn.microsoft.com/api/mcp`) | Developers building AI tools | Manual config (URL) |
| **Platform** | [Power Platform certified connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) | Agent builders in Copilot Studio | Connector gallery |
| **Product** | **M365 Copilot MCP plugin** (this project) | End users in Copilot Chat | Agent Store |
| **Product + UI** | MCP App (future) | End users, with interactive widgets | Agent Store |

Moving up the stack increases discoverability but adds packaging requirements (manifests, validation, marketplace submission). The protocol and platform layers are already live. This project is a proof of concept for the product layer.

## What this project is

A proof of concept for the product layer: a declarative agent — configuration only, no backend — that packages the Learn MCP Server as an installable M365 Copilot agent. Users find it in the store, add it, and ask documentation questions grounded in official Microsoft content.

This is an **MCP plugin** (tool-only, text responses with citations), not an **MCP App** (interactive UI widgets). We chose the plugin path deliberately:

- **MCP plugin** = agent calls MCP server tools, returns text with citation links. Simple, publishable to marketplace, and the AI synthesis (cross-article answers, contextual code samples) is the value add over browsing learn.microsoft.com directly.
- **MCP App** = agent renders rich HTML UI inline in chat. For Learn content, it's unclear what UI would add beyond what the browser already does well.

## Open questions

- **Is this useful as a product?** Does a "Learn Docs" agent in the store add enough value, or is opening learn.microsoft.com good enough? The hypothesis is that AI synthesis — cross-article answers, contextual code samples, conversational follow-ups — justifies the agent.
- **Should we publish to the marketplace?** The agent can go to the [Microsoft Commercial Marketplace](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/publish) via Partner Center. Worth pursuing?
- **ChatGPT store?** Same MCP server, different submission process. Read-only, public, no auth — straightforward if we want cross-platform reach.
- **MCP App later?** Only if we find a UI that adds something the browser can't: multi-doc synthesis views, interactive architecture diagrams, code playgrounds.

## Implementation

For technical details — architecture, project structure, file explanations, build gotchas, and step-by-step setup — see **[tech.md](tech.md)**.

## References

- [Build declarative agents with MCP](https://devblogs.microsoft.com/microsoft365dev/build-declarative-agents-for-microsoft-365-copilot-with-mcp/) — announcement blog
- [MCP Apps in Copilot Chat](https://devblogs.microsoft.com/microsoft365dev/mcp-apps-now-available-in-copilot-chat/) — UI widgets announcement
- [Build MCP plugins](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/build-mcp-plugins) — official guide
- [Publish agents](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/publish) — distribution options
- [Learn MCP Server connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) — certified Power Platform connector
