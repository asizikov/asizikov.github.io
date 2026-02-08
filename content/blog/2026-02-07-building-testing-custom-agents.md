---
title: "Building and Testing Custom Agents"
date: 2026-02-07
slug: "building-testing-custom-agents"
tags: ["github copilot", "developer-tools"]
draft: false
---

Building custom agents for GitHub Copilot (or any other AI assistant for that matter) sounds fun until you hit the reality of testing them. Let me walk you through the challenges and show you a better way to develop and test your agents locally.

## The Challenge with Custom Agents

Building custom agents isn't as straightforward as it might seem. Here are the pain points:

**Testing is tedious.** Every time you want to test your agent as a Copilot Coding Agent in the cloud, you're looking at a slow feedback loop. Upload, wait, test, repeat. Not exactly the rapid iteration cycle we're used to as developers. In a way, it reminds me of the old days of CI/CD pipeline testing where you had to wait for the entire pipeline to run just to find that typo in your YAML file.

**Prompt crafting is an art.** Writing effective system prompts is harder than it looks. You need to be precise, provide context, handle edge cases, and iterate multiple times before you get it right.

**Cloud testing takes forever.** Round-tripping to the cloud for every test run kills your momentum. When you're fine-tuning prompts, you need speed.

![Process of testing custom agents](/images/2026/02-testing-agents/00-process.png)

## The Solution: Build and Test Locally

Here's the good news: you don't need to test in the cloud. Both the GitHub CLI and VS Code let you run and test agents locally.

Yes, the local agent harness is not the same as in the cloud, but GitHub is actively working on a unified agent experience. Keeping that in mind, we need to be careful to use the common denominator features and avoid local-only tools and capabilities, to make sure our agents work both locally and in the cloud.

With local testing, you get:
- Instant feedback
- Rapid iteration on prompts
- Full control over the testing environment
- No waiting for cloud deployments

## A Practical Example: Release Notes Agent

Let me show you a real-world use case. Say you want to build an agent that:

1. Checks the current milestone in your repository
2. Checks all closed issues, PRs, and open bugs
3. Generates release notes from a template
4. Makes a go/no-go decision based on criteria you define

This is exactly the kind of task where it makes sense to have a specialized agent.

## Step 1: Generate the Agent with Copilot

Instead of writing the agent from scratch, let's ask Copilot CLI to generate it for us.

`/agent` command has a built-in agent generator.

![Create Agent in CLI](/images/2026/02-testing-agents/01-create-agent-cli.png)

```
Prompt: "Create a custom agent (Release Notes Writer) that uses GitHub MCP and checks the current GitHub milestone, 
 checks all closed issues, PRs and open bugs, generates release notes from a template (@docs/templates/release-notes-template.md), 
 and determines if we're ready to ship.                                           
 Output should be placed in /docs/release-notes/release-notes-[milestone-name].md                                   
 Note that we have Go/No-Go rules defined in the same template."
```

Copilot will scaffold the agent structure, including the system prompt, tools it needs, and the basic workflow.

## Step 2: Manual Testing

Now you can test the agent manually by invoking it directly in VS Code or via the CLI. This is already much faster than deploying Copilot Coding Agent and waiting for the cloud session to run.

![Invoke Agent in VS Code](/images/2026/02-testing-agents/02-invoke-agent-vscode.png)

Run a few test iterations:
- Does it find the right milestone?
- Does it format the release notes correctly?
- Are the go/no-go criteria working?

## Step 3: The True Power - Copilot CLI

Here's where things get interesting. With the Copilot CLI, you can run your agent-under-test as a subagent in an isolated context. The CLI main agent becomes the orchestrator.

Here's how it works:
- The CLI agent is your orchestrator
- It invokes your release notes agent as a subagent
- The subagent runs in an isolated context (it doesn't see the orchestrator's conversation and operates only with the tools and context you defined for it)
- You can specify a different model for the subagent
- The orchestrator reviews the results and iterates on the custom agent prompt based on the output

This separation is powerful—your orchestrator can test the same agent multiple times with different inputs, critique the outputs, and refine the approach without the subagent getting confused by the meta-conversation.


As a one-off exercise, I've started a session with this prompt: 

```
We need to test our @.github/agents/release-notes-writer.agent.md  Custom Agent.

you need to test it with GPT-5.2, Sonnet 4.5 and GPT-5-mini. Run each model as subagent.

then crosscheck their work, if any of them didn't do good job, you need to fine-tune the prompt and iterate again until all 
models prerform as expected
```

The session log would look something like this:

![Session log](/images/2026/02-testing-agents/03-session-log.png)

Sub-agentic tasks launched by the orchestrator, errors detected, and the orchestrator can start the iteration loop right there. 
My favorite part is that sub-agentic tasks are not billed; we only spend Premium Requests for the orchestrator session. Sweet, isn't it?

### Pro-tip: Fleet

While this process is going to work in vanilla Copilot CLI, we can also leverage Fleet functionality by enabling Experimental Features in the CLI.
This will instruct CLI to use the `sql` tool to coordinate the sessions and store the results in a local SQLite database, as well as enable better communication between subagents and the orchestrator.


![Enable Fleet in CLI](/images/2026/02-testing-agents/04-fleet.png)

### Testing Results

After a few iterations of testing and refining the prompt, we got to the point where all three models are performing well and generating the release notes as expected.

![Test Session Results](/images/2026/02-testing-agents/05-testing-results.png)

We can visualize the flow as below: 

![Testing Flow](/images/2026/02-testing-agents/06-flow.png)


## Why This Matters

This approach transforms agent development from a slow, painful process into a rapid, iterative workflow. You're no longer blocked by cloud deployment cycles. You can experiment freely, test quickly, and improve continuously.

The combination of local testing and CLI orchestrating multi-agent sessions gives you a complete toolkit for building production-quality custom agents.

## Taking It Further: The Agent Tester Meta Agent

Here's a mind-bending idea: this entire testing workflow can itself be codified as an "Agent Tester" meta agent.

Imagine an agent that:
1. Takes your agent definition as input
2. Generates test scenarios automatically
3. Runs your agent as a subagent with different models
4. Collects and analyzes the results
5. Suggests improvements to the system prompt
6. Iterates until quality thresholds are met

This is agents testing agents. It's meta-automation, and it's entirely possible with the tools we have today.

So, what I did was prompt the CLI agent to build an "Agent Tester" based on the successful testing session: 

![Agent Tester Created](/images/2026/02-testing-agents/07-meta-agent.png)

## What's Next?

If you want to try this workflow yourself, here are two layers of next steps.

### Do this next

- Build a small agent locally (a code reviewer or documentation helper is a great warm-up).
- Invoke it manually a couple of times to tighten the prompt.
- Then switch to the CLI orchestrator pattern: run your agent-under-test as a subagent (optionally with a different model), review the results, and iterate.

### Go deeper (optional)

Once the basics feel natural, make it repeatable:

- Build an evaluation pipeline around your Agent Tester.
- Run Copilot CLI as part of your CI/CD workflow, or run Copilot Coding Agent on GitHub Actions runners.
- For more complex evaluation scenarios, take a look at Copilot SDK and build a custom evaluation harness (TUI or Web UI).

The tooling is here. The only limit is your imagination.

## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
