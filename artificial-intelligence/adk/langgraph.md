# 🟢 LangGraph — Principal Engineer Reference

> **Principal Engineer Reference** — Deep-dive into LangGraph's graph-based state machine model, production architecture patterns, and engineering trade-offs.

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [What is LangGraph?](#1-what-is-langgraph) |
| 2 | [Core Mental Model: Graphs as Agents](#2-core-mental-model-graphs-as-agents) |
| 3 | [Graph Architecture](#3-graph-architecture) |
| 4 | [State Design](#4-state-design) |
| 5 | [Nodes, Edges & Conditionals](#5-nodes-edges--conditionals) |
| 6 | [Human-in-the-Loop](#6-human-in-the-loop) |
| 7 | [Multi-Agent Patterns](#7-multi-agent-patterns) |
| 8 | [Persistence & Checkpoints](#8-persistence--checkpoints) |
| 9 | [Streaming](#9-streaming) |
| 10 | [LangGraph Cloud & Studio](#10-langgraph-cloud--studio) |
| 11 | [Principal Engineer Patterns](#11-principal-engineer-patterns) |
| 12 | [Quick Reference](#12-quick-reference) |

---

## 1. What is LangGraph?

LangGraph is a **graph-based agent orchestration library** built on top of LangChain. It models agent workflows as **directed graphs** (or cyclic graphs) where:

- **Nodes** are units of work (LLM calls, tool invocations, human reviews, Python logic)
- **Edges** define transitions between nodes (conditional or unconditional)
- **State** is a typed Python dict shared across all nodes

```
Why graphs instead of simple chains?
  ✓ Cycles:     agents can loop, retry, and self-correct
  ✓ Branching:  conditional paths based on state
  ✓ Clarity:    the workflow is visualisable and inspectable
  ✓ Interrupts: pause at any node for human review
  ✓ Checkpoints: resume from any point after failure
```

---

## 2. Core Mental Model: Graphs as Agents

```mermaid
mindmap
  root((LangGraph))
    Graph Primitives
      StateGraph
        Stateful directed graph
        Most common
      MessageGraph
        Chat-specialised graph
        Manages message list
      CompiledGraph
        Compiled and validated
        Runnable object
    State
      TypedDict schema
      Annotations for merging
        add_messages
        custom reducers
      Shared across all nodes
    Nodes
      Python function or chain
      Receives full state
      Returns partial state update
      async supported
    Edges
      Normal edge
        Always transitions
      Conditional edge
        Function decides next node
      Entry point
        Where graph starts
      END sentinel
        Terminates graph
    Persistence
      Checkpointer interface
      MemorySaver
        In-memory dev
      SqliteSaver
        Local file
      PostgresSaver
        Production
      RedisSaver
        Fast resumption
    Human-in-the-Loop
      interrupt_before
      interrupt_after
      Command.RESUME
      Approval patterns
    Multi-Agent
      Subgraphs
        Nested graphs
      Supervisor pattern
      Swarm pattern
        Handoff tools
    Observability
      LangSmith tracing
      astream_events
      Studio visual debugger
```

---

## 3. Graph Architecture

### LangGraph Execution Model

```mermaid
flowchart TD
    subgraph Compile["Compile Phase"]
        BUILDER["StateGraph builder\n(add_node, add_edge)"]
        VALIDATE["Schema validation\nand graph validation"]
        COMPILED["CompiledGraph\n(runnable object)"]
        BUILDER --> VALIDATE --> COMPILED
    end

    subgraph Run["Run Phase"]
        INPUT["Input state\n(initial message / data)"]
        SUPER["Superstep\n(execute all ready nodes\nin parallel)"]
        STATE_UPDATE["Merge node outputs\ninto state\n(reducers applied)"]
        NEXT_NODE["Evaluate edges\n→ select next node(s)"]
        CHECK{"More nodes\nto run?"}
        OUTPUT["Final state / stream"]

        INPUT --> SUPER --> STATE_UPDATE --> NEXT_NODE --> CHECK
        CHECK -->|"Yes"| SUPER
        CHECK -->|"No"| OUTPUT
    end

    COMPILED --> Run

    style COMPILED fill:#27AE60,color:#fff
    style OUTPUT fill:#27AE60,color:#fff
```

### ReAct Agent as a Graph

```mermaid
flowchart TD
    START(["START"]) --> AGENT

    AGENT["agent node\n(LLM call)\nDecides: tool call or final answer"]
    
    AGENT --> ROUTE{{"should_continue?\n(conditional edge)"}}
    
    ROUTE -->|"tool_call in response"| TOOLS["tools node\n(execute tool calls)"]
    ROUTE -->|"no tool call (final)"| END(["END"])
    
    TOOLS -->|"tool results added to state"| AGENT

    style START fill:#3498DB,color:#fff
    style END fill:#27AE60,color:#fff
    style AGENT fill:#F39C12,color:#fff
    style TOOLS fill:#E74C3C,color:#fff
```

---

## 4. State Design

State design is the **most important architecture decision** in LangGraph.

### TypedDict State Schema

```python
from typing import Annotated, Sequence
from typing_extensions import TypedDict
from langchain_core.messages import BaseMessage
from langgraph.graph.message import add_messages

# Basic chat state
class AgentState(TypedDict):
    messages: Annotated[list[BaseMessage], add_messages]  # append-only
    current_step: str                                       # last write wins
    research_notes: str                                     # last write wins
    retry_count: int                                        # last write wins

# Complex workflow state
class WorkflowState(TypedDict):
    messages:        Annotated[list[BaseMessage], add_messages]
    task:            str                  # initial input, read-only by convention
    research:        str                  # written by researcher node
    analysis:        str                  # written by analyst node
    draft:           str                  # written by writer node
    review_comments: list[str]            # written by reviewer
    approved:        bool                 # set by human or supervisor
    iteration:       int                  # loop counter
    errors:          Annotated[list[str], lambda x, y: x + y]  # accumulate errors
```

### State Reducers

```mermaid
flowchart LR
    subgraph Reducers["State Reducers"]
        LW["Last-Write-Wins\n(default)\nNew value replaces old"]
        AM["add_messages\nAppends new messages\nto existing list"]
        CUSTOM["Custom reducer\nlambda old, new: ...\nAny merge logic"]
    end

    LW --> EX1["messages: 'hello' → 'world'\nResult: 'world'"]
    AM --> EX2["messages: [msg1] + [msg2]\nResult: [msg1, msg2]"]
    CUSTOM --> EX3["errors: [e1] + [e2]\nResult: [e1, e2]"]
```

---

## 5. Nodes, Edges & Conditionals

### Implementing a Full ReAct Graph

```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import ToolNode
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage

# Tools
tools = [search_tool, calculator_tool]
model = ChatOpenAI(model="gpt-4o").bind_tools(tools)

# Node: call the LLM
def agent_node(state: AgentState) -> dict:
    system = SystemMessage(content="You are a helpful assistant.")
    response = model.invoke([system] + state["messages"])
    return {"messages": [response]}

# Conditional edge: decide next step
def should_continue(state: AgentState) -> str:
    last_message = state["messages"][-1]
    if last_message.tool_calls:
        return "tools"        # go to tool node
    return END                # finish

# Build the graph
graph_builder = StateGraph(AgentState)

graph_builder.add_node("agent", agent_node)
graph_builder.add_node("tools", ToolNode(tools))

graph_builder.set_entry_point("agent")

graph_builder.add_conditional_edges(
    "agent",
    should_continue,
    {"tools": "tools", END: END},
)

graph_builder.add_edge("tools", "agent")  # always loop back

graph = graph_builder.compile()
```

### Node Types

```mermaid
flowchart TD
    subgraph NodeTypes["Node Types"]
        LLM_NODE["LLM Node\nfunction that calls LLM\nand returns state update"]
        TOOL_NODE["ToolNode (prebuilt)\nExecutes all tool calls\nin last message"]
        HUMAN_NODE["Human Node\ninterrupt_before / after\nWaits for external input"]
        LOGIC_NODE["Logic Node\nPure Python\nno LLM call\n(validation, formatting)"]
        SUBGRAPH["Subgraph Node\nAnother compiled graph\nembedded as a node"]
    end
```

---

## 6. Human-in-the-Loop

LangGraph's interrupt mechanism is one of its most powerful features.

### Interrupt Patterns

```mermaid
flowchart TD
    subgraph Pattern1["interrupt_before"]
        P1_RUN["Graph running..."]
        P1_INT["⏸️ Pause BEFORE 'approval' node\nSerialise state → persist"]
        P1_HUMAN["Human reviews state\nand approves or modifies"]
        P1_RES["Command(resume=True)\nGraph continues from checkpoint"]
        P1_RUN --> P1_INT --> P1_HUMAN --> P1_RES
    end

    subgraph Pattern2["interrupt_after"]
        P2_NODE["'draft' node runs and\nproduces draft output"]
        P2_INT["⏸️ Pause AFTER 'draft' node"]
        P2_HUMAN["Human edits draft\nin state['draft']"]
        P2_RES["Command(resume=edited_draft)\nGraph continues with edited value"]
        P2_NODE --> P2_INT --> P2_HUMAN --> P2_RES
    end
```

### Human-in-the-Loop Implementation

```python
from langgraph.types import Command, interrupt

# Option 1: interrupt() inside a node
def review_node(state: WorkflowState) -> dict:
    draft = state["draft"]
    
    # Execution pauses here — state is checkpointed
    human_decision = interrupt({
        "draft": draft,
        "instructions": "Please review the draft. Approve or provide feedback.",
    })
    
    if human_decision["approved"]:
        return {"approved": True}
    else:
        return {"review_comments": [human_decision["feedback"]], "approved": False}

# Compile with interruption points
graph = graph_builder.compile(
    checkpointer=MemorySaver(),
    interrupt_before=["send_email_node"],   # always pause before sending
)

# Resume after human acts
result = graph.invoke(
    Command(resume={"approved": True}),
    config={"configurable": {"thread_id": "thread-001"}},
)
```

---

## 7. Multi-Agent Patterns

### Supervisor Pattern

```mermaid
flowchart TD
    START(["START"]) --> SUPERVISOR

    SUPERVISOR["Supervisor LLM Node\nDecides which worker to call\nand when to finish"]

    SUPERVISOR --> ROUTE{{"next_worker\n(conditional edge)"}}

    ROUTE -->|"researcher"| RESEARCHER["Researcher Node\n(search + summarise)"]
    ROUTE -->|"coder"| CODER["Coder Node\n(write + execute code)"]
    ROUTE -->|"writer"| WRITER["Writer Node\n(draft + format)"]
    ROUTE -->|"FINISH"| END(["END"])

    RESEARCHER --> SUPERVISOR
    CODER --> SUPERVISOR
    WRITER --> SUPERVISOR

    style START fill:#3498DB,color:#fff
    style END fill:#27AE60,color:#fff
    style SUPERVISOR fill:#E74C3C,color:#fff
```

```python
from langgraph_supervisor import create_supervisor  # langgraph-supervisor package

supervisor_graph = create_supervisor(
    agents=[researcher_agent, coder_agent, writer_agent],
    model=ChatOpenAI(model="gpt-4o"),
    prompt="You are a supervisor managing a team of specialists. Route tasks to the right agent.",
)
```

### Swarm Pattern (Agent Handoffs)

```mermaid
flowchart LR
    ENTRY["Entry Agent\n(initial router)"]
    
    ENTRY -->|"transfer_to_billing"| BILLING["Billing Agent\n(handles payment issues)"]
    ENTRY -->|"transfer_to_technical"| TECH["Technical Agent\n(handles tech support)"]
    BILLING -->|"transfer_to_technical"| TECH
    TECH -->|"transfer_to_billing"| BILLING
    BILLING & TECH -->|"resolve"| END(["END"])
```

```python
from langgraph_swarm import create_handoff_tool, create_swarm

# Each agent can hand off to others via tools
billing_agent = create_react_agent(
    model,
    tools=[billing_tools, create_handoff_tool(agent_name="technical_agent")],
    name="billing_agent",
)

technical_agent = create_react_agent(
    model,
    tools=[tech_tools, create_handoff_tool(agent_name="billing_agent")],
    name="technical_agent",
)

swarm = create_swarm([billing_agent, technical_agent], default_active_agent="billing_agent")
```

---

## 8. Persistence & Checkpoints

### Checkpoint Architecture

```mermaid
flowchart LR
    subgraph Graph["LangGraph Execution"]
        N1["Node A"] --> N2["Node B"] --> N3["Node C"]
    end

    subgraph Checkpointer["Checkpointer"]
        CP1["Checkpoint after A\n(state snapshot + metadata)"]
        CP2["Checkpoint after B"]
        CP3["Checkpoint after C"]
    end

    subgraph Storage["Storage Backend"]
        MEM["MemorySaver\n(dict in process)"]
        SQLITE["SqliteSaver\n(local .db file)"]
        PG["PostgresSaver\n(Postgres + asyncpg)"]
        REDIS["RedisSaver\n(fast, ephemeral)"]
    end

    N1 -->|"save"| CP1
    N2 -->|"save"| CP2
    N3 -->|"save"| CP3
    CP1 & CP2 & CP3 --> Storage
```

### Thread-Based State

LangGraph uses **thread IDs** to separate independent conversations:

```python
from langgraph.checkpoint.postgres import PostgresSaver

# Production checkpointer
checkpointer = PostgresSaver.from_conn_string(
    "postgresql://user:pass@localhost/langgraph"
)
checkpointer.setup()  # create tables

graph = graph_builder.compile(checkpointer=checkpointer)

# Thread config identifies which conversation
config = {"configurable": {"thread_id": "user-123-session-456"}}

# Each invoke updates the thread's checkpoint
result = graph.invoke({"messages": [HumanMessage("hello")]}, config=config)

# View current state
current_state = graph.get_state(config)

# View history
for state_snapshot in graph.get_state_history(config):
    print(state_snapshot.created_at, state_snapshot.next)
```

---

## 9. Streaming

LangGraph supports three levels of streaming:

```python
# Level 1: Stream graph outputs (node results)
for chunk in graph.stream(inputs, config, stream_mode="values"):
    print(chunk)  # full state after each step

# Level 2: Stream updates (state diffs)
for chunk in graph.stream(inputs, config, stream_mode="updates"):
    print(chunk)  # only what changed

# Level 3: Stream all events (including LLM tokens)
async for event in graph.astream_events(inputs, config, version="v2"):
    if event["event"] == "on_chat_model_stream":
        print(event["data"]["chunk"].content, end="")
    elif event["event"] == "on_tool_start":
        print(f"\n[Calling tool: {event['name']}]")
```

---

## 10. LangGraph Cloud & Studio

### LangSmith + LangGraph Studio

```mermaid
flowchart LR
    subgraph Dev["Development"]
        CODE["Agent code\n(langgraph_app)"]
        STUDIO["LangGraph Studio\n• Visual graph editor\n• Live state inspection\n• Time-travel debugging\n• Interrupt UI"]
    end

    subgraph Cloud["LangGraph Cloud"]
        API["Managed API\n(REST + streaming)"]
        RUNS["Run management\n(async tasks)"]
        THREADS["Thread management\n(persistent state)"]
        CRONS["Cron jobs\n(scheduled runs)"]
    end

    subgraph Observability["LangSmith"]
        TRACES["Full trace per run"]
        EVALS["Eval datasets\n+ scoring"]
        MONITOR["Production monitoring"]
    end

    CODE --> STUDIO
    CODE --> Cloud
    Cloud --> Observability
    STUDIO --> Observability
```

### Deploying to LangGraph Cloud

```yaml
# langgraph.json — deployment manifest
{
  "dependencies": ["."],
  "graphs": {
    "my_agent": "./my_agent/graph.py:graph"
  },
  "env": ".env"
}
```

```bash
# Deploy to LangGraph Cloud
langgraph build -t my-agent-image
langgraph deploy
```

---

## 11. Principal Engineer Patterns

### Pattern 1: Typed State with Validation

```python
from pydantic import BaseModel, validator
from langgraph.graph import StateGraph

class StrictAgentState(BaseModel):
    """Pydantic model for validated state."""
    messages:      list = []
    task_id:       str
    status:        Literal["pending", "running", "done", "failed"] = "pending"
    output:        Optional[str] = None
    error:         Optional[str] = None
    token_count:   int = 0

    @validator("token_count")
    def token_count_non_negative(cls, v):
        if v < 0:
            raise ValueError("token_count cannot be negative")
        return v

graph = StateGraph(StrictAgentState)
```

### Pattern 2: Circuit Breaker Node

```python
MAX_RETRIES = 3

def circuit_breaker(state: AgentState) -> str:
    """Conditional edge: stop after too many retries."""
    if state.get("retry_count", 0) >= MAX_RETRIES:
        return "error_handler"
    if state.get("errors"):
        return "retry"
    return "continue"

graph_builder.add_conditional_edges(
    "agent",
    circuit_breaker,
    {
        "continue":      "tools",
        "retry":         "agent",
        "error_handler": "error_handler_node",
    }
)
```

### Pattern 3: Observability Wrapper

```python
import time
from langsmith import traceable

@traceable(name="agent-node", tags=["production"])
def agent_node(state: AgentState) -> dict:
    start = time.perf_counter()
    response = model.invoke(state["messages"])
    latency_ms = (time.perf_counter() - start) * 1000
    
    # Custom metadata in trace
    return {
        "messages": [response],
        "last_node_latency_ms": latency_ms,
    }
```

---

## 12. Quick Reference

```
Install:
  pip install langgraph langchain-openai
  pip install langgraph-checkpoint-postgres   # for production persistence

Key classes:
  StateGraph(StateSchema)
  MessageGraph()
  ToolNode(tools)                         # prebuilt tool execution node
  MemorySaver()                           # dev checkpointer
  PostgresSaver.from_conn_string(dsn)     # prod checkpointer

Key methods:
  graph.add_node(name, fn)
  graph.add_edge(from, to)
  graph.add_conditional_edges(from, condition_fn, mapping)
  graph.set_entry_point(name)
  graph.compile(checkpointer, interrupt_before, interrupt_after)
  graph.invoke(input, config)
  graph.stream(input, config, stream_mode="updates")
  graph.astream_events(input, config, version="v2")
  graph.get_state(config)
  graph.get_state_history(config)
  graph.update_state(config, values, as_node)

Config pattern:
  config = {"configurable": {"thread_id": "unique-conversation-id"}}

Human-in-the-loop:
  interrupt(value)                        # pause and return value to caller
  Command(resume=value)                   # resume with a value
  graph.compile(interrupt_before=["node_name"])

Docs:   https://langchain-ai.github.io/langgraph
GitHub: https://github.com/langchain-ai/langgraph
```
