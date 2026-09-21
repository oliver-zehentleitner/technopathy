---
title: "Keep the Why is not another workflow"
datePublished: 2026-09-21T16:24:05.588Z
cuid: cmubggd6g000009gmhocggtz7
slug: keep-the-why-is-not-another-workflow
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/b50b0f7d-b4c5-4684-98f3-99a873d3ad14.png
tags: ai, opensource, devtools, software-engineering, agentic-ai, coding-agents, keepthewhy, project-memory

---

There is a misconception about Keep the Why that I keep running into.

People see a structured `context/` directory containing decisions, rejected alternatives, workarounds, constraints and incident learnings and assume there must be discipline behind it.

Someone has to remember what to record.

Someone has to tell the agent when something is important.

Someone has to maintain another documentation system.

That would defeat the point.

**Keep the Why is designed so that after a very small initial setup, you keep working exactly as before.**

The agent does the remembering.

## Install the skill

[Keep the Why](https://github.com/oliver-zehentleitner/keep-the-why) is an Agent Skill using the open, cross-agent skill format.

For more than 70 supported agents, the recommended installation is documented on the [installation page](https://keepthewhy.com/installation/):

```bash
npx skills add https://github.com/oliver-zehentleitner/keep-the-why/tree/latest/skills/keep-the-why
```

There are also installation paths for GitHub CLI, Claude Code plugins, GitHub Copilot CLI, Codex plugins, Cursor, asm and manual installation.

There is no service to deploy. No database. No daemon. No account.

What you install is essentially the instructions that teach your coding agent how to recognize and preserve the reasoning behind a project.

If you start a session after installing the skill, you do not need a separate "load Keep the Why" step. Asking the agent to set it up in the project is enough to activate it.

If the agent session was already running before you installed the skill, tell that session to load Keep the Why once - or point it directly at its `SKILL.md`. After project setup, the start path handles future sessions automatically. The supported mechanisms are documented under [autostart](https://keepthewhy.com/autostart/).

## Initialize the project

Inside the project directory, follow the short [project setup](https://keepthewhy.com/setup/) and tell the agent something like:

> Initialize Keep the Why in this project.

This explicit request is intentional. Keep the Why does not silently turn itself on just because you installed it. A repository has to opt in once.

The agent then presents the project setup. With the defaults, there is very little to decide.

Among other things, the setup covers:

- where the project's why-knowledge should live - `context/` by default unless an existing decision location is a better fit
- how you want to start capturing project knowledge
- proactive capture during normal work
- asking before writing only when something is genuinely unclear
- no constant questions for issue or ticket references
- structural linting where supported and detected
- loading Keep the Why automatically in future sessions

The wizard is presented as a list with defaults already filled in where the setup defines them.

You can effectively answer:

> defaults

There is a second short list for your personal workflow preferences.

Again:

> defaults

And you are done.

This is not a configuration project. It is meant to be something you can set up in roughly the time it takes to explain why you wanted it.

## Then forget about Keep the Why

This is probably the most important part.

After setup, there is no Keep the Why workflow. You do not periodically stop coding to document decisions, decide whether something belongs in `context/`, or tell the agent:

> Store this in project memory.

You work with your coding agent normally. You ask it to implement something, debug a production problem, discuss an architectural choice, reject an apparently simpler implementation because of a constraint, or discover why a strange workaround exists.

That reasoning is already happening inside the conversation. Keep the Why's job is simply to stop throwing it away.

With the default setup, the agent proactively notices reasoning worth preserving and captures it while the work happens. If the situation is clear, it does not need to interrupt you. If something important is genuinely ambiguous, it asks.

That is the default.

## What actually gets left behind

The result is deliberately boring: plain Markdown in the project.

A real example from the Keep the Why documentation looks like this:

```markdown
context/retries.md

### Why retry_with_jitter isn't a plain retry loop

Type: constraint
Status: active
Evidence: confirmed
Source: discovered while considering simplifying it, 2026-07-22

The payment gateway's rate limiter returns 429 with a per-request
Retry-After header. A fixed-delay retry loop would frequently retry
before the limiter resets, causing repeated 429s under load.

Considered: replacing it with a plain retry loop, since the wrapper
looked like unnecessary complexity with nothing documenting why.
Not adopted once the Retry-After behavior surfaced during review.
```

That entry exists because somebody considered simplifying code that looked unnecessarily complicated.

Without the reason, a later agent sees complexity and may propose the same simplification again. With the reason in the repository, the next session can understand why the code looks the way it does without anyone having to retell the story.

That is what "the agent does the remembering" means in practice.

## Your agent stops being a goldfish

A coding agent usually enters a repository with an interesting asymmetry.

It has the source code, tests, documentation and Git history, but it does not necessarily have the experience that produced them.

A previous session may have spent an hour discovering why an obvious implementation does not work. The next session sees the same code and happily proposes the same implementation again.

That is the goldfish problem.

Keep the Why puts that missing experience into the repository too.

```text
project/
├── src/
├── tests/
├── docs/
├── context/
├── README.md
└── .keep-the-why
```

Now a future session can inherit more than the result.

It can inherit the reasoning.

## Git already solves the distribution problem

Keep the Why deliberately does not build another synchronization system.

The memory is part of the project. `context/` is Markdown. `.keep-the-why` records that the project has opted in and how it is configured. Both travel through Git like the rest of the repository.

Clone the repository somewhere else and the project memory comes with it. Switch machine and it comes with it. Open another branch and it comes with it. A different developer gets it. A different agent gets it. A different session gets it.

The personal workflow preferences of each developer remain personal, but the actual project knowledge is shared.

There is no memory database belonging to one AI provider and no conversation history that only one person can access.

The project carries the experience with it.

## The PR contains the why too

Yes, this means a change can include both:

```text
src/...
context/...
```

I increasingly think this is a feature, not a cost.

A reviewer normally receives the result of the coding process. The code changed. Maybe some tests changed. But much of the reasoning that led there disappeared with the coding session.

With repo-native project memory, some of that reasoning can arrive in the same pull request. That also helps during code review: the reviewer gets the implementation and the relevant project reasoning in the same diff.

The reviewer can see not just:

> This implementation changed.

but also:

> This is why it changed, what alternative was rejected and which constraint mattered.

And the reviewer does not have to be human.

A coding agent can produce a change and capture the reasoning that surfaced while producing it. A review agent can read the same `context/` when reviewing the change.

In that sense, `context/` becomes a small but useful agent-to-agent communication layer. The coder passes experience forward; the reviewer can challenge it, verify it or notice when the code contradicts it.

And because it is ordinary Markdown in the PR, humans can do exactly the same thing.

## Teams accumulate experience instead of isolated sessions

This becomes more interesting with multiple developers and agents.

Normally project knowledge spreads through conversations. One developer discovers something, another learns it in Slack, someone explains it during a call, and an agent discovers it independently three months later. Another agent may repeat the rejected approach six months after that.

Eventually the person who understood the original reason leaves. The code remains. The experience does not.

Keep the Why changes where that experience accumulates. That idea is the core of [repo-native project memory](https://oliver-zehentleitner.github.io/repo-native-project-memory/).

Not in one developer's head, one agent's session memory or one vendor's conversation database.

In the project.

That means agents and developers working at different times can benefit from each other's discoveries. The knowledge compounds without everyone having to become more disciplined about documentation.

## The complete workflow

The initial setup is basically this:

```text
Install Keep the Why
        ↓
"Initialize Keep the Why in this project"
        ↓
Accept the defaults
        ↓
Keep coding
```

If the current agent session was already running before the skill was installed, load it once in that session first. That is the exception, not the normal workflow.

After setup:

```text
normal development
        ↓
useful reasoning appears naturally
        ↓
the agent notices it
        ↓
context/ is updated
        ↓
Git carries it with the project
        ↓
future humans and agents inherit it
```

That is the system.

You should not have to become the librarian for your AI agent.

You should not have to remember to preserve the reasoning.

And you should not have to explain the same project history to every fresh session.

**Install the skill. Initialize the project. Keep coding.**

If Keep the Why requires you to constantly think about Keep the Why, it has failed at its job.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯