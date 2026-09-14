---
title: "Your repository already is your project's memory. One layer was missing."
datePublished: 2026-09-14T14:25:44.916Z
cuid: cmu1c57m800020agmba3s3imd
slug: your-repository-already-is-your-project-s-memory-one-layer-was-missing
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/8315714c-3004-4929-a611-ecb7fd7042cb.png
tags: ai, github, opensource, documentation, git, developer-tools, agents

---

Every coding agent starts the day as a goldfish. It can read the code, reason about it, and still have no idea why any of it is the way it is. So it proposes the simplification you rejected in March, for the reason you rejected it in March, and you explain it again.

A whole product category has grown around that: [project memory for agents](https://blog.technopathy.club/keep-the-why-project-memory-for-humans-and-ai-agents). Databases, MCP servers, knowledge graphs, subscriptions. I think much of it starts from the wrong question, because most of the memory already exists.

That is the idea behind [repo-native project memory](https://oliver-zehentleitner.github.io/repo-native-project-memory/).

It is called the repository.

## What a repository already remembers

Look at any well-kept project. `README.md` says what it is. `docs/` says how to use it. `tests/` say what it must do. `CHANGELOG.md` says what changed. `CONTRIBUTING.md` says how a change gets in. `pyproject.toml` or `package.json` says how it is built. `AGENTS.md` says where an agent should look first. Git says who changed what and when.

That is already a form of project memory: plain files, next to the code, versioned by Git, available in every clone. It has worked for people for decades without a separate platform, and it works for agents for the same reason: anything that can read a directory can read it.

Agents already consume the README, docs, tests and changelog. Increasingly, [they maintain them too](https://blog.technopathy.club/i-let-an-ai-agent-maintain-my-open-source-suite-for-a-week-here-s-what-actually-happened).

What the repository usually does not preserve systematically is [why](https://blog.technopathy.club/keep-the-why-code-becomes-legacy-when-nobody-remembers-why).

## The layer that was missing

Why is the retry loop this complicated? Why does that flag exist? Why must these two steps run in this order? Which alternative was tried and dropped?

That reasoning is produced in every working session, often out loud in the conversation with the agent, and a new session throws it away.

People have long had mechanisms for the big version of this. Architecture Decision Records work well for the handful of important architectural decisions worth documenting explicitly. Commit messages, issues and pull requests can also preserve pieces of rationale.

What was expensive was capturing the hundred small reasons that actually make a codebase what it is. Writing each one down manually often cost more than it seemed worth.

That is the part AI changed.

Not the need for the why, which is decades old, but the cost of capturing it.

The reasoning is spoken anyway. An agent that is already part of the conversation can write it down as a byproduct: short, synthesized, in a fixed form, committed with the code it explains.

Sometimes there is only a reason and no code at all. A change is started, investigated and abandoned once the reason not to make it becomes clear. No resulting diff, commit or PR necessarily records that dead end.

A `context/` entry can.

So the missing layer is one more part of the same repository. It lives where the rest lives, travels with every clone, appears in the pull request next to the code diff, and goes through the same review.

No account. No daemon. No database.

## Does it change anything?

I measured the abandoned-change case, because that is the one I cared about.

Twenty fresh agent sessions received the same codebase and the same request: simplify this retry wrapper. Ten had a `context/` entry recording why the wrapper looks the way it does and what had already been rejected; ten did not.

Without the entry, seven of ten offered the already-rejected simplification again.

With it, all ten found the entry and none did.

The Markdown file did not make the agent smarter. It gave it one piece of project history the code itself could not provide.

This also matches a broader lesson from my [agent and model evaluation matrix](https://blog.technopathy.club/same-skill-six-agents-nine-models-what-a-real-eval-matrix-taught-me): whether useful context is found and applied depends on the whole agent harness, not only on the underlying model.

## Where this is thin

A repository holds knowledge; it becomes useful memory only to the degree that its structure makes that knowledge findable. At hundreds or thousands of entries, that becomes a retrieval problem.

The layer also stays empty unless something fills it. And `context/` is not an established convention, so today an agent has to be told that it exists and when to use it.

A linter can check whether an entry has the right structure. It cannot check whether the reason is true. A confidently wrong why can be worse than none.

And much of the why never reaches the repository at all. It remains in chat, tickets, meetings and people's heads.

This is a documentation discipline with an agent as the writing hand, not a memory subsystem.

It wins on ownership, review and longevity.

It loses where nobody reads.

## Where to go from here

The argument, including the model and its limits, is on one page: [repo-native project memory](https://oliver-zehentleitner.github.io/repo-native-project-memory/).

It is a thesis, not a product, and it would hold for any implementation of the layer.

The implementation I use is [Keep the Why](https://keepthewhy.com): an agent skill and the file convention it maintains, a linter for the structure, and a dashboard to inspect it. The background and motivation are also covered in [Keep the Why: Project Memory for Humans and AI Agents](https://blog.technopathy.club/keep-the-why-project-memory-for-humans-and-ai-agents).

It is the why layer of repo-native project memory, and it does not claim to be more than that.

Session memory remembers what happened. Project state remembers where the project is. The why layer preserves why it became what it is.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯