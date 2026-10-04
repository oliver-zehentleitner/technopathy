---
title: "Keep the Why vs. Claude Code Auto Memory vs. MemoryCustodian vs. AgentsRoom"
datePublished: 2026-09-22T14:07:25.668Z
cuid: cmucr0gr000000agm17gnh0fp
slug: keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/339a0913-19a4-4ede-b5fe-1b431c99e0a0.png
tags: ai, opensource, documentation, developer-tools, ai-tools, ai-agents, agents, claude-code, keepthewhy

---

# Keep the Why vs. Claude Code Auto Memory vs. MemoryCustodian vs. AgentsRoom

"Project memory for coding agents" now means several different things, and most comparisons put them in one list as if they competed. They mostly don't.

Before comparing tools, ask three questions.

**Where does it live?** In the repository, in a directory on one machine, or on a server. This decides who gets it with a clone, whether a pull request can review it, and how much of the normal Git workflow applies to the memory itself.

**Who can read it?** One person, everyone using one particular app, or anyone who can open the repository.

**What does it remember?** This is the question that gets skipped. A memory can hold three different things, and they are layers, not rivals: what *happened* (a session, a transcript, a fix that worked), where the project *stands* (status, next action, what shipped), and *why* the project became what it is (the decision, the rejected alternative, the constraint the code doesn't show).

Run the names that usually appear in these lists through those questions and most of them leave the room. mem0, Letta, Zep and similar systems are external long-term memory systems centered on what an agent or user should remember across sessions, rather than rationale stored with the repository. Aider, Mastra, Dify and Langflow are a coding agent and frameworks with memory as a feature. Other tools focus on session observations, task state, project plans or summaries.

What is left here is a narrower comparison: two repo-native approaches that preserve durable project knowledge, Claude Code's built-in auto memory, and a server-backed project-memory implementation.

## The table

Checked on 2026-10-04 against [Claude Code's memory documentation](https://code.claude.com/docs/en/memory), [MemoryCustodian's README](https://github.com/waittim/MemoryCustodian), [AgentsRoom's Project Memory page](https://agentsroom.dev/features/project-memory), [the Keep the Why specification](https://keepthewhy.com/specification/), [repository structure](https://keepthewhy.com/repository-structure/), [dashboard documentation](https://keepthewhy.com/dashboard/) and [registry documentation](https://keepthewhy.com/registry/).

|  | Keep the Why | Claude Code Auto Memory | MemoryCustodian | AgentsRoom Project Memory |
| --- | --- | --- | --- | --- |
| **Where it lives** | Plain Markdown in `context/` inside each project, committed with the repository. `.keep-the-why` defines the project and its topology | By default `~/.claude/projects/<project>/memory/` on the user's machine; configurable to another local directory | Plain Markdown in `docs/memory/` in the repository, plus optional private local overlays | Server-side, keyed to a project id committed in the repo; the desktop app also keeps a gitignored local cache |
| **Who reads it** | Anyone with the clone, the reviewer in the pull request, any agent that can read the repository, and optionally humans through the read-only dashboard | Claude Code sessions that have access to that memory directory; all worktrees and subdirectories of the same Git repository share it | Anyone with the clone, plus supported agents through the skill / CLI workflow | Everyone logged in to AgentsRoom who opens the project and has the project id |
| **Versioned / auditable** | Git, alongside the code. Dashboard history is derived from Git | Not by default, unless the configured memory directory is versioned separately | Git, alongside the code, with additional CLI audit and conflict checks | Server-side note history; not Git history |
| **Vendor-independent** | Open Agent Skills format and plain Markdown; usable by supporting coding agents without a model-specific data store | Auto memory is a Claude Code feature | Plain Markdown plus a Python CLI and agent integrations for Codex, Claude Code, Gemini and generic agents | Provider-agnostic inside AgentsRoom through its MCP server, but the memory service itself is part of AgentsRoom |
| **What has to run** | Nothing continuously. The skill runs inside the coding agent when needed; linter and dashboard are optional. The dashboard server exists only while you run it, or it can export a static page | Nothing extra; built into Claude Code | A Python standard-library CLI / skill when reading, writing, checking, migrating or maintaining memory; no daemon or external server required | AgentsRoom desktop app, an account and its MCP / server path. Agent memory reads and writes require connectivity |
| **Selective loading** | `context/index.md` is a deterministic one-line-per-topic index under a fixed 36-heading skeleton; the agent opens only the topic the task touches. Family scope adds routing between related projects | The first 200 lines or 25 KB of `MEMORY.md` are loaded every session; detailed topic files are opened on demand | `manifest.md` routes a bounded context pack through task, path and scope, then loads `brief.md` and only relevant files / areas | `memory_list` returns notes and retrieval cues; `memory_get` fetches individual notes. The app adds fuzzy search over names, cues, bodies and tags |
| **What gets written, by whom** | Distilled rationale synthesized by the agent: decisions, rejected alternatives, workarounds, incidents and constraints, with stable UUIDs, Status, Evidence and optional cross-entry references. Superseded entries stay | Claude writes `user`, `feedback`, `project` and `reference` notes for itself. It deliberately skips things it can derive from the codebase | Typed, evidence-backed project memory with stable IDs, Subjects and Facets. Unconfirmed observations go to `inbox.md`; the CLI manages superseding, conflicts, forgetting and migrations | Agent-written notes in `architecture`, `decisions`, `conventions`, `pitfalls` and `features`, updated in place by default, with optional append / edit modes and no confirmation step |
| **Repository topology** | Single repo, shared-context mono repo, isolated-context mono repo, multi-repo families and nested repositories. Families can themselves nest | One auto-memory directory per Git repository, shared across its worktrees and subdirectories | Repository-scoped memory with optional area routing inside the repository; the current documented model does not define a multi-repository family topology | One project memory per project, plus Folder, Account and Agent scopes for knowledge that should span projects |
| **Cross-repository model** | **Family** for related projects with explicit routing, **friends** for independent repositories cited by rationale, and **thoughts** for chains of cited entries across those projects | No cross-repository relationship model in auto memory | Stable relations exist inside a project's memory, but the current documented model does not define cross-repository citations or a repository graph | Cross-project sharing is handled through Folder, Account and Agent scopes rather than repository-to-repository rationale citations |
| **Human view** | GitHub renders the Markdown directly. The optional dashboard adds Git authorship and status history, queues, search, timelines, graphs, family, friends, thoughts and the globe. It remains read-only | `/memory` exposes the files and opens them in an editor; no project-memory graph | Primarily files and CLI / audit output | Integrated Memory library with editor, fuzzy search, wiki-link graph and visual fading of older notes |

## The part that changed most in Keep the Why

Keep the Why started as a deliberately small idea: put durable rationale in `context/`, keep it in Git, and let the agent maintain it.

That part has not changed. What changed is the topology around it.

A `.keep-the-why` file now marks a project, and the nearest one above the working directory wins, similar to how Git finds `.git`. That one rule covers four layouts.

A **single repository** has one `.keep-the-why` and one `context/`.

A **mono repository** can either share one `context/` across the whole tree or give sub-projects isolated contexts of their own.

A **multi-repository project** can form a **family**. The parent lists its children with one scope line each, and each child points back to its parent. That relationship is not just for visualization. It is routing: rationale belongs once, in the family member whose scope owns it, and other members cite it instead of copying it.

A **nested repository**, such as a submodule or vendored checkout, remains its own project. The nearest `.keep-the-why` wins, so its memory does not leak into the surrounding repository.

Families can nest as well.

That solves the close relationships. Independent repositories use a different mechanism.

A **friend** is simply another repository cited by an entry through a stable entry UUID and that repository's canonical identity. Nothing is routed or synchronized between friends. The citation is the relationship.

Once entries can cite entries in other repositories, something else appears naturally: **thoughts**. A thought is a chain of rationale entries where each step cites or supersedes another one. The dashboard can follow those chains through a project, through its family, and into friends.

The [dashboard](https://keepthewhy.com/dashboard/) is still read-only. Locally it is a view over `context/`, `.keep-the-why`, linter findings and Git history. A static export is a rebuildable snapshot, not a second source of truth. Delete the dashboard and the project memory is still the Markdown in Git.

The **globe** takes that one step further. It loads published dashboard exports as a graph and can walk outward by repository hops. A repository can expose its family and the friends it cites without moving the rationale itself into a central service.

The [registry](https://keepthewhy.com/registry/) makes public exports discoverable. It stores the project pointers and derived metadata needed by the globe, not the rationale itself. It also provides the direction citations alone cannot: discovery of projects that cite you.

That distinction matters. Family is ownership and routing. Friends are citations. Thoughts are chains through those citations. The registry is discovery. The globe is a view over all of it.

None of those changes the authority model: each repository remains authoritative for its own rationale.

## Where the others are better

**Claude Code Auto Memory** still wins on setup. It is built in, on by default in normal local sessions, and it remembers a layer Keep the Why deliberately leaves out: your role, preferences, corrections and working patterns.

Claude Code has also moved since the first version of this comparison. It can now read `AGENTS.md` directly as project instructions, so shared repository instructions no longer necessarily need a Claude-specific import workaround. That makes the surrounding project-instruction story more portable, but auto memory itself is still machine-local by default and remains a Claude Code feature.

That is not a defect. It is personal and tool-local memory doing the job it was designed for.

**MemoryCustodian** is still the closest neighbour, but it has become a much stronger protocol since the earlier comparison.

It is repo-native, plain Markdown and deterministic about loading. It now has stable Subjects and Facets, evidence requirements, candidate memory in `inbox.md`, path-scoped areas, explicit forgetting, conflict checks, transaction journals, recovery, migrations, audit output and private local overlays.

The design difference is clearer now than it was before.

MemoryCustodian treats project memory as a controlled protocol operated through a CLI. Keep the Why treats the committed Markdown as the memory and the skill as the maintainer of that text. If you want stronger command-level enforcement, structural conflict handling and explicit mutation machinery, MemoryCustodian is the more opinionated tool. If you want the rationale to remain ordinary repository text with Git as the main system around it, Keep the Why stays deliberately thinner.

**AgentsRoom Project Memory** has the most integrated application experience of the four.

It has fuzzy search, a first-class editor, wiki-links, a graph, five project-memory folders, four scopes, automatic agent capture, update-in-place, append and edit operations, and a useful visual signal for notes that are getting old.

It also solves cross-project memory, but with a different ownership model. Folder, Account and Agent scopes let knowledge span projects inside AgentsRoom. Keep the Why instead keeps project rationale owned by repositories and connects repositories with explicit citations.

AgentsRoom stores Project Memory server-side. The local cache lets the UI show notes offline, but agent memory tools talk to the server on every call. The project id committed in the repository makes the shared memory follow the project inside AgentsRoom, but the notes themselves are not Git objects and are not reviewed in the repository's pull requests.

For a team already using AgentsRoom as the place where its agents live, that is a coherent trade. For a repository that should remain complete without the application, it is exactly the trade the repo-native approach avoids.

## What Keep the Why doesn't do, on purpose

* **No similarity search.** Retrieval is the index, topic names and explicit references. Entries are synthesized and read as Markdown, not embedded and ranked. Any external search tool can index `context/`; Keep the Why does not require one.

* **No server as the source of truth.** Family, friends, thoughts, dashboard exports, the registry and the globe add views and relationships without moving ownership away from the repositories. A published `state.json` is derived output. The canonical rationale is still the project's committed Markdown.

* **No automatic "this reasoning is stale" oracle.** Entries can carry `Revisit when`, statuses such as `needs-review`, and evidence state. The dashboard queues them, but it does not decide that an old decision is wrong merely because time passed. Age is a signal; validity is a project judgement.

* **No semantic commit gate.** The linter checks structure, references and security properties, not whether a decision is good. Prevention happens earlier, in the session, by giving the next agent the reason before it proposes the same rejected path again. In the retry-wrapper experiment, 7 of 10 fresh sessions proposed the rejected simplification without the recorded rationale; with the entry present, 0 of 10 did ([the experiment](https://blog.technopathy.club/what-happens-when-a-coding-agent-forgets-why-a-change-was-rejected)).

* **No UI that becomes another knowledge store.** GitHub can render `context/` directly. The dashboard adds a much richer read-only view over the same data and its Git history, including family, friends, thoughts and the globe, but writing still happens in the repository.

The reasons for those choices are in the [FAQ](https://keepthewhy.com/faq/).

## The part that is true of all four

Every one of these tools lives on the quality of what gets written into it.

A server full of notes, a `MEMORY.md`, a protocol full of stable IDs, or a `context/` full of Evidence fields is not useful merely because it persists. None of them can recover reasoning that was never captured, and none can guarantee that a claim remains true forever.

The differences are in what gets written, who owns it, how it is retrieved, and what happens when the project grows beyond one session, one machine or one repository.

Keep the Why's answer is deliberately narrow: preserve the reasoning behind the code as structured Markdown owned by the repository. The agent synthesizes rather than records. A rejected alternative is stored as a rejected alternative with the reason it lost, not as a transcript a later reader has to interpret.

The newer family / friends model extends the same rule instead of replacing it. Related repositories can be routed as one family. Independent repositories can cite each other. The dashboard can connect those citations into thoughts and walk them through the globe. The registry can make published projects discoverable.

But Git still owns the memory.

Session memory remembers what happened. Project state remembers where the project is. The why layer remembers why it became what it is.

Three layers. Pick one tool per layer if you need all three, and don't let a comparison, this one included, tell you they are rivals.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\_(ツ)_/¯
