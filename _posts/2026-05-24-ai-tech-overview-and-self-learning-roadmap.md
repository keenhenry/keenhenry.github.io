---
layout: post
title: Modern AI Knowledge Map Build Up
date:   2026-05-24
categories: [technology, self-learning, notes]
tags: artificial intelligence, AI
description: TODO
---

## Fundamentals

What is the current AI tech landscape? The overview and study roadmap.

TO READ:
- https://pub.towardsai.net/a-developers-guide-to-agentic-frameworks-in-2026-3f22a492dc3d

The answer from Gemini regarding the landscape and learning roadmap:

It is an exciting time to dive into agentic AI. The tech landscape has moved past single-prompt text generators and simple chatbots. Today, the focus is heavily on building stable, autonomous systems that can reason, break down multi-step tasks, correct their own mistakes, and interface with external software.

To build fully automatic workflows yourself, you will need to understand both the **conceptual architecture** of how an agent thinks and the **current software ecosystem** used to build them.

---

## 1. The Core Conceptual Architecture

Every modern AI agent—regardless of the framework you choose—is built on a combination of four foundational pillars: **Planning, Memory, Tools, and Reflection.**

```
               +--------------------------------------+
               |               BRAIN                  |
               |  (LLM + System Prompt + ReAct Loop)   |
               +-------------------+------------------+
                                   |
         +-------------------------+-------------------------+
         |                         |                         |
+--------v--------+       +--------v--------+       +--------v--------+
|    PLANNING     |       |     MEMORY      |       |      TOOLS      |
| • Goal Breakdown|       | • Short-term    |       | • APIs & Code   |
| • Reflection    |       | • Long-term/RAG |       | • MCP Protocols |
+-----------------+       +-----------------+       +-----------------+

```

### Planning & Reasoning (The Loop)

Instead of responding to a user immediately, an agent runs inside an execution loop. The most common pattern is **ReAct (Reason + Act)**.

* **Thought:** The agent analyzes the user's goal and reasons about what to do next.
* **Action:** It decides to call a specific tool or search for information.
* **Observation:** It looks at the result of that action, evaluates whether it succeeded, and determines the next step.

### Memory Systems

Stateless models cannot act as agents. Agents require a sophisticated memory architecture split into three layers:

* **Short-Term (Context):** Managing the immediate conversation or task state within the LLM's context window.
* **Long-Term (Semantic):** Utilizing a Vector Database (RAG) to pull in past experiences, company data, or domain knowledge when triggered.
* **Procedural:** Saving specific hardcoded routines or past successful execution paths (often mapped as state machines).

### Tools & Tool Calling (Action)

Agents interface with the physical and digital world via functions. The LLM doesn’t actually run code or call APIs itself; it outputs structured data (like JSON or specialized syntax) indicating *which* tool it wants to use and *what* parameters to pass. Your backend code catches this intent, executes the function, and feeds the results back to the LLM.

### Reflection & Self-Correction

A great agent checks its own work. Before delivering a final response, a reflection step passes the output back to the LLM (or a secondary "critic" LLM) with a prompt like: *"Review this code/data against the original goal. Are there errors?"* If errors are found, it loops back to the planning phase.

---

## 2. The Tech Landscape (Frameworks Ecosystem)

The agent ecosystem has consolidated into clear categories based on how you want to handle control and scale.

### Production & Code-First Frameworks

* **LangGraph (LangChain Ecosystem):** The current heavyweight for production enterprise systems. It models agents as **state machines** using directed graphs (nodes are actions, edges are conditional paths). It gives you precise control over loop execution, retries, and human-in-the-loop validation, offering excellent debugging via LangSmith.
* **Claude Agent SDK / MCP:** Released by Anthropic, this is the same engine underlying their autonomous developer tools. It heavily emphasizes the **Model Context Protocol (MCP)**, an open standard that lets agents securely connect to data sources, local files, and secure sandboxed environments.
* **Pydantic AI & Smolagents:** Highly popular, lightweight, code-first alternatives. They eschew heavy abstractions in favor of pure, clean Python and type hinting, giving you the fastest path to highly reliable single-agent workflows.

### Multi-Agent Orchestration

* **CrewAI:** Best for role-based multi-agent setups. You intuitively define a "crew" of individual agents (e.g., a *Researcher* agent, a *Writer* agent, and a *Reviewer* agent), assign them tasks, and let them sequentially or hierarchically collaborate.
* **AutoGen (Microsoft / AG2):** A powerful framework centered around conversational multi-agent systems where agents talk to each other to solve a coding or analytical problem.

---

## 3. Your Learning Roadmap

To go from zero to building fully automated agentic workflows, focus your learning on these progressive blocks:

### Phase 1: Tool Use Mastery (The Foundation)

Master how to write custom Python functions and expose them to an LLM.

* Learn how the **Function Calling API** works natively in OpenAI or Anthropic.
* Practice writing tight, self-documenting code with clear docstrings, because the LLM relies entirely on your docstrings and type hints to understand *how* and *when* to use your tools.

### Phase 2: Building from Scratch (No Frameworks)

Do not jump straight into LangGraph or CrewAI. Build a basic ReAct loop yourself using pure Python.

* Write a `while` loop that takes a user prompt, feeds it to an LLM, parses the model's JSON request to use a local tool, executes that tool, appends the result to the conversation history, and sends it back to the LLM until the model outputs a final answer.
* *Why?* Building this loop manually demystifies what agent frameworks are actually doing under the hood, making debugging infinitely easier later.

### Phase 3: Adopt a State Machine Framework

Once your native loops get too messy with conditional branches and error handling, adopt a framework.

* Learn **LangGraph** if your goal is deterministic, auditable, enterprise-grade workflows where you must dictate strict paths and handle complex state persistence.
* Learn **CrewAI** if you want to build cross-functional pipelines where multiple LLMs hand off work to each other using simple declarative code.

### Phase 4: Observability and Guardrails

An autonomous agent can easily get stuck in infinite loops or hallucinate tool inputs.

* Learn to use tracing tools like **LangSmith** or **Langfuse** to monitor exactly what thoughts, actions, and token costs your agents are incurring.
* Implement structured inputs/outputs using **Pydantic** to guarantee that the data flowing between your tools and your LLMs never breaks your application code.


What is a skill?
What is a harness?


## Applications

Different types of agents:
- passive
- Proactive, can use tools. But what do we need to do to make the tool usable for an AI agent?
- agent-to-agent working together; how to set up this way of working

What do I need to do to create agentic workflow; like a real hands-off mode to become an 100x engineer?
Can I create 'local' agentic AI workflows without relying on the tech giants?

Is project 'LocalGPT' a solution to my answer?
