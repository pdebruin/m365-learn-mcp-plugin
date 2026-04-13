# Technical Details

## Architecture

```
┌─────────────────────┐     MCP (Streamable HTTP)     ┌──────────────────────────┐
│  M365 Copilot Chat  │ ──────────────────────────────▶│  learn.microsoft.com/api │
│                      │     POST, no auth              │         /mcp             │
│  Declarative Agent   │◀──────────────────────────────│  (Learn MCP Server)      │
│  (this project)      │     Tool results + citations   │                          │
└─────────────────────┘                                └──────────────────────────┘
```

The agent has **no backend**. It's a configuration-only app package that tells M365 Copilot:
- What MCP server to connect to (`https://learn.microsoft.com/api/mcp`)
- What tools are available (3 tools, read-only, no auth)
- How to behave (instructions, conversation starters)

The Learn MCP Server is public and requires no authentication.

## Project structure

```
learn-mcp-app/
├── appPackage/
│   ├── manifest.json            # Teams app manifest (v1.25)
│   ├── declarativeAgent.json    # Agent config (v1.6) — name, starters, actions
│   ├── ai-plugin.json           # Plugin manifest (v2.4) — tool definitions, MCP runtime
│   ├── mcp-tools.json           # Tool schemas from Learn MCP Server's tools/list
│   ├── instruction.txt          # Agent system instructions
│   ├── color.png                # App icon (192x192)
│   └── outline.png              # App icon outline (32x32)
├── env/
│   └── .env.dev                 # Provisioned app IDs (TEAMS_APP_ID, M365_TITLE_ID, etc.)
├── m365agents.yml               # ATK lifecycle config (v1.11)
└── .vscode/                     # VS Code settings for ATK
```

## Key files explained

### `ai-plugin.json` — the heart of the agent

Declares the 3 functions and maps them to the MCP server runtime:

```json
{
  "schema_version": "v2.4",
  "runtimes": [{
    "type": "RemoteMCPServer",
    "spec": {
      "url": "https://learn.microsoft.com/api/mcp",
      "mcp_tool_description": { "file": "mcp-tools.json" }
    },
    "run_for_functions": [
      "microsoft_docs_search",
      "microsoft_code_sample_search",
      "microsoft_docs_fetch"
    ],
    "auth": { "type": "None" }
  }]
}
```

The `response_semantics` on search functions map `$.results[].title` and `$.results[].contentUrl` to citation links in Copilot Chat.

### `mcp-tools.json` — tool schemas

**Critical format requirement:** Must be `{"tools": [...]}` (object with a `tools` key), NOT a bare JSON array `[...]`. This was the primary cause of 400 errors during our initial hand-crafted attempts. ATK's "Fetch action from MCP" button produces the correct format.

Includes `inputSchema`, `outputSchema`, and `annotations` (with `readOnlyHint: true`) for each tool.

### `instruction.txt` — agent behavior

Tells the agent to always search documentation before answering, use the right tool for the right task (search → code samples → fetch), and include source links.

## How we built this

### Timeline

MCP server support for M365 Copilot declarative agents was announced at **Ignite 2025** (preview) and reached **GA in March 2026** with ATK v6.6.0. This project was built in April 2026.

### What worked

1. **ATK wizard scaffolding** — Created a new declarative agent via "Start with an MCP Server" in the M365 Agents Toolkit VS Code extension
2. **ATK "Fetch action from MCP" button** — Pulled tool schemas directly from the live server, generating correctly-formatted `ai-plugin.json` and `mcp-tools.json`
3. **Iterative provisioning** — Small changes, provision, test, repeat

### What didn't work (lessons learned)

We initially hand-crafted all manifest files from documentation. This failed with persistent HTTP 400 errors during `extendToM365`. The errors were unhelpful ("Internal Error - Failed to make a successful HTTP request"). After extensive debugging, the root causes were:

| Problem | Symptom | Fix |
|---|---|---|
| `mcp-tools.json` was a bare array `[...]` | 400 during `extendToM365` | Must be `{"tools": [...]}` |
| Missing `$schema` in `ai-plugin.json` | Validation failure | Add `"$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.4/schema.json"` |
| Missing `namespace` and `contact_email` | Validation failure | Required fields in ai-plugin.json |
| `run_for_functions` was empty `[]` | Explicit validation error | Must list all function names |
| Old schema versions (manifest v1.19, agent v1.3) | Silent failures | Use manifest v1.25, agent v1.6, YAML v1.11 |
| Config file named `teamsapp.yml` | ATK didn't recognize project | Must be `m365agents.yml` for ATK v6.3+ |

**Lesson:** Use the ATK wizard to scaffold and fetch tools. Don't hand-craft manifest files — the validation is opaque and the format requirements are underdocumented.

### Prerequisites that blocked us

- **Copilot license** — Even on M365 Developer Program sandbox tenants, you must purchase and assign a Copilot license. Without it: "Microsoft 365 account administrator hasn't enabled Copilot access for this account"
- **Custom app upload** — Must be enabled in Teams admin center → Setup policies → Upload custom apps
- **Version bumping** — Every `extendToM365` call requires a higher `version` in manifest.json than the previous deployment

## Getting started

### Prerequisites

- [VS Code](https://code.visualstudio.com/) with [M365 Agents Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) extension
- M365 tenant with:
  - Copilot license assigned to your account
  - Custom app upload enabled
- Both checkmarks green in ATK: "Custom App Upload Enabled" ✅ and "Copilot Access Enabled" ✅

### Provision & test

1. Open `learn-mcp-app/` in VS Code
2. Open the ATK sidebar → click **Provision** → select **dev**
3. ATK runs 4 steps: create app → zip package → update app → extend to M365
4. Open [m365.cloud.microsoft/chat](https://m365.cloud.microsoft/chat)
5. Find "Learn Docs" in the agent picker and start chatting

### Making changes

After editing any file in `appPackage/`:
1. Bump `version` in `manifest.json` (e.g., `1.0.4` → `1.0.5`)
2. Re-provision via ATK

Changes to `instruction.txt` and `declarativeAgent.json` require re-provisioning. The MCP server itself is external — no deployment needed on our side.

## Distribution options

| Method | Supported |
|---|---|
| Sideload (personal dev/test) | ✅ |
| Organizational catalog (your tenant) | ✅ via Teams Admin Center |
| Microsoft Commercial Marketplace (public) | ✅ via [Partner Center](https://partner.microsoft.com) |

For marketplace submission, the agent must pass:
- [Commercial Marketplace certification policies](https://learn.microsoft.com/en-us/legal/marketplace/certification-policies)
- [Store validation guidelines for agents](https://learn.microsoft.com/en-us/microsoftteams/platform/concepts/deploy-and-publish/appsource/prepare/review-copilot-validation-guidelines)
- [Responsible AI validation checks](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/rai-validation)

## References

- [Build declarative agents with MCP](https://devblogs.microsoft.com/microsoft365dev/build-declarative-agents-for-microsoft-365-copilot-with-mcp/) — announcement blog post
- [Build MCP plugins](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/build-mcp-plugins) — official guide
- [Plugin manifest schema v2.4](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/api-plugin-manifest) — ai-plugin.json reference
- [Declarative agent schema v1.6](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/declarative-agent-manifest-1.6) — agent manifest reference
- [Publish agents](https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/publish) — distribution options
- [Learn MCP Server connector](https://learn.microsoft.com/en-us/connectors/microsoftlearndocsmcpserver/) — certified connector (different from this agent)
