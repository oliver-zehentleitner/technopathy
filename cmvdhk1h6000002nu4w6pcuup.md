---
title: "Seven Coding Agents. One Chat Window."
datePublished: 2026-10-18T07:10:11.343Z
cuid: cmvdhk1h6000002nu4w6pcuup
slug: seven-coding-agents-one-chat-window
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/d3e80af0-06ba-4ff6-9543-8308293ad850.png
tags: software-engineering, agentic-ai, coding-agents, project-memory

---

After the planning was done, seven agents started working.

Infrastructure, ingestion, retrieval, frontend, accounts, shared library, coordination. Separate sessions, working in parallel and talking to each other. Watching the whole thing get moving was fascinating. I was surprised by how well it worked, and by how good the results became.

The agents ran on a Linux VM, in separate GNU screen sessions over SSH.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/0d155593-90b5-4ae5-a0a4-9706cd651b07.png align="center")

![Seven coding agents running in GNU screen on a Linux VM](article-assets/fleet-screen.png align="center")

*The fleet: AGENT\_A through AGENT\_G. Fairly ordinary terminal windows for a surprisingly productive setup.*

On my side, I used Claude Desktop, with AGENT\_A pinned as my main conversation. The other agents had their own chats, available whenever I wanted to talk to one directly. Most of the time, A handled the coordination.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f7972a81-ea57-457f-b1cc-1926507b6d5d.png align="center")

![The agent chats in Claude Desktop, with AGENT_A pinned](article-assets/fleet-claude-desktop.png align="center")

That was the key. I told A what I needed. It worked out who should do what, coordinated the work and brought the results and questions back to me. I did not have to remember which session owned which problem.

It felt like working with one agent, except that long implementation tasks happened in parallel while our conversation continued. Efficient, comfortable, and a workflow I want to use again.

**What we built**

[lasteverLM](https://github.com/oliver-zehentleitner/lasteverLM) is a NotebookLM-style application: documents, web pages, images and audio go in; cited answers, reports, quizzes, charts, mind maps and spoken overviews come out. It includes login, credit accounting, administration and deployment to Kubernetes. Outputs expose their supporting passages, model and costs so the user can inspect how they were made.

From the first commit to handover, about **78 hours elapsed**. I spent roughly **20–24 hours** on planning, decisions, feedback and testing. Recorded agent activity totalled **138.5 hours across seven sessions**, with work happening in parallel. That is a transcript-based activity measure, not an estimate of equivalent human labour.

The resulting repository contained roughly **81,000 lines of Python + TypeScript**, alongside **208 test files** and **300 recorded decisions across 84 files**. The [build account](https://oliver-zehentleitner.github.io/lasteverLM/how-it-was-built/) records over **650 merged pull requests, over 1,930 commits and over 80 release tags**. Across the seven agent sessions, there were **13,703 model requests**. Those numbers describe the scale of the work; review, measurements and live testing tell us whether it worked.

**Six builders, one head, and checks on everyone**

Each implementation agent owned one part of the repository and worked in its own checkout. A coordinated, reviewed and merged pull requests, and handled releases. It did no service implementation work.

The merge rule was to review the change and wait for every required check to pass. A release tag triggered image builds, migrations, deployment and a smoke test against the public domain. Median time from tag to deployed and smoke-tested: 3.6 minutes.

The coordinator needed checking too. Parallel work collided in shared files. Release tooling had bugs. A frontend change could land before its backend dependency. Each failure needed a concrete correction in the tools or review process. Having a head made coordination easier; it did not make the head infallible.

Some authority stayed explicitly with me. Reading or changing real user data required my approval in the session doing the work. Another agent passing on an approval did not count. At one point, an agent refused the head’s request because that approval was missing. The head corrected itself.

My walkthroughs also found things the tests had missed: a chart built from sources I had deselected, an audio file reporting only its first segment’s duration, and report paragraphs silently escaping verification. Each became a fix and a regression test.

The short deployment loop helped here. I could find something on the phone, discuss it with A, and test the deployed correction while the problem was still fresh.

**Shared reasoning made the parallel work easier**

The agents used Keep the Why to preserve decisions, rejected alternatives and constraints beside the code. They read the other components’ context before building against them. By the end, there were 300 entries across 84 files.

[![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e12bf0bc-b1e1-4422-a2c6-583b65dcc3a8.png align="center")](https://oliver-zehentleitner.github.io/lasteverLM/keep-the-why-dashboard/)

![Decision records grouped by agent and project in the Keep the Why dashboard](article-assets/ktw-authors.png align="center")

This supplied information the implementation alone cannot: why PostgreSQL also serves as the job queue, why generated outputs should not feed back into new outputs, and when an earlier decision should be reconsidered.

It required upkeep. At this speed, reasoning can fall behind the implementation. Two maintenance rounds still found gaps. Recording it helped; checking it against the code remained necessary.

One investigation shows why I also wanted measurements. A bad image transcription suggested we might need a stronger vision model. Testing ten models showed that large, dense screenshots were a problem across the platform’s models. Cutting the image into overlapping tiles changed the result: one model went from reading 68 of 160 lines correctly to 159. A scanned page improved from 37 of 60 lines to all 60.

We changed how the image was supplied and kept the model. The [evaluation includes the method, results and limitations](https://github.com/oliver-zehentleitner/lasteverLM/blob/main/ingest/docs/vision-eval.md).

**What I want from fleet tooling now**

The head worked so well that my next wish is mostly about the interface around it.

Chats contain a lot of output: progress narration, command results, explanations. Somewhere between them is a question that actually needs me. I want those questions collected in one place, attached to the coding project.

A card might need a yes/no approval, a choice between options, a written answer or a discussion. The list should be ordered by priority and by what is blocking work. Most requests would come through the head; others would come directly from an agent that needs my personal go-ahead. If the team resolves something itself, it should close the request. Logs and conversations should remain available behind that view.

Joining should be straightforward too:

> Join this project’s team.

The agent should receive its identity, role, hierarchy, project and team knowledge, and communication protocol.

Or:

> Here is the SSH connection. Start Luke there, using Claude or Pi, with model X and role Y.

The platform should start the session, connect it to the team and collect its logs.

I will look at existing fleet tools before deciding what to build. Paperclip is already close to what I have in mind. I want something shaped around a coding project and this particular way of working. This build gave me a much clearer picture of what that means.

**An unofficial Agent D**

A small postscript made me smile. An outside contributor opened [PR #685](https://github.com/oliver-zehentleitner/lasteverLM/pull/685). When I asked my agent to inspect it, it reported a branch named `agent-D/audio-request-errors` and a commit using our own `OZ-AGENT_D` author identity. The contribution even followed our changelog convention.

Apparently, Agent D had acquired an unofficial understudy.

There is a useful distinction here for the tooling I want: a contributor can pick up the project’s conventions from the repository. Joining the team, receiving an identity and gaining permissions need an explicit process.

The head already made seven agents feel like one conversation. Now I want every remaining request for my attention to be just as easy to find and handle.

* * *

The [repository](https://github.com/oliver-zehentleitner/lasteverLM), [build account](https://oliver-zehentleitner.github.io/lasteverLM/how-it-was-built/) and [decision dashboard](https://oliver-zehentleitner.github.io/lasteverLM/keep-the-why-dashboard/) are public, including the measurements and what went wrong.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯