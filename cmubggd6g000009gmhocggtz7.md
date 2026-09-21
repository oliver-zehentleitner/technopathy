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

There are also installation paths for GitHub CLI, Claude Code plugins, Codex plugins, Cursor, asm and manual installation.

There is no service to deploy.

No database.

No daemon.

No account.

What you install is essentially the instructions that teach your coding agent how to recognize and preserve the reasoning behind a project.

And you do not have to abandon the session you are already working in.

Once the skill is available, tell your agent to load Keep the Why - or point it directly at its `SKILL.md` and tell it to follow it.

If you want it loaded automatically in future sessions, the supported approaches are documented under [autostart](https://keepthewhy.com/autostart/).

Then continue in the same session.

## Initialize the project

Inside the project directory, follow the short [project setup](https://keepthewhy.com/setup/) and tell the agent something like:

> Initialize Keep the Why in this project.

This explicit request is intentional.

Keep the Why does not silently turn itself on just because you installed it. A repository has to opt in once.

The agent then presents the project setup.

With the defaults, there is very little to decide.

Among other things, it proposes:

- `context/` for the project's why-knowledge
- capturing from now on
- proactive capture during normal work
- asking before writing only when something is genuinely unclear
- no constant questions for issue or ticket references
- structural linting where the project supports it
- loading Keep the Why automatically in future sessions

The wizard is presented as a list with the defaults already filled in.

You can effectively answer:

> defaults

There is a second short list for your personal workflow preferences.

Again:

> defaults

And you are done.

This is not a configuration project.

It is meant to be something you can set up in roughly the time it takes to explain why you wanted it.

## Then forget about Keep the Why

This is probably the most important part.

After setup, there is no Keep the Why workflow.

You do not periodically stop coding to document decisions.

You do not have to say:

> Store this in project memory.

You do not have to decide whether something belongs in `context/`.

You work with your coding agent normally.

You ask it to implement something.

You debug a production problem.

You discuss an architectural choice.

You reject an apparently simpler implementation because of a constraint.

You discover why a strange workaround exists.

You start removing something and then realize why it must stay.

That reasoning is already happening inside the conversation.

Keep the Why's job is simply to stop throwing it away.

With the default setup, the agent proactively notices reasoning worth preserving and captures it while the work happens.

If the situation is clear, it does not need to interrupt you.

If something important is genuinely ambiguous, it asks.

That is the default.

## Your agent stops being a goldfish

A coding agent usually enters a repository with an interesting asymmetry.

It has the source code.

It has the tests.

It has the documentation.

It has the Git history.

But it does not have the experience that produced them.

A previous session may have spent an hour discovering why an obvious implementation does not work.

The next session sees the same code and happily proposes the same implementation again.

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

The memory is part of the project.

`context/` is Markdown.

`.keep-the-why` records that the project has opted in and how it is configured.

Both travel through Git like the rest of the repository.

Clone the repository somewhere else and the project memory comes with it.

Switch machine and it comes with it.

Open another branch and it comes with it.

A different developer gets it.

A different agent gets it.

A different session gets it.

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

A reviewer normally receives the result of the coding process.

The code changed.

Maybe some tests changed.

But much of the reasoning that led there disappeared with the coding session.

With repo-native project memory, some of that reasoning can arrive in the same pull request.

The reviewer can see not just:

> This implementation changed.

but also:

> This is why it changed, what alternative was rejected and which constraint mattered.

And the reviewer does not have to be human.

A coding agent can produce a change and capture the reasoning that surfaced while producing it.

A review agent can read the same `context/` when reviewing the change.

In that sense, `context/` becomes a small but useful agent-to-agent communication layer.

The coder passes experience forward.

The reviewer can challenge it, verify it or notice when the code contradicts it.

And because it is ordinary Markdown in the PR, humans can do exactly the same thing.

## Teams accumulate experience instead of isolated sessions

This becomes more interesting with multiple developers and agents.

Normally project knowledge spreads through conversations.

One developer discovers something.

Another learns it in Slack.

Someone explains it during a call.

An agent discovers it independently three months later.

Another agent repeats the rejected approach six months after that.

Eventually the person who understood the original reason leaves.

The code remains.

The experience does not.

Keep the Why changes where that experience accumulates. That idea is the core of [repo-native project memory](https://oliver-zehentleitner.github.io/repo-native-project-memory/).

Not in one developer's head.

Not in one agent's session memory.

Not in one vendor's conversation database.

In the project.

That means agents and developers working at different times can benefit from each other's discoveries.

The knowledge compounds.

And it does so without everyone having to become more disciplined about documentation.

## The complete workflow

The initial setup is basically this:

```text
Install Keep the Why
        ↓
Tell the current agent to load it
        ↓
"Initialize Keep the Why in this project"
        ↓
Accept the project defaults
        ↓
Accept the personal defaults
        ↓
Keep coding
```

After that:

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