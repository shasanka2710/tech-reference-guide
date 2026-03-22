# 🔵 Google Agent Development Kit (ADK)

> **Principal Engineer Reference** — Architecture, internals, production patterns, and hands-on guidance for Google's open-source Agent Development Kit.

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [What is Google ADK?](#1-what-is-google-adk) |
| 2 | [Architecture Overview](#2-architecture-overview) |
| 3 | [Core Concepts](#3-core-concepts) |
| 4 | [Agent Types & Composition](#4-agent-types--composition) |
| 5 | [Tool System](#5-tool-system) |
| 6 | [Session & State Management](#6-session--state-management) |
| 7 | [Multi-Agent with A2A Protocol](#7-multi-agent-with-a2a-protocol) |
| 8 | [Streaming & Live Agents](#8-streaming--live-agents) |
| 9 | [Deployment: Vertex AI Agent Engine](#9-deployment-vertex-ai-agent-engine) |
| 10 | [Observability & Evaluation](#10-observability--evaluation) |
| 11 | [Principal Engineer Patterns](#11-principal-engineer-patterns) |
| 12 | [Quick Reference](#12-quick-reference) |

---

## 1. What is Google ADK?

Google ADK (Agent Development Kit) is Google's **open-source Python framework** for building, composing, evaluating, and deploying AI agents. Released in 2025, it is designed to be:

- **Model-agnostic** — works with Gemini, GPT-4o, Claude, and any Vertex AI model
- **Deployment-agnostic** — run locally, on Cloud Run, or on Vertex AI Agent Engine
- **Framework-agnostic** — integrates with LangGraph, CrewAI via the A2A protocol
- **Production-grade** — built-in session management, streaming, evaluation, and tracing

```
google-adk (core)
├── Agents           — LlmAgent, SequentialAgent, ParallelAgent, LoopAgent
├── Tools            — FunctionTool, built-in tools, MCP tools, OpenAPI tools
├── Sessions         — InMemorySessionService, VertexAI session service
├── Runners          — InProcessRunner, VertexAI runner
├── Evaluation       — AgentEvaluator, eval datasets
└── CLI              — adk web, adk run, adk eval, adk deploy
```

---

## 2. Architecture Overview

### High-Level Architecture

```mermaid
flowchart TD
    subgraph Client["Client"]
        SDK["Python SDK\n/ REST Client\n/ Web UI"]
    end

    subgraph ADK_Runtime["Google ADK Runtime"]
        RUNNER["Runner\n(orchestrates the agent loop)"]
        
        subgraph Agent["Agent"]
            LLM_AGENT["LlmAgent\n(reasoning brain)"]
            INSTR["Instruction\n(system prompt template)"]
            TOOLS_REG["Tool Registry"]
        end

        subgraph Session["Session Service"]
            STATE["State\n(mutable dict)"]
            EVENTS["Event History\n(immutable log)"]
            ARTIFACTS["Artifacts\n(files / blobs)"]
        end

        SAFETY["Safety / Guardrails\n(before/after callbacks)"]
    end

    subgraph Model["Model Layer"]
        GEMINI["Gemini\n(default)"]
        VERTEX["Vertex AI\n(any model)"]
        LITELLM["LiteLLM\n(GPT, Claude, etc.)"]
    end

    subgraph Tools_Ext["Tools"]
        FUNC["Python Functions"]
        MCP_T["MCP Servers"]
        OPENAPI["OpenAPI / REST"]
        BUILTIN["Built-in\n(google search, code exec)"]
        AGENT_T["Sub-Agents as Tools"]
    end

    subgraph Deploy["Deployment"]
        LOCAL["Local\n(adk run / adk web)"]
        CLOUDRUN["Cloud Run"]
        VERTEX_ENG["Vertex AI\nAgent Engine"]
    end

    Client --> RUNNER
    RUNNER --> Agent
    RUNNER <--> Session
    LLM_AGENT --> Model
    LLM_AGENT --> SAFETY
    TOOLS_REG --> Tools_Ext
    RUNNER --> Deploy

    style ADK_Runtime fill:#EBF5FB
    style Agent fill:#FEF9E7
    style Session fill:#EAFAF1
    style Tools_Ext fill:#FDEDEC
```

### Request Lifecycle

```mermaid
sequenceDiagram
    participant U as User / App
    participant R as Runner
    participant S as SessionService
    participant A as LlmAgent
    participant M as Gemini / LLM
    participant T as Tool

    U->>R: runner.run(user_message, session_id)
    R->>S: get_or_create session
    S-->>R: session (state + history)
    
    loop Agent Loop
        R->>A: run_step(context)
        A->>A: build prompt (instructions + history + state)
        A->>M: generate(prompt + tool_definitions)
        
        alt LLM returns tool_call
            M-->>A: tool_call { name, args }
            A->>T: invoke tool(args)
            T-->>A: tool_result
            A->>S: append tool event to history
        else LLM returns final answer
            M-->>A: text response
            A->>S: append response event
            A-->>R: final_response (stop loop)
        end
    end

    R-->>U: stream of events / final response
```

---

## 3. Core Concepts

### ADK Concept Map

```mermaid
mindmap
  root((Google ADK))
    Agent
      LlmAgent
        instruction
        model
        tools
        sub_agents
        callbacks
      SequentialAgent
        Runs sub-agents in order
        Passes state between them
      ParallelAgent
        Runs sub-agents concurrently
        Merges results into state
      LoopAgent
        Repeats sub-agent
        Until exit condition
    Tool
      FunctionTool
        Any Python function
        Auto schema extraction
      Built-in Tools
        google_search
        code_execution
        vertex_ai_search
      MCP Tool
        Any MCP server
      OpenAPITool
        REST API from spec
      AgentTool
        Another agent as tool
    Session
      session_id
      user_id
      state dict
        read/write by agents
      events list
        immutable audit log
      artifacts
        files and blobs
    Runner
      InProcessRunner
        For local dev and testing
      VertexAiRunner
        Managed cloud execution
    Callbacks
      before_model_call
      after_model_call
      before_tool_call
      after_tool_call
      on_agent_start
      on_agent_end
```

### Event System

ADK models all agent activity as an ordered stream of **events**. This is key to traceability and streaming.

```mermaid
flowchart LR
    subgraph Events["Event Types"]
        E1["UserMessageEvent\n(user input)"]
        E2["ModelResponseEvent\n(LLM output text)"]
        E3["ToolCallEvent\n(LLM requests tool)"]
        E4["ToolResultEvent\n(tool output)"]
        E5["StateUpdateEvent\n(state mutation)"]
        E6["AgentTransferEvent\n(hand-off to sub-agent)"]
        E7["FinalResponseEvent\n(terminal event)"]
    end

    E1 --> E3 --> E4 --> E2 --> E7
    E2 -.->|"intermediate"| E3

    style E7 fill:#27AE60,color:#fff
    style E1 fill:#3498DB,color:#fff
```

---

## 4. Agent Types & Composition

### Four Native Agent Types

```mermaid
flowchart TD
    subgraph LLM["LlmAgent — The Reasoning Core"]
        direction TB
        LA["LlmAgent\n• Uses an LLM to reason\n• Selects and calls tools\n• Can delegate to sub_agents\n• The most common agent type"]
    end

    subgraph SEQ["SequentialAgent — Pipeline"]
        direction LR
        S1["Sub-Agent 1"] --> S2["Sub-Agent 2"] --> S3["Sub-Agent 3"]
        SN["Runs sub-agents left to right\nEach can read/write shared state\nDeterministic, auditable"]
    end

    subgraph PAR["ParallelAgent — Fan-out/Fan-in"]
        direction TB
        P0["Input State"] --> PA1["Sub-Agent A"]
        P0 --> PA2["Sub-Agent B"]
        P0 --> PA3["Sub-Agent C"]
        PA1 & PA2 & PA3 --> MERGE["Merged State\n(all writes reconciled)"]
    end

    subgraph LOOP["LoopAgent — Iterative Refinement"]
        direction LR
        L1["Sub-Agent"] --> CHECK{"exit condition\nin state?"}
        CHECK -->|"No"| L1
        CHECK -->|"Yes"| DONE["Done"]
    end
```

### Composition Example: Research Pipeline

```python
from google.adk.agents import LlmAgent, SequentialAgent, ParallelAgent

# Leaf agents
researcher = LlmAgent(
    name="researcher",
    model="gemini-2.0-flash",
    instruction="Research the given topic and populate state['research_notes'].",
    tools=[google_search_tool],
)

analyst = LlmAgent(
    name="analyst",
    model="gemini-2.0-flash",
    instruction="Analyse state['research_notes'] and write insights to state['analysis'].",
)

writer = LlmAgent(
    name="writer",
    model="gemini-2.0-pro",
    instruction="Write an executive report from state['analysis']. Save to state['report'].",
)

# Compose into a pipeline
pipeline = SequentialAgent(
    name="research_pipeline",
    sub_agents=[researcher, analyst, writer],
)
```

---

## 5. Tool System

### Tool Architecture

```mermaid
flowchart TD
    subgraph ToolTypes["Tool Types in Google ADK"]
        FT["FunctionTool\n• Python function → tool\n• Type hints → JSON schema\n• Docstring → description"]
        BI["Built-in Tools\n• google_search()\n• code_execution()\n• vertex_ai_search()"]
        MCPT["MCPTool\n• Connect any MCP server\n• stdio or SSE transport\n• Huge ecosystem"]
        OAT["OpenAPITool\n• Generate tools from\n  OpenAPI spec\n• Auto-maps endpoints"]
        AT["AgentTool\n• Wrap another agent\n  as a callable tool\n• Enables composability"]
    end

    subgraph ToolSystem["Tool System"]
        TR["Tool Registry\n(per-agent)"]
        SC["Schema Extraction\n(JSON Schema)"]
        INV["Tool Invocation\n(with auth + retry)"]
        CB["before/after_tool_call\ncallbacks"]
    end

    FT & BI & MCPT & OAT & AT --> TR
    TR --> SC --> INV --> CB
```

### Defining a Function Tool

```python
from google.adk.tools import FunctionTool

def get_weather(city: str, units: str = "celsius") -> dict:
    """
    Get current weather for a city.

    Args:
        city: The city name (e.g., "London", "Tokyo")
        units: Temperature units: "celsius" or "fahrenheit"

    Returns:
        dict with temperature, condition, humidity
    """
    # implementation
    return {"city": city, "temperature": 22, "condition": "Sunny", "humidity": 55}

weather_tool = FunctionTool(func=get_weather)
# ADK auto-extracts the JSON schema from type hints and docstring
```

### Connecting an MCP Server

```python
from google.adk.tools.mcp_tool import MCPToolset, StdioServerParameters

# Connect to any MCP-compatible tool server
mcp_tools = MCPToolset(
    connection_params=StdioServerParameters(
        command="npx",
        args=["-y", "@modelcontextprotocol/server-filesystem", "/workspace"],
    )
)

# Use in an agent
file_agent = LlmAgent(
    name="file_agent",
    model="gemini-2.0-flash",
    instruction="You can read and write files in the workspace.",
    tools=[mcp_tools],
)
```

---

## 6. Session & State Management

### State Scope Prefixes

Google ADK has a powerful **scoped state** system using key prefixes:

```
state["key"]              → Session scope   (current session only)
state["user:key"]         → User scope      (persists across user's sessions)
state["app:key"]          → App scope       (shared across all users)
state["temp:key"]         → Temp scope      (this agent step only, not persisted)
```

### State Flow in Multi-Agent Systems

```mermaid
flowchart LR
    subgraph SessionState["Session State (shared dict)"]
        SK["session_key: value"]
        UK["user:profile: {...}"]
        AK["app:config: {...}"]
    end

    A1["Agent 1\nWrites state['research']"] -->|"updates"| SessionState
    SessionState -->|"reads"| A2["Agent 2\nReads state['research']"]
    A2 -->|"updates"| SessionState
    SessionState -->|"reads"| A3["Agent 3\nReads state['research']\nWrites state['report']"]
```

### Session Persistence Options

| Service | Use Case | Persistence |
|---------|----------|------------|
| `InMemorySessionService` | Local dev, unit tests | None — lost on restart |
| `VertexAiSessionService` | Production on GCP | Firestore-backed, durable |
| `DatabaseSessionService` | Self-hosted production | Your own DB |

```python
from google.adk.sessions import InMemorySessionService, VertexAiSessionService

# Development
session_service = InMemorySessionService()

# Production
session_service = VertexAiSessionService(
    project="my-project",
    location="us-central1",
)
```

---

## 7. Multi-Agent with A2A Protocol

### Agent-to-Agent (A2A) Protocol

A2A is a Google-led **open standard** for agent interoperability. It allows agents built on different frameworks (ADK, LangGraph, CrewAI) to communicate via a standard HTTP API.

```mermaid
sequenceDiagram
    participant OA as Orchestrator Agent (ADK)
    participant A2A_S as A2A Server (Agent Card)
    participant WA as Worker Agent (any framework)

    OA->>A2A_S: GET /.well-known/agent.json
    A2A_S-->>OA: Agent Card (capabilities, schema)
    
    OA->>A2A_S: POST /tasks/send { task, context }
    A2A_S->>WA: Execute task
    WA-->>A2A_S: Task result
    A2A_S-->>OA: TaskResult { output, artifacts }
```

### A2A Agent Card

Every A2A-compatible agent exposes a `/.well-known/agent.json` describing itself:

```json
{
  "name": "weather-agent",
  "description": "Fetches real-time weather for any location",
  "url": "https://weather-agent.example.com",
  "version": "1.0.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": false
  },
  "skills": [
    {
      "id": "get-weather",
      "name": "Get Weather",
      "description": "Returns current weather for a city",
      "inputModes": ["text"],
      "outputModes": ["text", "data"]
    }
  ]
}
```

### Calling a Remote A2A Agent

```python
from google.adk.agents import LlmAgent
from google.adk.tools.agent_tool import AgentTool
from google.adk.a2a import A2AClient

# Wrap a remote agent as a local tool
remote_weather_agent = AgentTool(
    agent=A2AClient(agent_url="https://weather-agent.example.com"),
    name="get_weather_from_remote",
    description="Get weather from the remote weather agent service",
)

orchestrator = LlmAgent(
    name="orchestrator",
    model="gemini-2.0-flash",
    instruction="Help users with travel planning using available tools.",
    tools=[remote_weather_agent],
)
```

---

## 8. Streaming & Live Agents

### Streaming Architecture

Google ADK supports **bidirectional real-time streaming** via Gemini's Live API — enabling voice-in/voice-out and live video understanding.

```mermaid
flowchart LR
    subgraph Client["Client"]
        MIC["🎤 Microphone\n/ Audio Stream"]
        CAM["📷 Camera\n/ Video Stream"]
        SPK["🔊 Speaker\n/ Audio Output"]
    end

    subgraph ADK_Live["ADK Live Session"]
        LIVE_API["Gemini Live API\n(bidirectional WebSocket)"]
        VAD["Voice Activity\nDetection"]
        TOOL_EXEC["Tool Execution\n(async)"]
        INTERRUPT["Interruption\nHandling"]
    end

    subgraph Output["Output"]
        TEXT_OUT["📝 Text Stream"]
        AUDIO_OUT["🔊 Audio Stream"]
        TOOL_OUT["🛠️ Tool Results"]
    end

    MIC -->|"audio bytes"| LIVE_API
    CAM -->|"video frames"| LIVE_API
    LIVE_API --> VAD
    LIVE_API --> TOOL_EXEC
    LIVE_API --> INTERRUPT
    LIVE_API --> TEXT_OUT
    LIVE_API --> AUDIO_OUT
    TOOL_EXEC --> TOOL_OUT
    AUDIO_OUT --> SPK
```

### Text Streaming Example

```python
from google.adk.runners import InProcessRunner

runner = InProcessRunner(agent=my_agent, session_service=session_service)

# Streaming — yields events as they arrive
async for event in runner.run_async(
    user_message="Research the latest trends in quantum computing",
    session_id="session-001",
):
    if event.is_text_chunk():
        print(event.text, end="", flush=True)
    elif event.is_tool_call():
        print(f"\n[Tool: {event.tool_name}({event.args})]")
    elif event.is_final():
        print("\n[Done]")
```

---

## 9. Deployment: Vertex AI Agent Engine

### Deployment Architecture

```mermaid
flowchart TD
    subgraph Dev["Development"]
        LOCAL["adk run / adk web\n(local InProcessRunner)"]
    end

    subgraph Staging["Staging"]
        CLOUDRUN["Cloud Run\n(containerised agent)"]
    end

    subgraph Prod["Production"]
        subgraph VAAE["Vertex AI Agent Engine"]
            MANAGED["Managed Runtime\n(auto-scaled)"]
            SESS["Managed Sessions\n(Firestore-backed)"]
            TRACING["Cloud Trace\nintegration"]
            IAM["IAM-based auth"]
        end
    end

    Dev -->|"adk deploy cloud_run"| Staging
    Staging -->|"adk deploy agent_engine"| Prod

    style Dev fill:#FEF9E7
    style Staging fill:#EBF5FB
    style Prod fill:#EAFAF1
```

### Deploying to Agent Engine

```python
from google.adk.deploy import VertexAiAgentEngineDeployer

deployer = VertexAiAgentEngineDeployer(
    project="my-gcp-project",
    location="us-central1",
    agent=my_agent,
    session_service=VertexAiSessionService(...),
    display_name="My Production Agent",
    description="Customer support agent for ACME Corp",
)

# Deploy — returns a managed agent endpoint
endpoint = deployer.deploy()
print(f"Agent deployed at: {endpoint.resource_name}")
```

---

## 10. Observability & Evaluation

### Evaluation Framework

```mermaid
flowchart LR
    subgraph EvalDataset["Eval Dataset (.test.json)"]
        CASE1["test_case_1:\n  input: 'What is..'\n  expected_output: '..'\n  reference_tools: [..]"]
        CASE2["test_case_2: ..."]
    end

    subgraph EvalRun["adk eval"]
        RUN["Run agent against\neach test case"]
        SCORE["Score:\n• Exact match\n• Similarity\n• LLM-as-judge\n• Tool trajectory match"]
        REPORT["HTML / JSON Report\n(pass/fail per case)"]
    end

    EvalDataset --> RUN --> SCORE --> REPORT
```

### Eval Dataset Format

```json
[
  {
    "query": "What is the weather in Tokyo?",
    "expected_tool_use": [
      {
        "tool_name": "get_weather",
        "tool_input": { "city": "Tokyo" }
      }
    ],
    "reference_final_response": "The current weather in Tokyo is..."
  }
]
```

```bash
# Run evaluation
adk eval \
  --agent_module my_agent.agent \
  --eval_set tests/weather_eval.test.json \
  --output_dir reports/
```

---

## 11. Principal Engineer Patterns

### Pattern 1: Hierarchical Agent Delegation

```mermaid
flowchart TD
    ROOT["Root LlmAgent\n(user-facing, routes tasks)"]
    ROOT -->|"complex research"| RESEARCH["Research SubAgent\n(search + synthesis)"]
    ROOT -->|"data analysis"| ANALYSIS["Analysis SubAgent\n(code execution)"]
    ROOT -->|"document creation"| WRITING["Writing SubAgent\n(generation + formatting)"]
    RESEARCH --> ROOT
    ANALYSIS --> ROOT
    WRITING --> ROOT
    ROOT --> USER["Response to User"]
```

```python
research_agent = LlmAgent(name="researcher", ...)
analysis_agent = LlmAgent(name="analyst", ...)
writing_agent  = LlmAgent(name="writer", ...)

# Root agent delegates via sub_agents list
root_agent = LlmAgent(
    name="root",
    model="gemini-2.0-flash",
    instruction="""
    You are a helpful assistant. For complex tasks, delegate:
    - Research tasks → researcher agent
    - Data analysis  → analyst agent
    - Document creation → writer agent
    """,
    sub_agents=[research_agent, analysis_agent, writing_agent],
)
```

### Pattern 2: Guardrail Callbacks

```python
from google.adk.agents import CallbackContext
from google.adk.models import LlmResponse

def safety_check_before_model(ctx: CallbackContext, llm_request) -> LlmResponse | None:
    """Block if user asks about competitor products."""
    user_text = ctx.user_message.text or ""
    blocked_terms = ["competitor_a", "competitor_b"]
    
    if any(term in user_text.lower() for term in blocked_terms):
        return LlmResponse(
            text="I can only assist with questions about our products.",
        )
    return None  # Allow the call to proceed

def cost_guard_before_model(ctx: CallbackContext, llm_request):
    """Limit context window to control cost."""
    # Trim history if too long
    if len(ctx.session.events) > 50:
        llm_request.messages = llm_request.messages[-20:]  # Keep last 20
    return None

my_agent = LlmAgent(
    name="safe_agent",
    model="gemini-2.0-flash",
    instruction="You are a helpful assistant.",
    before_model_callback=safety_check_before_model,
)
```

### Pattern 3: Stateful Workflow with SequentialAgent

```python
from google.adk.agents import SequentialAgent, LlmAgent

# Each agent reads/writes shared state keys
step1 = LlmAgent(
    name="extractor",
    instruction="Extract key facts from the document in state['document'] and write to state['facts'].",
)

step2 = LlmAgent(
    name="classifier",
    instruction="Classify each fact in state['facts'] by category. Write to state['classified_facts'].",
)

step3 = LlmAgent(
    name="summariser",
    instruction="Create an executive summary from state['classified_facts']. Write to state['summary'].",
)

doc_pipeline = SequentialAgent(
    name="document_pipeline",
    sub_agents=[step1, step2, step3],
)

# Set initial state
session = session_service.create_session(
    app_name="doc-processor",
    user_id="user-1",
    initial_state={"document": "Annual report text here..."},
)
```

---

## 12. Quick Reference

```
Install:
  pip install google-adk

Key classes:
  LlmAgent(name, model, instruction, tools, sub_agents, callbacks)
  SequentialAgent(name, sub_agents)
  ParallelAgent(name, sub_agents)
  LoopAgent(name, sub_agents, max_iterations)
  FunctionTool(func)
  InProcessRunner(agent, session_service)
  InMemorySessionService()

CLI commands:
  adk run   <agent_module>          — run agent in terminal
  adk web   <agent_module>          — launch web UI at localhost:8080
  adk eval  --agent_module ... --eval_set ...
  adk deploy cloud_run ...
  adk deploy agent_engine ...

State scopes:
  state["key"]       → session
  state["user:key"]  → user
  state["app:key"]   → app-wide
  state["temp:key"]  → single-step only

Default model: gemini-2.0-flash
Docs:          https://google.github.io/adk-docs
GitHub:        https://github.com/google/adk-python
```
