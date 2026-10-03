---
title: "Dynamic Workflows: first look"
date: 2026-10-03
slug: "dynamic-workflows-first-look"
tags: ["agentic workflows", "github copilot", "ai"]
draft: false
---

Just about [10 months ago](blog.cloud-eng.nl/2026/01/09/my-github-copilot-powered-development-workflow/), I was talking about the tools I use. A lot of things have changed since then. The biggest one for me is a switch to GitHub Copilot App (yep, I remember I wasn't so impressed with the idea of Orchestrators back in January, yet here I am) and completely new ways of working that got unlocked with it. 

Today, I'd like to talk about Dynamic Workflows, [which recently landed](https://github.blog/changelog/2026-10-01-dynamic-workflows-in-copilot-cli-and-the-copilot-app/) in the Copilot app and CLI. 


## Let's look back for a second.

The pace of innovation in the industry is insane, and to be fair, if you have anything else to do in addition to reading about all the new ideas and approaches that "revolutionize the way we work", you can easily get lost and struggle with three questions: 

* What is this new tech about? 
* What problem does it solve? 
* Do I have this problem? 

Since the name is somewhat cryptic, I'd like to give some context first. 

The more capable our models and harnesses become, the less manual steering they require. 
Only last December, I was demoing [this "agent handoff"](https://github.com/asizikov-demos/agent-handoff-demo) playground: a simple workflow that passes processed information between several sub-agents under human supervision at every step. A very limited, UI-driven system that felt like a toy, to be fair. 

The way we used to define reusable agentic primitives has evolved as well. Remember "Reusable Prompts"? They are obsolete already. We just simply don't need to prompt that hard. 

At the same time, a lot of useful work was applied to HOW the agent accomplishes the task: Skills that mix non-deterministic invocation rules, deterministic scripts and an LLM-driven process gluing that together. 

The complexity of parallel work was reduced to `/fleet` or `/orchestrate` commands where the actual coordination was delegated to the agent, and the human in charge simply describes the outcome. 

This gave us a state where Agents were given a mix of deterministic and non-deterministic tools that can execute tasks, but the whole workflow and communication were purely non-deterministic (even though Agents are pretty good at following instructions, they still fail sometimes, albeit rarely). 

And that's the problem Dynamic Workflows solve: 

* Be able to codify coordination and orchestration of several agents
* Share and reuse these codified workflows

Now let's take a look at the example. 

## Sample workflow

I'll take the sample workflow from the demo repository linked above.

![Agent workflow with translation, sentiment analysis, and sentiment-based handoffs](/images/2026/10-dynamic-workflows-first-look/00-workflow-handoffs.png)

This workflow is only implicit. If we look at the `.github/agents/` dir, we'll see that these agents are not coupled really, and even the `user-support-operator.agent.md` file was added to describe the whole flow end-to-end. 

But there is a problem here: we have a Markdown file that pretends to be a program. Well, Markdown is fine, but we have better ways to define the deterministic algorithm. You know, like a programming language. 

And that's what makes Dynamic Workflows unique: while other ways to invoke a multi-agent flow delegate the actual flow design and orchestration to the agent, which is free to analyze the problem and build the plan (or use the plan provided externally), DW pin the flow and allow each agent to be non-deterministic within the boundaries of its own task. 

The best way to understand something is to build it, so let's dive in!

First, I'm going to check out the older demo project and ask the Copilot app to build me a DW based on the repo content. 

![Copilot converting an agent handoff flow into a Dynamic Workflow and testing it](/images/2026/10-dynamic-workflows-first-look/01-workflow-conversion.png)

## Anatomy of a Dynamic Workflow

Once completed, this will place two new files into the `.github/extensions/` directory: 

```
.github/
└── extensions/
    └── user-support-operator/
        ├── config.mjs
        └── extension.mjs
```

`user-support-operator` is a dynamic workflow that is registered as a Copilot Extension now. 
`extension.mjs` is an entry point. We'll take a look later. Let's focus on `config.mjs` first. 

```js
export const workflowMeta = {
  name: "user-support-operator",
  description: "End-to-end support flow: translate → sentiment → routed response ...",
  phases: [
    { title: "Translate", detail: "translator agent → English + original language" },
    { title: "Sentiment", detail: "sentiment agent → Positive/Negative/Neutral" },
    { title: "Respond", detail: "acknowledgment / supporter / information agent" },
  ],
  argsSchema: { /*SKIPPED*/ },
};
export const translationSchema = { /*SKIPPED*/ };
export const sentimentSchema = { /*SKIPPED*/ };
export const responderAgents = {
  Positive: "acknowledgment",
  Negative: "supporter",
  Neutral: "information",
};
```

As we can see, it's a graph configuration. We have metadata here (`name`, `description`), phases and schema contracts defined. 

The rest is built into `extension.mjs`. 

If I drop the SDK-related code (extension registration), retries, logs and other technical logic, the DW will be boiled down to this: 

```js
async (ctx) => {
  const message = typeof ctx.args?.message === "string" ? ctx.args.message.trim() : "";

  ctx.phase("Translate");
  const translation = await callWithFallback("translator", .. );
  
  ctx.phase("Sentiment");
  const analysis = await callWithFallback("sentiment",..);
  const sentiment = analysis?.sentiment ?? "Neutral";
  
  ctx.phase("Respond");
  const responder = responderAgents[sentiment];
  const response = await callWithFallback(responder, ..);
  
  return {
    response: response ?? null,
    sentiment,
    explanation,
    responder,
    originalLanguage: t.originalLanguage,
    translatedMessage: t.translatedMessage,
    degraded: !translation || !analysis,
  };
};
```

Here we drive agent invocation step-by-step via the codified flow, switching between phases. 

I like that it's not yet another visual programming tool, where we define steps on a canvas. We can use natural language instead and make an LLM generate code for us. 

## Configuration options

Copilot SDK exposes quite a lot of configuration options for us. I'm not going to list them all. I'll share the [doc link](https://docs.github.com/en/enterprise-cloud@latest/copilot/concepts/agents/dynamic-workflows#limiting-a-dynamic-workflow) instead. But, at the time of me writing this page, it's possible to control different aspects of the workflow, such as: 

* AI Credit limits
* Maximum number of total and concurrent subagents
* Timeouts

Just enough primitives. The rest we can define in code. 

## Invocation options

Apart from direct invocation in the session (just tell Copilot to run the workflow by name), we can [trigger dynamic workflows in scripts](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-dynamic-workflows#running-a-dynamic-workflow-from-the-command-line) by passing the name of the workflow and its arguments to the Copilot CLI tool. 


```
copilot --experimental --allow-all-tools \ 
        workflow run user-support-operator \
        --args '{"message":"Hola, mi pedido llegó roto y estoy muy molesto"}'
```

results in 

```
● Translate

{
    "translatedMessage":"Hello, my order arrived broken and I am very upset.",
    "originalLanguage":"Spanish",
    "originalMessage":"Hola, mi pedido llegó roto y estoy muy molesto"
}

● Detected language: Spanish

● Sentiment
{
    "sentiment":"Negative",
    "explanation":"The user is very upset because their order arrived broken."
}
● Sentiment: Negative

● Respond

Hola, lamento mucho que tu pedido haya llegado roto. 
Entiendo perfectamente tu molestia y te pido disculpas por el inconveniente. 
Por favor, compártenos el número de pedido y, si es posible, 
una foto del daño para ayudarte cuanto antes con un reemplazo o reembolso.

{
    "response":"Hola, lamento mucho que tu pedido haya llegado roto. ...",
    "sentiment":"Negative",
    "explanation":"The user is very upset because their order arrived broken.",
    "responder":"supporter",
    "originalLanguage":"Spanish",
    "translatedMessage":"Hello, my order arrived broken and I am very upset.",
    "degraded":false
}
```

## Distribution and Discoverability

Dynamic workflows are extensions, meaning that they can (and should) be distributed as such. 
GitHub Copilot has a working [plugin ecosystem](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/copilot-cli/customize-copilot/plugins-creating). 

This means all standard options are available for us: 
* personal vs project installations
* plugin marketplace support for discoverability and distribution 

## Looking forward 

I hope at this point you've got a solid understanding of what Dynamic Workflows are, how they work and what they can do. 

Now let's speculate about the future. Predicting anything in Tech is fun now. The cycle is so short that you'll know if you were wrong in no time. 

As we saw today, DWs are a very niche tool: resumable, codified, well-structured workflows that serve a particular purpose. 

They are obviously a way to build predictable chains of agentic steps (which makes a lot of sense outside of a human-driven code gen flow, where reproducibility is important). They are useful when we optimize for token (AI Credits) consumption, as the orchestration process is scripted and doesn't "burn" through your limits. 

Dynamic Workflows are a good primitive. I do not expect them to become as ubiquitous as Skills, but I'd be pleased to see them around. 

## Disclosure

> *I'm employed by GitHub at the time of writing this post. All opinions are my own.*
