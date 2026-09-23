---
title: "Keep the Why vs. Claude Code Auto Memory vs. MemoryCustodian vs. AgentsRoom"
datePublished: 2026-09-22T14:07:25.668Z
cuid: cmucr0gr000000agm17gnh0fp
slug: keep-the-why-vs-claude-code-auto-memory-vs-memorycustodian-vs-agentsroom
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/339a0913-19a4-4ede-b5fe-1b431c99e0a0.png
tags: ai, opensource, documentation, developer-tools, ai-tools, ai-agents, agents, claude-code, keepthewhy

---

"Project memory for coding agents" now means at least four different things, and most comparisons put them in one list as if they competed. They mostly don't. Before comparing tools, ask three questions.

**Where does it live?** In the repository, in a directory on one machine, or on a server. This decides who gets it with a clone and whether a pull request can review it.

**Who can read it?** One person, everyone who opens the project in one particular app, or anyone who can open the repository.

**What does it remember?** This is the question that gets skipped. A memory can hold three different things, and they are layers, not rivals: what *happened* (a session, a transcript, a fix that worked), where the project *stands* (status, next action, what shipped), and *why* the project became what it is (the decision, the rejected alternative, the constraint the code doesn't show).

Run the names that usually appear in these lists through those questions and most of them leave the room. mem0, Letta, Zep and MemPalace are external long-term memory systems centered on what an agent or user should remember across sessions, rather than rationale stored with the repository. Aider, Mastra, Dify and Langflow are a coding agent and three frameworks with memory as a feature. AgentMemory primarily captures and retrieves session observations in an external local store. Beads, projectmemory.app and Storyflow are closer to *where the project stands*: Beads through a persistent dependency-aware task graph, the latter two through project state living primarily in their own workspace rather than in repository-native rationale files. projectmem keeps a gitignored event log and commits a digest of it, a session layer with a repo-native summary. What is left is two repo-native tools that hold rationale, Claude Code's built-in memory, and a server-backed project-memory implementation. Those four are the table.

## The table

Checked on 2026-09-22 against [Claude Code's memory page](https://code.claude.com/docs/en/memory), [MemoryCustodian's README](https://github.com/waittim/MemoryCustodian), [AgentsRoom's Project Memory page](https://agentsroom.dev/features/project-memory) and [the Keep the Why specification](https://keepthewhy.com/specification/).

|  | Keep the Why | Claude Code Auto Memory | MemoryCustodian | AgentsRoom Project Memory |
| --- | --- | --- | --- | --- |
| **Where it lives** | `context/` in the repository (or an existing decisions folder) | by default `~/.claude/projects/<project>/memory/` on the user's machine; the auto-memory directory can be configured | `docs/memory/` in the repository | AgentsRoom's server, keyed by a project id committed in the repo |
| **Who reads it** | anyone with the clone, the reviewer in the pull request, any agent | one user in Claude Code wherever that memory directory is available | anyone with the clone | anyone logged in to AgentsRoom who has the project id |
| **Versioned** | Git, with the code | no, unless the configured directory is versioned separately | Git, with the code | a note history on the server; not Git |
| **Vendor-independent** | Agent Skills format, any supporting agent; the content is Markdown | Claude Code only | Python CLI plus skills for Codex, Claude Code and Gemini; the content is Markdown | any CLI, but only inside the AgentsRoom app, logged in |
| **What has to run** | nothing; a linter and a read-only dashboard are optional | nothing, built in | a Python CLI: `init`, `add`, `check`, `migrate` | a desktop app, an account, an MCP server; agents need a connection for every read and write |
| **Selective loading** | `context/index.md`, one line per topic under a fixed 36-heading skeleton; the agent opens the topic the task touches | the first 200 lines or 25 KB of `MEMORY.md`, every session | `manifest.md` routing rules, then `brief.md`, then the task's files | `memory_list` at the start of a task, `memory_get` per note |
| **What gets written, by whom** | rationale synthesized by the agent from the conversation, with Type, Status and Evidence fields; superseded entries kept; confirmation is a project setting | notes Claude writes for itself, four kinds: user, feedback, project, reference; no confirmation | typed files for decisions, constraints and rejected paths, with stable IDs and evidence attribution; the agent writes through the CLI | free-form notes in five folders, wiki-linked, written by the agent through `memory_save`, no confirmation |

## Where the others are better

**Claude Code Auto Memory** has zero setup, is on by default, and holds the layer Keep the Why deliberately leaves out: who you are, how you like to work, the corrections you gave last Tuesday. Keep the Why keeps that out of the repository on purpose, nobody's preferences belong in a shared `context/`, so the two are complementary. By default, auto memory does not follow a change of tool, a teammate's clone or a second machine. That is not a bug. It was never meant to be repository-native project memory.

**MemoryCustodian** is the closest neighbour: repo-native, plain Markdown, manifest routing, evidence per entry, a `do-not-use.md` for rejected paths. The differences are design choices. A protocol-centered set of core files plus routed area files, or topic files the project names itself. CLI-enforced evidence and identity checks, or a project-level confirmation setting. If you want the memory to be a program with commands, MemoryCustodian is the honest pick. If you want it to be text the agent maintains and nothing else, that is Keep the Why.

**AgentsRoom Project Memory** has the best reading experience of the four: a graph with wiki-links as edges, fuzzy search, four scopes, update-in-place with an append mode. All of it runs on AgentsRoom's server. The notes are not in Git, not reviewable in a pull request, not readable outside the app, and the feature page says it plainly: "treat the memory of a public repository as public." It is project memory that lives with the project id, not in the project. For a team that has moved its whole agent workflow into AgentsRoom, a fine trade. For a repository that will outlive the app, the trade the repo-native idea exists to avoid.

## What Keep the Why doesn't do, on purpose

*   **No similarity search.** Retrieval is the index and the topic name; entries are synthesized and read in full, not stored verbatim and ranked. `context/` is Markdown, any search tool can index it, none is shipped.
    
*   **No commit gating.** No Git hooks; the linter checks the form of an entry, never a decision. Prevention happens in the session, before the change: with the entry in place, 0 of 10 fresh sessions proposed a rejected simplification; without it, 7 of 10 did ([the experiment](https://blog.technopathy.club/what-happens-when-a-coding-agent-forgets-why-a-change-was-rejected)).
    
*   **No UI that holds anything.** `context/` renders on GitHub for anyone who can open the repository; the [dashboard](https://keepthewhy.com/dashboard/) reads and holds nothing.
    

The reasons for each are in the [FAQ](https://keepthewhy.com/faq/).

## The part that is true of all four

Every one of these tools lives on the quality of what gets written into it. A store full of transcripts, five folders of notes, a `decisions.md` with stable IDs, a `context/` with Evidence fields: none is worth anything if the content is thin, and none can fix that afterwards. The differences are in who writes, when, and what shape the writing has to take.

Keep the Why's answer is less about who writes the memory than what gets written. The agent synthesizes rather than records: a rejected alternative is written as a rejected alternative with the reason it lost, not as a transcript or general note a later reader has to interpret. The linter keeps the form honest. The rest is Markdown in Git, and Git already knows how to version it, distribute it and put it in front of a reviewer.

Session memory remembers what happened. Project state remembers where the project is. The why layer remembers why it became what it is. Three layers. Pick one tool per layer if you need all three, and don't let a comparison, this one included, tell you they are rivals.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯