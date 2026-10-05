---
title: "Keep the Why vs. OKF Agent Memory vs. RepoWise: Three Approaches to Git-Native Project Memory"
datePublished: 2026-10-05T15:21:42.738Z
cuid: cmuvee2hd000006licknc3rbq
slug: keep-the-why-vs-okf-agent-memory-vs-repowise-three-approaches-to-git-native-project-memory
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/dea28336-b78f-4621-8036-b39982cdd5cb.png
tags: software-architecture, git, ai-agents, context-engineering, coding-agents, keepthewhy, project-memory

---

## Three approaches to Git-native project memory for coding agents

"Project memory for coding agents" is becoming a broad category. Three projects are especially interesting because all are repository-oriented, all preserve knowledge beyond individual AI sessions, and all can deal with architectural decisions:

- [Keep the Why](https://keepthewhy.com/)
- [OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory)
- [RepoWise](https://repowise.dev/)

They overlap, but they start from different questions.

**Keep the Why preserves rationale.**

**OKF Agent Memory structures durable knowledge.**

**RepoWise derives codebase intelligence.**

I previously compared [Keep the Why with Claude Code Auto Memory, MemoryCustodian, and AgentsRoom](https://blog.technopathy.club/keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom). This comparison is different because OKF Agent Memory and RepoWise operate much closer to the repository itself.

A note before going further: I built Keep the Why, so this is not a neutral product review. I am interested in the architectures, their boundaries, and the tradeoffs behind them rather than declaring a winner.

This article is a snapshot from **October 5, 2026**.

## The short version

**[Keep the Why 0.20.0](https://github.com/oliver-zehentleitner/keep-the-why)** is a complete, deliberately narrow rationale system. It preserves the why behind a codebase as Markdown in the repository.

**[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory)** is a broader Git-native knowledge architecture. It structures persistent knowledge and provides dedicated local retrieval over it.

**[RepoWise](https://repowise.dev/)** is a codebase intelligence system. It indexes the repository, derives multiple intelligence layers, and includes architectural decisions as one of them.

The feature overlap is real. The architectural responsibility is different.

## Keep the Why 0.20.0: preserve the part the repository usually loses

A software repository already contains a large amount of memory. Source code tells us what exists, tests describe expected behavior, documentation explains how things work, and Git records what changed.

What is often missing is the reasoning behind all of it.

Why was this architecture chosen? Why is an awkward workaround still there? Which alternative was tried and rejected? Which operational constraint is invisible from the code?

[Keep the Why](https://keepthewhy.com/) exists only for that layer.

It stores rationale as plain Markdown under `context/`. The files live with the project and can be versioned, reviewed, branched, merged, and distributed through Git like the rest of the repository.

The key mechanism is capture while the reasoning exists. Keep the Why is an agent skill, so when durable reasoning surfaces during normal development, the agent records it instead of letting it disappear with the session.

The current skill covers continuous capture during development, retrospective recovery from existing evidence, knowledge-transfer interviews, and maintenance of recorded rationale.

With version **0.20.0**, I consider that core complete: the problem, boundaries, structure, and fundamental workflow are deliberately stable.

### Deliberately narrow

Keep the Why is not a chat archive, task manager, agent orchestrator, vector database, project-management system, or general company knowledge base.

It has one job: **preserve the why behind a codebase**, including architectural decisions, rejected alternatives, workarounds, incident learnings, and constraints the code cannot explain.

A general memory system asks what information an agent may need again. Keep the Why asks what reasoning would be expensive or impossible to reconstruct later.

That narrow scope is intentional, not a missing feature set.

### From ADR tool to continuous why layer

Keep the Why grew out of the Architecture Decision Record problem. Classic ADRs are excellent for a small number of major, discrete decisions, but they depend on somebody deliberately writing a record and usually model one decision per file. Keep the Why expanded that idea into continuous, agent-maintained rationale: smaller decisions, rejected changes, workarounds, constraints, and incident learnings captured while the reasoning is present.

The ADR connection is still explicit today. Keep the Why is listed in the [Architecture Decision Record community project's tools and resources](https://github.com/architecture-decision-record/architecture-decision-record), while its own [ADR comparison](https://keepthewhy.com/faq/#how-is-this-different-from-an-adr-architecture-decision-record) describes ADRs as complementary rather than obsolete.

### Structure without moving the source of truth

The [Keep the Why specification](https://keepthewhy.com/specification/) defines stable IDs, status, evidence, sources, relationships, and revisit conditions. A linter checks the mechanical rules locally and in CI.

The [read-only dashboard](https://keepthewhy.com/dashboard/) visualizes `context/` and Git history without becoming another source of truth. Delete it and the rationale is still there.

The same principle extends across repositories through monorepos, families, nested repositories, external friends, and stable cross-repository links. The [Keep the Why Registry](https://keepthewhy.com/registry/) adds discovery and backlinks without centralizing the underlying data.

I wrote more about that model in [The reasoning behind a codebase, as a web you can walk](https://blog.technopathy.club/the-reasoning-behind-a-codebase-as-a-web-you-can-walk) and [The globe: following your reasoning into other people's repositories](https://blog.technopathy.club/the-globe-following-your-reasoning-into-other-people-s-repositories).

### Measured behavior, not only philosophy

Keep the Why also publishes a [full eval suite](https://keepthewhy.com/evals/) that runs the skill against fresh fixture projects and agent sessions.

A separate controlled [rejected-change experiment](https://github.com/oliver-zehentleitner/keep-the-why/tree/main/experiments/rejected-change) tested the core claim directly: twenty fresh sessions received the same request to simplify a retry wrapper. Without the recorded rationale, **7 of 10** sessions proposed the already-rejected simplification. With the rationale present in `context/`, **0 of 10** did.

It does not prove universal behavior, but it makes the core claim testable rather than purely philosophical.

## OKF Agent Memory: persistent structured knowledge

[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory) treats the repository as a persistent structured knowledge corpus. Its `knowledge/` bundle can contain decisions, facts, observations, processes, research, people, goals, resources, and project-specific concepts.

It implements the [Open Knowledge Format](https://github.com/GoogleCloudPlatform/open-knowledge-format), the Markdown-plus-YAML format that originated in Google's knowledge-catalog project. OKF defines the representation; OKF Agent Memory adds the behavior and tooling for persistent agent memory.

### Retrieval is part of the architecture

OKF separates compact push memory from a larger pull corpus retrieved when needed. Its local `okf` tool provides in-memory BM25 search, lookup, validation, graph operations, and mutations, with equivalent MCP tools for search, show, create, update, relate, and validate.

No external database or embedding API is required. Repository-owned text stays canonical while retrieval is derived locally.

Four scopes extend the model beyond one repository: **Project**, **Vendor**, **User**, and **System**, with project knowledge taking precedence on ID collisions. An optional Hub adds synchronization. OKF Agent Memory is licensed under **MIT**.

The result is broader than Keep the Why: representation, retrieval, provenance, trust, lifecycle, graph relationships, scopes, and synchronization all belong to the architecture.

## RepoWise: derive intelligence from the codebase

[RepoWise](https://repowise.dev/) is primarily a codebase intelligence system. It derives code structure, dependency relationships, Git history, code health, dead-code signals, generated documentation, and architectural decisions, then exposes them through MCP tools such as `get_context`, `search_codebase`, `get_risk`, and [`get_why`](https://docs.repowise.dev/mcp/get-why).

Unlike the other two, its core product is an additional intelligence model derived from the repository.

### Decisions are a first-class layer

The [architectural decision layer](https://docs.repowise.dev/intelligence/decisions) is what makes RepoWise especially relevant here.

Current decision sources include six index-time lanes: ADR files, inline markers, Git archaeology, PR bodies, code comments, and a harvest during LLM documentation generation. Decisions can also come from coding-agent sessions and manual CLI capture. Earlier CHANGELOG and README mining have been retired.

Session mining exists as a separate source. The current [configuration documentation](https://docs.repowise.dev/configuration/reference) describes `session_mining` as enabled by default, and it can be disabled explicitly.

RepoWise distinguishes candidates from accepted decisions. Its CLI documentation states:

> "A candidate governs nothing. A decision is a candidate a person accepted with `repowise decision confirm`."

Acceptance is not just a status flip. A candidate needs a reason, a scope, and an evidence reference, otherwise acceptance is refused. The acceptance history is append-only, so granting, reviewing, superseding, or withdrawing authority remains visible.

This is a strong design choice because inferred evidence and governing decisions are not treated as the same thing.

### Retrieval, hooks, and semantic search

Accepted decisions are linked to the files they govern and surfaced through [`get_why`](https://docs.repowise.dev/mcp/get-why). When explicit rationale is missing, RepoWise can fall back to Git archaeology and rationale comments instead of presenting inference as authority.

RepoWise also has proactive agent hooks: editing a governed file can inject a short rationale notice into the agent's context.

Its retrieval combines full-text, graph, and specialized code intelligence. Semantic search is available only when an embedder such as Gemini, OpenAI, or local Ollama is configured; otherwise the vector layer remains empty.

Accepted decisions can be exported to `.repowise/decisions.yaml` and committed to Git. On import, RepoWise follows a simple authority rule: **the file wins; the store is the copy**.

That creates a hybrid architecture: accepted decisions can become durable repository data while the larger intelligence model remains derived and indexed. RepoWise's open-source core is **AGPL-3.0**, with hosted and commercial options.

## Three architectures

The simplest way to separate the projects is by data flow.

### Keep the Why

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

The agent is the interface. The files are the memory. No retrieval infrastructure is required.

### OKF Agent Memory

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

The files remain canonical. A dedicated local retrieval layer helps the agent find and operate on relevant concepts.

### RepoWise

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

RepoWise builds a derived intelligence model and exposes that model to the agent.

## Side by side

| | Keep the Why 0.20.0 | OKF Agent Memory | RepoWise |
|---|---|---|---|
| Primary purpose | Preserve software rationale | Persistent structured agent knowledge | Codebase intelligence |
| Core unit | Rationale entry | Knowledge concept | Derived intelligence / decision record |
| Scope | Deliberately narrow | Broad and domain-neutral | Broad software-engineering analysis |
| Canonical project knowledge | Markdown in `context/` | Plain-text OKF bundle in `knowledge/` | Repository plus optional Git-tracked decision manifest |
| Main capture model | Agent captures reasoning while it happens | Agent creates and maintains structured concepts | Mine, derive, capture, review, and confirm |
| Retrospective recovery | Explicit workflow | Possible by creating concepts from existing evidence | Strong through repository archaeology |
| Dedicated retrieval required | No | No external service, but local search is central | Yes, the derived index is central |
| Local search | Normal file tools | Built-in BM25 | Full-text, graph, specialized retrieval, semantic with configured embedder |
| Database required for core project memory | No | No | Derived local index |
| MCP required | No | No, optional | Main agent integration |
| Architectural decisions | Core purpose | One knowledge type among many | One intelligence layer among many |
| Rejected alternatives | First-class | Representable as structured knowledge | Supported in decision records |
| Evidence / provenance | Explicit | Provenance and trust metadata | Explicit evidence and acceptance authority |
| Cross-project model | Families, friends, thoughts | Project, vendor, user, system scopes | Multi-repo workspaces and cross-repo queries |
| Human-readable without special tooling | Yes | Yes | Decision manifest yes; full intelligence requires RepoWise |
| External service required | No | No | No for local self-hosted operation |
| License | MIT | MIT | AGPL-3.0 core, hosted/commercial options |
| Core idea | Preserve the missing why | Build durable structured knowledge | Derive understanding from the codebase |

## Capture, structure, reconstruction

A small example shows the difference better than a larger feature matrix.

Imagine a developer and an agent spend an hour simplifying a retry mechanism. The obvious replacement looks good but causes duplicate orders under one specific failure mode. They revert it and keep the existing implementation.

No production code changes. No useful commit. Possibly no pull request.

But the project learned something important.

### Keep the Why: capture the learning while it exists

For Keep the Why, the conversation itself contains durable rationale. The agent records what was tried, why it failed, which constraint matters, and when the decision should be revisited.

The valuable artifact is what the project learned, even if no code change survives.

### OKF Agent Memory: make it part of durable knowledge

OKF can represent the same learning as a structured decision or related concepts, with provenance, relationships, lifecycle metadata, and retrieval through the knowledge layer.

The information becomes part of a broader persistent knowledge corpus.

### RepoWise: connect evidence to governed code

RepoWise is strongest when useful evidence exists in a source it can inspect: an ADR, commit, PR body, inline marker, code comment, session transcript, or manually entered decision.

It can connect that evidence to affected code, track authority and staleness, and surface the decision when the agent touches the relevant files.

These approaches are complementary in an important sense: **capture prevents knowledge loss, structure makes knowledge manageable, and reconstruction extracts value from evidence that already exists.**

## Where the approaches overlap

The boundaries are not absolute. Keep the Why has grown structure and cross-repository relationships around a narrow rationale core. OKF's general knowledge model naturally includes software decisions. RepoWise's derived intelligence now includes rationale, alternatives, evidence, authority, hooks, and a Git-tracked manifest.

RepoWise's proactive decision hooks are especially close to Keep the Why's goal of surfacing rationale before an agent repeats an old mistake. The difference is where the knowledge comes from and which layer is responsible for delivering it.

## What happens when the memory gets large?

OKF makes local BM25 retrieval part of the design. RepoWise makes retrieval fundamental to the product through a broader derived index.

Keep the Why currently relies on files, stable IDs, `grep`, `rg`, and its indexes. An open [scaling issue](https://github.com/oliver-zehentleitner/keep-the-why/issues/256) deliberately waits for real repositories to reveal the limit instead of inventing one.

If retrieval ever becomes a real problem, an optional, derived, disposable local index would preserve the core rule: **`context/` remains the source of truth.** There is no reason to build that infrastructure before the need exists.

## "Git-native" means different things here

All three can reasonably describe themselves as Git-native, but they mean different things by it.

For **Keep the Why**, repository files are the durable memory layer. For **OKF Agent Memory**, repository files remain canonical while tooling adds retrieval and additional scopes. For **RepoWise**, Git is both evidence and an integration point: accepted decisions can be tracked there while the larger intelligence model is computed separately.

## Which one solves which problem?

If your problem is:

> "Our coding agents keep forgetting why this code exists, repeat rejected approaches, or lose reasoning that never became a commit."

That is the problem [Keep the Why](https://keepthewhy.com/) is designed for.

If your problem is:

> "We want a broader structured knowledge architecture for agents, including project knowledge, reusable packages, multiple scopes, and local retrieval."

[OKF Agent Memory](https://github.com/okf-memory/okf-agent-memory) is closer to that problem.

If your problem is:

> "We want agents to understand a large existing codebase, including architecture, history, dependencies, health, decisions, and risk."

That is where [RepoWise](https://repowise.dev/) becomes especially interesting.

These systems could also coexist. RepoWise could index a repository that already contains `context/`. OKF could hold broader project or organizational knowledge while Keep the Why remains responsible specifically for rationale.

The important question is not whether tools overlap. It is **which layer owns which truth**.

## Conclusion

Keep the Why, OKF Agent Memory, and RepoWise overlap enough to look similar from a distance. Up close, they answer different questions:

**Keep the Why:** How do we preserve the reasoning behind a codebase without turning it into another platform?

**OKF Agent Memory:** How do we represent, retrieve, compose, and govern durable knowledge for agents?

**RepoWise:** How do we derive a useful intelligence model from a codebase and surface it when an agent needs it?

My shorthand remains:

**Keep the Why is a complete, deliberately narrow rationale system.**

**OKF Agent Memory is a general knowledge architecture.**

**RepoWise is a codebase intelligence system with a sophisticated decision layer.**

They optimize for different things. Watching those boundaries evolve is more interesting than forcing all three into one definition of "AI memory."

## Links and further reading

### Keep the Why

- [Keep the Why](https://keepthewhy.com/)
- [GitHub repository](https://github.com/oliver-zehentleitner/keep-the-why)
- [Specification](https://keepthewhy.com/specification/)
- [Full eval suite](https://keepthewhy.com/evals/)
- [Rejected-change experiment](https://github.com/oliver-zehentleitner/keep-the-why/tree/main/experiments/rejected-change)
- [Dashboard](https://keepthewhy.com/dashboard/)
- [Registry](https://keepthewhy.com/registry/)
- [Scaling observation issue #256](https://github.com/oliver-zehentleitner/keep-the-why/issues/256)
- [Architecture Decision Record community resources](https://github.com/architecture-decision-record/architecture-decision-record)

### OKF Agent Memory

- [OKF Agent Memory on GitHub](https://github.com/okf-memory/okf-agent-memory)
- [Open Knowledge Format](https://github.com/GoogleCloudPlatform/open-knowledge-format)
- [Agent Memory Convention](https://github.com/okf-memory/okf-agent-memory/blob/develop/docs/spec/CONVENTION.md)

### RepoWise

- [RepoWise](https://repowise.dev/)
- [Architectural decisions](https://docs.repowise.dev/intelligence/decisions)
- [`get_why`](https://docs.repowise.dev/mcp/get-why)
- [Decision CLI and manifest](https://docs.repowise.dev/cli/decision)
- [Proactive agent hooks](https://docs.repowise.dev/intelligence/hooks)
- [Licensing](https://docs.repowise.dev/licensing)

### Related articles

- [Keep the Why vs. Claude Code Auto Memory vs. MemoryCustodian vs. AgentsRoom](https://blog.technopathy.club/keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom)
- [The reasoning behind a codebase, as a web you can walk](https://blog.technopathy.club/the-reasoning-behind-a-codebase-as-a-web-you-can-walk)
- [The globe: following your reasoning into other people's repositories](https://blog.technopathy.club/the-globe-following-your-reasoning-into-other-people-s-repositories)

---

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), Bluesky, Mastodon, X, and LinkedIn, or join Telegram for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\_(ツ)_/¯