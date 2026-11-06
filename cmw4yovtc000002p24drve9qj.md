---
title: "Keep the Why's dashboard is now a library, and Maity is its second host"
datePublished: 2026-11-06T12:39:37.432Z
cuid: cmw4yovtc000002p24drve9qj
slug: keep-the-why-s-dashboard-is-now-a-library-and-maity-is-its-second-host
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/3ecaa25d-6213-493d-b7e5-8d686ad7159a.png
tags: opensource, documentation, developer-tools, ai-agents, keepthewhy

---

[Keep the Why](https://keepthewhy.com) keeps the reasoning behind a codebase in the repository: decisions, rejected alternatives, workarounds, incidents and constraints, as plain Markdown in `context/`, committed with the code. Reading it back has always had its own tool, [keep-the-why-dashboard](https://pypi.org/project/keep-the-why-dashboard/). It is a read-only viewer with a graph of topics and references, a timeline that plays the project day by day, the *thoughts* (chains of entries that cite each other), and the project's family and friends.

Until now that viewer was one page. Now its views are a library that any page can load and mount, and [Maity](https://maity.dev) is the first host besides the dashboard itself.

## What changed

The dashboard's page is now built from the same files another page can use:

- **The files:** ten classic scripts and one stylesheet, listed in load order in `ktw-views.json` and shipped in the `keep-the-why-dashboard` wheel.
- **No build step, no framework:** no ES modules either, so a page opened from `file://` still works.
- **On load:** each script adds one frozen object to `globalThis.KeepTheWhy`: `core`, `ui`, `scene`, `markdown`, and the views (`graph`, `timeline`, `family`, `search`, `thoughts`, `pages`).

A host gives the views three things:

1. **A provider.** It answers the views' questions: the project's state, an entry's text, the graph of a family. The views reach nothing on their own. They do not fetch, store, post or delete. Everything that touches the outside world goes through the provider the host hands them.
2. **Navigation.** Routes are a small grammar in `core.routes`. The host decides what an address looks like in its own URL space.
3. **Its look.** The views read only `--ktw-*` custom properties, about colours, fonts and the corner radius. A host maps its own theme onto them, and the views look like the host, not like a foreign widget.

The dashboard is now just one host among others. Its three columns, its sidebar and its globe belong to the dashboard, not to the library. Another host puts the views wherever its layout has room.

The full contract is in [docs/dashboard-views.md](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/docs/dashboard-views.md). It covers the files, the provider, the scene the views share, the tokens, and what a host has to guarantee.

## Read-only, and safe to embed

A library that renders text written by people and agents into someone else's page has to be boring about safety. These views are:

- **No markup from strings.** No file turns a string into markup: there is no `innerHTML`, no `eval`, and no string timers. A test fails the build if one appears. A host that vendors the files can run the same check over its copy.
- **Checked addresses.** Every address the views link to goes through one gate: an absolute `https://` URL with a host and no credentials, or a relative reference into the host's own places. Anything else is shown as text.
- **Contrast in both themes.** Text on the accent and accent-coloured text meet WCAG AA in the dark and light themes. A test measures those pairs straight from the shipped stylesheets, so a host that changes the colours knows what to keep.

The views render and never write. Embedding them does not give them a way to change anything.

## Maity: the reasons next to the work

[Maity](https://maity.dev) is an open-source hub where developers and their coding agents, from any vendor, work as one team: a shared board, one inbox for the human, and review across vendors. Maity holds the coordination, and the project's knowledge stays in the repository. Keep the Why is its first provider for that knowledge.

A Maity project now has a **Why** view. It loads the Keep the Why library on the first visit and shows the project's own reasoning, read from the repository through the hub, in Maity's colours:

- the overview,
- the graph,
- the timeline,
- the thoughts,
- the family.

It is the same graph and the same timeline the dashboard shows, not a second implementation. When the library gets a fix or a new view, Maity takes over the new release, and nothing is maintained twice.

That matters because of who reads it. On a board where several agents pick up cards, review each other's pull requests and hand work over between sessions, *why* something is the way it is answers the question that comes up most. Why was this approach rejected? Which constraint does this workaround protect? The answer used to be one tab away, in the dashboard. Now it sits next to the card, in the tool where the work is coordinated.

## What it means for you

- **If you use Keep the Why:** nothing changes in your repository. `context/` stays plain Markdown, the dashboard works as before, and an update comes with `pip install -U keep-the-why-dashboard`.
- **If you use Maity:** the Why view shows what your repository already records. Without a `context/`, it stays empty and tells you so. Keep the Why is optional.
- **If you build tools for developers:** the views are yours to host. Write a provider over wherever your data comes from, map the tokens onto your theme, and mount a view into any element. The dashboard, Maity and a small test host in the repository are the three examples to start from.

The principle stays the same as before: the reasoning lives in the repository, and every tool only reads it. Now more tools can.

**Links:**
- [Keep the Why](https://keepthewhy.com)
- [The dashboard, live on Keep the Why's own `context/`](https://keepthewhy.com/dashboard/live/)
- [The host contract: docs/dashboard-views.md](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/docs/dashboard-views.md)
- [Maity](https://maity.dev), and [start free on maity.team](https://maity.team/register)

---

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯
