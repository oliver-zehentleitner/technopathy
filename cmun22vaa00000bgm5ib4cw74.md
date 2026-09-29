---
title: "The reasoning behind a codebase, as a web you can walk"
datePublished: 2026-09-29T19:14:55.362Z
cuid: cmun22vaa00000bgm5ib4cw74
slug: the-reasoning-behind-a-codebase-as-a-web-you-can-walk
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c4cedd97-d2b4-4f15-8390-bb54bfdb8986.png
tags: ai, opensource, documentation, git, developer-tools, ai-tools, ai-agents, agents, claude-code, keepthewhy, project-memory

---

Open this before you read on:

**[keepthewhy.com/dashboard/live/#graph](https://keepthewhy.com/dashboard/live/#graph)**

Give it a few seconds to settle.

Then come back.

It looks like a knowledge graph.

It isn't.

There is no graph database behind it. Nobody maintains nodes and edges. There is no central service collecting project knowledge.

What you are looking at is reasoning that was already sitting in Git repositories as Markdown.

The graph is just what appears when those reasons start citing each other.

## What you are looking at

The project in the centre is Keep the Why itself.

Its hubs are topics from `context/`. Around them are individual entries: decisions, rejected alternatives, workarounds, incident learnings and constraints — things the code can tell you happened, but usually cannot tell you **why**.

Each entry is an ordinary Markdown section in the repository.

For example:

```markdown
## Keep dashboard exports read-only

Id: ...
Type: decision
Status: active
Evidence: confirmed
See: ...

Reason:
...

Rejected alternative:
...
```

The colour of an entry shows how well its reason is supported. Its shape tells you whether it is still active, superseded, or needs another look.

Git provides the history: who first recorded it, who touched it later, when its status changed.

The dashboard adds none of that information.

It reads what is already there.

That distinction matters to me: delete the dashboard and the project has lost nothing.

## Then the graph escaped the repository

The more interesting part is further out.

You will see other projects around Keep the Why.

One is [repo-native project memory](https://oliver-zehentleitner.github.io/repo-native-project-memory/), the thesis Keep the Why grew out of.

Another is the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite), a family of related repositories.

Nobody added those projects to a graph configuration.

They appear because an entry in one repository cites an entry in another.

That citation is still just text:

```text
See: https://github.com/... <entry-id>
```

The entry ID is stable. A heading can be reworded, a topic file can be split, and the reference still identifies the same piece of reasoning.

Once those references existed, something unexpected became possible:

the reasoning of several repositories could be viewed as one connected structure without moving any of it into a central store.

## Family, friends, thoughts

I ended up with three different kinds of connection.

They sound informal, but the distinction turned out to be useful.

### Family: where does this reasoning belong?

A family is a group of projects that belong together.

A backend, frontend, shared library and infrastructure repository may all be parts of one product.

The family describes that structure and, importantly, gives the agent routing information.

If a decision belongs to the shared library, it gets recorded there.

If it applies to the whole product, it belongs higher in the family.

The other repositories cite it instead of copying it.

That sounds like a small rule, but it avoids one of the ugliest problems in multi-repository documentation:

five slightly different copies of the same decision.

The reasoning has one owner.

Everything else can point to it.

Families can nest, so the same idea still works for a suite containing a cluster containing another project.

### Friends: what does this project relate to?

Not every relationship means two repositories belong to the same system.

Any Keep the Why entry can cite an entry in any other repository.

Those repositories are **friends**.

Nothing is routed between them.

Nothing is merged.

They simply know about each other because their reasoning crossed paths.

A friend may be one repository or an entire family.

The dashboard follows those references and shows the relevant entries on both sides.

That is why the graph around Keep the Why contains projects that were never configured as part of Keep the Why itself.

They are there because the data says they are related.

### Thoughts: how did one reason lead to another?

This is the part I find most interesting.

An entry can cite an earlier entry with `See`, or say that another entry superseded it.

Once several of those links line up, they form a chain:

```text
incident
   ↓
constraint discovered
   ↓
architecture decision
   ↓
later workaround
   ↓
replacement
```

Keep the Why calls such a chain a **thought**.

Nobody writes a thought.

There is no `thoughts.md`.

The individual decisions were recorded when they happened. Their references are enough for the chain to emerge later.

And importantly, a `See` link does not claim formal causality.

It says these pieces of rationale are related — often that one followed from the other, but not necessarily.

So a thought is not a proof.

It is a trail through the project's recorded reasoning.

Open **Thoughts** in the dashboard and you can read one from beginning to end, with every entry in full, even when the chain crosses repository boundaries.

## The web is deliberately incomplete

There is an important limitation here.

There is no global Keep the Why index.

No service knows every repository that has ever cited yours.

From one project, the dashboard can see what that project points to.

Its friends form the first neighbourhood.

Click one and you can walk there, making it the new centre. The path you walked remains visible, so you can move back through the reasoning the same way you came.

Thoughts can go further.

If a chain continues into another repository, the dashboard says so. Ask it to continue and it loads exactly the repositories needed to follow that chain, hop by hop.

What it does **not** do is crawl every project reachable from every project.

That is intentional.

Each repository remains responsible for publishing its own state.

There is no central graph to submit your project's reasoning to.

The web exists because projects link to one another, not because a platform owns the web.

That also means there are things it cannot know.

If some repository elsewhere cites the newest entry in your chain, but you have never loaded that repository, there is no magic global backlink index that can tell you.

I prefer that limitation to needing one.

## Once reasoning has links, you can ask different questions

At first I built the dashboard mostly because I wanted a quick way to understand what was already in `context/`.

Then the references made new questions possible.

One is:

**Which chains of reasoning started from something nobody ever confirmed?**

Every entry carries an evidence level.

If the first step of a thought is only `inferred` or `unknown`, every later decision may be perfectly reasonable — but the chain started on shaky ground.

The Thoughts view can now surface those origins.

Another question is:

**What later reasoning is connected to something we no longer trust?**

Suppose an old assumption becomes `needs-review`.

The interesting part is not only that one entry.

What came after it?

Which later decisions link back to it?

Do those relationships cross into another repository?

The dashboard can show the later entries resting on that point and the projects they live in.

It still does not claim those decisions are wrong.

A link records a relationship, not a theorem.

But it tells a human where to look.

There are other shapes hiding in the same data:

places where many thoughts start;

entries several thoughts pass through;

chains that evolved over months;

decisions that were superseded but are still being cited.

None required another documentation process.

They became visible because the original reasoning was recorded once, with identity and relationships.

## The graph is not the product

This distinction is important.

Keep the Why does not ask developers to maintain a graph.

The actual workflow is much more boring.

You work with a coding agent.

During that work a real reason surfaces:

- an architectural decision,
- an alternative that lost,
- a workaround whose purpose is not visible from the code,
- a production constraint,
- an attempted change that gets abandoned.

The agent records it in `context/`.

Git versions it with the code.

A later session reads it before repeating the same discussion.

That is the product.

The dashboard is only a lens over the traces this process leaves behind.

And that is probably why I like the graph now more than I expected to.

I originally considered the dashboard almost secondary.

The value was in the agent using the reasoning.

It still is.

But the dashboard makes something visible in twenty seconds that is harder to explain in twenty paragraphs:

a project is not just a collection of files.

It is the result of a long sequence of decisions.

And once those decisions keep their reasons and can refer to one another, you can actually see that sequence.

## Try it on your own project

Everything behind the page remains plain Markdown in `context/`, versioned by Git beside the code.

The dashboard only reads it.

```bash
pip install keep-the-why-dashboard
ktw-dashboard
```

Or export the whole view as a static page and publish it with the project's documentation.

No Keep the Why account.

No graph database.

No central memory service.

No new source of truth.

Just the reasoning the repository already carried — connected strongly enough that you can finally walk through it.

**Keep a Changelog records what changed.**

**Keep the Why preserves why it changed.**

And now you can see how those reasons connect.

[Keep the Why](https://keepthewhy.com/) · [Live dashboard](https://keepthewhy.com/dashboard/live/#graph) · [Thoughts](https://keepthewhy.com/dashboard/live/#thoughts) · [Dashboard explained](https://keepthewhy.com/dashboard/#family-friends-thoughts) · [GitHub](https://github.com/oliver-zehentleitner/keep-the-why)

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯