---
title: "Keep the Why: Code Becomes Legacy When Nobody Remembers Why"
datePublished: 2026-07-11T05:22:37.714Z
cuid: cmrfx4dst00000akmfrid7bwk
slug: keep-the-why-code-becomes-legacy-when-nobody-remembers-why
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/1be315d8-3566-4c00-96bc-9ac02675d254.png
tags: ai, opensource, documentation, developer-tools, legacy-code, agents

---

> **Keep a Changelog records what changed. Keep the Why preserves why it changed.**

I've been maintaining the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite) since 2019. It is a set of open-source Python packages for building automated trading systems on Binance, and parts of it have been running in production for years.

For the last few months, I have also been working with an AI coding agent on these projects every day.

Early on, I invested quite a bit of time into the harness around it:

*   conventions
    
*   project structure
    
*   persistent context
    
*   documentation the agent could actually use
    
*   rules for how it should work inside a mature codebase
    

That effort paid off.

Today I can point the agent at a piece of code and ask why something looks the way it does. In many cases, it can answer correctly without guessing.

Not because the model somehow remembers seven years of development history. It does not.

It can answer because the reasoning is still present in the repository.

That distinction matters.

The more I worked with this setup, the more obvious something became:

If structured context makes an AI agent useful in a codebase it did not grow up with, the same context should also help a human developer entering that project later.

A new developer should not have to reverse-engineer every strange retry loop, compatibility workaround or architectural boundary from the code alone. They should be able to ask:

*   Why is this retry logic so defensive?
    
*   Why are we not using the obvious library?
    
*   Why does this component wait for a snapshot before processing buffered events?
    
*   What broke the last time somebody simplified this?
    

And ideally, the answer should already live with the project.

That is the idea behind **Keep the Why**.

## The problem

Tests preserve expected behaviour.

Git preserves changes.

Issue trackers preserve discussions, sometimes.

What often disappears is the reasoning behind the final implementation.

That creates a few very familiar problems.

### Re-debate

A team discusses the same architecture question again because nobody remembers that it was already settled eighteen months ago.

The previous decision may still be correct. The reasons are simply gone.

### Silent regression

Somebody finds a workaround that looks unnecessary and cleans it up.

Unfortunately, it was not unnecessary. It was the fix for a bug nobody documented properly.

The code looked ugly because reality was ugly.

### Onboarding stall

A new developer sees code they do not understand and avoids touching it.

That is usually a reasonable decision. Changing unfamiliar code without context is how repeat incidents are born.

AI coding agents have the same problem, only faster. A fresh session starts with little or no project history. Without preserved rationale, the agent either spends time reconstructing it or confidently proposes the exact approach that already failed once.

## This is not a new problem

Architecture Decision Records have existed for years, and they are useful.

But they require somebody to notice that a decision deserves an ADR, stop the current work and document it deliberately. Large architectural choices may get that treatment. Smaller constraints often do not.

Those smaller decisions are frequently the ones that become dangerous later:

*   a defensive timeout
    
*   an odd startup sequence
    
*   a compatibility branch
    
*   a strange cache invalidation rule
    
*   a library that was evaluated and rejected
    
*   a workaround linked to a production incident
    

They are too small for ceremony and too important to forget.

The important change is not that reasoning suddenly became valuable.

The change is that an AI agent already participates in many of the conversations where that reasoning appears. Capturing it no longer has to be a completely separate documentation task.

## What Keep the Why is

[Keep the Why](https://keepthewhy.com/) is an open-source agent skill.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/066f1c95-1ec4-48ab-bd8a-96b39d3ca8c6.png align="center")

It is a `SKILL.md` file with supporting references and examples, based on the open Agent Skills format. There is no service, database, account or MCP server behind it.

It teaches a coding agent four related workflows.

### 1\. Continuous capture

During normal development, the agent notices when a conversation contains useful rationale:

*   an architectural choice
    
*   a rejected alternative
    
*   a production constraint
    
*   an incident finding
    
*   an intentional workaround
    
*   a behaviour that looks strange but is deliberate
    

It then updates the relevant project context at a natural checkpoint.

The goal is not to document every line of code. That would just create a second codebase, only written in Markdown.

The goal is to preserve the parts that are not obvious from the implementation itself.

### 2\. Retrospective recovery

The skill can also be applied to an existing repository.

The agent inspects:

*   code
    
*   git history
    
*   issues
    
*   pull requests
    
*   existing documentation
    

It reconstructs what the available evidence supports and clearly marks what remains uncertain.

It should not invent a clean historical narrative just because that would look nice in a document.

Sometimes the honest result is:

> We know what this does. We do not yet know why it was designed this way.

That is useful information too.

### 3\. Knowledge-transfer interviews

This is the part I find especially interesting.

Before a long-term maintainer changes teams, leaves or retires, the agent can first analyse the repository and identify areas where the reasoning is missing.

It can then prepare focused questions:

> Why does this synchronization process wait for the snapshot before applying buffered updates?

instead of:

> Please explain the synchronization system.

The first question has a chance of recovering useful knowledge. The second usually produces a meeting nobody wants to attend.

For broad, mostly tacit knowledge, the process can also work the other way around: let the maintainer explain the history in their own words, then extract decisions, constraints and open questions from the conversation.

The important part is that the result becomes project knowledge, not another recording nobody will ever watch again.

### 4\. Maintenance

Documentation can become legacy too.

So the skill also defines how to:

*   update existing topic files
    
*   resolve contradictions
    
*   mark old decisions as superseded
    
*   split files before they become too large
    
*   keep indexes lean enough for humans and agents
    
*   avoid creating five documents for one small decision
    

This is living documentation, not an archive of frozen decisions.

## The structure

Keep the Why deliberately separates two kinds of project knowledge.

```text
docs/
context/
```

### `docs/`

This answers:

> How do I use, operate, test, deploy or troubleshoot this project?

Examples:

*   installation
    
*   configuration
    
*   API usage
    
*   testing
    
*   deployment
    
*   operations
    
*   troubleshooting
    

### `context/`

This answers:

> Why is the project built this way?

Examples:

*   architectural rationale
    
*   rejected alternatives
    
*   historical constraints
    
*   incident learnings
    
*   compatibility decisions
    
*   deliberate workarounds
    
*   superseded approaches
    

Both layers are plain Markdown and versioned with the repository.

There is no separate AI-only knowledge store waiting to drift away from the human documentation.

The context is organised by topic, for example:

```text
context/
├── index.md
├── architecture.md
├── synchronization.md
├── compatibility.md
└── incidents.md
```

I prefer this over a file-by-file shadow tree because decisions often span several files and survive refactors.

I also prefer it over one file per decision for smaller, evolving topics. Formal ADRs still make sense where they fit. Keep the Why is not trying to ban them.

## What it looks like in practice

Imagine a normal implementation discussion:

```text
You: The simpler retry mechanism caused duplicate orders during
     reconnects. Keep the stateful version even though it looks
     more complicated.
```

That is not just an implementation detail.

It contains:

*   the rejected alternative
    
*   the reason it was rejected
    
*   the failure mode
    
*   the constraint future maintainers must preserve
    

The agent should recognise that and update the relevant topic in `context/`.

Six months later, somebody finds the code and thinks:

> This looks over-engineered. I can simplify it.

Chesterton's Fence, except this time the fence has a sign on it.

## The rule that matters most

The core rule of Keep the Why is simple:

> Never invent rationale.

Plausible-sounding historical explanations are worse than missing documentation. A blank page creates caution. A confident false explanation creates bad decisions.

Each relevant rationale entry should make clear whether its evidence is:

*   **confirmed** — supported by a maintainer or authoritative project evidence
    
*   **inferred** — reasonably derived, but not confirmed
    
*   **unknown** — the available evidence is insufficient
    

Separately, an entry can be marked as **superseded** when it was once valid but has since been replaced.

This distinction matters: evidence confidence and temporal status are two different things.

This is not decoration.

It is the difference between useful project knowledge and hallucination wearing documentation's clothes.

The second important rule is proportionality.

A self-explanatory decision may need one sentence.

A surprising architecture boundary may need:

*   context
    
*   decision
    
*   alternatives
    
*   consequences
    
*   evidence
    
*   conditions under which it should be revisited
    

Not every decision deserves a ceremony. Not every strange-looking line deserves to be forgotten either.

## How it relates to existing approaches

Keep the Why is not a replacement for:

*   tests
    
*   README files
    
*   `CONTRIBUTING.md`
    
*   ADRs
    
*   issue trackers
    
*   [Keep a Changelog](https://keepachangelog.com/)
    

The name is deliberately related to Keep a Changelog.

Keep a Changelog records **what changed**.

Keep the Why preserves **why it changed**.

There is also prior work in this space:

- Architecture Decision Records
- [git-why](https://github.com/hexapode/git-why)
- [Addy Osmani's `documentation-and-adrs` skill](https://github.com/addyosmani/agent-skills/blob/main/skills/documentation-and-adrs/SKILL.md)
- [Agent Decision Records (AgDR)](https://github.com/me2resh/agent-decision-record)
- repository-memory and agent-context tools
    

I did not invent the idea that software decisions should be documented.

What I wanted was a practical combination of:

*   continuous capture
    
*   retrospective recovery
    
*   code-guided knowledge-transfer interviews
    
*   topic-based living documentation
    
*   repository-native storage
    
*   no required external platform
    

That combination grew out of how I was already working.

## Where the project stands

The underlying practice is not completely new to me.

I have been testing variations of this approach in my own AI-assisted development workflow for roughly four months.

But the public project is not a copy of that private setup.

It is a new incarnation of it:

*   cleaned up
    
*   generalised
    
*   made cross-agent
    
*   turned into an explicit method
    
*   separated from my own repository conventions
    
*   packaged as an open-source skill
    

That means it now has to prove itself in real use — and mature through it.

The initial structure looks reasonable to me. The real test is what happens across different repositories, teams and agents.

I expect some parts to change.

For example:

*   When does the agent capture too much?
    
*   When does it ask too little?
    
*   Which structure works for a small library and which one works for a large multi-repository system?
    
*   How much classification is useful before it becomes bureaucracy?
    
*   When should a topic be split?
    
*   How do we keep old rationale from becoming trusted but stale?
    

There is also a small but relevant body of supporting research. [A 2026 position paper](https://arxiv.org/abs/2601.21116) audited 62 architectural decisions across two internal projects and found that roughly 23% had stale supporting evidence within two months. The sample is small and the result is not a universal law, but it supports something maintainers already know: documentation does not stay correct by existing. It has to be maintained.

I do not yet have automated cross-agent eval results or a published before-and-after study.

The eval cases exist. The public evidence does not, yet.

That is the honest state of the project.

## Try it

Install it with:

```bash
npx skills add oliver-zehentleitner/keep-the-why
```

Or inspect and install it through GitHub:

```bash
gh skill preview oliver-zehentleitner/keep-the-why keep-the-why
gh skill install oliver-zehentleitner/keep-the-why keep-the-why
```

It is instructions only:

*   no scripts
    
*   no network calls of its own
    
*   no external service
    
*   no account
    
*   no database
    

Documentation and manual installation:

[keepthewhy.com](https://keepthewhy.com/)

Source, license and the actual skill:

[github.com/oliver-zehentleitner/keep-the-why](https://github.com/oliver-zehentleitner/keep-the-why)

## Feedback is welcome

Keep the Why is young.

The base is there, but the method now needs real use outside my own projects.

I am especially interested in feedback from people working with:

*   mature codebases
    
*   long-running production systems
    
*   AI coding agents
    
*   developer onboarding
    
*   knowledge transfer
    
*   maintainers who have inherited code nobody wants to touch
    

Bug reports are useful.

Conceptual criticism is even more useful.

If the structure is wrong for your project, I want to know why. If an important workflow is missing, propose it. If another project already solves part of this better, point me to it.

The project is open source because I do not think this should become another private knowledge silo.

The whole point is to keep the reasoning with the code.

Because “ask Bob” is not documentation.