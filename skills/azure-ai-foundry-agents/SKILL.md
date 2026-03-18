---
name: azure-ai-foundry-agents
description: Guides creation, deployment, governance, and observability of AI agents and multi-agent systems on Azure AI Foundry using Microsoft Agent Framework. Use when building agents with function calling, Databricks Genie, vector databases (Azure AI Search), AI Gateway governance, or Application Insights monitoring. Triggers on phrases like "create agent on Azure", "deploy agent Foundry", "multi-agent Azure", "agent with tools Azure", "Databricks Genie agent", "vector search agent Foundry", "Azure AI agent", "Microsoft Agent Framework", "AI Gateway agent governance", "monitor agent App Insights", "token limit agent Azure", "content safety agent", "agent observability Azure", "agent telemetry Azure".
license: MIT
compatibility: Requires Azure subscription and azure-ai-projects Python SDK (>=2.0.0b4, v2 beta line) or Microsoft.Agents.AI .NET package. SDK v2 has breaking changes vs v1 — see SDK Migration Notes below. Works with Claude Code, Claude Desktop, VS Code with GitHub Copilot, and Cursor.
metadata:
  author: Alessandro Armillotta
  version: 1.2.0
  category: azure-ai
  tags: [azure, ai-foundry, agents, multi-agent, databricks, vector-db, microsoft-agent-framework, ai-gateway, app-insights, observability]
  dependencies:
    - name: azure-microsoft-foundry
      repo: MicrosoftDocs/Agent-Skills
      raw_base: https://raw.githubusercontent.com/MicrosoftDocs/Agent-Skills/main/skills/azure-microsoft-foundry
      files: [SKILL.md]
    - name: azure-cognitive-search
      repo: MicrosoftDocs/Agent-Skills
      raw_base: https://raw.githubusercontent.com/MicrosoftDocs/Agent-Skills/main/skills/azure-cognitive-search
      files: [SKILL.md]
    - name: azure-ai-services
      repo: MicrosoftDocs/Agent-Skills
      raw_base: https://raw.githubusercontent.com/MicrosoftDocs/Agent-Skills/main/skills/azure-ai-services
      files: [SKILL.md]
    - name: azure-ai
      repo: microsoft/azure-skills
      raw_base: https://raw.githubusercontent.com/microsoft/azure-skills/main/.github/plugins/azure-skills/skills/azure-ai
      files: [SKILL.md]
    - name: azure-aigateway
      repo: microsoft/azure-skills
      raw_base: https://raw.githubusercontent.com/microsoft/azure-skills/main/.github/plugins/azure-skills/skills/azure-aigateway
      files: [SKILL.md]
    - name: appinsights-instrumentation
      repo: microsoft/azure-skills
      raw_base: https://raw.githubusercontent.com/microsoft/azure-skills/main/.github/plugins/azure-skills/skills/appinsights-instrumentation
      files: [SKILL.md]
    - name: azure-diagnostics
      repo: microsoft/azure-skills
      raw_base: https://raw.githubusercontent.com/microsoft/azure-skills/main/.github/plugins/azure-skills/skills/azure-diagnostics
      files: [SKILL.md]
  mcp_servers:
    - name: microsoft-learn
      type: http
      url: https://learn.microsoft.com/api/mcp
---

# Azure AI Foundry Agents

Guide for creating, configuring, and deploying AI agents and multi-agent systems on Azure AI Foundry using Microsoft Agent Framework.

> **Live documentation:** If `mcp_microsoftdocs:microsoft_docs_fetch` is available (VS Code with MCP enabled), use it to retrieve live Microsoft docs for limits, quotas, SDK references, and changelogs. This skill handles the implementation workflow; the MicrosoftDocs skills (`azure-ai-foundry-local`, `azure-cognitive-search`, `azure-ai-services`) handle live documentation retrieval.

## Prerequisites

Before starting, gather from the user:
1. **Azure project endpoint** — format: `https://<resource>.ai.azure.com/api/projects/<project>`
2. **Model deployment name** — e.g. `gpt-4o`, `gpt-4o-mini`, `claude-sonnet`
3. **Agent type** — single agent or multi-agent orchestration
4. **Integrations needed** — function calling, Databricks Genie, Azure AI Search (vector DB), MCP tools
5. **Language** — Python or C#
6. **Governance needed?** — AI Gateway (token limits, content safety, semantic caching, rate limiting)
7. **Observability needed?** — Application Insights (telemetry, traces, latency, error rates)

CRITICAL: Always ask these questions before generating any code.

> **Related skills to load alongside this one:**
> - `azure-microsoft-foundry` (from MicrosoftDocs/Agent-Skills) — live Microsoft Foundry docs and SDK references
> - `azure-cognitive-search` (from MicrosoftDocs/Agent-Skills) — Azure AI Search patterns for RAG
> - `azure-ai-services` (from MicrosoftDocs/Agent-Skills) — Azure AI services reference
> - `azure-ai` (from microsoft/azure-skills) — AI Search, Speech, OpenAI, Document Intelligence
> - `azure-aigateway` (from microsoft/azure-skills) — AI Gateway governance: token limits, content safety, caching, MCP rate limiting
> - `appinsights-instrumentation` (from microsoft/azure-skills) — App Insights SDK, telemetry patterns, APM for agents
> - `azure-diagnostics` (from microsoft/azure-skills) — debug and troubleshoot deployed agents (Container Apps, Function Apps, KQL)

## Instructions

### Step 1: Project Setup

**Python:**
```bash
pip install "azure-ai-projects>=2.0.0b4" azure-identity
```

**C#:**
```bash
dotnet add package Azure.AI.Projects --prerelease
dotnet add package Azure.Identity
```

Initialize the client:

**Python:**
```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

project = AIProjectClient(
    endpoint="https://<resource>.ai.azure.com/api/projects/<project>",
    credential=DefaultAzureCredential()
)
openai = project.get_openai_client()
```

### Step 2: Create a Single Agent

**Minimal agent (Python):**
```python
from azure.ai.projects.models import PromptAgentDefinition

agent = project.agents.create_version(
    agent_name="my-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="You are a helpful assistant.",
        tools=[]
    )
)
```

**Invoke the agent:**
```python
response = openai.responses.create(
    input="Hello, what can you do?",
    extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}}
)
print(response.output_text)
```

**Cleanup (always delete agent versions when done testing):**
```python
project.agents.delete_version(agent_name=agent.name, agent_version=agent.version)
```

### Step 3: Add Tools (Function Calling)

For each custom tool, define name, description, and JSON schema parameters. The agent decides when to invoke each tool.

See `references/function-calling.md` for the complete 5-step workflow and full examples.

**Quick example:**
```python
from azure.ai.projects.models import FunctionTool

weather_tool = FunctionTool(
    name="get_weather",
    description="Get current weather for a city",
    parameters={
        "type": "object",
        "properties": {
            "location": {"type": "string", "description": "City and state, e.g. Milan, IT"},
            "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
        },
        "required": ["location"],
        "additionalProperties": False
    },
    strict=True
)

agent = project.agents.create_version(
    agent_name="weather-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o-mini",
        instructions="You are a weather assistant. Use get_weather to answer weather questions.",
        tools=[weather_tool]
    )
)
```

After the agent responds with a function call, execute the function and submit output:
```python
from openai.types.responses.response_input_param import FunctionCallOutput
import json

for item in response.output:
    if item.type == "function_call" and item.name == "get_weather":
        result = get_weather(**json.loads(item.arguments))
        final = openai.responses.create(
            input=[FunctionCallOutput(type="function_call_output", call_id=item.call_id, output=json.dumps(result))],
            previous_response_id=response.id,
            extra_body={"agent_reference": {"name": agent.name, "type": "agent_reference"}}
        )
        print(final.output_text)
```

### Step 4: Add Vector Search (Azure AI Search)

For RAG (Retrieval-Augmented Generation) use cases, attach Azure AI Search as vector store.

See `references/vector-db.md` for full configuration.

```python
from azure.ai.projects.models import FileSearchTool, VectorStore

# Attach file search tool to agent
file_search = FileSearchTool(vector_store_ids=["<vector_store_id>"])

agent = project.agents.create_version(
    agent_name="rag-agent",
    definition=PromptAgentDefinition(
        model="gpt-4o",
        instructions="Answer questions using the knowledge base. Always cite sources.",
        tools=[file_search]
    )
)
```

### Step 5: Add Databricks Genie Integration

Databricks Genie exposes data analytics through natural language via MCP or direct API.

See `references/databricks-genie.md` for full setup.

**Key constraints:**
- Rate limit: 5 questions per minute
- Authentication via Entra ID (brokered automatically in Azure)
- Requires Azure Databricks workspace with Genie enabled

```python
# Connect Databricks Genie via MCP tool endpoint
from azure.ai.projects.models import McpTool

genie_tool = McpTool(
    server_label="databricks-genie",
    server_url="https://<workspace>.azuredatabricks.net/api/2.0/genie/mcp",
    allowed_tools=["genie_query"]
)
```

### Step 6: Multi-Agent Orchestration

Build an orchestrator agent that delegates to specialist sub-agents.

See `references/multi-agent-patterns.md` for full patterns.

**Architecture:**
```
User Request
    ↓
Orchestrator Agent
    ├→ Data Agent (Databricks Genie + Azure AI Search)
    ├→ Analysis Agent (function calling + code interpreter)
    └→ Report Agent (document generation)
    ↓
Aggregated Response
```

**Orchestrator definition:**
```python
# Create specialist agents first
data_agent = project.agents.create_version(
    agent_name="data-specialist",
    definition=PromptAgentDefinition(
        model="gpt-4o",
        instructions="You are a data specialist. Query Databricks Genie and Azure AI Search to answer data questions.",
        tools=[genie_tool, file_search]
    )
)

# Create orchestrator with connected agents
from azure.ai.projects.models import ConnectedAgentTool

orchestrator = project.agents.create_version(
    agent_name="orchestrator",
    definition=PromptAgentDefinition(
        model="gpt-4o",
        instructions="""You are an orchestrator. Delegate tasks to specialist agents:
- Data questions → data-specialist
- Analysis tasks → analysis-specialist
Always synthesize results into a coherent final response.""",
        tools=[
            ConnectedAgentTool(agent_name="data-specialist", description="Handles data queries and retrieval"),
            ConnectedAgentTool(agent_name="analysis-specialist", description="Performs data analysis and calculations")
        ]
    )
)
```

## Governance: Azure AI Gateway

Usa Azure API Management (APIM) come AI Gateway per governare gli endpoint degli agenti. Fornisce: token limits, content safety, semantic caching, rate limiting, jailbreak detection, e observability centralizzata.

### Quando usarlo
- Produzione con più agenti o utenti che condividono lo stesso endpoint
- Controllo costi (token budget per utente/team)
- Content safety obbligatoria (compliance)
- Load balancing tra più deployment dello stesso modello

### Setup rapido: esporre un agente dietro AI Gateway

```bash
# Prerequisiti: APIM instance già provisioned
az apim api import \
  --resource-group <rg> \
  --service-name <apim-name> \
  --specification-format OpenApi \
  --specification-url "https://<resource>.ai.azure.com/api/projects/<project>/openapi.json" \
  --path "agents/my-agent"
```

**Aggiungere policy LLM in APIM (bicep):**
```bicep
resource aiGatewayPolicy 'Microsoft.ApiManagement/service/apis/policies@2023-05-01-preview' = {
  name: 'policy'
  properties: {
    value: '''
    <policies>
      <inbound>
        <!-- Token limit: max 10k tokens/min per subscription -->
        <azure-openai-token-limit tokens-per-minute="10000" counter-key="@(context.Subscription.Id)" />
        <!-- Semantic cache: risposta cached per query simili (soglia 0.85) -->
        <azure-openai-semantic-cache-lookup score-threshold="0.85" />
        <!-- Content safety: blocca jailbreak e contenuti dannosi -->
        <azure-openai-content-safety />
      </inbound>
      <outbound>
        <azure-openai-semantic-cache-store duration="3600" />
        <!-- Emit token metrics verso App Insights -->
        <azure-openai-emit-token-metric />
      </outbound>
    </policies>
    '''
  }
}
```

> Per la configurazione completa di AI Gateway con agent endpoints, load balancing multi-backend e MCP rate limiting, carica la skill `azure-aigateway`.

---

## Observability: Application Insights

Strumenta gli agenti con Azure Application Insights per tracciare latenze, token usage, errori, e flussi conversazionali.

### Install

```bash
pip install "azure-ai-projects>=2.0.0b4" azure-monitor-opentelemetry
```

### Enable tracing su AIProjectClient

```python
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.monitor.opentelemetry import configure_azure_monitor

# Configura App Insights — APPLICATIONINSIGHTS_CONNECTION_STRING da env o Key Vault
configure_azure_monitor()

project = AIProjectClient(
    endpoint="https://<resource>.ai.azure.com/api/projects/<project>",
    credential=DefaultAzureCredential()
)

# Abilita tracing OpenTelemetry sul client (cattura ogni call all'agente)
project.telemetry.enable()
```

### Cosa viene tracciato automaticamente
- Ogni invocazione dell'agente (input, output, latenza)
- Tool calls (nome tool, argomenti, risposta)
- Token usage (prompt + completion tokens per request)
- Errori e retry

### Query KQL utili (Azure Monitor / Log Analytics)

```kql
// Latenza media per agente (ultimi 7 giorni)
dependencies
| where timestamp > ago(7d)
| where type == "Azure AI Foundry"
| summarize avg_latency_ms=avg(duration), p95_latency_ms=percentile(duration, 95)
    by agent_name=tostring(customDimensions["agent.name"])
| order by avg_latency_ms desc

// Token usage per giorno
customMetrics
| where name == "gen_ai.client.token.usage"
| summarize total_tokens=sum(value) by bin(timestamp, 1d), tostring(customDimensions["gen_ai.token.type"])
| render timechart

// Errori agente per tipo
exceptions
| where timestamp > ago(24h)
| summarize count() by type, outerMessage
| order by count_ desc
```

> Per pattern avanzati di instrumentazione (custom spans, sampling, SDK setup), carica la skill `appinsights-instrumentation`. Per debugging di Container Apps o Function Apps che ospitano gli agenti, usa `azure-diagnostics`.

---

## IDE Compatibility Notes

### VS Code / GitHub Copilot
- Place this skill in `.github/` as `copilot-instructions.md` to use with Copilot Chat
- Reference Azure SDK docs via `@docs` in Copilot Chat
- Use `#codebase` to let Copilot analyze existing agent code

### Cursor
- Add skill content to `.cursor/rules/azure-agents.mdc`
- Works with Cursor's `@azure` context for SDK autocompletion

## Common Issues

**Error: `DefaultAzureCredential` fails**
- Run `az login` in terminal
- Or set `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`, `AZURE_TENANT_ID` env vars

**Error: Model not found**
- Verify the deployment name in Azure AI Foundry portal (Settings > Models)
- Model name ≠ deployment name (e.g., deployment `my-gpt4o` runs model `gpt-4o`)

**Function calls not triggered**
- Improve tool `description` — be explicit about when to use it
- Add example phrases to the description
- Set `strict=True` in FunctionTool definition

**Databricks Genie rate limit exceeded**
- Implement exponential backoff: wait 60s after 5 calls/minute
- Cache frequent queries in Azure AI Search vector store

**Multi-agent: sub-agent not invoked**
- Improve `description` in `ConnectedAgentTool`
- Be explicit in orchestrator instructions about which agent handles what

## Performance Notes
- Take time to correctly define tool schemas — precision here prevents errors downstream
- For multi-agent systems, always define clear boundaries between agents in their instructions
- Do not skip cleanup of agent versions after testing

---

## SDK Migration Notes (v2.0.0b4+)

The `azure-ai-projects` SDK v2 (Python `2.0.0b4+`, .NET `2.0.0-beta.1+`) has breaking changes vs v1:

| Area | Old (v1) | New (v2) |
|------|----------|----------|
| Authentication | `ad_token`, `ad_token_provider` params | Unified `credential` param (`DefaultAzureCredential`) |
| Session management | `AgentThread` type | Removed — use `openai.responses` API directly |
| Checkpoints | Previous format | Redesigned — migrate or regenerate existing artifacts |
| Event property | `source_executor_id` | Renamed to `executor_id` in `WorkflowOutputEvent` |

Install command (`pip install "azure-ai-projects>=2.0.0b4"`) already targets v2. If upgrading from v1, follow the [Microsoft upgrade guide](https://learn.microsoft.com/en-us/agent-framework/support/upgrade/python-2026-significant-changes).

---

## New Features (2026)

### Managed Long-Term Memory
The Foundry Agent Service now includes a fully managed memory store — no custom vector DB or retrieval pipeline required. Enable per agent:
```python
# Memory is configured via the agent definition — consult live docs via microsoft-learn MCP
# Tool: mcp_microsoftdocs:microsoft_docs_fetch
# Query: "azure ai foundry agent memory configuration"
```
Memory runs in 4 phases: Extract → Consolidate → Retrieve → Customize. Use `user_profile_details` to focus extraction on your domain.

### Voice Live API (Public Preview)
Build voice-first, multimodal, real-time agents via the Voice Live API integration with Foundry Agent Service. Consult live docs for the current SDK interface.

### Durable Agent Orchestration
New HITL (Human-in-the-Loop) pattern: pair Azure Durable Functions with Agent Framework and SignalR for agents that survive restarts and wait for human approval across sessions.
