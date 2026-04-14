# Learn Docs — M365 Copilot MCP Plugin for Microsoft Learn

A declarative agent for Microsoft 365 Copilot Chat that connects to the [Microsoft Learn MCP Server](https://learn.microsoft.com/api/mcp) via an MCP plugin, giving users the ability to search, browse, and read official Microsoft documentation directly from Copilot.

| Agent store listing | Agent in action |
|---|---|
| ![Agent store listing showing Learn Docs agent with description and permissions](splash.png) | ![Copilot Chat showing a grounded response about creating an Azure Storage Account via CLI](chat.png) |

## Why this exists

The Microsoft Learn MCP Server (`https://learn.microsoft.com/api/mcp`) exposes 3 read-only tools via the Model Context Protocol:

| Tool | What it does |
|---|---|
| `microsoft_docs_search` | Search Learn for articles — returns titles, URLs, and content excerpts |
| `microsoft_code_sample_search` | Find code snippets by query and language |
| `microsoft_docs_fetch` | Fetch full article content by URL |

These tools are already available to AI assistants (VS Code Copilot, CLI agents, Copilot Studio via connector). But they aren't directly available to **end users** in M365 Copilot Chat — someone has to package them as a declarative agent first.

This project is that packaging. It's a tool-only agent (no custom UI) that wraps the Learn MCP Server as an installable Copilot agent. Users open Copilot Chat, select the agent, and ask questions grounded in official Microsoft documentation.

### How is this different from the certified connector?

Learn MCP Server already has a [certified connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) for Copilot Studio. Different audience:

| | Certified connector | This agent |
|---|---|---|
| **Who uses it** | Agent builders in Copilot Studio | End users in M365 Copilot Chat |
| **Discovery** | Found when building an agent | Found in the Agent Store |
| **Setup required** | User creates agent, adds connector, configures | User installs, starts chatting |
| **Tools exposed** | `microsoft_docs_search` only | All 3 tools |

The connector serves builders. This agent serves users. Both can coexist.

For architecture, project structure, build notes, and lessons learned, see [tech.md](tech.md).

## Future work

- [ ] Custom icons (currently ATK default placeholders)
- [ ] Publish to organizational catalog for broader testing
- [ ] Marketplace submission via Partner Center
- [ ] Interactive UI widgets (MCP Apps) — e.g., architecture diagram viewer, code sample playground — only if the UI adds value beyond linking to learn.microsoft.com

## References

- [Build declarative agents with MCP](https://devblogs.microsoft.com/microsoft365dev/build-declarative-agents-for-microsoft-365-copilot-with-mcp/) — announcement blog post
- [Build MCP plugins](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/build-mcp-plugins) — official guide
- [Plugin manifest schema v2.4](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api-plugin-manifest) — ai-plugin.json reference
- [Declarative agent schema v1.6](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/declarative-agent-manifest-1.6) — agent manifest reference
- [Publish agents](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/publish) — distribution options
- [Learn MCP Server connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) — certified connector (different from this agent)
