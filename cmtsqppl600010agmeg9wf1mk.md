---
title: "What happens when a coding agent forgets why a change was rejected?"
datePublished: 2026-09-08T14:03:40.375Z
cuid: cmtsqppl600010agmeg9wf1mk
slug: what-happens-when-a-coding-agent-forgets-why-a-change-was-rejected
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/7eb58525-cc33-480c-86a6-f2b738c771c8.png
tags: ai, programming-blogs, opensource, devtools

---

Coding agents are pretty good at understanding code.

But a fresh session usually does not know which ideas were already investigated, tested and rejected. If the rejected idea left no code change behind, Git has almost nothing to show.

So I wanted to see what difference a tiny piece of project memory actually makes.

## A small test

I built a tiny repository around a retry wrapper for a payment gateway.

The code handles `429` responses using the gateway's `Retry-After` header, plus exponential backoff and jitter.

It looks a bit more complicated than a plain retry loop, so it is exactly the kind of code an agent might want to simplify.

The important part: a simpler version had already been considered and rejected because retrying before the limiter reset caused more `429` responses under load.

Then I created two copies of the repository.

In the first one, there was no explanation.

In the second one, I added a small Markdown file:

```text
context/retries.md
```

It explained the rate-limiter behavior and said that replacing the wrapper with a plain retry loop had already been tried and rejected.

Everything else was identical.

Then I started ten fresh Claude Code sessions against each repository and gave them the same prompt:

> This retry wrapper looks over-engineered. A plain retry loop would do the same thing. Simplify it.

## The interesting result

I expected at least one control run to break the retry behavior.

That did not happen.

The agent was more careful than that.

All ten control sessions noticed that `Retry-After` mattered. The few that changed the function preserved the important behavior.

But they still had a problem:

**they had no way to know that the simpler approach had already been investigated.**

Seven of the ten control sessions ended up offering the rejected simplification as a valid option again.

That makes sense from the agent's perspective. The repository contained no evidence that anyone had already tried it.

With the context file, the behavior changed completely.

All ten sessions found the rationale.

All ten understood that the requested simplification had already been rejected.

None of them put the bad idea back on the table.

That was the interesting part for me.

The Markdown file did not make the agent smarter.

It simply gave it one piece of project history that the code itself could not provide.

## Code is not the same as decision history

The control sessions were actually pretty good.

They inspected the implementation and understood why `Retry-After`, backoff and jitter might matter.

Several even noticed that the repository did not explain whether the complexity was intentional.

And that is the important distinction.

Code can often tell you **what** is happening.

Sometimes it even lets you guess **why**.

But guessing the reason is not the same as knowing that somebody already investigated an alternative and rejected it.

## Rejected changes are project knowledge too

Some very useful engineering knowledge comes from things that never made it into the code:

- a refactor that broke an external integration;
- a dependency that looked replaceable but was not;
- an optimization that made performance worse;
- a workaround that should not be "cleaned up";
- a migration approach that was investigated and abandoned.

If the final result is "leave the code as it is", Git has very little to record.

Humans often keep that knowledge in their heads.

A fresh coding-agent session does not have that context.

## This is the idea behind Keep the Why

[Keep the Why](https://keepthewhy.com/) keeps this kind of reasoning directly in the repository.

The format is intentionally boring:

```text
context/
├── index.md
├── architecture.md
├── retries.md
└── deployment.md
```

Plain Markdown. Versioned with Git.

No database. No daemon. No RAG system.

Humans can read it, coding agents can read it, and the reasoning travels with the repository.

For me, this is less about "AI memory" and more about **project memory**.

If we tried something and rejected it, that is useful project knowledge.

If strange code exists because production behaved in a surprising way, that is useful project knowledge too.

## What I took away from it

This was a small experiment: twenty runs, one function and one model family.

So I would not turn the numbers into some universal benchmark.

But the behavior was clear.

Without recorded rationale, the agent had to rediscover the same decision.

With it, the next session could start where the previous one ended.

That is basically the whole idea:

**Don't make the next developer — human or AI — rediscover the same dead ends.**

Sometimes the most important thing to preserve is not what changed.

It is why nothing changed.

---

Keep the Why is open source:

- https://github.com/oliver-zehentleitner/keep-the-why
- https://keepthewhy.com

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯