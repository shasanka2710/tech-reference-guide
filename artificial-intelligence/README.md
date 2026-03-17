# 🤖 Artificial Intelligence — Consumer & Solution Designer Reference

> **Audience:** Engineers, architects, and product builders who consume Gen AI and Agentic workflows to design and build AI-powered solutions — not focused on ML theory or model training internals.  
> **Goal:** A top-down, practical reference — from foundational concepts to advanced solution patterns — with visual diagrams throughout.

---

## 📋 Table of Contents

| # | Topic | File |
|---|-------|------|
| 1 | [Gen AI Fundamentals](#gen-ai-fundamentals) | [gen-ai-fundamentals.md](./gen-ai-fundamentals.md) |
| 2 | [Agentic Workflows](#agentic-workflows) | [agentic-workflows.md](./agentic-workflows.md) |
| 3 | [Gen AI Solution Design](#gen-ai-solution-design) | [gen-ai-solution-design.md](./gen-ai-solution-design.md) |

---

## 🗺️ The AI Landscape — Consumer Perspective

```mermaid
mindmap
  root((AI for Solution Designers))
    Gen AI Fundamentals
      What is a Foundation Model
      How LLMs work conceptually
      Tokens & Context Window
      Prompting Techniques
      Embeddings & Vector Search
      RAG - Retrieval-Augmented Generation
    Agentic Workflows
      What is an AI Agent
      Agent Components
        LLM Brain
        Tools & Actions
        Memory
        Planning
      Agent Patterns
        ReAct
        Plan-and-Execute
        Multi-Agent
      Orchestration Frameworks
    Gen AI Solution Design
      Architecture Patterns
      Evaluation & Testing
      Safety & Guardrails
      Cost Optimisation
      Observability & Monitoring
      Common Use Cases
```

---

## 🔭 Top-Down Learning Path

```mermaid
flowchart TD
    A["🧠 Start Here:\nWhat is Gen AI?"] --> B["📖 Foundation Models\n& LLMs"]
    B --> C["🔤 Tokens, Context\n& Parameters"]
    C --> D["💬 Prompting\nTechniques"]
    D --> E["🔍 Embeddings\n& Vector Search"]
    E --> F["📚 RAG — Retrieval-\nAugmented Generation"]
    F --> G["🤖 AI Agents\n& Tool Use"]
    G --> H["🔗 Agentic Workflow\nPatterns"]
    H --> I["🏗️ Solution Design\n& Architecture"]
    I --> J["🛡️ Safety, Cost\n& Observability"]
    J --> K["🚀 Production-Ready\nGen AI Systems"]

    style A fill:#4A90D9,color:#fff
    style K fill:#27AE60,color:#fff
```

---

## Gen AI Fundamentals

**File:** [gen-ai-fundamentals.md](./gen-ai-fundamentals.md)

Covers the essential concepts every solution designer must know when working with Large Language Models and generative AI services:

- What foundation models are and how they differ from traditional AI
- Tokens, context windows, and what they mean for your application
- Temperature, top-p, and other inference parameters that control output
- Prompting strategies: zero-shot, few-shot, chain-of-thought, system prompts
- Embeddings and semantic search
- Retrieval-Augmented Generation (RAG) — the most important pattern for production apps
- Multimodal models and their capabilities

---

## Agentic Workflows

**File:** [agentic-workflows.md](./agentic-workflows.md)

Deep-dives into AI agents — autonomous systems that can plan, use tools, and complete multi-step tasks:

- The anatomy of an AI agent (LLM + Tools + Memory + Planning)
- The ReAct loop: Reason → Act → Observe → Repeat
- Tool use and function calling
- Memory types: short-term, long-term, episodic
- Multi-agent architectures: orchestrator-worker, peer-to-peer
- Human-in-the-loop patterns
- Popular frameworks: LangGraph, AutoGen, CrewAI

---

## Gen AI Solution Design

**File:** [gen-ai-solution-design.md](./gen-ai-solution-design.md)

Architectural guidance for building reliable, cost-effective, and safe Gen AI solutions:

- Decision framework: when to use which AI pattern
- Core architectural patterns: RAG, fine-tuning, prompt chaining, router chains
- Evaluation and testing strategies (LLM-as-judge, evals pipelines)
- Safety, guardrails, and responsible AI practices
- Cost optimisation techniques
- Observability, tracing, and monitoring in production
- Reference architectures for common use cases

---

## 🔑 Key Concepts at a Glance

| Concept | One-line Definition |
|---------|---------------------|
| **LLM** | Large Language Model — a neural network trained on vast text to predict and generate language |
| **Foundation Model** | A large pre-trained model adaptable to many tasks via prompting or fine-tuning |
| **Token** | The basic unit of text an LLM processes (~¾ of a word on average) |
| **Context Window** | Maximum tokens the model can "see" at once (input + output combined) |
| **Prompt** | The input text you send to a model |
| **Embedding** | A numerical vector representing the semantic meaning of text |
| **RAG** | Pattern that retrieves relevant documents and injects them into the prompt |
| **AI Agent** | An LLM that can reason, plan, and use tools to complete multi-step tasks |
| **Tool / Function Calling** | Ability for the LLM to invoke external APIs, code, or databases |
| **Agentic Workflow** | A pipeline where one or more agents autonomously complete a task |
| **Guardrails** | Input/output validation layers that enforce safety and quality |
| **Hallucination** | When a model generates confident but factually incorrect content |
| **Fine-tuning** | Further training a model on domain-specific data to specialise behaviour |
| **Inference** | Running the model to generate output (as opposed to training) |
| **Latency / TTFT** | Time-to-First-Token — a key responsiveness metric for streaming apps |
