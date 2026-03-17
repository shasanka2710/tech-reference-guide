# 🤖 Agentic Workflows — Consumer Reference

> **Goal:** Understand AI agents, how they work, common patterns, and how to design agentic systems — from a *solution designer's* perspective.

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [What is an AI Agent?](#1-what-is-an-ai-agent) |
| 2 | [Anatomy of an Agent](#2-anatomy-of-an-agent) |
| 3 | [The ReAct Loop](#3-the-react-loop) |
| 4 | [Tool Use & Function Calling](#4-tool-use--function-calling) |
| 5 | [Agent Memory](#5-agent-memory) |
| 6 | [Planning Strategies](#6-planning-strategies) |
| 7 | [Multi-Agent Patterns](#7-multi-agent-patterns) |
| 8 | [Human-in-the-Loop](#8-human-in-the-loop) |
| 9 | [Agentic Workflow Patterns](#9-agentic-workflow-patterns) |
| 10 | [Frameworks & Ecosystem](#10-frameworks--ecosystem) |
| 11 | [When to Use Agents vs. Simpler Patterns](#11-when-to-use-agents-vs-simpler-patterns) |
| 12 | [Common Failure Modes & Mitigations](#12-common-failure-modes--mitigations) |

---

## 1. What is an AI Agent?

```
Traditional LLM call:
  User Input → LLM → Response    (single round-trip, stateless)

AI Agent:
  User Goal → Agent loop → [Plan → Act → Observe → Repeat] → Final Answer
  
  Key differences:
  ✓ Multi-step: can take many actions to complete a task
  ✓ Tool use:   can call APIs, search the web, run code, query DBs
  ✓ Memory:     can retain information across steps
  ✓ Planning:   can decompose complex goals into sub-tasks
  ✓ Adaptive:   reacts to observations and errors in real-time
```

### Agents on the Autonomy Spectrum

```mermaid
flowchart LR
    A["💬 Single LLM Call\n(Prompt → Response)"]
    B["🔗 Prompt Chains\n(fixed multi-step)"]
    C["🔀 Router\n(LLM picks next step)"]
    D["🛠️ Tool-Using LLM\n(function calling)"]
    E["🤖 Agent\n(reason + plan + act loop)"]
    F["🌐 Multi-Agent System\n(autonomous collaboration)"]

    A -->|"more autonomy"| B --> C --> D --> E --> F

    style A fill:#85C1E9
    style F fill:#E74C3C,color:#fff
```

---

## 2. Anatomy of an Agent

```mermaid
flowchart TD
    subgraph Agent["🤖 AI Agent"]
        LLM["🧠 LLM Brain\n(reasoning, planning,\ndecision making)"]
        
        subgraph Memory["💾 Memory"]
            STM["Short-Term\n(conversation / scratchpad)"]
            LTM["Long-Term\n(vector store / database)"]
            EP["Episodic\n(past task outcomes)"]
        end

        subgraph Tools["🛠️ Tools / Actions"]
            WEB["Web Search"]
            CODE["Code Execution"]
            API["API Calls"]
            DB["Database Queries"]
            FILE["File Read/Write"]
            AGENT2["Sub-Agents"]
        end

        subgraph Planning["📋 Planning"]
            DECOMP["Task Decomposition"]
            REFL["Reflection & Self-Critique"]
            REPLAN["Re-planning on Failure"]
        end
    end

    USER["👤 User Goal / Task"] --> LLM
    LLM <--> Memory
    LLM <--> Tools
    LLM <--> Planning
    LLM --> RESP["✅ Final Response"]

    style Agent fill:#EBF5FB
    style Memory fill:#FEF9E7
    style Tools fill:#EAFAF1
    style Planning fill:#FDEDEC
```

---

## 3. The ReAct Loop

ReAct (**Re**ason + **Act**) is the foundational pattern for most agents.

### ReAct Cycle

```mermaid
flowchart TD
    START["🎯 Receive Task / Goal"] --> THINK
    
    THINK["💭 THOUGHT\nReason about the current state.\nWhat do I know? What do I need?\nWhat action should I take?"]
    
    THINK --> ACT["⚡ ACTION\nCall a tool or output an answer.\nExamples:\n- search('query')\n- code_exec('python code')\n- api_call(endpoint, params)"]
    
    ACT --> OBS["👁️ OBSERVATION\nReceive the result of the action.\n(tool output, error, data)"]
    
    OBS --> CHECK{"Is the\ngoal\nachieved?"}
    
    CHECK -->|"No"| THINK
    CHECK -->|"Yes"| FINAL["✅ FINAL ANSWER\nSynthesize all observations\ninto a coherent response"]

    style START fill:#3498DB,color:#fff
    style FINAL fill:#27AE60,color:#fff
    style THINK fill:#F39C12,color:#fff
    style ACT fill:#E74C3C,color:#fff
    style OBS fill:#9B59B6,color:#fff
```

### ReAct Example Trace

```
Task: "What is the current stock price of Apple and how does it compare to last year?"

Thought: I need to get the current stock price of Apple. I'll use the stock_price tool.
Action: stock_price(ticker="AAPL", period="current")
Observation: AAPL current price = $195.40

Thought: Now I need the price from a year ago to compare.
Action: stock_price(ticker="AAPL", period="1y_ago")
Observation: AAPL price 1 year ago = $178.20

Thought: I have both data points. I can now calculate the change and answer.
Final Answer: Apple (AAPL) is currently trading at $195.40, up 9.1% 
             from $178.20 one year ago.
```

---

## 4. Tool Use & Function Calling

### How Function Calling Works

```mermaid
sequenceDiagram
    participant App as Your Application
    participant LLM as LLM (e.g., GPT-4o)
    participant Tool as External Tool / API

    App->>LLM: Prompt + Tool Definitions (JSON schema)
    LLM-->>App: tool_call { name: "search", args: {"query": "..."} }
    
    Note over App: App detects tool_call response
    
    App->>Tool: Execute search("...")
    Tool-->>App: Search results
    
    App->>LLM: Tool result injected into conversation
    LLM-->>App: Final response using tool results

    Note over App,LLM: This loop can repeat multiple times
```

### Tool Definition (OpenAI format)

```json
{
  "type": "function",
  "function": {
    "name": "search_knowledge_base",
    "description": "Search the internal knowledge base for relevant information. Use this when you need to find specific company policies, procedures, or product information.",
    "parameters": {
      "type": "object",
      "properties": {
        "query": {
          "type": "string",
          "description": "The search query to find relevant documents"
        },
        "top_k": {
          "type": "integer",
          "description": "Number of results to return (default: 5)",
          "default": 5
        }
      },
      "required": ["query"]
    }
  }
}
```

### Common Agent Tools

| Tool Category | Examples | Use Case |
|---------------|----------|----------|
| **Search** | Web search, RAG search, SQL query | Grounding answers in real data |
| **Code Execution** | Python sandbox, shell commands | Data analysis, calculations, automation |
| **API Calls** | REST APIs, GraphQL, webhooks | Interact with external systems |
| **File Operations** | Read/write files, parse PDFs | Document processing |
| **Communication** | Send email, Slack, calendar | Action in the real world |
| **Browser** | Playwright, Selenium | Web scraping, UI automation |
| **Sub-Agent** | Delegate to specialised agent | Complex multi-domain tasks |

### Tool Design Best Practices

```
✓ Clear, specific descriptions — the LLM reads these to decide when to call
✓ Well-typed parameters with descriptions
✓ Return structured data (JSON) not free-form text
✓ Include error information in return value
✓ Idempotent tools where possible (safe to retry)
✓ Rate limit and timeout every tool
✓ Log all tool calls for observability and debugging
✗ Do not give agents destructive tools without confirmation (delete, send, publish)
```

---

## 5. Agent Memory

```mermaid
mindmap
  root((Agent Memory))
    Short-Term Memory
      In-context window
      Current conversation
      Scratchpad / scratch notes
      Limited by context window
    Long-Term Memory
      Vector store
        Semantic search over past experiences
      Key-Value store
        Fast lookup of facts
      Relational DB
        Structured query of history
    Episodic Memory
      Past task outcomes
      What worked, what failed
      Enables learning from experience
    Semantic Memory
      Factual knowledge base
      RAG retrieval
      Domain knowledge
    Procedural Memory
      How to do things
      Stored as tools or prompts
      Few-shot examples
```

### Memory Management Strategies

```
Problem: context window fills up in long agent runs

Solutions:
  1. Summarisation
     - Periodically summarise old conversation turns
     - Replace old messages with summary
     
  2. Sliding Window
     - Keep only last N messages in context
     - Risk: lose important early context
     
  3. Memory-Augmented RAG
     - Embed and store all messages in vector DB
     - Retrieve relevant past context at each step
     
  4. Structured Memory
     - Extract entities/facts into structured store
     - Query as needed (e.g., "remember user's name is Alice")
```

---

## 6. Planning Strategies

### Plan Types

```mermaid
flowchart TD
    subgraph Static["Static Planning"]
        SP["Pre-defined steps\nbefore execution begins\n\nPros: predictable, auditable\nCons: fails on unexpected situations"]
    end

    subgraph Dynamic["Dynamic Planning (ReAct)"]
        DP["Plan and act interleaved\nNext step decided based on\nprevious observation\n\nPros: adaptive, handles surprises\nCons: can loop, harder to predict"]
    end

    subgraph PAE["Plan-and-Execute"]
        PE["1. Planner LLM creates full plan\n2. Executor carries out each step\n3. Re-planner updates plan if step fails\n\nPros: clear structure + adaptability\nCons: two LLM calls per major task"]
    end

    subgraph ToT["Tree of Thoughts"]
        TOT["Explore multiple reasoning\npaths simultaneously\nSelect best path\n\nPros: better for complex puzzles\nCons: expensive (many LLM calls)"]
    end
```

### Plan-and-Execute Pattern

```mermaid
flowchart LR
    GOAL["🎯 Complex Goal"] --> PLANNER
    
    subgraph PLANNER["🗺️ Planner LLM"]
        P1["Step 1: Research topic"]
        P2["Step 2: Analyse findings"]
        P3["Step 3: Draft report"]
        P4["Step 4: Review & refine"]
    end

    PLANNER --> EXEC

    subgraph EXEC["⚙️ Executor"]
        direction TB
        E1["Execute Step 1"]
        E2["Execute Step 2"]
        E3["Execute Step 3"]
        E4["Execute Step 4"]
        E1 --> E2 --> E3 --> E4
    end

    E1 -->|"Failure"| REPLANNER["🔄 Re-planner\nAdjust remaining steps"]
    REPLANNER --> E2

    E4 --> RESULT["✅ Final Result"]
```

---

## 7. Multi-Agent Patterns

### Why Multiple Agents?

```
Single agent limitations:
  - One context window → limited working memory
  - Single model → limited specialisation
  - Sequential → slow for parallelisable tasks
  - Hard to audit and maintain as tasks grow complex

Multi-agent benefits:
  ✓ Specialisation: each agent optimised for its domain
  ✓ Parallelism: independent sub-tasks run concurrently
  ✓ Scale: complex tasks broken into manageable pieces
  ✓ Quality: agents can review each other's work
```

### Multi-Agent Topologies

```mermaid
flowchart TD
    subgraph Orchestrator["🎭 Orchestrator Pattern"]
        direction TB
        O["Orchestrator Agent\n(plans and delegates)"]
        O --> W1["Worker Agent\n(web search)"]
        O --> W2["Worker Agent\n(data analysis)"]
        O --> W3["Worker Agent\n(report writing)"]
        W1 --> O
        W2 --> O
        W3 --> O
    end

    subgraph Pipeline["🔗 Pipeline Pattern"]
        direction LR
        P1["Agent 1\n(Research)"] --> P2["Agent 2\n(Analyse)"] --> P3["Agent 3\n(Write)"] --> P4["Agent 4\n(Review)"]
    end

    subgraph Peer["👥 Peer / Debate Pattern"]
        direction TB
        A1["Agent A\n(propose solution)"] -->|"critique"| A2["Agent B\n(critique & improve)"]
        A2 -->|"revised solution"| A1
        A1 & A2 --> JUDGE["Judge Agent\n(select best)"]
    end

    subgraph Supervisor["🔍 Supervisor Pattern"]
        direction TB
        SUP["Supervisor Agent\n(quality control)"]
        WORK["Worker Agent\n(executes task)"]
        WORK --> SUP
        SUP -->|"reject: needs\nimprovement"| WORK
        SUP -->|"approve"| OUT["Output"]
    end
```

### Orchestrator-Worker Example: Research Assistant

```mermaid
sequenceDiagram
    participant U as User
    participant O as Orchestrator
    participant R as Research Agent
    participant A as Analysis Agent
    participant W as Writer Agent

    U->>O: "Write a competitive analysis report on electric vehicles"
    O->>R: "Research top EV companies, market share, recent news"
    R-->>O: Structured research findings
    O->>A: "Analyse findings, identify trends and competitive gaps"
    A-->>O: Analysis with insights
    O->>W: "Write executive report from research + analysis"
    W-->>O: Draft report
    O->>U: Final competitive analysis report
```

---

## 8. Human-in-the-Loop

### When to Include Humans

```
Include human checkpoint when:
  - Action is irreversible (send email, delete data, make payment)
  - High stakes or compliance-sensitive decisions
  - Agent confidence is low
  - Output will be customer-facing
  - Regulatory requirements mandate human approval

Automate fully when:
  - Action is reversible / low-risk
  - High volume, speed is priority
  - Established accuracy with tested evals
```

### Human-in-the-Loop Patterns

```mermaid
flowchart TD
    subgraph Review["Review Pattern"]
        RV_A["Agent completes task\nfully"] --> RV_H["Human reviews\ncompleted output"]
        RV_H -->|"approve"| RV_OUT["Publish / Act"]
        RV_H -->|"reject"| RV_A
    end

    subgraph Approval["Approval Gate Pattern"]
        AP_A["Agent runs until\nit needs to act\n(irreversible action)"] --> AP_H["Pause ⏸️\nHuman approves\nspecific action"]
        AP_H -->|"approved"| AP_CONT["Agent continues"]
        AP_H -->|"denied"| AP_STOP["Agent takes\nalternative path"]
    end

    subgraph Correction["Correction Pattern"]
        CO_A["Agent produces\nintermediate result"] --> CO_H["Human reviews\nand edits if needed"]
        CO_H --> CO_CONT["Agent continues\nwith corrected input"]
    end

    subgraph Escalation["Escalation Pattern"]
        ES_A["Agent attempts task"] --> ES_CONF{"Confidence\n> threshold?"}
        ES_CONF -->|"Yes"| ES_AUTO["Proceed automatically"]
        ES_CONF -->|"No"| ES_H["Escalate to\nhuman expert"]
    end
```

---

## 9. Agentic Workflow Patterns

### Pattern Catalogue

```mermaid
mindmap
  root((Agentic Patterns))
    Simple Patterns
      Single Agent ReAct
      Tool-Augmented LLM
      Prompt Chain
    Retrieval Patterns
      RAG Agent
        Retrieves before every answer
      Adaptive RAG
        Decides when to retrieve
      Self-RAG
        Critiques its own retrieval
    Coordination Patterns
      Orchestrator-Worker
      Pipeline
      Supervisor-Critic
      Peer Debate
    Planning Patterns
      ReAct
      Plan-and-Execute
      Tree of Thoughts
      Reflexion
    Memory Patterns
      Memory-Augmented Agent
      Episodic Memory
      Shared Memory Team
    Human Collaboration
      Human-in-the-Loop
      Human-on-the-Loop
      Mixed Initiative
```

### Reflexion Pattern (Self-Improvement)

```mermaid
flowchart TD
    TASK["🎯 Task"] --> ACT["Agent attempts task"]
    ACT --> EVAL["Evaluator\n(LLM or external)"]
    EVAL --> REFLECT{"Score\nacceptable?"}
    REFLECT -->|"Yes"| DONE["✅ Output accepted"]
    REFLECT -->|"No"| REFLECT_STEP["💭 Reflection Step\nAgent reasons about\nwhat went wrong\nand how to improve"]
    REFLECT_STEP --> MEMORY["Store reflection\nin memory"]
    MEMORY --> ACT

    style DONE fill:#27AE60,color:#fff
    style REFLECT_STEP fill:#E67E22,color:#fff
```

---

## 10. Frameworks & Ecosystem

```mermaid
mindmap
  root((Agent Frameworks))
    Orchestration
      LangGraph
        Graph-based workflows
        State machines
        Human-in-the-loop built-in
      LangChain
        Chains and agents
        Large ecosystem
      LlamaIndex
        Data-focused agents
        Strong RAG integration
    Multi-Agent
      AutoGen
        Microsoft
        Conversational agents
        Strong code execution
      CrewAI
        Role-based agents
        Task assignment
      Agency Swarm
        Structured teams
    Low-Code / Managed
      Amazon Bedrock Agents
        AWS managed
        Native AWS integration
      Azure AI Foundry
        Azure ecosystem
      Google Vertex AI Agents
        GCP ecosystem
    Open Source
      OpenAgents
      SuperAGI
      AgentBench
```

### Framework Selection Guide

| Need | Recommended |
|------|-------------|
| Production workflows with state management | **LangGraph** |
| Quick prototyping with many integrations | **LangChain** |
| RAG-heavy applications | **LlamaIndex** |
| Code-writing / execution agents | **AutoGen** |
| Role-based collaborative agents | **CrewAI** |
| AWS ecosystem | **Amazon Bedrock Agents** |
| Azure ecosystem | **Azure AI Foundry** |
| GCP ecosystem | **Vertex AI Agents** |

---

## 11. When to Use Agents vs. Simpler Patterns

```mermaid
flowchart TD
    START["New AI Feature Request"] --> Q1{"Is the task\ncompletable in a\nsingle LLM call?"}
    Q1 -->|"Yes"| SIMPLE["✅ Simple Prompt\nor Prompt + RAG\n(cheapest, most reliable)"]
    Q1 -->|"No"| Q2{"Are the steps\nknown in advance\nand fixed?"}
    Q2 -->|"Yes"| CHAIN["✅ Prompt Chain\n(predictable, auditable)"]
    Q2 -->|"No"| Q3{"Does the task require\nexternal data or\nactions?"}
    Q3 -->|"No"| COT["✅ Chain-of-Thought\nor Few-Shot Prompting"]
    Q3 -->|"Yes"| Q4{"Can one agent\nhandle it or\nneeds specialisation?"}
    Q4 -->|"One agent"| SINGLE_AGENT["✅ Single Agent\nwith tools"]
    Q4 -->|"Needs specialisation"| Q5{"Is parallelism\nor consensus\nimportant?"}
    Q5 -->|"No (sequential)"| PIPELINE["✅ Pipeline Pattern\n(agent1 → agent2 → ...)"]
    Q5 -->|"Yes"| MULTI["✅ Multi-Agent System\n(orchestrator + workers)"]

    style SIMPLE fill:#27AE60,color:#fff
    style CHAIN fill:#27AE60,color:#fff
    style COT fill:#27AE60,color:#fff
    style SINGLE_AGENT fill:#F39C12,color:#fff
    style PIPELINE fill:#E67E22,color:#fff
    style MULTI fill:#E74C3C,color:#fff
```

**Rule of thumb:** Start with the simplest solution that works. Move to agents only when simpler patterns are insufficient. Agents add latency, cost, and complexity.

---

## 12. Common Failure Modes & Mitigations

| Failure Mode | Description | Mitigation |
|-------------|-------------|------------|
| **Infinite Loop** | Agent loops without progress | Max iteration limit; break condition check |
| **Tool Hallucination** | Agent calls a tool that doesn't exist | Strict tool definition validation; fallback |
| **Goal Drift** | Agent pursues sub-goals, forgets original | Include original goal in every step's prompt |
| **Context Overflow** | Long runs exhaust context window | Periodic summarisation; external memory |
| **Cascading Errors** | Error in step 2 corrupts all downstream steps | Checkpoint validation; restart capability |
| **Over-calling Tools** | Agent calls expensive tools unnecessarily | Tool use cost budget; caching |
| **Prompt Injection** | Malicious tool output hijacks agent | Sanitise tool returns; sandboxed execution |
| **Non-determinism** | Same input → different plans → different results | Lower temperature; evals; human checkpoints |
| **Deadlock** | Multi-agent system waits on each other | Timeouts; async orchestration |
| **Scope Creep** | Agent takes unintended actions in the world | Principle of least privilege for tools |

---

## 📌 Quick Reference

```
ReAct loop:     Thought → Action → Observation → (repeat) → Final Answer
Agent = LLM + Tools + Memory + Planning loop

Tool call flow:
  1. LLM decides to call tool (returns JSON)
  2. Your app executes the tool
  3. Result injected back into LLM context
  4. LLM continues reasoning

Memory types:
  Short-term: in-context (tokens)
  Long-term:  vector DB, key-value store
  Episodic:   past outcomes stored for reference

Multi-agent patterns:
  Orchestrator-Worker: central agent delegates
  Pipeline:            agent1 → agent2 → agent3
  Supervisor:          critic agent reviews worker
  Peer Debate:         agents argue, judge selects

When NOT to use agents:
  - Task is completable in one LLM call
  - Steps are fixed and predictable (use chains)
  - Latency is critical (agents are slow)
  - Cost is a primary constraint
```
