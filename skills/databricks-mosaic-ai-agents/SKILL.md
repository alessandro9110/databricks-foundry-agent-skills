---
name: databricks-mosaic-ai-agents
description: Guides building and deploying custom AI agents on Databricks using Mosaic AI Agent Framework with LangGraph or LangChain. Use when creating agents with MLflow tracing, Unity Catalog functions as tools, Vector Search retrieval, or deploying agents via Model Serving (agents.deploy) or Databricks Apps. Triggers on phrases like "build agent Databricks", "LangGraph Mosaic AI", "LangChain Databricks agent", "deploy agent MLflow", "UC function tool", "agent asset bundle", "Databricks agent job deployment", "Mosaic AI LangGraph", "Databricks Apps agent".
license: MIT
compatibility: Requires Databricks workspace with Unity Catalog, MLflow 3.1.3+, databricks-langchain, langgraph or langchain, databricks-agents SDK. MCP server requires uv (install: curl -LsSf https://astral.sh/uv/install.sh | sh) — the ai-dev-kit repo is auto-cloned to ~/.databricks-ai-dev-kit during install. Works with Claude Code, Claude Desktop, VS Code with GitHub Copilot, and Cursor. Complements databricks-bundles, databricks-app-python, and langgraph-fundamentals skills.
metadata:
  author: Alessandro Armillotta
  version: 1.1.1
  category: databricks
  tags: [databricks, mosaic-ai, langgraph, langchain, mlflow, unity-catalog, agents, asset-bundle, databricks-apps]
  dependencies:
    - name: databricks-bundles
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-bundles
      files: [SKILL.md]
    - name: databricks-app-python
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-app-python
      files: [SKILL.md]
    - name: databricks-model-serving
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-model-serving
      files: [SKILL.md]
    - name: databricks-vector-search
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-vector-search
      files: [SKILL.md]
    - name: databricks-mlflow-evaluation
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-mlflow-evaluation
      files: [SKILL.md]
    - name: langgraph-fundamentals
      repo: langchain-ai/langchain-skills
      raw_base: https://raw.githubusercontent.com/langchain-ai/langchain-skills/main/config/skills/langgraph-fundamentals
      files: [SKILL.md]
    - name: langgraph-persistence
      repo: langchain-ai/langchain-skills
      raw_base: https://raw.githubusercontent.com/langchain-ai/langchain-skills/main/config/skills/langgraph-persistence
      files: [SKILL.md]
    - name: langchain-fundamentals
      repo: langchain-ai/langchain-skills
      raw_base: https://raw.githubusercontent.com/langchain-ai/langchain-skills/main/config/skills/langchain-fundamentals
      files: [SKILL.md]
    - name: framework-selection
      repo: langchain-ai/langchain-skills
      raw_base: https://raw.githubusercontent.com/langchain-ai/langchain-skills/main/config/skills/framework-selection
      files: [SKILL.md]
    - name: databricks-lakebase-provisioned
      repo: databricks-solutions/ai-dev-kit
      raw_base: https://raw.githubusercontent.com/databricks-solutions/ai-dev-kit/main/databricks-skills/databricks-lakebase-provisioned
      files: [SKILL.md]
  mcp_servers:
    - name: databricks
      type: stdio
      command: ~/.ai-dev-kit/.venv/bin/python
      args: ~/.ai-dev-kit/repo/databricks-mcp-server/run_server.py
      auto_clone: https://github.com/databricks-solutions/ai-dev-kit.git
---

### Databricks Mosaic AI Agents

Guide for building custom AI agents on Databricks using LangGraph or LangChain, and deploying them via **Model Serving** (production-grade REST endpoint) or **Databricks Apps** (rapid iteration with built-in UI).

> **Related skills to load alongside this one:**
> - `databricks-bundles` (from databricks-solutions/ai-dev-kit) — for bundle structure and deployment commands
> - `langgraph-fundamentals` (from langchain-ai/langchain-skills) — for LangGraph graph design patterns
> - `databricks-model-serving` (from databricks-solutions/ai-dev-kit) — for serving endpoint concepts
> - `databricks-lakebase-provisioned` (from databricks-solutions/ai-dev-kit) — for persistent agent memory via managed PostgreSQL
> - `databricks-app-python` (from databricks-solutions/ai-dev-kit) — for Databricks Apps patterns, OAuth, and FastAPI deployment

### Prerequisites

Ask the user for:
1. **Workspace URL** — `https://<workspace>.azuredatabricks.net`
2. **Unity Catalog target** — `catalog.schema` for model registration
3. **Agent framework** — LangGraph (recommended for stateful/multi-step) or LangChain
4. **Model endpoint** — e.g. `databricks-meta-llama-3-70b-instruct` or custom
5. **Tools needed** — UC functions, Vector Search, custom Python tools, Genie
6. **Deployment target** — Model Serving endpoint or Databricks App

CRITICAL: Always confirm the UC catalog.schema before generating any deployment code.

---

## Choose Your Deployment Path

| | **Model Serving** | **Databricks Apps** |
|---|---|---|
| **Best for** | Production, SLA, auto-scaling | Rapid iteration, CI/CD, custom UI |
| **Agent pattern** | MLflow pyfunc / `predict()` | `@invoke` / `@stream` decorators (async) |
| **Local dev** | Not supported | `uv run start-app` → localhost:8000 |
| **Built-in UI** | MLflow Review App | Chat UI included |
| **Deploy command** | `agents.deploy()` Python API | `databricks apps deploy` or DABs |
| **Dependencies** | `pip_requirements` in `log_model()` | `pyproject.toml` |
| **Auth** | Service principal auto-provisioned | App auth or per-user workspace client |
| **Streaming** | Supported | Native async streaming |
| **Stateful memory** | Lakebase checkpointer | Lakebase checkpointer |

---

## Deployment Path 1: Model Serving

### Step 1: Install Dependencies

```bash
pip install databricks-langchain langgraph mlflow databricks-agents databricks-sdk
```

### Step 2: Choose the Agent Framework

| Use LangGraph when | Use LangChain when |
|--------------------|--------------------|
| Multi-step reasoning with state | Simple ReAct agent loop |
| Human-in-the-loop needed | Straightforward tool calling |
| Complex conditional routing | Rapid prototyping |
| Persistent memory across turns | Chain-based workflows |

### Step 3: Build the Agent (Models from Code Pattern)

Create a standalone `agent.py` file — MLflow logs the file, not an object.

**LangGraph agent (`src/agent.py`):**
```python
import mlflow
from databricks_langchain import ChatDatabricks
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

mlflow.langchain.autolog()

# Define tools
@tool
def query_catalog(sql: str) -> str:
    """Execute a SQL query against Unity Catalog tables.
    Use for structured data retrieval, aggregations, and business metrics.

    Args:
        sql: Valid Spark SQL query against Unity Catalog tables
    Returns:
        Query results as formatted string
    """
    from databricks import sql as dbsql
    from databricks.sdk import WorkspaceClient
    ws = WorkspaceClient()
    conn = dbsql.connect(
        server_hostname=ws.config.host,
        http_path="/sql/1.0/warehouses/<warehouse-id>",
        credentials_provider=lambda: {"Authorization": f"Bearer {ws.config.token}"}
    )
    cursor = conn.cursor()
    cursor.execute(sql)
    rows = cursor.fetchall()
    cols = [d[0] for d in cursor.description]
    cursor.close(); conn.close()
    return "\n".join([str(dict(zip(cols, r))) for r in rows])

# Initialize model
llm = ChatDatabricks(
    endpoint="databricks-meta-llama-3-70b-instruct",
    temperature=0.1,
    max_tokens=2000
)

# Create agent
tools = [query_catalog]
agent = create_react_agent(
    llm,
    tools,
    state_modifier="You are a data analyst assistant. Use query_catalog to answer data questions. Always explain your SQL logic."
)

# REQUIRED: set_model so MLflow knows what to log
mlflow.models.set_model(agent)
```

### Step 4: Log and Register Agent with MLflow

See `references/bundle-deployment.md` for the full driver script pattern.

```python
import os
import mlflow

mlflow.set_registry_uri("databricks-uc")
mlflow.set_tracking_uri("databricks")

UC_MODEL_NAME = os.environ.get("UC_MODEL_NAME", "catalog.schema.agent_name")
AGENT_CODE_PATH = "./src/agent.py"
ENDPOINT_NAME = os.environ.get("ENDPOINT_NAME", "my-agent-endpoint")

input_example = {
    "messages": [{"role": "user", "content": "How many orders were placed last month?"}]
}

# Log agent as MLflow model
with mlflow.start_run():
    logged_info = mlflow.langchain.log_model(
        lc_model=AGENT_CODE_PATH,
        artifact_path="agent",
        input_example=input_example,
        example_no_conversion=True,
        pip_requirements=[
            "databricks-langchain",
            "langgraph",
            "mlflow",
            "databricks-agents",
        ]
    )
    print(f"Logged model URI: {logged_info.model_uri}")

# Register in Unity Catalog
model_version = mlflow.register_model(
    model_uri=logged_info.model_uri,
    name=UC_MODEL_NAME
)
print(f"Registered: {UC_MODEL_NAME} version {model_version.version}")

# Deploy to Model Serving endpoint
from databricks import agents

deployment = agents.deploy(
    model_name=UC_MODEL_NAME,
    model_version=model_version.version,
    endpoint_name=ENDPOINT_NAME,
    scale_to_zero=True,       # reduces cost when idle; ~15 min cold start
    workload_size="Small",    # Small | Medium | Large
    deploy_feedback_model=True  # enables the Review App for stakeholder feedback
)
print(f"Endpoint URL:  {deployment.query_endpoint}")
print(f"Review App:    {deployment.review_app_url}")
```

> **Note:** `agents.deploy()` requires `databricks-agents >= 1.1.0` when running outside a Databricks notebook.
> Deployment takes up to 15 minutes — do not set job timeout below 20 minutes.

**Check deployment status:**
```python
from databricks.agents import get_deployments

for d in get_deployments(model_name=UC_MODEL_NAME):
    print(f"Version {d.model_version}: {d.endpoint_url} — {d.state}")
```

### Step 5: Deploy via Asset Bundle Job

**`databricks.yml`** (main config):
```yaml
bundle:
  name: my-agent-bundle

include:
  - resources/*.yml

variables:
  catalog:
    default: "dev_catalog"
  schema:
    default: "dev_schema"
  endpoint_name:
    default: "my-agent-endpoint-dev"
  model_name:
    default: "my_agent"

targets:
  dev:
    default: true
    mode: development
    workspace:
      profile: dev-profile
    variables:
      catalog: "dev_catalog"
      schema: "dev_schema"
      endpoint_name: "my-agent-endpoint-dev"

  prod:
    mode: production
    workspace:
      profile: prod-profile
    variables:
      catalog: "prod_catalog"
      schema: "prod_schema"
      endpoint_name: "my-agent-endpoint"
```

**`resources/deploy_job.yml`** (deployment job):
```yaml
resources:
  jobs:
    deploy_agent:
      name: "[${bundle.target}] Deploy Agent - ${var.model_name}"
      tasks:
        - task_key: log_register_deploy
          spark_python_task:
            python_file: ./src/deploy_agent.py
          libraries:
            - pypi:
                package: "databricks-langchain"
            - pypi:
                package: "langgraph"
            - pypi:
                package: "mlflow>=3.1.3"
            - pypi:
                package: "databricks-agents>=1.1.0"
          new_cluster:
            spark_version: "15.4.x-cpu-ml-scala2.12"
            node_type_id: "i3.xlarge"
            num_workers: 1
            spark_env_vars:
              UC_MODEL_NAME: "${var.catalog}.${var.schema}.${var.model_name}"
              ENDPOINT_NAME: "${var.endpoint_name}"
      permissions:
        - level: CAN_MANAGE_RUN
          group_name: "users"
      timeout_seconds: 1800  # 30 min — deployment takes up to 15 min
```

**Deploy and run:**
```bash
# Validate
databricks bundle validate -t dev

# Deploy bundle (creates job)
databricks bundle deploy -t dev

# Run the deployment job (logs, registers, and creates endpoint)
databricks bundle run deploy_agent -t dev

# Deploy to prod
databricks bundle deploy -t prod
databricks bundle run deploy_agent -t prod
```

**Query the deployed endpoint:**
```bash
curl -X POST "https://<workspace>.azuredatabricks.net/serving-endpoints/my-agent-endpoint/invocations" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "How many orders last month?"}]}'
```

---

## Deployment Path 2: Databricks Apps

Use Databricks Apps when you need rapid iteration, local debugging, a built-in chat UI, or Git-based CI/CD. The agent runs as an async FastAPI server using MLflow's `@invoke`/`@stream` decorators.

### Project Structure

```
my-agent-app/
├── agent_server/
│   ├── __init__.py
│   └── agent.py          # @invoke / @stream handlers
├── app.yaml              # Databricks Apps config (start command + env)
├── databricks.yml        # Bundle config (app resource + permissions)
├── pyproject.toml        # Python dependencies
└── .env                  # Local dev env vars (never commit)
```

### Option A — LangGraph Agent for Apps

**`agent_server/agent.py`:**
```python
import mlflow
from typing import AsyncGenerator
from databricks_langchain import ChatDatabricks
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent
from mlflow.genai.agent_server import invoke, stream
from mlflow.types.responses import (
    ResponsesAgentRequest,
    ResponsesAgentResponse,
    ResponsesAgentStreamEvent,
)

mlflow.langchain.autolog()

@tool
def query_catalog(sql: str) -> str:
    """Execute a SQL query against Unity Catalog tables."""
    # ... same implementation as Model Serving path
    pass

llm = ChatDatabricks(endpoint="databricks-meta-llama-3-70b-instruct", temperature=0.1)
agent_graph = create_react_agent(
    llm,
    [query_catalog],
    state_modifier="You are a helpful data analyst assistant."
)

@invoke()
async def invoke_handler(request: ResponsesAgentRequest) -> ResponsesAgentResponse:
    messages = [{"role": m.role, "content": m.content} for m in request.input]
    result = await agent_graph.ainvoke({"messages": messages})
    final_text = result["messages"][-1].content
    return ResponsesAgentResponse(output=[{
        "type": "message",
        "role": "assistant",
        "content": [{"type": "output_text", "text": final_text}]
    }])

@stream()
async def stream_handler(
    request: ResponsesAgentRequest,
) -> AsyncGenerator[ResponsesAgentStreamEvent, None]:
    messages = [{"role": m.role, "content": m.content} for m in request.input]
    item_id = "langgraph_msg"
    full_text = ""

    # astream_events v2 emits per-token chunks from the LLM
    async for event in agent_graph.astream_events({"messages": messages}, version="v2"):
        if event["event"] == "on_chat_model_stream":
            delta = event["data"]["chunk"].content
            if delta and isinstance(delta, str):
                full_text += delta
                yield ResponsesAgentStreamEvent(
                    type="response.output_text.delta",
                    item_id=item_id,
                    content_index=0,
                    delta=delta,
                )

    # Required final event to signal completion
    yield ResponsesAgentStreamEvent(
        type="response.output_item.done",
        item_id=item_id,
        item={
            "type": "message",
            "role": "assistant",
            "content": [{"type": "output_text", "text": full_text}],
        },
    )
```

### Option B — OpenAI Agents SDK for Apps

The official Databricks app-templates use the OpenAI Agents SDK, which has native async support and integrates with Databricks MCP servers.

**`agent_server/agent.py`:**
```python
import mlflow
from datetime import datetime
from typing import AsyncGenerator

import litellm
from agents import Agent, Runner, function_tool, set_default_openai_api, set_default_openai_client
from agents.tracing import set_trace_processors
from databricks_openai import AsyncDatabricksOpenAI
from mlflow.genai.agent_server import invoke, stream
from mlflow.types.responses import (
    ResponsesAgentRequest,
    ResponsesAgentResponse,
    ResponsesAgentStreamEvent,
)

# Use Databricks-hosted models via the OpenAI-compatible API
set_default_openai_client(AsyncDatabricksOpenAI())
set_default_openai_api("chat_completions")
set_trace_processors([])  # use MLflow only for tracing
mlflow.openai.autolog()
litellm.suppress_debug_info = True

@function_tool
def get_current_time() -> str:
    """Get the current date and time."""
    return datetime.now().isoformat()

def create_agent() -> Agent:
    return Agent(
        name="DataAssistant",
        instructions="You are a helpful data analyst assistant.",
        model="databricks-meta-llama-3-70b-instruct",
        tools=[get_current_time],
    )

@invoke()
async def invoke_handler(request: ResponsesAgentRequest) -> ResponsesAgentResponse:
    agent = create_agent()
    messages = [i.model_dump() for i in request.input]
    result = await Runner.run(agent, messages)
    return ResponsesAgentResponse(output=[item.to_input_item() for item in result.new_items])

@stream()
async def stream_handler(
    request: ResponsesAgentRequest,
) -> AsyncGenerator[ResponsesAgentStreamEvent, None]:
    agent = create_agent()
    messages = [i.model_dump() for i in request.input]
    result = Runner.run_streamed(agent, input=messages)
    item_id = "oai_msg"
    full_text = ""

    async for event in result.stream_events():
        if hasattr(event, "delta") and event.delta:
            full_text += event.delta
            yield ResponsesAgentStreamEvent(
                type="response.output_text.delta",
                item_id=item_id,
                content_index=0,
                delta=event.delta,
            )

    yield ResponsesAgentStreamEvent(
        type="response.output_item.done",
        item_id=item_id,
        item={
            "type": "message",
            "role": "assistant",
            "content": [{"type": "output_text", "text": full_text}],
        },
    )
```

> **Tip:** For Databricks UC function tools via MCP, use `databricks_openai.agents.McpServer` inside an async context manager — see the official template for the full pattern.

### Configuration Files

**`app.yaml`:**
```yaml
command: ["uv", "run", "start-app"]

env:
  - name: MLFLOW_TRACKING_URI
    value: "databricks"
  - name: MLFLOW_REGISTRY_URI
    value: "databricks-uc"
  - name: API_PROXY
    value: "http://localhost:8000/invocations"
  - name: CHAT_APP_PORT
    value: "3000"
  - name: CHAT_PROXY_TIMEOUT_SECONDS
    value: "300"
  - name: MLFLOW_EXPERIMENT_ID
    valueFrom: "experiment"
```

**`databricks.yml`:**
```yaml
bundle:
  name: my-agent-app

resources:
  apps:
    my_agent_app:
      name: "my-agent-app"
      description: "LangGraph data analyst agent"
      source_code_path: ./
      config:
        command: ["uv", "run", "start-app"]
        env:
          - name: MLFLOW_TRACKING_URI
            value: "databricks"
          - name: MLFLOW_REGISTRY_URI
            value: "databricks-uc"
          - name: MLFLOW_EXPERIMENT_ID
            value_from: "experiment"

      resources:
        - name: "experiment"
          experiment:
            experiment_id: ""       # fill in after creating the experiment
            permission: "CAN_MANAGE"
        # Add other resources as needed — see Resource Permission Mapping below

targets:
  dev:
    mode: development
    default: true
  prod:
    mode: production
    resources:
      apps:
        my_agent_app:
          name: "my-agent-app-prod"
```

**`pyproject.toml`** (LangGraph variant):
```toml
[project]
name = "my-agent-app"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.129.0",
    "uvicorn>=0.41.0",
    "databricks-langchain>=0.4.0",
    "databricks-agents>=1.9.0",
    "langgraph>=0.2.0",
    "mlflow>=3.1.3",
    "python-dotenv>=1.2.1",
]

[project.scripts]
start-app = "scripts.start_app:main"
start-server = "agent_server.start_server:main"
agent-evaluate = "agent_server.evaluate_agent:evaluate"
```

### Resource Permission Mapping

When migrating from Model Serving or adding resources, map them in `databricks.yml`:

| MLmodel Resource Type | `databricks.yml` Equivalent | Permission |
|---|---|---|
| `serving_endpoint` | `serving_endpoint` | `CAN_QUERY` |
| `lakebase` (Lakebase Provisioned) | `database` | `CAN_CONNECT_AND_CREATE` |
| `vector_search_index` | `uc_securable` (type: TABLE) | `SELECT` |
| `function` (UC function) | `uc_securable` (type: FUNCTION) | `EXECUTE` |
| `table` | `uc_securable` (type: TABLE) | `SELECT` / `MODIFY` |
| `sql_warehouse` | `sql_warehouse` | `CAN_USE` |

Example with multiple resources:
```yaml
resources:
  - name: "experiment"
    experiment:
      experiment_id: "..."
      permission: "CAN_MANAGE"
  - name: "vector-index"
    uc_securable:
      securable_type: "TABLE"
      securable_full_name: "catalog.schema.my_vector_index"
      permission: "SELECT"
  - name: "warehouse"
    sql_warehouse:
      id: "<warehouse-id>"
      permission: "CAN_USE"
```

### Local Development

```bash
# Install uv if needed
curl -LsSf https://astral.sh/uv/install.sh | sh

# Clone official template (optional, for reference)
git clone https://github.com/databricks/app-templates.git

# Install dependencies
uv sync

# Set local env vars
cat > .env << EOF
DATABRICKS_HOST=https://<workspace>.azuredatabricks.net
DATABRICKS_TOKEN=<your-pat>
MLFLOW_TRACKING_URI=databricks
EOF

# Start the agent server locally
uv run start-app
# Agent available at http://localhost:8000
# Chat UI at http://localhost:3000

# Run evaluation
uv run agent-evaluate
```

Test locally with curl:
```bash
curl -X POST http://localhost:8000/invocations \
  -H "Content-Type: application/json" \
  -d '{"input": [{"role": "user", "content": "What time is it?"}], "stream": false}'
```

### Deploy to Databricks Apps

**Option A — via Databricks CLI:**
```bash
# Create the app (first time only)
databricks apps create my-agent-app

# Sync code to workspace
DATABRICKS_USERNAME=$(databricks current-user me | jq -r .userName)
databricks sync . "/Users/$DATABRICKS_USERNAME/my-agent-app"

# Deploy
databricks apps deploy my-agent-app \
  --source-code-path "/Workspace/Users/$DATABRICKS_USERNAME/my-agent-app"
```

**Option B — via Databricks Asset Bundles (recommended for CI/CD):**
```bash
databricks bundle validate
databricks bundle deploy -t dev
databricks bundle run my_agent_app -t dev

# Promote to prod
databricks bundle deploy -t prod
databricks bundle run my_agent_app -t prod
```

**Query the deployed app (OAuth only — PAT not supported):**
```bash
databricks auth login --host https://<workspace>.azuredatabricks.net
TOKEN=$(databricks auth token | jq -r .token)

curl -X POST "<app-url>/invocations" \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"input": [{"role": "user", "content": "How many orders last month?"}], "stream": true}'
```

### Migrate an Existing Model Serving Agent to Apps

1. **Download your model artifacts:**
```bash
DATABRICKS_CONFIG_PROFILE=<profile> uv run --no-project \
  --with "mlflow[databricks]>=3.1.3" \
  python3 -c "
import mlflow
mlflow.set_tracking_uri('databricks')
mlflow.artifacts.download_artifacts(
    artifact_uri='models:/<model-name>/<version>',
    dst_path='./original_mlflow_model'
)"
```

2. **Convert `predict()` → `@invoke()` / `@stream()`:**

| Model Serving | Databricks Apps |
|---|---|
| `class MyAgent(ResponsesAgent):` | standalone `@invoke()` / `@stream()` functions |
| `def predict(self, request):` | `async def invoke_handler(request):` |
| `def predict_stream(self, request):` | `async def stream_handler(request):` |
| `mlflow.models.set_model(agent)` | not needed |

3. **Map MLflow resources to `databricks.yml`** — see Resource Permission Mapping table above.

4. **Test locally** with `uv run start-app`, then deploy.

---

## Shared Steps (apply to both deployment paths)

### Add Unity Catalog Functions as Tools

For reusable, governed tools stored in Unity Catalog. See `references/uc-tools.md` for full setup including SQL function definitions and access grants.

```python
from databricks_langchain import UCFunctionToolkit
from databricks.sdk import WorkspaceClient

ws = WorkspaceClient()

toolkit = UCFunctionToolkit(
    warehouse_id="<warehouse-id>",
    client=ws
)
uc_tools = toolkit.get_tools(
    tool_names=["catalog.schema.calculate_metrics", "catalog.schema.get_customer_info"]
)

agent = create_react_agent(llm, uc_tools)
```

### Add Vector Search (RAG)

```python
from databricks_langchain import DatabricksVectorSearch
from langchain.tools.retriever import create_retriever_tool

vs = DatabricksVectorSearch(
    index_name="catalog.schema.my_vector_index"
)
retriever = vs.as_retriever(search_kwargs={"k": 5})

search_tool = create_retriever_tool(
    retriever,
    "search_knowledge_base",
    "Search internal documentation and knowledge base. Use for policy questions, product info, and unstructured content."
)

agent = create_react_agent(llm, [search_tool, query_catalog])
```

### Persistent Memory with Lakebase

Model Serving endpoints and Databricks Apps are stateless — LangGraph state does not persist across invocations. Use **Lakebase Provisioned** (managed PostgreSQL on Databricks) as the external store for agent memory, chat history, and LangGraph checkpoints.

#### Create Lakebase Instance

If the `databricks` MCP server is active, create the instance directly from the IDE:

```
Tool: create_or_update_lakebase_database
Input: {
  "type": "provisioned",
  "name": "my-agent-memory",
  "capacity": "CU_1",
  "stopped": false
}
```

Or via SDK:

```python
from databricks.sdk import WorkspaceClient

w = WorkspaceClient()
instance = w.database.create_database_instance(
    name="my-agent-memory",
    capacity="CU_1",
    stopped=False
)
print(f"Endpoint: {instance.read_write_dns}")
```

#### Install Memory Dependencies

```bash
pip install "databricks-langchain[memory]" "psycopg[binary]>=3.0"
```

#### LangGraph Checkpointer (Stateful Agent)

```python
import mlflow
from databricks_langchain import ChatDatabricks
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.postgres import PostgresSaver
from databricks.sdk import WorkspaceClient
import uuid

mlflow.langchain.autolog()

w = WorkspaceClient()
instance = w.database.get_database_instance(name="my-agent-memory")
cred = w.database.generate_database_credential(
    request_id=str(uuid.uuid4()),
    instance_names=["my-agent-memory"]
)

conn_string = (
    f"host={instance.read_write_dns} "
    f"dbname=postgres "
    f"user={w.current_user.me().user_name} "
    f"password={cred.token} "
    f"sslmode=require"
)

checkpointer = PostgresSaver.from_conn_string(conn_string)
checkpointer.setup()  # Creates checkpoint tables on first run

llm = ChatDatabricks(endpoint="databricks-meta-llama-3-70b-instruct")
agent = create_react_agent(llm, tools, checkpointer=checkpointer)

# Invoke with thread_id to maintain state across calls
config = {"configurable": {"thread_id": "user-session-123"}}
result = agent.invoke({"messages": [{"role": "user", "content": "Hello"}]}, config)

mlflow.models.set_model(agent)
```

#### Declare Lakebase as MLflow Resource (Model Serving only)

```python
from mlflow.models.resources import DatabricksLakebase

with mlflow.start_run():
    logged_info = mlflow.langchain.log_model(
        lc_model="./src/agent.py",
        artifact_path="agent",
        input_example=input_example,
        example_no_conversion=True,
        resources=[
            DatabricksLakebase(database_instance_name="my-agent-memory")
        ],
        pip_requirements=[
            "databricks-langchain[memory]",
            "langgraph",
            "mlflow",
            "databricks-agents",
            "psycopg[binary]>=3.0",
        ]
    )
```

For Databricks Apps, add the database resource to `databricks.yml`:
```yaml
resources:
  - name: "agent-memory"
    database:
      name: "my-agent-memory"
      permission: "CAN_CONNECT_AND_CREATE"
```

#### Check Instance Status via MCP

```
Tool: get_lakebase_database
Input: { "type": "provisioned", "name": "my-agent-memory" }
```

---

## MLflow Tracing

MLflow auto-tracing captures every agent step. View traces in the Databricks UI:
- **Experiments** → select your experiment → **Traces** tab
- Each trace shows: input, tool calls, intermediate steps, output, latency

```python
# Enable auto-tracing (add to agent.py)
mlflow.langchain.autolog()

# Or manually set experiment
mlflow.set_experiment("/Shared/my-agent-experiment")
```

---

## Common Issues

**`ModuleNotFoundError` on serving endpoint**
- Add all dependencies to `pip_requirements` in `log_model()`
- Pin versions: `databricks-langchain==0.3.0`

**MLflow tracing not working from Git folder**
- Set experiment before deployment:
  ```python
  mlflow.set_experiment("/Shared/my-agent-experiment")  # non-Git path
  ```

**UC function tool not found**
- Verify the function exists: `SELECT * FROM system.information_schema.routines WHERE routine_name = 'my_function'`
- Check warehouse has access to the catalog

**Deployment job times out (Model Serving)**
- `agents.deploy()` takes ~15 min — set job timeout to at least 30 minutes (`timeout_seconds: 1800`)
- Poll status programmatically:
  ```python
  from databricks.agents import get_deployments
  for d in get_deployments(model_name=UC_MODEL_NAME):
      print(f"Version {d.model_version}: {d.state}")
  ```
- If the `databricks` MCP is active: `Tool: get_serving_endpoint_status`

**`agents.deploy()` fails with version error**
- Requires `databricks-agents >= 1.1.0` when running outside a Databricks notebook
- Requires `mlflow >= 3.1.3`

**LangGraph state not persisting across invocations**
- Use Lakebase Provisioned as PostgreSQL checkpointer — see Persistent Memory section
- For Model Serving: declare `DatabricksLakebase` as MLflow resource for automatic credential provisioning
- For Databricks Apps: add `database` resource to `databricks.yml`

**Databricks Apps — OAuth token required**
- Apps do not accept PATs for querying — use OAuth: `databricks auth login` then `databricks auth token`

**Databricks Apps — Review App not available**
- MLflow Review App is not supported for Databricks Apps deployments
- Use labeling sessions on existing traces for evaluation instead

**Databricks Apps — `@stream` handler not returning responses**
- Must yield a `response.output_item.done` event as the final event — without it the stream never closes
- Ensure `item_id` is consistent between delta and done events
