---
title: "Harness Engineering. Part 1: Evolution of AI-assisted workflows"
date: 2026-11-24
slug: "harness-engineering-part-1-evolution-of-ai-assisted-workflows"
tags: ["agentic workflows", "harness engineering", "github copilot"]
draft: false
---

<!-- TODO: hook. Something like: "Same model, same agent, wildly different results between two teams. The difference is rarely the model. It's everything around it." -->
Ever noticed how, despite all of us using the same models and the same agents, we all get different results? Some teams are successful and ship at unprecedented speed, while others struggle to make agents work. What's the difference? 

Software Engineering as a discipline is in the middle of a massive transformation process. A lot of new capabilities are unlocked, but many new (tricky and exciting) challenges stand in our way.

The models evolve, and the harnesses, tools and techniques change. In this series, I'm planning to walk you through these challenges and discuss what tools we have at our disposal, what problems they solve and how they stack on top of each other. 

I'll use GitHub Copilot as my main example, but many of these techniques apply to other agents too.

## What I mean by "harness engineering"

Let's fix the terminology first. 

- **Inner loop**: what the harness vendors build. The agent loop, the system prompt, tool calling, compaction internals, model routing. Copilot CLI, Claude Code, Codex CLI... You don't own it, you pick it.
- **Outer loop**: everything your team owns and can customize around that agent. Instructions, skills, MCP servers, hooks, guardrails, review flows, memory, workspace setup.
- In this series, *harness engineering* = engineering the **outer loop**.

The outer loop is the part that encodes *your* project, *your* conventions, *your* definition of done. It's everything that enables agents to be autonomous and produce results that are up to your standards. 

## The map

Let's map out a typical evolution of AI-assisted workflows:

![Evolution of AI-assisted workflows](/images/2026/11-harness-engineering-p1/00-map.png)


The central axis displays the **autonomy** of agent work.
  - Left: low autonomy, *the human is the loop*.
  - Right: high autonomy, *human out of the loop*.

We can split the evolution into two major iterations - session scope increases in the first iteration and the number of sessions grows once we manage to have successful and reasonably autonomous sessions. 

The overall evolution can be broken into four stages: 

1. IDE-first: low autonomy
2. Session-first: higher autonomy, larger scope
3. Multiple sessions supervised and orchestrated by the human
4. High autonomy, high level of parallelism: software factories


## Quick detour: the context window

Every technique in this post is, one way or another, about what ends up in the context window. A longer version is in [Context Engineering: An Interactive Tutorial](/2026/04/18/context-engineering-interactive-tutorial/); here is the short one.

- What goes in: system prompt → `copilot-instructions.md` / `AGENTS.md` / `CLAUDE.md` → tool definitions → user prompt → in/out/in/out...
- Everything the outer loop adds competes for the same space.

## Stage 1: One project, one branch, IDE-first

> *Human is the loop. You tell the agent what to do.*

The most basic, the OG flow, is IDE-first, where agentic capabilities are enabled via the IDE extension. 

When working with your IDE, you're typically focusing on one project and one branch that is checked out. Often you need an IDE when you want to use code navigation, debuggers and other plugins. This usually means agents cannot perform tasks unsupervised and you need to step in often. 

We can define several traits of this stage:

- one session per task, 
- few sessions in flight, 
- typically short sessions.
- You see every step; you can steer every step.

When you manually drive your sessions, your typical lever is **context management**. When the session takes your full attention, you can afford to have a plan in your head and guide the agent through the process.

Challenges that you have all revolve around that context management: 

- providing project context to every session and task-specific context on demand
- preventing your agents from doing silly things
- teaching agents your internal tools and conventions
- managing the context window: keeping it lean, isolating some tasks, compacting when needed

## Stage 2: One project, one branch, no IDE

> *You stop telling the agent what to do and start telling it where to go.*

With the evolution of the inner loop of our harness and increased capabilities of the models, we can shift towards more complex and more autonomous sessions. Essentially, when most of the challenges from the previous stage are solved, we can switch our prompting approach towards being goal-based. 

When the agent is given the task, it can iterate towards this goal and validate stages. 

With this shift, we delegate smaller decision points to the agent, and we focus on the task and higher-level steering of the workflow. When that happens, we can drop the IDE and use CLI tools, where the agent activity is the primary focus. We still can open the IDE and perform surgical changes here and there, but treat that as an exception. 

Traits of this stage:
- You don't watch every edit anymore. 
- The agent runs longer, and you check the result.

That only works if the environment can tell the agent when it's wrong. This is where harness engineering really starts.

Typical challenges at this stage are: 

- providing reliable feedback to the agent
- managing context automatically
- managing complex multi-model flows
- parallelization of sub-agents
- keeping the agent within the repository conventions
- managing access permissions and tool-call approvals



## The break: where attention runs out

> *You cannot control every step anymore.*

In Stage 2, one session can run without your constant supervision. In Stage 3, several sessions compete for your attention at once. When the tooling lets us manage multiple sessions in parallel, we hit the next block - mental capacity. You physically can't review every step.

This requires us to **delegate**:
  - decision-making,
  - verification,
  - act → learn → improve loops.

Everything in Stages 1 and 2 still applies, but it now has to work without you watching.

## Stage 3: Many projects, many branches, many sessions

> *Multiple sessions in progress, local and remote (cloud), longer sessions.*

Self-contained, autonomous sessions require a lot of "stand-by" time, where the engineer simply waits until the agent asks for direction or until it's time to accept the work produced and move it to the next stage. Naturally, we want to minimize idle time and optimize it as much as we can. That's where parallel and multi-stage session orchestration is required. 

![Attention patterns: one session versus parallel sessions](/images/2026/11-harness-engineering-p1/01-attention-patterns.png)

Traits of this stage can be defined as: 

- longer, more complex workflows with a high level of autonomy
- higher-level orchestration: workflows, dependencies and interception points
- dependency on reproducible isolated environments
- need for automatic approvals and security guardrails

New problems, mostly about the *environment*, not the prompt:

- Secure isolated environments are desperately needed
- Work across multiple repositories
- Agent-to-agent communication
- End-to-end verification of larger chunks of work
- Local environment performance - can you really run 10 copies of your application? 
- Local environment uptime
- Need for automated improvement/self-healing processes
- Observability and control

When these challenges are solved, we pretty much have unlocked the path to the higher level of autonomous work, where agents can collaborate and move large-scope work forward in a reliable, predictable and secure way. 

## Stage 4: Software factories

> *Many projects, many sessions, high autonomy. Human out of the loop.*

At the moment of writing this page, we can say that Software Factories are the ultimate goal for our journey. Will it stay this way for a long time? I don't know. 

We have to be honest: many things that you are planning and building for this stage right now will most probably be obsolete in the foreseeable future, but what's important is the foundation that leads to this stage. 

The term itself is pretty old, something from the 60s, but the way we look at it is both new and old. 

We want to build a process where the intent (issues, specs, goals) is turned into tested and deployable software. Now agents take care of most of the flow, relying on a well-tested harness and reproducible and scalable infrastructure, providing us with a high level of visibility over the process. 

Human work shifts towards setting directions, outcome reviews and managing the factory itself, just like engineers manage Dark (lights-off) Factories in manufacturing. "Human out of the loop" means out of the step-by-step execution loop, not absent from oversight: people still set direction and review outcomes. 

At this point, it's the most vague part of the story: a lot of innovation and research is happening right now. 

## How the techniques stack

As we hopefully see, each stage doesn't replace the previous one; it depends on it.

- Context management (Stage 1) is what keeps autonomous sessions coherent.
- Guardrails and outer loop harness engineering (Stage 2) are what make delegation (Stage 3) safe.
- Environment isolation, orchestration and agent-to-agent communication (Stage 3) are what make scale (Stage 4) possible.

Skip a layer and the next one falls over. Our goal, as engineers, is to build this reliable foundation to make transitions possible. 


## What's next in the series

In the next posts in the series, we'll walk through the entire journey all the way towards Agentic Factories, getting a closer look at solutions to the challenges outlined in this episode. What tools do we have to battle them? How do they stack on top of each other? 

Stay tuned.

## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
