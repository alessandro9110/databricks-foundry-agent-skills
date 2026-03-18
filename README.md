# databricks-foundry-agent-skills

> A curated collection of Claude skills for building production-grade AI agents on **Databricks** and **Azure AI Foundry** — with LangGraph, LangChain, Deep Agents, and OpenAI Agent SDK.

Works with Claude Code, Claude Desktop, VS Code with GitHub Copilot, and Cursor.

---

## What you can build

### On Databricks (Mosaic AI)

- **Stateful agents** with LangGraph — custom graph nodes, conditional routing, persistent memory
- **Multi-step planning agents** with Deep Agents — automatic task breakdown, subagent delegation, file context management
- **Human-in-the-loop workflows** — interrupt/resume patterns with Lakebase checkpointer
- **RAG agents** — Unity Catalog Vector Search + UC function tools
- **Supervisor agents** — multi-agent orchestration with dedicated specialist subagents (GA)
- **Production deployment** via Model Serving (`agents.deploy()`) or Databricks Apps (FastAPI + chat UI)
- **Full MLflow tracing** — every agent step traced and visible in the Databricks UI

### On Azure AI Foundry

- **Single agents** with function calling, Azure AI Search (RAG), MCP tools
- **Multi-agent orchestration** — orchestrator + specialist agents via `ConnectedAgentTool`
- **Databricks Genie integration** — natural language data analytics from Azure agents via MCP
- **Managed long-term memory** — per-user context extraction, consolidation, and retrieval across sessions
- **Enterprise governance** — AI Gateway (token limits, content safety, semantic caching, jailbreak detection)
- **Production observability** — Application Insights telemetry, token metrics, KQL queries
- **Durable orchestration** — HITL workflows with Azure Durable Functions + SignalR

---

## Skills

### `databricks-mosaic-ai-agents` · v1.3.0

Build and deploy custom AI agents on Databricks using Mosaic AI Agent Framework.

| | |
|---|---|
| **Frameworks** | LangGraph · LangChain · Deep Agents (on LangGraph) · OpenAI Agent SDK |
| **Deployment** | Model Serving (`agents.deploy()`) · Databricks Apps · deploy from Git |
| **Tools** | Unity Catalog functions · Vector Search (RAG) · Databricks SQL · Genie |
| **Memory** | Lakebase Provisioned (managed PostgreSQL checkpointer) |
| **Orchestration** | Supervisor Agent (GA) · SubAgent delegation · TodoList planning |
| **HITL** | `interrupt()` / `Command(resume=)` with Lakebase checkpointer |
| **Observability** | MLflow auto-tracing · Experiment UI · Agent evaluation |

**Trigger phrases:**
`"build agent Databricks"` · `"LangGraph Mosaic AI"` · `"Deep Agents Databricks"` · `"multi-agent planning Databricks"` · `"subagent delegation"` · `"deploy agent MLflow"` · `"human-in-the-loop Databricks"` · `"Databricks Apps agent"` · `"UC function tool"` · `"Supervisor Agent Databricks"`

---

### `azure-ai-foundry-agents` · v1.2.0

Create, deploy, govern, and monitor AI agents on Azure AI Foundry using Microsoft Agent Framework.

| | |
|---|---|
| **SDK** | `azure-ai-projects` Python `>=2.0.0b4` (v2 beta line) · `Azure.AI.Projects` .NET |
| **Agent types** | Single agent · Multi-agent orchestration (`ConnectedAgentTool`) |
| **Integrations** | Function calling · Azure AI Search · Databricks Genie (MCP) · MCP tools |
| **Memory** | Managed long-term memory (extract → consolidate → retrieve → customize) |
| **Governance** | Azure AI Gateway — token limits, content safety, semantic caching, rate limiting |
| **Observability** | Application Insights — traces, token usage, latency, error KQL queries |
| **New patterns** | Voice Live API (preview) · Durable Agent Orchestration |

**Trigger phrases:**
`"create agent on Azure"` · `"deploy agent Foundry"` · `"multi-agent Azure"` · `"Databricks Genie agent"` · `"AI Gateway agent governance"` · `"monitor agent App Insights"` · `"agent telemetry Azure"` · `"Microsoft Agent Framework"`

> ⚠️ **SDK v2 breaking changes:** `ad_token`/`ad_token_provider` replaced by unified `credential` param · `AgentThread` removed · checkpoint format redesigned. See [Microsoft upgrade guide](https://learn.microsoft.com/en-us/agent-framework/support/upgrade/python-2026-significant-changes).

---

## How skills work

Skills are `SKILL.md` files loaded into the AI assistant's context at session startup. When you describe a task, the assistant matches it against each skill's `description` field and applies the relevant guidance automatically — no manual activation needed.

All skills (both from this repo and external dependencies) are installed into the same local directory and loaded together:

```
.claude/skills/
├── databricks-mosaic-ai-agents/SKILL.md   ← this repo
├── azure-ai-foundry-agents/SKILL.md       ← this repo
├── deep-agents-core/SKILL.md              ← auto-downloaded from langchain-ai/langchain-skills
├── langgraph-fundamentals/SKILL.md        ← auto-downloaded from langchain-ai/langchain-skills
├── azure-aigateway/SKILL.md               ← auto-downloaded from microsoft/azure-skills
├── databricks-bundles/SKILL.md            ← auto-downloaded from databricks-solutions/ai-dev-kit
└── ...                                    ← all other dependencies
```

The main skill sets the implementation workflow. Dependency skills provide supporting context — the assistant references them when it needs specific details (e.g. LangGraph graph design, APIM policy syntax, App Insights SDK setup).

---

## What gets installed

### Skills from this repo

| Skill | Version | Description |
|-------|---------|-------------|
| `databricks-mosaic-ai-agents` | v1.3.0 | Build and deploy Mosaic AI agents on Databricks |
| `azure-ai-foundry-agents` | v1.2.0 | Create and deploy agents on Azure AI Foundry |

### External dependencies — auto-downloaded at install

Dependencies are declared in each `SKILL.md` frontmatter and downloaded automatically by `install.sh`. No manual configuration required.

#### From [`databricks-solutions/ai-dev-kit`](https://github.com/databricks-solutions/ai-dev-kit)

| Skill | Why it's included |
|-------|-------------------|
| `databricks-bundles` | DAB structure, `databricks.yml` patterns, deploy/run commands |
| `databricks-app-python` | Databricks Apps patterns: OAuth, FastAPI, Streamlit, resource permissions |
| `databricks-model-serving` | Model Serving endpoint concepts, scaling, traffic routing |
| `databricks-vector-search` | Vector Search index creation, querying, embedding management |
| `databricks-mlflow-evaluation` | Agent evaluation with MLflow: judges, metrics, labeling sessions |
| `databricks-lakebase-provisioned` | Managed PostgreSQL on Databricks — used as LangGraph checkpointer for persistent agent memory |

#### From [`langchain-ai/langchain-skills`](https://github.com/langchain-ai/langchain-skills)

| Skill | Why it's included |
|-------|-------------------|
| `langgraph-fundamentals` | LangGraph StateGraph, nodes, edges, and conditional routing patterns |
| `langgraph-persistence` | Checkpointer setup (InMemorySaver for dev, PostgresSaver for prod) |
| `langgraph-human-in-the-loop` | `interrupt()` / `Command(resume=)` patterns, 4-tier error handling |
| `langchain-fundamentals` | LangChain agent creation, tool definition, middleware |
| `framework-selection` | Decision guide: LangGraph vs LangChain vs Deep Agents vs OpenAI Agent SDK |
| `deep-agents-core` | `create_deep_agent()` harness — planning, subagents, file context, middleware configuration |
| `deep-agents-memory` | Deep Agents persistent memory: Store setup, MemoryMiddleware, cross-session context |
| `deep-agents-orchestration` | SubAgentMiddleware, TodoListMiddleware, HumanInTheLoopMiddleware |

#### From [`MicrosoftDocs/Agent-Skills`](https://github.com/MicrosoftDocs/Agent-Skills)

| Skill | Why it's included |
|-------|-------------------|
| `azure-microsoft-foundry` | Live Microsoft Foundry docs: SDK references, limits, quotas, changelogs |
| `azure-cognitive-search` | Azure AI Search: indexes, skillsets, vector/semantic search, RAG patterns |
| `azure-ai-services` | Azure AI services: Speech, Document Intelligence, Vision |

#### From [`microsoft/azure-skills`](https://github.com/microsoft/azure-skills)

| Skill | Why it's included |
|-------|-------------------|
| `azure-ai` | Azure AI Search, Speech, OpenAI, Document Intelligence — used as tools within agents |
| `azure-aigateway` | Configure APIM as AI Gateway: token limits, content safety, semantic caching, MCP rate limiting |
| `appinsights-instrumentation` | App Insights SDK setup, telemetry patterns, APM configuration for deployed agents |
| `azure-diagnostics` | Debug and troubleshoot agents deployed on Container Apps or Function Apps using KQL |

### MCP servers (optional)

Configured when choosing **Skills + MCP servers** during install.

| Server | Type | What it enables |
|--------|------|-----------------|
| `databricks` | stdio | Live access to your Databricks workspace: SQL execution, job management, model serving endpoints, Vector Search indexes, Lakebase instances — usable as tools during coding |
| `microsoft-learn` | http | Live Microsoft documentation retrieval — the `azure-microsoft-foundry` skill uses this to fetch up-to-date SDK references and API limits |

---

## Prerequisites

The installer checks these automatically and shows install hints for anything missing.

### Required

| Tool | macOS | Linux | Windows |
|------|-------|-------|---------|
| `bash` | built-in | built-in | WSL or Git Bash |
| `curl` | built-in | `sudo apt install curl` | built-in (Win 10+) |
| `python3` | `brew install python` | `sudo apt install python3` | [python.org](https://www.python.org/downloads/) |
| `git` | `brew install git` | `sudo apt install git` | [git-scm.com](https://git-scm.com/downloads) |

> **Windows:** use [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) (`wsl bash install.sh`) or Git Bash.

### Recommended (needed for MCP server setup)

| Tool | Install |
|------|---------|
| `uv` | `curl -LsSf https://astral.sh/uv/install.sh \| sh` — faster venv creation; falls back to `python3 -m venv` |
| `databricks` CLI | `brew tap databricks/tap && brew install databricks` — needed for `~/.databrickscfg` OAuth auth |

### Python libraries (for your own projects)

**Databricks Mosaic AI agents:**
```bash
pip install databricks-langchain langgraph deepagents mlflow databricks-agents databricks-sdk
```

**Azure AI Foundry agents:**
```bash
pip install "azure-ai-projects>=2.0.0b4" azure-identity azure-monitor-opentelemetry
```

---

## Installation

### Quick install

```bash
bash <(curl -sL https://raw.githubusercontent.com/alessandro9110/databricks-foundry-agent-skills/main/install.sh)
```

The installer is fully interactive with arrow-key menus. No flags required.

### What the installer does

| Step | What happens |
|------|-------------|
| **1. Prerequisites check** | Verifies `curl`, `python3`, `git`, `uv`, `databricks` CLI |
| **2. Scope** | Choose project (`./.claude/skills`) or global (`~/.claude/skills`) |
| **3. Summary** | Lists all skills, dependencies, and MCP servers before proceeding |
| **4. Skill install** | Copies skills from this repo + downloads all external dependencies via `curl` |
| **5. MCP choice** | Optionally clones `ai-dev-kit`, creates Python venv, configures `.mcp.json` |
| **6. Auth** | `~/.databrickscfg` via OAuth (recommended) or env vars written to shell profile |

### Install for multiple AI tools

```bash
git clone https://github.com/alessandro9110/databricks-foundry-agent-skills
cd databricks-foundry-agent-skills
bash install.sh --tools claude,cursor,copilot
```

### CLI flags

| Flag | Description |
|------|-------------|
| `--global, -g` | Force global install (`~/.claude/skills`) |
| `--tools, -t` | Comma-separated: `claude`, `cursor`, `copilot` (default: `claude`) |
| `--yes, -y` | Skip all prompts — project scope, skills only, no MCP |

---

## Usage

Once installed, skills activate automatically based on what you type. No slash commands or manual loading needed.

### Example prompts — Databricks

```
"Build a LangGraph agent that queries Unity Catalog and deploys to Model Serving"
"Create a Deep Agents setup on Databricks with a researcher subagent and SQL specialist"
"Add human-in-the-loop approval before any write operation in my Databricks agent"
"Deploy my agent to Databricks Apps with streaming and a built-in chat UI"
"Set up persistent memory for my agent using Lakebase"
```

### Example prompts — Azure AI Foundry

```
"Create an Azure AI agent with function calling and Azure AI Search for RAG"
"Build a multi-agent system on Foundry with an orchestrator and a data specialist"
"Connect my Azure agent to Databricks Genie for natural language data queries"
"Set up AI Gateway with token limits and content safety for my agent endpoint"
"Add Application Insights telemetry to my deployed Azure AI agent"
```

### IDE compatibility

| Feature | Claude Code | Claude Desktop | VS Code + Copilot | Cursor |
|---------|:-----------:|:--------------:|:-----------------:|:------:|
| Databricks skill | ✅ | ✅ | ✅ | ✅ |
| Azure AI Foundry skill | ✅ | ✅ | ✅ | ✅ |
| Databricks MCP server | ✅ | ✅ | ✅ | ✅ |
| microsoft-learn MCP | ✅ | ✅ | ✅ | ✅ |

---

## Maintenance

This repo includes a local `/deps-health` slash command (Claude Code only, not distributed) that checks all declared dependencies and searches for recent changes in the covered technologies:

```
/deps-health                    # check all skills
/deps-health azure-ai-foundry-agents   # check one skill
```

Run it periodically to catch broken dependency URLs and stay up to date with SDK changes.

---

## Contributing

1. Fork this repository
2. Follow the branch naming convention:

| Change type | Prefix | Example |
|-------------|--------|---------|
| New skill | `feature/` | `feature/add-openai-agents-skill` |
| Fix existing skill | `fix/` | `fix/update-azure-sdk-version` |
| Update dependencies | `fix/` | `fix/update-dependencies` |
| Documentation | `docs/` | `docs/update-readme` |

3. Create your skill under `skills/<skill-name>/SKILL.md` with valid YAML frontmatter (`name` + `description` required)
4. If your skill depends on external skills, declare them in the `metadata.dependencies` block — `install.sh` reads dependencies dynamically from the frontmatter, no manual script changes needed
5. Open a pull request against `main`

### Dependency declaration format

```yaml
metadata:
  dependencies:
    - name: skill-name
      repo: owner/repo
      raw_base: https://raw.githubusercontent.com/owner/repo/main/path/to/skill
      files: [SKILL.md]
```

> Always verify that `raw_base/SKILL.md` returns HTTP 200 before declaring a dependency. Use `curl -o /dev/null -s -w "%{http_code}" <url>` to check.

---

## Sources & External Repositories

| Repository | Maintainer | Skills provided |
|------------|------------|-----------------|
| [databricks-solutions/ai-dev-kit](https://github.com/databricks-solutions/ai-dev-kit) | Databricks | `databricks-bundles`, `databricks-app-python`, `databricks-model-serving`, `databricks-vector-search`, `databricks-mlflow-evaluation`, `databricks-lakebase-provisioned` |
| [langchain-ai/langchain-skills](https://github.com/langchain-ai/langchain-skills) | LangChain | `langgraph-fundamentals`, `langgraph-persistence`, `langgraph-human-in-the-loop`, `langchain-fundamentals`, `framework-selection`, `deep-agents-core`, `deep-agents-memory`, `deep-agents-orchestration` |
| [MicrosoftDocs/Agent-Skills](https://github.com/MicrosoftDocs/Agent-Skills) | Microsoft | `azure-microsoft-foundry`, `azure-cognitive-search`, `azure-ai-services` |
| [microsoft/azure-skills](https://github.com/microsoft/azure-skills) | Microsoft | `azure-ai`, `azure-aigateway`, `appinsights-instrumentation`, `azure-diagnostics` |
| [langchain-ai/deepagents](https://github.com/langchain-ai/deepagents) | LangChain | Source library for Deep Agents patterns (not a skill dependency — referenced for documentation) |
| [microsoft/agent-framework](https://github.com/microsoft/agent-framework) | Microsoft | Source library for Microsoft Agent Framework (not a skill dependency — referenced for documentation) |

---

## License

MIT
