---
title: "Harness Engineering. Part 2: The human is the loop"
date: 2026-11-25
slug: "harness-engineering-part-2-the-human-is-the-loop"
tags: ["harness engineering", "github copilot"]
draft: false
---

This is the second post in the Harness Engineering series. Start with [Part 1: Evolution of AI-assisted workflows](/2026/11/24/harness-engineering-part-1-evolution-of-ai-assisted-workflows/).

Let's start with a short recap of what we discussed in the previous part and then continue with steps that need to be taken before moving to the next Stage. 
<!-- Before publishing: choose the Copilot surface and version, then verify the product-specific details marked below. -->
## Stage 1: One project, one branch, IDE-first

> *Human is the loop. You tell the agent what to do.*

In Stage 1, you have one session per task, a few sessions in flight, and typically short sessions. You see and can steer each step.

The main lever is **context management**. The plan lives in your head, and you guide the agent through it.

## The challenges

- providing project context to every session and task-specific context on demand
- preventing your agents from doing silly things
- teaching agents your internal tools and conventions
- managing the context window: keeping it lean, isolating some tasks, compacting when needed

## Tools for the job

We can group existing tools by the problem they are designed to solve. 

| Challenge | Tools and techniques |
|---|---|
| Project and task context | Repository and path-scoped instructions, explicit references, Skills, MCPs |
| Preventing silly things | Clear boundaries in instructions, tool permissions and human and LLM-assisted approvals, local sandboxes |
| Internal tools and conventions | Skills, MCP servers, manually selected custom agents |
| Managing the context window | Lean instructions, sub-agents, compaction, fresh sessions with a handoff |

Now let's discuss them one by one. 
### Project and task context

The first and most impactful improvement to the quality of the agent's work is relevant, project-aware context that we can add to the session.
There is a lot that the agent can discover itself by exploring the repository; our job is to provide what's hard to discover. 
#### Repository instructions
Recurring project context can be put in a shared instruction file, such as `.github/copilot-instructions.md` or `AGENTS.md`. 
The most important thing to keep in mind is that these instructions are added to every session regardless of the task it was given. 
If we want to optimise for quality and token efficiency, we need to keep them focused so that recurring context doesn't crowd out the task.

Example: "TypeScript monorepo. Use `pnpm`, not `npm`. API handlers live in `services/*/handlers`."

> NOTE: Instruction files tend to get stale when the project evolves. Noisy, outdated instructions negatively impact the work of the agent.

Many harnesses provide options to automatically generate shared instruction files, but those are usually generated based on what the agent can discover, so often they are not that helpful. Ideally, we need to keep track of agent misses, situations that get the agent confused; they are perfect candidates to be codified in the `AGENTS.md` file. 
#### Path-scoped instructions

Put specialized conventions in an instruction file associated with matching paths, rather than in general instructions.

Example: `applyTo: "**/*.test.ts"`: "Use the shared test factories; never hit the real database."

These instructions are loaded on-demand when the agent is about to edit files matching the pattern. 

#### Explicit references

Refer to the files a task needs.

Example: "Add pagination to `orders.ts`, following the pattern in `customers.ts`," with both files attached.

When we control the process manually, it's fine to do so. However, that won't fly when we give more autonomy to our agents. Something to fix in the next stage. 

### Preventing silly things

When we're fully in control, when the session has our attention, making sure the agent does what's expected is not that hard. 
But if we keep in mind the end goal, we can start building towards more independent agentic sessions. 

We have several levers at our disposal that can help us tighten up the process and establish boundaries. 

#### Boundaries in instructions

Instructions can provide agents with declarations of what must never happen. 

Example: "Never edit files under `generated/`. Never commit secrets."

These are guidelines, not guaranteed enforcement. But that's something that helps us pay less attention to the workflow.

#### Tool permissions and human approval

Agents ask permission to invoke certain tools. The longer and more complex sessions become, the more often we feel the cognitive overload of constant approvals. 

Approval comes in levels, from the most to the least human involvement:

1. **Ask every time.** The agent shows the exact tool call and you choose **Allow Once** or **Skip**.
2. **Scoped approval.** Approve a command prefix, a host or a folder for the current session, or always.
3. **Rules.** Allow and deny lists for commands and tools. Deny wins.
4. **Model-assisted approval.** A separate model reviews each tool call and asks you only about the risky ones. VS Code calls this **Assisted permissions**.
5. **Allow all.** No ordinary confirmations.

Example: 

Rules in Copilot CLI: allow all `git` commands except `git push`.

```bash
copilot --allow-tool='shell(git:*)' --deny-tool='shell(git push)'
```

Rules in VS Code: let lint run without asking, but always ask before a build.

```json
{
  "chat.tools.terminal.autoApprove": {
    "npm run lint": true,
    "npm run build": false
  }
}
```


#### Local sandboxes

Sandboxing allows us to separate the agent process from the rest of the operating system. We can define how the file system is accessed by the agent and what network capabilities it may use. And they can be distributed via Managed Settings and enforced at the Enterprise level. 

Combining different approval levels, using defaults configured per repository or centrally managed by Managed Settings and setting up local sandboxes allows us to reduce dependency on human supervision while keeping the agent sessions secure and reliable. 

### Internal tools and conventions

Another part of the knowledge that the agent cannot discover by itself consists of various internal conventions, tools and data sources. This is everything that lives outside of the codebase and that a human needs to tell the agent during the session. 

Capturing these bits of information in pluggable building blocks for the harness lets us delegate more work to the agent. 

#### Skills

A skill packages a prompt/instructions, with optional scripts and templates, for the agent to use when relevant.

Example: a `release-notes` skill that knows the team's changelog format and calls a script to list merged PRs.

Observing existing sessions and capturing what could be codified and packaging that into reusable skills for the team should become a standard exercise. 

#### MCP servers

MCP servers can give the agent access to internal systems. They can expose only the tools needed for the task.

Example: an issue-tracker server lets the agent read the acceptance criteria itself instead of having them pasted in.

MCP servers abstract away complex APIs, authentication and authorization rules. A team should be responsible for defining what external services should be accessible to the agent. 
Combining that with sandboxes can allow us to limit the attention scope and potential security surface for the agent. 

#### Manually selected custom agents

Manually select a named agent with instructions for a specific role, a narrower set of tools and MCP access, and a specified model where the surface supports these settings.

Example: select a "docs" agent configured for file editing and a documentation MCP server, without shell or unrelated MCP tools, using a chosen model. 

### Managing the context window

#### Lean instruction files

Move rarely needed detail out of general instructions and into path-scoped instructions or skills, where supported.
Follow the progressive loading pattern where possible. Let the agent know that additional information can be loaded from other documents. 

Example: keep the repository layout in general instructions and test-specific conventions in a scoped file.

#### Sub-agents

Where supported, isolate noisy work and bring a concise result back to the main conversation.

Example: ask a sub-agent to find callers of `legacyAuth()` and return a list rather than the full search transcript.

Subagents run in the same branch, but they have their own isolated context window. We can spawn and create many parallel sub-agents and assign them different models. This unlocks more complex workflows (cross-checking the work with multiple models, using high-tier models as judges, splitting research, planning and implementation), but it comes at the cost of us managing them manually. 

#### Compaction

Where available, summarize the conversation to reclaim context. The mechanism belongs to the vendor's inner loop; the user may be able to trigger it.

Example: `/compact` after a completed investigation, preserving the relevant decisions and next step.

#### Fresh sessions with a handoff

When the task changes, start over with a short summary of the goal, the decisions so far, and the next step.

Example: carry the agreed API change and the next implementation step into a new session.

### Improvement loop

Copilot sessions are stored in the user's home directory; that means we can index them, analyze and improve our customizations. 

The built-in `/chronicle` skill can guide us through the process and provide us with feedback based on the local session runs: 

- Where did our agents struggle? 
- Why didn't the agent load the skill? 
- How can we improve our `AGENTS.md` or any of the path-based instructions? 
- What MCPs were not so helpful? 
- What do our agents miss, and what do we have to explicitly provide over and over?
 
## What's next in the series

These techniques support a human-supervised session. When we iterate through the same act → learn → improve loop, we may end up with a reliable, yet simple harness that gives us confidence to delegate more complex tasks to our agents. 

In Stage 2, the question becomes: how does the agent find out it's wrong when you stop watching every edit?

Stay tuned!

## Further reading

- [Building Skills: How to Write a Good Agent Skill](/2026/02/01/building-skills-how-to-write-a-good-agent-skill/)
- [Building and Testing Custom Agents](/2026/02/07/building-testing-custom-agents/)
- [VS Code: Manage approvals and permissions](https://code.visualstudio.com/docs/agents/run/approvals)
- [GitHub Docs: Allowing and denying tool use in Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/allowing-tools)

## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
