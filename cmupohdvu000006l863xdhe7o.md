---
title: "The Globe: Following Your Reasoning Into Other People's Repositories"
datePublished: 2026-10-01T15:17:36.552Z
cuid: cmupohdvu000006l863xdhe7o
slug: the-globe-following-your-reasoning-into-other-people-s-repositories
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c46b4b49-b30a-483c-97f3-48f6397f61af.png
tags: ai, github, opensource, documentation, git, developer-tools, ai-tools, knowledge-management, ai-agents, claude, claude-code

---

A decision rarely stands alone.

**We retry three times** may exist because **the payment provider counts every attempt against our error budget**.

**We pinned this WebSocket library** may exist because **the upstream fix has not been released yet**.

And sometimes the reasoning behind your decision does not live in your repository at all.

It lives in a sibling package.

An upstream library.

Another service maintained by your team.

Or a project maintained by someone you have never met.

That became increasingly interesting while working on [Keep the Why](https://keepthewhy.com/), the [open-source project on GitHub](https://github.com/oliver-zehentleitner/keep-the-why).

Keep the Why stores project reasoning as plain Markdown in the repository's `context/` directory. Decisions, rejected alternatives, constraints, workarounds and incident learnings stay next to the code they explain.

An entry can also reference another entry.

And that other entry can live in another repository.

Once that existed, something slightly unexpected happened.

The project memory stopped looking like a collection of documents.

It started looking like a graph.

## The graph was already there

The Keep the Why [dashboard](https://keepthewhy.com/dashboard/) has visualized these relations for a while.

Load a project and you can see its entries, their relations, parent and child repositories, and references to rationale elsewhere. The [live graph](https://keepthewhy.com/dashboard/live/#graph) shows this directly on Keep the Why's own project memory.

Originally, that graph had a natural boundary: the project you opened.

External references appeared at the edge.

But if a decision says:

```text
See: https://github.com/example/upstream#ktw:some-decision-id
```

then the interesting question is obvious:

What happens if we follow it?

And then follow the references from that repository?

And the references from those repositories?

That is what the **Globe** does.

[**Open the Globe**](https://keepthewhy.com/dashboard/live/#globe)

[![Keep the Why Globe - cross-repository reasoning graph](https://keepthewhy.com/assets/dashboard-globe-screenschot.png align="center")](https://keepthewhy.com/dashboard/live/#globe)

**The Globe follows explicit reasoning links across repositories. Click the screenshot to open the live dashboard.**

The public graph is still small. We are just starting to connect projects, so right now the Globe is more a demonstration of the model than a map of a mature ecosystem.

That is also why the registry exists: to give this network a place to start growing.

The Globe is not a new database or another memory backend.

It is the same project reasoning, with fewer artificial boundaries.

## Follow the reasoning, one hop at a time

The Globe starts with the project you already loaded.

From there you decide how far it should follow external references:

*   **Hop 1** loads repositories directly referenced by the current project.
    
*   **Hop 2** follows references found inside those repositories.
    
*   The process continues outward, up to ten hops.
    

Each hop is a separate wave.

That matters because the next wave cannot be known in advance.

To know what repository B references, the browser first has to load repository B.

So before every wave, the Globe pauses and shows what it is about to fetch: which repositories were discovered and how many files that represents.

You can continue or stop there.

There is no background crawler walking the internet.

There is also no reason to load ten hops just because ten are possible.

Often one or two are enough to understand where a decision came from.

And **off really means off**.

With external loading disabled, the dashboard stays inside the project you opened.

`Clear` removes what the Globe loaded and leaves your own project family, friends and navigation path intact.

Reload the page and the external graph is gone.

Nothing is persisted between visits.

## Repository families travel together

Repositories are not always independent units.

A project may be a suite with several packages, or a parent repository may explicitly declare child projects.

Keep the Why already supports those relationships through `parent` and `children`, defined as part of the [project format](https://keepthewhy.com/specification/).

The Globe keeps them.

If an external reference reaches one member of a declared project family, the family arrives together and remains connected according to the repositories' own metadata.

That is important because otherwise the graph would create a misleading picture.

A package that belongs to a larger project should not suddenly look like an isolated external dependency just because that happened to be the repository containing the cited decision.

You can also click any loaded project and make it the new centre.

The route you followed remains visible.

So if you started in repository A, followed a decision into B, then discovered C through B, you can still see how you got there.

The graph is not only showing what is connected.

It is showing how you walked through the reasoning.

## There is still no server behind it

This was the part I cared about most.

I did not want the graph to turn Keep the Why into the kind of infrastructure it was originally designed to avoid.

There is no central service resolving these links.

There is no account.

No graph database.

No daemon crawling repositories.

No API storing everybody's project memory.

Each repository that publishes a dashboard export already declares where that export lives in its `.keep-the-why` file.

The export can simply be a static `state.json` hosted on GitHub Pages or another static host.

When the Globe finds another repository, the browser reads its `.keep-the-why` at `HEAD`, finds the `dashboard-state` location, fetches the export and draws what it contains.

The dashboard server never fetches the other repository.

Your browser does.

That distinction sounds small, but it keeps the architecture pleasantly boring.

The repository remains authoritative.

The published export is only a view of that repository.

And the graph exists because the repositories link to each other, not because a central system reconstructed the relationships afterwards.

## A graph does not need all the prose

Following repositories also exposed a very practical problem.

Keep the Why entries contain text.

Sometimes quite a lot of it.

But the graph does not need the full explanation of every decision just to know that entry A references entry B.

Loading all prose for every repository in every wave would waste bandwidth quickly.

So the dashboard export was split.

`state.json` contains the structure:

```text
projects
topics
entries
fields
relations
links
```

The actual entry bodies live beside it in:

```text
state.body.json
```

Those bodies are fetched only when you open an entry.

For a prose-heavy project, that reduces what a graph wave needs to load to roughly a third of the previous size.

It is a small architectural change with a useful property:

The farther you explore, the less unnecessary text you move around.

## Then there was one missing direction

Following references works well, but it has a blind spot.

You can discover what your project cites.

You can discover what those projects cite.

But you cannot discover a repository that nothing in your current graph points to.

More importantly, you cannot discover who cites **you**.

That information exists somewhere out there, but there is no link you can follow backwards to find it.

This is where the [Keep the Why Registry](https://keepthewhy.com/registry/) came from.

Not as a replacement for the distributed graph.

As a discovery mechanism for the one thing the graph itself cannot do.

## The registry is intentionally boring

The registry is a text file in the Keep the Why repository.

One canonical repository URL per line.

That is basically it.

To add a project, you open a pull request and add its repository URL.

A workflow then reads the repository's `.keep-the-why`, follows its `dashboard-state` declaration, loads the published export and verifies that the export actually identifies the repository being registered.

The registry therefore stores where to start looking.

It does not store your project memory.

It does not copy your entries.

And it does not even need to permanently store the location of your export.

If you move the published dashboard state later, the next registry build follows the repository configuration again and finds the new location.

Project families work here too.

Register the root project and its declared children arrive through the relationships already stored in those repositories.

If an export temporarily stops responding, it is not deleted immediately either.

It remains listed and marked as unavailable for a month before it is dropped.

Repositories disappear for temporary reasons.

Infrastructure should not turn one bad day into permanent metadata.

## The registry is optional

This part matters.

Keep the Why works without the registry.

The dashboard works without it.

The Globe works without it.

Cross-repository references work without it.

Nothing requires a project to join a central list.

The registry only solves discovery.

Tick `registry` in the Globe and it becomes another wave you can load.

Suddenly you are not limited to projects already reachable from your own references.

You can see the first published Keep the Why projects joining the graph and then explore their reasoning exactly the same way.

Once loaded, there is no special registry graph.

They are just repositories again.

That was the design I wanted.

Centralize discovery where central discovery is useful.

Do not centralize the project memory itself.

## It only draws relations that actually exist

A graph can become misleading very quickly if it starts guessing.

So the Globe does not infer relationships from prose.

It does not connect two repositories because they mention the same library.

It does not connect entries because they use similar words.

It does not create a relation because two projects share a topic name.

A connection exists when someone recorded one.

For example through `See` or `Superseded by`.

If two projects obviously belong together but no explicit relation exists in their project memory, they appear as separate islands.

I prefer that.

The graph may know less, but what it shows has provenance.

It came from the project records themselves.

## The interesting part is not the visualization

The Globe looks nice.

But the visualization is not really the part I find interesting.

The useful part is what happens when reasoning crosses repository boundaries without changing ownership.

Repository A remains authoritative for its decisions.

Repository B remains authoritative for its decisions.

Neither needs to copy the other's rationale.

They can simply point at each other.

Git already distributes the files.

Static hosting distributes the read-only views.

The browser follows the references.

The graph appears as a consequence.

Nobody has to maintain one global knowledge graph containing everybody's project history.

And nobody owns the web of reasoning that emerges between repositories.

## A decision three repositories away can still matter to you

There is another consequence of following these links.

A decision is not automatically trustworthy forever just because somebody wrote it down.

Keep the Why already records things such as status, evidence and supersession.

The Globe carries that information across repository boundaries.

So if a chain of reasoning starts from an unconfirmed entry, that remains visible.

If something in the chain is still open, that remains visible.

If a decision gets superseded, that remains visible too.

This becomes particularly useful when the decision is no longer in your own repository.

Imagine your local workaround exists because of an upstream constraint.

Later, the upstream project supersedes that constraint.

Your own code does not magically become wrong.

But the reasoning chain leading to it has changed.

That is exactly the kind of thing worth looking at again.

Not because a graph decided your code is stale.

Because the evidence your decision depended on changed.

## Project memory does not have to stop at the repository boundary

When I started Keep the Why, the idea was intentionally narrow:

Keep the reasoning that is expensive to reconstruct close to the code.

Plain Markdown.

Git.

A small convention.

An agent skill that writes and reads it.

The repository remains the source of truth.

Cross-repository links did not change that model.

They made the consequence of it more visible.

Once repositories can cite reasoning in other repositories, project memory no longer has to mean isolated project memory.

It can stay local, independently owned and Git-native while still participating in something larger.

No shared database is required.

No global account is required.

No project has to hand its memory to another service.

The files were already there.

The links were already there.

The Globe just follows them.

## Try it

Start with Keep the Why itself:

[**Open the Globe**](https://keepthewhy.com/dashboard/live/#globe)

Try one or two hops first.

Then load the registry and explore the first projects joining the graph.

The [registry](https://keepthewhy.com/registry/) explains how discovery works and how to add a project.

The [dashboard documentation](https://keepthewhy.com/dashboard/) covers publishing an export and the rest of the graph features.

The complete format is documented in the [Keep the Why specification](https://keepthewhy.com/specification/).

If your project already uses Keep the Why and publishes a dashboard, adding it to the registry is one line in a pull request.

If it does not use Keep the Why yet, the [installation guide](https://keepthewhy.com/installation/) covers all supported methods. The recommended Skills CLI installation currently starts with:

```bash
npx skills add https://github.com/oliver-zehentleitner/keep-the-why/tree/latest/skills/keep-the-why
```

Then tell your coding agent to initialize Keep the Why in the project.

The reasoning graph is not something you have to maintain separately.

It is a byproduct of projects remembering why they became what they are.

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯