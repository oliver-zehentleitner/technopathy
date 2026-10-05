---
title: "Keep the Why vs. OKF Agent Memory vs. RepoWise: Three Approaches to Git-Native Project Memory"
datePublished: 2026-10-05T15:21:42.738Z
cuid: cmuvee2hd000006licknc3rbq
slug: keep-the-why-vs-okf-agent-memory-vs-repowise-three-approaches-to-git-native-project-memory
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/dea28336-b78f-4621-8036-b39982cdd5cb.png
tags: software-architecture, git, ai-agents, context-engineering, coding-agents, keepthewhy, project-memory

---

# Keep the Why vs. OKF Agent Memory vs. RepoWise

## Three approaches to Git-native project memory for coding agents

"Project memory for coding agents" is becoming a surprisingly broad category.

A few months ago, many tools in this space were easy to separate. Some stored chat history. Some put memories into a database. Some built vector stores. Others added persistent files to a repository.

That distinction is getting harder.

Three current projects are particularly interesting because all of them are repository-oriented, all of them care about knowledge surviving individual AI sessions, and all of them can represent or recover architectural decisions:

- [Keep the Why](https://keepthewhy.com/)
- [OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory)
- [RepoWise](https://repowise.dev/)

At first glance, they overlap heavily.

They do.

But after looking at how each one actually works, I think they represent three different interpretations of project memory:

**Keep the Why preserves rationale.**

**OKF Agent Memory structures durable knowledge.**

**RepoWise derives codebase intelligence.**

That difference matters much more than the feature lists.

I previously compared [Keep the Why with Claude Code Auto Memory, MemoryCustodian, and AgentsRoom](https://blog.technopathy.club/keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom). This comparison is different. OKF Agent Memory and RepoWise operate much closer to the repository itself, which makes the architectural boundaries more interesting.

A note before going further: I built Keep the Why, so this obviously is not a neutral product review. I am interested in comparing the architectures, their boundaries, and the tradeoffs behind them rather than declaring a winner.

This article is a snapshot from **October 5, 2026**.

---

## The short version

If I had to reduce all three projects to one sentence each:

**[Keep the Why 0.20.0](https://github.com/oliver-zehentleitner/keep-the-why)** is a complete, deliberately narrow rationale system for preserving the why behind a codebase as Markdown in the repository.

**[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory)** is a broader Git-native knowledge architecture that structures persistent agent knowledge and provides dedicated local retrieval.

**[RepoWise](https://repowise.dev/)** is a codebase intelligence system that indexes the repository, derives multiple intelligence layers, and includes architectural decisions as one of them.

The overlap is real.

The center of gravity is different.

---

# Keep the Why 0.20.0: preserve the part the repository usually loses

Keep the Why starts from a simple observation:

**A software repository already contains a remarkable amount of memory.**

The source tells us what exists.

Tests tell us what behavior is expected.

Documentation tells us how things work.

Git tells us what changed, when, and by whom.

What is often missing is the reasoning behind all of it.

Why was this architecture chosen?

Why is this awkward workaround still there?

What alternative was already tried?

Why did it fail?

Which operational constraint is invisible from the source code?

[Keep the Why](https://keepthewhy.com/) exists only for that layer.

It stores the reasoning behind the code as plain Markdown under `context/`. The files live with the project and can be versioned, reviewed, distributed, branched, and merged through Git like the rest of the repository.

The important part is how that information gets there.

Keep the Why is an agent skill. During normal development, reasoning already appears in the conversation between developer and coding agent. Instead of throwing durable reasoning away when the session ends, the agent recognizes what belongs to the project and records it while the context is still fresh.

The current skill covers four core workflows:

1. **Continuous capture** during normal development.
2. **Retrospective recovery** for existing projects where rationale has to be reconstructed from code, Git, issues, documentation, and other evidence.
3. **Knowledge-transfer interviews** when important reasoning still exists mainly in someone's head.
4. **Maintenance** of rationale already recorded in the project.

With version **0.20.0**, I consider that core complete.

That does not mean Keep the Why can never change again. It means the problem, the boundaries, the structure, and the fundamental workflow are now deliberately stable.

Adding functionality simply because it is technically possible would work against the design.

---

## Keep the Why is intentionally not general memory

This boundary is important.

Keep the Why does not try to remember everything an agent ever knew.

It is not:

- a chat archive
- a task manager
- an agent orchestrator
- a general company knowledge base
- a vector database
- a replacement for documentation
- a replacement for Git history
- a project-management system

It has one job:

**Preserve the why behind a codebase.**

That includes architectural decisions, rejected alternatives, workarounds, incident learnings, constraints, and other reasoning that the code itself cannot explain.

The distinction is more important than it may initially sound.

A general memory system naturally asks:

> What information might this agent need again?

Keep the Why asks a narrower question:

> What reasoning would be expensive or impossible to reconstruct later?

That narrower scope is the architecture.

---

## More than a folder of Markdown

Calling Keep the Why "Markdown in `context/`" is technically correct, but no longer tells the whole story.

The [Keep the Why specification](https://keepthewhy.com/specification/) defines the structure and semantics of the project memory.

Entries have stable IDs. They can express status, evidence, sources, relationships, revisit conditions, and links to other rationale.

The [linter](https://github.com/oliver-zehentleitner/keep-the-why) validates the mechanically checkable parts locally and in CI. It deliberately does not pretend to determine whether a rationale is true. That remains a human and agent responsibility.

There is also a [read-only dashboard](https://keepthewhy.com/dashboard/) over `context/`, `.keep-the-why`, linter findings, and Git history.

It can show:

- rationale entries and topics
- history and status changes
- authors
- unresolved or unconfirmed reasoning
- relationships between entries
- family members
- friends
- cross-repository thoughts
- the project graph

The architectural rule is more important than the UI:

**The dashboard writes nothing into the project and owns none of the project memory.**

Delete it and nothing is lost.

It is a lens over Markdown and Git.

I wrote more about that graph model in [The reasoning behind a codebase, as a web you can walk](https://blog.technopathy.club/the-reasoning-behind-a-codebase-as-a-web-you-can-walk).

---

# Project memory beyond one repository

Keep the Why also no longer assumes that one project always means one repository.

A project can be:

- a single repository
- a monorepo
- multiple related repositories
- nested repositories

Related repositories can form a **family**.

Repositories outside that family can still become **friends** when rationale in one repository references rationale in another.

Stable entry IDs allow those relationships to survive heading changes and file reorganizations.

When linked entries form a chain across repositories, the dashboard can display that chain as a **thought**.

This eventually led to the [Keep the Why Registry](https://keepthewhy.com/registry/), which adds discovery and reverse references between published projects without becoming a central storage system.

The registry does not own the rationale either.

It only helps projects find one another.

I described that model in more detail in [The globe: following your reasoning into other people's repositories](https://blog.technopathy.club/the-globe-following-your-reasoning-into-other-people-s-repositories).

The principle stays consistent:

**The tools read the project memory. They do not become the project memory.**

---

# OKF Agent Memory: treat project memory as structured knowledge

[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory) starts from a much broader premise.

Instead of asking only how to preserve software rationale, it asks how an agent can maintain a persistent body of structured knowledge.

Its `knowledge/` bundle can represent architectural decisions, facts, processes, domain knowledge, API contracts, operational information, and many other forms of durable knowledge.

That immediately puts it in a different category from Keep the Why.

An architectural decision is one type of knowledge in OKF.

In Keep the Why, decision rationale is the reason the system exists.

OKF Agent Memory implements the [Open Knowledge Format](https://github.com/okf-memory/okf-agent-memory) and builds conventions and tooling around it.

The project describes a dual-memory architecture:

- a compact push layer for rules and invariants that should always reach the agent
- a larger pull layer containing durable project knowledge that is retrieved when needed

This is a very different response to context growth.

Instead of expecting the agent to navigate the knowledge bundle entirely through normal file operations, OKF makes retrieval a first-class capability.

---

## Search is part of the OKF architecture

The local `okf` tool provides in-memory BM25 retrieval over the knowledge bundle.

For example:

```bash
okf search "authentication architecture" knowledge
okf show decisions/auth knowledge
```

It can also expose the same functionality through an embedded MCP server.

The current tooling includes operations for searching, showing, creating, updating, relating, and validating concepts.

No external vector database or embedding API is required for that retrieval path.

That is worth emphasizing.

OKF adds a dedicated search layer, but it remains local and operates over repository-owned plain-text knowledge.

The search mechanism is infrastructure, but not a new canonical source of truth.

The knowledge files remain the knowledge.

---

# OKF has a broader memory hierarchy

OKF Agent Memory also extends beyond project-local knowledge.

Its current architecture defines four scopes:

**Project**

Knowledge belonging directly to the current repository.

**Vendor**

Reusable knowledge packages or external knowledge dependencies.

**User**

Personal knowledge and preferences that can survive across projects.

**System**

Machine-level or organizational knowledge such as policies and infrastructure constraints.

Those scopes have deterministic precedence.

Project knowledge can override a more general concept coming from a vendor, user, or system layer.

There is also an optional Hub architecture for synchronization.

This makes OKF considerably more ambitious than a repository convention.

It is building a general architecture for persistent agent knowledge: representation, provenance, trust, lifecycle, retrieval, scopes, packaging, synchronization, and composition.

That breadth is one of its strengths.

It is also why its center of gravity is fundamentally different from Keep the Why.

---

# RepoWise: derive an intelligence model from the codebase

[RepoWise](https://repowise.dev/) starts somewhere else again.

It is not primarily a memory format.

It is a **codebase intelligence engine**.

RepoWise indexes a repository and derives several layers of information from it, including:

- dependency and symbol structure
- Git history and co-change information
- code health signals
- generated documentation
- architectural decisions
- dead-code and risk information
- cross-repository intelligence in workspace mode

That information is exposed to coding agents through dedicated MCP tools such as `get_context`, `search_codebase`, `get_risk`, and [`get_why`](https://docs.repowise.dev/mcp/get-why).

This means RepoWise tries to build an additional model of the repository rather than requiring all useful knowledge to exist explicitly as repository documents.

That is the biggest architectural difference in this comparison.

---

# RepoWise Decisions come surprisingly close to the Why

The reason RepoWise belongs in this article is its [architectural decisions layer](https://repowise.dev/features/decisions).

It is very close to the problem Keep the Why is interested in.

RepoWise can gather decision evidence from several places where teams already leave reasoning, including ADRs, Git history, pull requests, comments, inline rationale markers, documentation, and coding-agent sessions.

A decision record can contain:

- context
- the decision itself
- rationale
- rejected alternatives
- consequences
- affected files
- tags
- evidence
- currency and authority information

RepoWise then connects those records to the files they govern.

An agent can call [`get_why`](https://docs.repowise.dev/mcp/get-why) before modifying a file and retrieve accepted decisions, candidates, history, evidence, related documentation, and fallback archaeology.

When no accepted decision exists, RepoWise can fall back to Git history and rationale comments instead of pretending certainty exists.

That distinction is important.

RepoWise separates **candidates** from **accepted decisions**.

Something extracted from history or an agent transcript does not govern the project just because the system found it.

Acceptance is an explicit authority boundary.

The current documentation is very clear about that:

A candidate governs nothing until somebody explicitly confirms it.

---

## RepoWise also has a Git-tracked decision manifest

RepoWise has an interesting hybrid architecture.

Accepted decisions can be exported to:

```text
.repowise/decisions.yaml
```

The file can be committed with the repository.

When it is imported again, RepoWise treats the tracked file as authoritative over the local store.

In other words:

**The file wins. The index is the copy.**

That moves the accepted decision layer closer to the philosophy behind Keep the Why and OKF.

But the wider RepoWise system remains different.

The dependency graph, search indices, wiki, Git-derived signals, code health, and other intelligence are still computed layers built by RepoWise.

That is not a criticism.

It is exactly where much of RepoWise's power comes from.

It simply solves a different problem.

---

# Three architectures

The easiest way to understand the difference is to reduce each project to its fundamental flow.

## Keep the Why

```text
developer + coding agent
          |
          v
reasoning surfaces while working
          |
          v
       context/
          |
          v
   Markdown + Git
          |
          v
future agent / human
```

The agent is the interface.

The files are the memory.

Git can provide storage, history, review, and distribution, but Keep the Why also works as plain local files without Git.

No retrieval infrastructure is required.

---

## OKF Agent Memory

```text
project knowledge
       |
       v
structured concepts
       |
       v
    knowledge/
       |
       v
 local BM25 / CLI / MCP
       |
       v
      agent
```

The files remain the canonical knowledge corpus.

A dedicated local retrieval layer helps the agent find and manipulate relevant concepts.

---

## RepoWise

```text
code + Git + ADRs + PRs + comments + sessions
                      |
                      v
                RepoWise analysis
                      |
                      v
 graph + git + wiki + decisions + health + index
                      |
                      v
                     MCP
                      |
                      v
                    agent
```

RepoWise builds a derived intelligence model from the repository and surrounding evidence.

---

# Side by side

| | Keep the Why 0.20.0 | OKF Agent Memory | RepoWise |
|---|---|---|---|
| Primary purpose | Preserve software rationale | Persistent structured agent knowledge | Codebase intelligence |
| Core unit | Rationale entry | Knowledge concept | Derived intelligence / decision record |
| Scope | Deliberately narrow | Broad and domain-neutral | Broad software-engineering analysis |
| Canonical project knowledge | Markdown in `context/` | Plain-text OKF bundle in `knowledge/` | Repository plus optional Git-tracked decision manifest |
| Main capture model | Agent captures reasoning while it happens | Agent creates and maintains structured concepts | Mine, derive, capture, review, and confirm |
| Continuous session capture | Core workflow | Supported through agent workflow | Session mining available |
| Retrospective recovery | Explicit workflow | Possible by creating concepts from existing evidence | Core strength through repository archaeology |
| Knowledge-transfer interviews | Explicit workflow | Not a primary specialization | Not a primary specialization |
| Dedicated retrieval required | No | No external service, but local search is central | Yes, the derived index is central |
| Local search | Normal file tools today | Built-in BM25 | Keyword, semantic, graph, and specialized retrieval |
| Database required for core project memory | No | No | Derived index uses local storage |
| Vector retrieval required | No | No | Used by parts of the intelligence/retrieval system |
| MCP required | No | No, optional | Main agent integration |
| Architectural decisions | Core purpose | One knowledge type among many | One intelligence layer among many |
| Rejected alternatives | First-class | Representable as structured knowledge | Supported in decision records |
| Evidence / provenance | Explicit | Strong provenance and trust model | Explicit evidence and authority model |
| Staleness | Maintenance and review | Lifecycle metadata and validation | Code-linked staleness and health |
| Cross-project model | Families, friends, thoughts | Project, vendor, user, system scopes | Multi-repo workspaces and cross-repo queries |
| Human-readable without special tooling | Yes | Yes | Accepted manifest yes, full intelligence requires RepoWise |
| External service required | No | No | No for local self-hosted operation |
| Core philosophy | Preserve only the missing why | Build durable structured knowledge | Derive understanding from the codebase |

---

# The most important distinction: capture, structure, and reconstruction

Imagine a developer and a coding agent spend an hour investigating a retry mechanism.

The obvious simplification looks good.

They try it.

Under one specific failure mode it causes duplicate orders.

They revert the experiment and decide to leave the existing ugly implementation alone.

No production code changes.

No pull request.

Possibly no commit worth keeping.

But the project just learned something extremely valuable.

What happens to that knowledge?

---

## Keep the Why: capture the learning when it exists

For Keep the Why, this is exactly the event it was designed around.

The conversation contains durable rationale.

The agent records:

- what was considered
- what was tried
- why it failed
- which constraint matters
- when the decision should be revisited

The interesting artifact is not a code change.

There may not even be one.

The artifact is what the project learned.

This is why continuous capture matters.

Once the working context disappears, Git archaeology may never be able to reconstruct what happened.

---

## OKF Agent Memory: turn the learning into durable knowledge

OKF can represent the same discovery as a structured decision or a set of related knowledge concepts.

Those concepts can carry provenance, relationships, lifecycle metadata, and descriptions optimized for later retrieval.

The focus is broader than preserving rationale alone.

The new information becomes part of the project's durable knowledge corpus.

Later, the agent retrieves it through the OKF search layer.

---

## RepoWise: connect recorded evidence to the code

RepoWise is strongest when useful evidence exists somewhere it can inspect.

That might be:

- an ADR
- a commit
- a PR discussion
- a rationale comment
- an agent transcript
- a manually accepted decision

RepoWise can connect those pieces, associate them with governed code, track whether the code moved afterwards, and surface them when an agent touches the affected area.

If explicit rationale does not exist, it can perform archaeology and keep inferred candidates separate from accepted authority.

That is powerful.

But capture and archaeology solve different failure modes.

If knowledge was never written anywhere, reconstruction always has a ceiling.

---

# Where the three projects genuinely overlap

This comparison stops being a simple taxonomy because the edges are moving closer together.

Keep the Why began with an intentionally small premise, but it now has a defined schema, evidence, relationships, lifecycle semantics, a linter, multi-repository structure, a dashboard, graphs, and a registry.

OKF starts with general structured knowledge, but architectural decisions and repository-local project memory are obvious central use cases.

RepoWise begins with derived codebase intelligence, but its decision system now supports explicit rationale, alternatives, evidence, governed files, staleness, review, authority, and a Git-tracked decision manifest.

So yes, these systems overlap.

Quite substantially in places.

But their centers of gravity remain different.

That matters more than whether two rows in a feature matrix happen to contain checkmarks.

---

# What happens when the memory gets large?

This is one of the most interesting architectural differences.

OKF already assumes that a sufficiently large knowledge corpus benefits from dedicated retrieval.

Its answer is intentionally lightweight: local in-memory BM25 search over the knowledge bundle.

RepoWise goes much further because retrieval is fundamental to the product. It builds several indices and intelligence layers to support code search, architectural context, decisions, risk analysis, and other queries.

Keep the Why currently does neither.

That is intentional.

`context/` is normal project data.

Agents already know how to read files, inspect an index, follow stable IDs, and use tools such as `grep` or `rg`.

It would be easy to add a database or a retrieval service now.

But that would mean building infrastructure for a scaling problem that has not actually been observed yet.

There is already an open [Keep the Why issue about learning scaling limits from real projects](https://github.com/oliver-zehentleitner/keep-the-why/issues/256).

The principle behind it is simple:

**Do not invent a threshold before real repositories show where the threshold is.**

I think retrieval should follow the same rule.

If a future `context/` corpus becomes large enough that normal file access stops being convenient, the answer may be extremely small.

For example:

```bash
ktw search "retry duplicate orders"
```

That command could scan `context/`, optionally maintain a disposable local cache, and return a shortlist of likely relevant entries:

```text
8f31...  Retry strategy rejected after duplicate-order incident
a72c...  Queue consumer idempotency constraint
19de...  Why retries happen above the transport layer
```

The agent can then read the selected Markdown files normally.

The crucial architectural rule would remain:

**`context/` is the source of truth.**

Any search index should be local, optional, derived, disposable, and completely rebuildable from the files.

Maybe that tool will eventually be useful.

Maybe `grep` will remain sufficient for far larger projects than expected.

There is no need to decide before the problem exists.

---

# "Git-native" means different things here

All three projects can reasonably describe themselves as Git-native or repository-native.

But that phrase hides meaningful differences.

## Keep the Why

For Keep the Why, the files themselves are the entire durable project-memory layer.

Git adds versioning, review, collaboration, branching, and distribution.

There is no separate authoritative store.

## OKF Agent Memory

For OKF, Git also stores the canonical project knowledge.

Dedicated tooling provides structured retrieval, validation, graph operations, and multiple memory scopes around those files.

## RepoWise

For RepoWise, the repository is both source material and an integration point.

Accepted decisions can live in Git as `.repowise/decisions.yaml`, while the larger codebase-intelligence model is derived and indexed separately.

All three relationships with Git are legitimate.

They are not the same architecture.

---

# Which one solves which problem?

The useful question is not:

> Which project has the most memory features?

It is:

> What am I trying to preserve or understand?

If your problem is:

> "Our coding agents keep forgetting why this code exists, repeat rejected approaches, or lose reasoning that never became a commit."

That is exactly the problem [Keep the Why](https://keepthewhy.com/) is designed for.

If your problem is:

> "We want a broader, structured, persistent knowledge architecture for agents, including project knowledge, reusable packages, user knowledge, and local retrieval."

[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory) is much closer to that problem.

If your problem is:

> "We want agents to understand a large existing codebase, including architecture, history, dependencies, risks, decisions, documentation, and why particular files are shaped the way they are."

That is where [RepoWise](https://repowise.dev/) becomes especially interesting.

These are not mutually exclusive requirements either.

A project could theoretically use Keep the Why as its human-readable rationale layer and another system for broader retrieval or code intelligence.

The important question is which system owns which truth.

---

# Could they coexist?

Yes, and this is where the boundaries become useful.

A tool like RepoWise could index a repository that already contains `context/`.

OKF could represent broader organizational or project knowledge while Keep the Why remains responsible specifically for code rationale.

A future local Keep the Why search command could improve retrieval without changing the underlying storage model at all.

Architectures become messy when multiple systems silently compete to be authoritative for the same information.

They become much easier to reason about when each layer has a clearly defined responsibility.

That is why I increasingly think "project memory" is not one thing.

It is a stack.

Source code is memory.

Tests are memory.

Documentation is memory.

Git history is memory.

General project knowledge can be memory.

Derived codebase intelligence can be memory.

And rationale is memory.

The interesting design question is not how to move all of them into one database.

It is how to make sure each kind of information has the simplest durable home that fits it.

---

# What I like about all three approaches

The most encouraging part of this comparison is that all three projects move away from the assumption that durable agent memory has to mean sending everything into an opaque external memory service.

They take repository ownership seriously.

They make durable information inspectable.

They try to preserve provenance.

And they acknowledge, in different ways, that an AI agent should not have to reconstruct the entire history and reasoning of a project from scratch every time a new session starts.

That is a much more interesting direction than simply making context windows larger.

---

# Conclusion

Keep the Why, OKF Agent Memory, and RepoWise overlap enough to look like competitors from a distance.

Up close, they are answers to different questions.

**Keep the Why asks:**

> How do we preserve the reasoning behind a codebase without turning that reasoning into another platform?

**OKF Agent Memory asks:**

> How do we represent, retrieve, compose, and govern durable knowledge for agents?

**RepoWise asks:**

> How do we derive a useful intelligence model from a codebase and put that understanding in front of an agent when it matters?

My current shorthand is:

**Keep the Why is a complete, deliberately narrow rationale system.**

**OKF Agent Memory is a general knowledge architecture.**

**RepoWise is a codebase intelligence system with a sophisticated decision layer.**

None of those descriptions makes one inherently better than the others.

They optimize for different things.

And I suspect watching those boundaries evolve will be more interesting than watching another wave of tools compete on how many memories they can store.

---

## Links and further reading

### Keep the Why

- [Keep the Why](https://keepthewhy.com/)
- [GitHub repository](https://github.com/oliver-zehentleitner/keep-the-why)
- [Specification](https://keepthewhy.com/specification/)
- [Installation](https://keepthewhy.com/installation/)
- [Dashboard](https://keepthewhy.com/dashboard/)
- [Registry](https://keepthewhy.com/registry/)
- [Scaling observation issue #256](https://github.com/oliver-zehentleitner/keep-the-why/issues/256)

### OKF Agent Memory

- [OKF Agent Memory on GitHub](https://github.com/okf-memory/okf-agent-memory)
- [Getting Started](https://github.com/okf-memory/okf-agent-memory/blob/develop/docs/guides/GETTING_STARTED.md)
- [Current README and memory scopes](https://github.com/okf-memory/okf-agent-memory/blob/develop/README.md)

### RepoWise

- [RepoWise](https://repowise.dev/)
- [Architectural Decisions](https://repowise.dev/features/decisions)
- [Decision intelligence documentation](https://docs.repowise.dev/intelligence/decisions)
- [`get_why` documentation](https://docs.repowise.dev/mcp/get-why)
- [Decision CLI and Git-tracked manifest](https://docs.repowise.dev/cli/decision)
- [MCP tools overview](https://docs.repowise.dev/mcp/overview)

### Related articles

- [Keep the Why vs. Claude Code Auto Memory vs. MemoryCustodian vs. AgentsRoom](https://blog.technopathy.club/keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom)
- [The reasoning behind a codebase, as a web you can walk](https://blog.technopathy.club/the-reasoning-behind-a-codebase-as-a-web-you-can-walk)
- [The globe: following your reasoning into other people's repositories](https://blog.technopathy.club/the-globe-following-your-reasoning-into-other-people-s-repositories)

---

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), Bluesky, Mastodon, X, and LinkedIn, or join Telegram for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\_(ツ)_/¯