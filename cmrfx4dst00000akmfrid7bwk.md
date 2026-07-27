---
title: "Keep the Why: Code Becomes Legacy When Nobody Remembers Why"
datePublished: 2026-07-11T05:22:37.714Z
cuid: cmrfx4dst00000akmfrid7bwk
slug: keep-the-why-code-becomes-legacy-when-nobody-remembers-why
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/1be315d8-3566-4c00-96bc-9ac02675d254.png
tags: ai, opensource, documentation, developer-tools, legacy-code, agents

---

> **Keep a Changelog records what changed. Keep the Why preserves why it changed.**

I've been maintaining the [UNICORN Binance Suite](https://blog.technopathy.club/page/unicorn-binance-suite) since 2019. It is a set of open-source Python packages for building automated trading systems on Binance, and parts of it have been running in production for years.

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
    
*   Which alternatives were already tried and rejected?
    

And ideally, the answer should already live with the project.

That is the idea behind **Keep the Why**.

## The problem

Important project knowledge is constantly created in conversation.

A developer explains why a timeout cannot be reduced. A production incident reveals why a strange compatibility branch exists. A team rejects the obvious library because it breaks under a specific workload. An AI agent proposes a simplification, then discovers why the existing code must remain defensive.

The resulting code is committed.

The reasoning often is not.

Tests preserve expected behaviour.

Git preserves changes.

Issue trackers preserve discussions, sometimes.

What frequently disappears is the reasoning behind the final implementation.

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

### Repeated agent mistakes

AI coding agents have the same problem, only faster.

A fresh session starts with little or no project history. Without preserved rationale, the agent either spends time reconstructing it or confidently proposes the exact approach that already failed once.

The conversation in which the mistake was explained may have been excellent.

But once the session ends, that knowledge is gone unless something writes it down.

## Documentation as a byproduct

Traditional documentation is usually treated as a separate task.

Finish the implementation, reload the reasoning from memory, rewrite it for another format and put it somewhere people may or may not find later.

That is exactly why it so often does not happen.

When an AI agent is already part of the development process, the situation changes.

The decision, alternatives, constraints and trade-offs are already being discussed in the same conversation that produces the code. The reasoning has already been expressed.

Capturing it does not have to mean creating another documentation ceremony.

It can become a byproduct of the work that was happening anyway.

Keep the Why has a deliberately narrow job:

> Do not let useful project reasoning disappear when the conversation ends.

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
    
*   an attempted cleanup that was abandoned after discovering a hidden constraint
    

They are too small for ceremony and too important to forget.

That last category is particularly easy to lose.

Imagine a developer starts removing what looks like redundant code, discovers why it exists and stops the change.

No commit is created.

There may be no pull request, no issue and no code comment.

The developer learned something important, but the repository gained no trace of it.

The next person can make exactly the same attempt.

A complete walkthrough of this case is available in [`examples/abandoned-change.md`](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/examples/abandoned-change.md).

The repository also contains practical examples for continuous capture, retrospective recovery and knowledge-transfer interviews in the [`examples/`](https://github.com/oliver-zehentleitner/keep-the-why/tree/main/examples) directory.

The important change is not that reasoning suddenly became valuable.

The change is that an AI agent already participates in many of the conversations where that reasoning appears. Capturing it no longer has to be a completely separate documentation task.

## What Keep the Why is

[Keep the Why](https://keepthewhy.com/) is an open-source, repository-native agent skill.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/066f1c95-1ec4-48ab-bd8a-96b39d3ca8c6.png align="center")

It follows the open `SKILL.md`\-based Agent Skills format and works across compatible agents such as Claude Code, Codex CLI, Gemini CLI, Cursor and others.

There is no service, database, account, daemon, dashboard or MCP server behind it.

The installable package contains instructions, references, examples and evaluation cases. It does not contain executable scripts or binaries and has no network access of its own.

The skill teaches an agent four related workflows.

### 1\. Continuous capture

During normal development, the agent notices when a conversation contains useful rationale:

*   an architectural choice
    
*   a rejected alternative
    
*   a production constraint
    
*   an incident finding
    
*   an intentional workaround
    
*   a behaviour that looks strange but is deliberate
    
*   a change that was abandoned after discovering why the existing behaviour must remain
    

It then updates the relevant project context at a natural checkpoint, once the decision has actually settled.

The goal is not to document every line of code. That would just create a second codebase, only written in Markdown.

The goal is to preserve the parts that are not obvious from the implementation itself.

### 2\. Retrospective recovery

The skill can also be applied to an existing repository.

The agent inspects available evidence such as:

*   code
    
*   git history
    
*   issues
    
*   pull requests
    
*   existing documentation
    
*   incident records
    

It reconstructs only what the evidence supports and clearly marks what remains uncertain.

It should not invent a clean historical narrative just because that would look nice in a document.

Sometimes the honest result is:

> We know what this does. We do not yet know why it was designed this way.

That is useful information too.

For a large or unfamiliar codebase, the skill does not pretend it can recover everything in one pass. It can prioritise high-risk areas such as authentication, payments, data integrity, recent incidents, unusually defensive code or components with a low bus factor.

### 3\. Knowledge-transfer interviews

This is the part I find especially interesting.

Before a long-term maintainer changes teams, leaves or retires, the agent can first analyse the repository and identify areas where the reasoning is missing.

It can then prepare focused questions:

> Why does this synchronization process wait for the snapshot before applying buffered updates?

instead of:

> Please explain the synchronization system.

The first question has a chance of recovering useful knowledge. The second usually produces a meeting nobody wants to attend.

For broad, mostly tacit knowledge, the process can also work the other way around.

A maintainer who has worked on one system for ten years may not know which isolated questions matter most. In that case, the agent can let them narrate the history freely, extract the real decision forks from that story and only then ask targeted follow-up questions.

Free narration and targeted questions are not competing techniques. They are sequential steps for different kinds of missing knowledge.

The important part is that the result becomes project knowledge, not another recording nobody will ever watch again.

### 4\. Maintenance

Documentation can become legacy too.

So the skill also defines how to:

*   update existing topic files
    
*   resolve contradictions
    
*   merge duplicate entries
    
*   mark old decisions as superseded
    
*   identify decisions whose assumptions need review
    
*   split files before they become too large
    
*   keep indexes lean enough for humans and agents
    
*   avoid creating five documents for one small decision
    
*   migrate existing context when the documented format evolves
    

This is living documentation, not an archive of frozen decisions.

## The structure

Keep the Why deliberately separates different kinds of project knowledge.

A typical project can look like this:

```text
project/
├── README.md
├── AGENTS.md
├── AGENTS.local.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── docs/
│   └── index.md
└── context/
    ├── README.md
    ├── index.md
    ├── architecture.md
    ├── synchronization.md
    ├── compatibility.md
    └── incidents.md
```

Each part answers a different question.

| Location | Question |
| --- | --- |
| `README.md` | What is this, and should I care? |
| `AGENTS.md` | Where should an agent look when working in this repository? |
| `docs/` | How do I use, operate, test or deploy this project? |
| `CONTRIBUTING.md` | How do I contribute? |
| Tests | Did this change break something? |
| `CHANGELOG.md` | What changed, release by release? |
| `context/` | Why is the project built this way? |
| `AGENTS.local.md` | What is specific to this developer's local workflow? |

These layers complement each other. None replaces the others.

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

A human opening `context/synchronization.md` sees the same information as an agent reading it.

The context is organised by topic rather than by source file:

```text
context/
├── README.md
├── index.md
├── architecture.md
├── synchronization.md
├── compatibility.md
└── incidents.md
```

`context/README.md` explains the directory to anyone landing there without prior knowledge.

`context/index.md` remains deliberately lean. It helps humans and agents find the relevant topic without loading the complete project history into every session.

I prefer this over a file-by-file shadow tree because decisions often span several files and survive refactors.

I also prefer it over one file per decision for smaller, evolving topics. Formal ADRs still make sense where they fit. Keep the Why is not trying to ban them.

It adapts to an existing repository structure rather than forcing every project into the same template.

## First-time setup

Installing the skill is not the same as deciding how one specific project should use it.

On first activation, Keep the Why runs a short setup instead of silently guessing.

The project-level questions include:

*   Where should the why-knowledge live?
    
*   Should the project begin with continuous capture, retrospective recovery, an interview or a combination?
    
*   Should the Keep the Why badge be added to the README?
    
*   How much confirmation is required before the agent writes to `context/`?
    

Project-wide settings are stored in the committed `AGENTS.md`, for example:

```markdown
<!-- keep-the-why:config -->
- context: `context/`
- init: complete
- context-schema: 0.4.2
- capture-confirmation: confirm-when-unsure
<!-- /keep-the-why:config -->
```

Personal workflow preferences are stored separately in `AGENTS.local.md`, which is excluded from Git:

```markdown
<!-- keep-the-why:local -->
- capture-mode: proactive
- confirmation-flow: sequential
- update-check: every 14 days
- consistency-check: every 30 days
<!-- /keep-the-why:local -->
```

This distinction matters.

Where shared project knowledge lives and how writes to it are governed are project decisions.

Whether one developer wants proactive capture, questions one at a time or occasional update checks is a personal preference.

## How much control should the agent have?

Different developers and teams want different levels of interruption.

Keep the Why therefore separates three questions.

### When should it look?

`capture-mode` controls whether the skill proactively notices capture-worthy reasoning during normal work or only acts when explicitly asked.

The options are:

*   `proactive`
    
*   `explicit-only`
    

### Should it ask before writing?

`capture-confirmation` controls whether an identified entry requires permission before it is written.

The options are:

*   `automatic` — write without asking once the evidence and proportionality checks are satisfied
    
*   `confirm-always` — ask before every write
    
*   `confirm-when-unsure` — write clear cases directly and ask when something is genuinely unclear
    

`confirm-when-unsure` is the default.

Automatic does not mean guessing.

It only removes the permission question when the agent already has enough evidence to write something honest.

A factual clarification remains independent:

> Was this timeout introduced because of a provider limit or because of internal load?

That question may still be necessary even in automatic mode.

“May I write this?” and “What actually happened?” are not the same question.

### One question or a batch?

`confirmation-flow` controls how multiple pending questions are presented.

The options are:

*   `sequential` — one question, one answer, then the next
    
*   `batch` — a numbered list that can be reviewed together
    

Sequential is the default for a developer whose preference is not yet known.

The same preference applies to setup questions, retrospective findings and interview results. The skill does not force ten questions into one message when the developer prefers a simple one-at-a-time dialogue.

## What it looks like in practice

Imagine a normal implementation discussion:

```text
You: The simpler retry mechanism caused duplicate orders during
     reconnects. Keep the stateful version even though it looks
     more complicated.
```

That is not just an implementation detail.

It contains:

*   the chosen behaviour
    
*   the rejected alternative
    
*   the reason it was rejected
    
*   the observed failure mode
    
*   the constraint future maintainers must preserve
    

The agent should recognise that and update the relevant topic in `context/`.

For example:

```markdown
## Stateful retry handling

**Status:** active  
**Evidence:** confirmed

Reconnect retries preserve order state instead of simply repeating the
request.

A stateless retry mechanism was previously considered and rejected because
it produced duplicate orders during reconnects.

The additional state tracking is intentional and must not be removed merely
to simplify the control flow.
```

Six months later, somebody finds the code and thinks:

> This looks over-engineered. I can simplify it.

Chesterton's Fence, except this time the fence has a sign on it.

## The rule that matters most

The core rule of Keep the Why is simple:

> Never invent rationale.

Plausible-sounding historical explanations are worse than missing documentation.

A blank page creates caution.

A confident false explanation creates bad decisions.

Each rationale entry makes two separate classifications.

### Evidence

How strongly is the explanation supported?

*   **confirmed** — supported by a maintainer or authoritative project evidence
    
*   **inferred** — reasonably derived, but not confirmed
    
*   **unknown** — the available evidence is insufficient
    

### Status

Is the decision still current?

*   **active**
    
*   **superseded**
    
*   **open**
    
*   **needs-review**
    

These are different dimensions.

A decision can be superseded today and still be backed by confirmed evidence about why it was correct at the time.

A decision can be active but based only on an inference that should eventually be verified.

Mixing evidence confidence with temporal status hides important information.

The optional source can identify where the explanation came from: a maintainer interview, commit, issue or other evidence.

Old rationale is marked as superseded rather than silently deleted. The history of “we used to do X, then Y happened, so now we do Z” is often exactly what a future maintainer needs.

## Record the fork, not only the outcome

A useful decision has two halves:

1.  What was chosen?
    
2.  What was not chosen, and why?
    

Documentation often records only the final state:

> We use Redis for caching.

That says what exists but not why.

A more useful entry might explain:

> Redis is deliberately used only as a disposable cache. Persistence was evaluated and rejected because recovery complexity would exceed the value of retaining this data.

The rejected path does not have to become a formal essay.

Even one honest sentence can prevent the same alternative from being reconsidered without the information that previously ruled it out.

This does not mean inventing alternatives for every obvious implementation detail. A rejected alternative belongs in the record only when it was genuinely in contention.

## Proportionality

Not every decision deserves a structured entry.

A self-explanatory convention may need one sentence.

A surprising architectural boundary may need:

*   context
    
*   decision
    
*   alternatives
    
*   consequences
    
*   evidence
    
*   failure modes
    
*   conditions under which it should be revisited
    

“Prevents a breaking API change” probably deserves an entry.

“Formats the code more nicely” probably does not.

The goal is useful context, not context bloat.

When the line is genuinely unclear, the skill asks a focused question instead of silently deciding in either direction.

## Ambiguity must be clarified

The rule against guessing applies beyond historical rationale.

It also applies to configuration and user instructions.

A missing setting with an explicitly documented default can use that default.

A present but invalid value cannot.

For example:

```text
confirmation-flow: grouped
```

The valid values are `sequential` and `batch`.

The skill should not silently translate `grouped` into whichever option appears closest. It should name the valid choices and ask what was intended.

The same applies to contradictory settings, ambiguous instructions and likely typos.

Guessing correctly by luck is still guessing.

## Context versioning and migrations

The skill itself can evolve, and so can the structure of existing `context/` entries.

Keep the Why therefore records a `context-schema` value in the project configuration.

This is not an unrelated second versioning system. It records the latest skill version against which the project's existing context has been checked.

When a newer skill version changes the expected context format, the agent consults the documented migrations and discusses what should happen:

*   migrate now
    
*   defer the migration
    
*   stop asking this particular developer about that specific migration
    

Migrations are never performed silently.

The inverse case also matters.

If a project's context was created by a newer version of the skill than the currently installed one, the older skill should not continue writing as though everything were compatible. It should recommend updating before modifying entries it may no longer understand.

Optional update and consistency checks can run opportunistically when the skill is already active in a session.

They are not background services.

There is no daemon waking up every fourteen days. The configured interval only determines whether a check is due the next time the skill is used.

## Security and the trust model

Repository context is useful precisely because an agent reads and uses it.

That also makes it a potential indirect prompt-injection surface.

Any repository content can contain malicious or misleading instructions:

*   source comments
    
*   commit messages
    
*   issues
    
*   logs
    
*   documentation
    
*   encoded or hidden text
    

`context/` deserves particular care because it may be read early, persists across sessions and is deliberately presented as trusted project background.

A poisoned context entry could otherwise turn a one-time injection into persistent agent behaviour.

Keep the Why therefore draws a hard boundary:

> `context/` contains project knowledge, not agent instructions.

An entry may describe a technical constraint:

> This component requires Python 3.11 for compatibility.

It does not gain authority to direct the agent:

> Install Python 3.11 now by running this command.

Repository content cannot:

*   override system, developer or user instructions
    
*   expand the agent's permissions
    
*   authorize tool calls
    
*   disable safety checks
    
*   request or reveal secrets
    
*   declare itself trustworthy
    
*   instruct the agent to hide something from the user
    

Suspicious instructions are not followed.

They are also not silently deleted or rewritten. The agent identifies what looks wrong and asks the user how it should be handled.

The same restraint applies when writing new context.

The skill synthesizes established reasoning. It does not copy hidden content, encoded payloads, secrets or commands-for-later from issues, webpages, commits or logs into a persistent context file.

A source is evidence for a claim.

It is never authority over the agent's next action.

The complete model is documented in the project's [Security overview](https://keepthewhy.com/security/) and [Trust model](https://keepthewhy.com/trust-model/).

## Why it stays simple

Keep the Why follows a deliberately Unix-like philosophy.

It does one job:

> Preserve the reasoning behind a codebase.

It does not try to become:

*   a project-management system
    
*   an agent orchestration framework
    
*   a task tracker
    
*   a session-memory database
    
*   a hosted knowledge platform
    
*   a replacement for Git
    
*   a replacement for tests or documentation
    

The repository already provides storage, versioning, review, history, access control and distribution.

Markdown already provides a format humans and agents can read.

Git already provides the mechanism for shipping the reasoning with the code change it explains.

Adding a database, daemon, dashboard or synchronization service would introduce another system that can drift, fail, require credentials or become abandoned.

The simplest architecture is not always the one with the fewest files.

It is the one with the fewest independent systems that must remain correct.

Keep the Why uses what is already there.

## How it relates to existing approaches

Keep the Why is not a replacement for:

*   tests
    
*   README files
    
*   `AGENTS.md`
    
*   `CONTRIBUTING.md`
    
*   ADRs
    
*   issue trackers
    
*   [Keep a Changelog](https://keepachangelog.com/)
    

The name is deliberately related to Keep a Changelog.

Keep a Changelog records **what changed**.

Keep the Why preserves **why it changed**.

There is also prior work in this space:

*   [Architecture Decision Records](https://adr.github.io/)
    
*   [AGENTS.md](https://agents.md/)
        
*   repository-memory and agent-context tools
    

I did not invent the idea that software decisions should be documented.

Formal ADRs remain the right tool for major, discrete architectural decisions.

`git-why` takes a useful source-file-oriented approach.

Agent Decision Records provide more deliberate, checkpointed records.

`AGENTS.md` provides a common entry point that tells agents where to look.

What I wanted was a practical combination of:

*   continuous capture
    
*   retrospective recovery
    
*   code-guided knowledge-transfer interviews
    
*   topic-based living documentation
    
*   explicit evidence and status
    
*   configurable human confirmation
    
*   repository-native storage
    
*   cross-agent compatibility
    
*   no required external platform
    
*   an explicit trust model for persistent context
    

That combination grew out of how I was already working.

## What this is not

Keep the Why is not a guarantee.

It cannot preserve reasoning that was never expressed, and it cannot recover historical knowledge for which no evidence remains.

It does not magically keep documentation correct forever. Entries still need review, contradiction resolution and maintenance.

It is not session memory and does not attempt to record everything an agent or developer did.

It is not an activity log or conversation transcript.

It is not project management, task tracking or workflow orchestration.

It is not a trust boundary around repository content.

It is not a replacement for tests.

Tests tell you whether a change broke expected behaviour.

Keep the Why tells you why the behaviour existed in the first place.

## Where the project stands

The underlying practice is not completely new to me.

I have been testing variations of this approach in my own AI-assisted development workflow for several months.

But the public project is not a copy of that private setup.

It is a new incarnation of it:

*   cleaned up
    
*   generalised
    
*   made cross-agent
    
*   turned into an explicit method
    
*   separated from my own repository conventions
    
*   packaged as an open-source skill
    
*   tested against its own repository
    
*   backed by examples, references and evaluation cases
    

Since the initial release, the project has gained:

*   first-time project setup
    
*   personal workflow preferences
    
*   configurable capture confirmation
    
*   sequential or batch question flows
    
*   context status and evidence as separate dimensions
    
*   schema tracking and documented migrations
    
*   optional update and staleness checks
    
*   safer pinned installation methods
    
*   validation against the Agent Skills specification
    
*   an explicit security and prompt-injection trust model
    

That does not mean the method has finished proving itself.

The real test is what happens across different repositories, teams and agents.

Questions still remain:

*   When does the agent capture too much?
    
*   When does it ask too little?
    
*   Which structure works for a small library and which one works for a large multi-repository system?
    
*   How much classification is useful before it becomes bureaucracy?
    
*   When should a topic be split?
    
*   How do we keep old rationale from becoming trusted but stale?
    
*   How well do different agents follow the same skill?
    
*   Which parts of the process should eventually be checked automatically?
    

There is also a small but relevant body of supporting research.

[A 2026 position paper](https://arxiv.org/abs/2601.21116) audited 62 architectural decisions across two internal projects and found that roughly 23% had stale supporting evidence within two months.

The sample is small, non-replicated and concerned traditional ADRs rather than AI-generated decisions. It is not proof of anything universal.

But it supports something maintainers already know:

Documentation does not stay correct merely by existing. It has to be maintained.

I do not yet have published cross-agent evaluation results or a formal before-and-after study.

The evaluation cases exist.

The public empirical evidence is still limited.

That is the honest state of the project.

## Try it

`main` is active development and is not guaranteed to be release-ready at every commit.

The recommended installation therefore pins to the floating `latest` release tag. Use an exact version tag instead when full reproducibility matters.

With the [skills CLI](https://skills.sh/):

```bash
npx skills add https://github.com/oliver-zehentleitner/keep-the-why/tree/latest/skills/keep-the-why
```

The CLI lets you choose a supported agent and whether the skill should be installed for one project or for your user account.

With GitHub CLI 2.90.0 or newer, you can inspect the skill before installing it:

```bash
gh skill preview oliver-zehentleitner/keep-the-why keep-the-why
```

Then install the latest released version:

```bash
gh skill install oliver-zehentleitner/keep-the-why keep-the-why@latest
```

After installation, start a new agent session and say:

```text
Initialize Keep the Why in this project.
```

That explicit prompt is needed only once.

The setup writes the project pointers into `AGENTS.md`, which compatible tools can read in later sessions without being reminded again.

It remains instructions only:

*   no executable scripts
    
*   no binaries
    
*   no network access of its own
    
*   no external service
    
*   no account
    
*   no API key
    
*   no database
    
*   no daemon
    

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

If the structure is wrong for your project, I want to know why.

If an important workflow is missing, propose it.

If another project already solves part of this better, point me to it.

The project is open source because I do not think this should become another private knowledge silo.

The whole point is to keep the reasoning with the code.

Because “ask Bob” is not documentation.