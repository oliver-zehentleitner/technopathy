---
title: "From `pip install` to a Redundant Binance Order Book Cluster — UBDCC + Dashboard Quickstart"
datePublished: 2026-04-26T17:44:30.731Z
cuid: cmog25pmt006w1qjpf2ntay5s
slug: from-pip-install-to-a-redundant-binance-order-book-cluster-ubdcc-dashboard-quickstart
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/ccc79faa-d213-4696-b024-e2b1e4c95da5.png
tags: python, trading, binance, unicorn-binance-depth-cache-cluster

---

> This is the quickstart.
> 
> If you want the architectural “why” behind it — why UBDCC is not just a cache, why `#6000` is a trust signal, and how UBWA → UBLDC → UBDCC turns Binance depth streams into an observable trust layer — read the companion deep-dive:
> 
> [**UBDCC Deep-Dive: Building a Trust Layer for Binance Order Books**](https://blog.technopathy.club/ubdcc-deep-dive-building-a-trust-layer-for-binance-order-books)

If your goal is simple — *“I need a Binance DepthCache and I want it running now”* — this is probably the shortest path I know.

With [**UBDCC**](https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster), you can start live, correctly synchronized Binance DepthCaches in minutes, manage and automate the setup by code or REST, and use the browser dashboard as an additional operational interface with a built-in API Builder.

You do **not** need Kubernetes to start.  
You do **not** need Redis.  
You do **not** need PostgreSQL.  
You do **not** need to build your own cache manager, failover logic, or API layer first.

The local quickstart is really just this:

```bash
pip install ubdcc
ubdcc start --dcn 4
ubdcc-dashboard start
```

And a few minutes later, you have live Binance DepthCaches running locally.

That is the point of this article.

Not theory.  
Not architecture for architecture’s sake.  
Just getting a real Binance DepthCache up and running quickly — and then understanding why the setup is stronger than it looks at first.

> **Not a Python developer?**
> 
> That is fine.
> 
> UBDCC is implemented in Python, but once it is running, you use it like a local service over HTTP/JSON.
> 
> Your application can stay in Node.js, PHP, Go, Java, Rust, C#, or anything else that can make HTTP requests.

This is the hands-on quickstart. If you first want the technical background, read these two companion posts:

*   [Your Binance Order Book Is Wrong — Here's Why](https://blog.technopathy.club/your-binance-order-book-is-wrong-here-s-why) — why naive local Binance order books can silently accumulate stale levels.
    
*   [Why Your Binance Order Book Should Not Live Inside Your Bot](https://blog.technopathy.club/why-your-binance-order-book-should-not-live-inside-your-bot) — why shared market-data infrastructure is cleaner than per-bot cache ownership.

## Video walkthrough

If you prefer to see the full setup once before going through the steps, here is the uncut quickstart video:

%[https://youtu.be/o1-nCravKnc]    

## Who this is for

This is useful if you want a Binance DepthCache for things like:

*   trading bots
    
*   arbitrage systems
    
*   market making
    
*   dashboards
    
*   alerting
    
*   execution services
    
*   research tools
    
*   mixed-language setups where not everything is written in Python
    

If you only have one Python process and one local consumer, [**UBLDC**](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache) may already be enough.

UBDCC is what you reach for when you want more than one in-process cache and do not want to build the rest of the infrastructure yourself.

## What UBDCC is — and what it is not

UBDCC turns Binance DepthCaches into a running service.

Instead of every script opening its own WebSocket connection, downloading its own snapshot, and maintaining its own copy of the same market, you start UBDCC once and query the DepthCache over HTTP/JSON whenever you need it.

The cluster has three parts:

*   **mgmt** — the coordinator that keeps cluster state and distributes work
    
*   **restapi** — the HTTP interface your clients talk to
    
*   **dcn** — short for **DepthCache Node**, a worker process that actually holds and maintains DepthCaches in memory
    

A **DCN** is where the real DepthCache work happens.

It fetches snapshots, consumes Binance depth streams, keeps books synchronized, re-syncs when needed, and can hold replicas for failover.

If you remember only one thing:

> A DCN is the part of the cluster where the actual DepthCaches live.

UBDCC is **not** a trading bot.

It does not place orders, route execution, generate signals, manage portfolios, or decide what to trade. It is the market-data layer underneath those things.

## Why this is useful in practice

Once a DepthCache is synchronized inside UBDCC, your consumers are no longer pulling fresh Binance snapshots again and again.

They are reading from the cache that is already running in your local cluster.

That gives you some very practical advantages:

*   much lower latency than repeatedly going back to Binance REST
    
*   far less pressure on Binance rate limits
    
*   one running DepthCache can serve many consumers
    
*   strategy restarts do not mean rebuilding the whole market view from scratch
    
*   you can hit your local cluster as hard as you want without hammering Binance the same way
    

This is the same architectural shift I describe in more detail here: [Why Your Binance Order Book Should Not Live Inside Your Bot](https://blog.technopathy.club/why-your-binance-order-book-should-not-live-inside-your-bot)

That is the practical value.

The “shared service” part is not the first thing the user wants.  
The first thing the user wants is a Binance DepthCache that is up, correct, fast, and usable.

UBDCC just happens to solve the rest at the same time.

## Step 1 — Install Python if needed

UBDCC is distributed as a Python package, so the machine that runs it needs Python 3.9 or newer.

That does **not** mean your own application has to be written in Python.

If you are a Node.js, PHP, Go, Java, Rust, or C# developer, think of Python here as a one-time runtime dependency: install it once, start UBDCC, and then use the running service over HTTP/JSON from your own stack.

Check whether Python is already available:

```bash
python3 --version
```

On Windows, if Python is missing, download and install it once from python.org.

Optional but recommended:

```bash
python3 -m venv ubdcc-env
source ubdcc-env/bin/activate
# Windows:
# ubdcc-env\Scripts\activate
```

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/044b8038-22e0-4a42-b258-4d9b3a0c6684.png align="center")

## Step 2 — Install UBDCC

```bash
python3 -m pip install ubdcc
```

That one package gives you the cluster components and the dashboard.

For the local quickstart, there is no long dependency story and no separate UI installation dance.

Install once and move on.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/4a73cc74-5602-4fbf-aa24-9f2d59051fad.png align="center")

## Step 3 — Start the cluster

```bash
ubdcc start --dcn 4
```

This gives you:

*   1 management process
    
*   1 REST API process
    
*   4 DCNs
    

That is already enough to get real Binance DepthCaches running locally with redundancy and failover potential.

After startup, note the REST endpoint, usually:

```text
http://127.0.0.1:42081/
```

A setup that would normally take a fair amount of engineering work is available here as a package, a CLI, a REST API, and an optional browser dashboard.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f0ecdcfb-c8bf-4fc7-b418-779a518518f2.png align="center")

## Step 4 — Open the dashboard

In a second terminal:

```bash
ubdcc-dashboard start
```

By default, the dashboard binds to `127.0.0.1:8080` and opens in your browser.

Connect it to:

```text
http://127.0.0.1:42081
```

The dashboard is optional, but it makes the first-run experience much easier and comes with a built-in API Builder for quick multi-language onboarding.

You can see nodes, markets, replicas, credentials, sync state, and generated API calls. That is why it matters so much: it turns a DepthCache cluster into something you can inspect and operate immediately.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/38c5c7a5-6bf6-4892-8d7e-ccb89da88e0a.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e1623753-de9a-445d-a4e8-f5ef66277a2a.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/2cd9aeb0-79a0-4969-a5f5-46e6dd0775c4.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f28fa5cb-c46a-4b55-aa2f-0b46de9bc88c.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/14882bbf-34c1-423d-a267-f710cfb0e7ba.png align="center")

## Step 5 — Add credentials if needed

For pure Spot mainnet markets, you do **not** need credentials.

Credentials become relevant for some other Binance environments, some snapshot flows, and for getting more headroom where authenticated rate limits apply.

You can handle that through the dashboard or directly through code / REST.

In the dashboard:

*   click **Credentials**
    
*   choose the account group
    
*   add an API key / secret pair
    

The cluster can then distribute credential assignments across nodes.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/42323883-b37a-4792-b156-18e865330e61.png align="center")

## Step 6 — Create DepthCaches with replicas

Now create your first markets.

You can do this through the dashboard, through code, or directly through REST.

In the dashboard:

*   click **DepthCaches**
    
*   choose an exchange such as `binance.com`
    
*   add symbols like `BTCUSDT`, `ETHUSDT`, `BNBUSDT`
    
*   set **Replicas** to `2` or `3`
    
*   create them
    

This is the moment where the setup usually gets its first “okay, that is actually cool” reaction.

You are not creating one local cache in one process.

You are creating **running Binance DepthCaches** that are:

*   synchronized
    
*   visible
    
*   replicated if you want
    
*   ready to query over REST
    

Watch the tiles after creation.

They should move into a synchronized state. That is one of the strongest parts of the dashboard: it does not just show that a cache exists, it shows whether it is actually ready to trust.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/8b40d86b-32d3-4f7e-afff-fa39906cc977.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6d4b1264-bab4-44d8-b7de-5f10393f4cc2.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/3cd0362b-eeb9-4874-8732-995cee0fd938.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/7370fc3f-80de-4c5a-9f51-3a2727559acf.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e9118bec-87ff-4a7d-94d7-180bced083ac.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/ef807ddd-e224-4c29-9f6a-ea5d633fb45e.png align="center")

The replicas are intentionally started with a slight delay. After a few minutes, they should all be in sync:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/4c8b3e41-407b-4d9a-a6bf-dab40c969c82.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c2ec65c5-473e-4790-94de-2af71d1af37f.png align="center")

## Step 7 — Kill a node and watch failover happen

Now for the part that makes the architecture feel real.

Create at least one market with replica count `3`.

Then:

1.  open the **DCNs** tab in the dashboard
    
2.  find a DCN that hosts that market
    
3.  copy the DCN name
    
4.  go back to the operator console
    
5.  remove that DCN by name
    

Example:

```text
ubdcc> remove-dcn <dcn-name>
```

What should happen:

*   the node disappears
    
*   replica count temporarily drops
    
*   the scheduler redistributes work
    
*   a new replica comes up
    
*   the market remains available through another replica
    

That is the difference between “I have a cache” and “I have a cache that survives node loss”.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f1f2420c-d47d-45cc-a9fe-2dc2d1ee7083.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/fa868a44-d74c-4f21-ae72-ba85e60ee594.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c89f0a86-3338-4853-8b23-c63fb4a34892.png align="center")

## Step 8 — Query the DepthCache over REST

Before touching application code, test from the shell.

```bash
curl 'http://127.0.0.1:42081/get_asks?exchange=binance.com&market=BTCUSDT&limit_count=5'
```

Or with HTTPie:

```bash
http GET 'http://127.0.0.1:42081/get_asks?exchange=binance.com&market=BTCUSDT&limit_count=5'
```

This is where the practical benefit becomes obvious.

You can hammer the local cluster with reads as much as you want.

Your consumers are no longer going back to Binance for fresh snapshots over and over again. They are reading from the synchronized DepthCache that UBDCC is already maintaining.

That is faster, easier on rate limits, and especially useful once several bots, dashboards, and services all need the same market view at the same time.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/47307998-6cd0-490b-9208-2ef3566d64ba.png align="center")

## Step 9 — Generate client code for your language

Now open the **API Builder** in the dashboard.

This is one of the strongest parts of the whole setup.

The dashboard can generate snippets for:

*   curl
    
*   HTTPie
    
*   Python
    
*   JavaScript
    
*   Go
    
*   C#
    
*   Java
    
*   Rust
    
*   PHP
    
*   C/C++
    

So if your stack is mixed, the onboarding flow becomes very simple:

*   create the DepthCaches once
    
*   query them from any language
    
*   copy the snippet
    
*   paste it into your project
    

That is exactly how a system like this becomes useful outside one developer’s Python process.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/94a2a76b-161a-4b8c-9d6e-29f577c61d4b.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/7f026979-386f-4ac1-95dc-3717acd159cf.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/70c30e19-5bdd-48cb-b908-c956d89da465.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6507d516-d30f-4a5d-86c2-f76d7974c979.png align="center")

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6d4afc7a-f40b-4763-a493-8961c2a282d8.png align="center")

## Simple Python example

Python users can go two ways.

### Raw HTTP

```python
import requests

response = requests.get(
    "http://127.0.0.1:42081/get_asks",
    params={
        "exchange": "binance.com",
        "market": "BTCUSDT",
        "limit_count": 5
    },
    timeout=10
)

print(response.json())
```

### Official cluster client

If you already live in the UNICORN Binance world, you can also use the official cluster-aware client path instead of raw HTTP.

That gives Python users a smooth migration path from local caches to shared cluster-backed access.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/22cc4df2-6ff4-4e07-8d28-46c387a201a9.png align="center")

## Related projects

If you want to go deeper into the stack:

*   [**UBLDC**](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache) — standalone local DepthCaches for single-process Python use
    
*   [**UBWA**](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) — Binance WebSocket layer underneath the stack
    
*   [**UBTSL**](https://github.com/oliver-zehentleitner/unicorn-binance-trailing-stop-loss) — trailing stop-loss engine from the same suite
    
*   [**UNICORN Binance Suite**](https://github.com/oliver-zehentleitner/unicorn-binance-suite) — the umbrella project
    

## Final thoughts

The nice thing about UBDCC is that it starts simple.

You come for a working Binance DepthCache.

What you also get is:

*   correct synchronization
    
*   explicit cache state
    
*   replicas
    
*   failover
    
*   browser management
    
*   multi-language access
    
*   fast local startup
    
*   room to scale later
    

That is a lot to get from a `pip install`.

Related background:

*   [Your Binance Order Book Is Wrong — Here's Why](https://blog.technopathy.club/your-binance-order-book-is-wrong-here-s-why)
    
*   [Why Your Binance Order Book Should Not Live Inside Your Bot](https://blog.technopathy.club/why-your-binance-order-book-should-not-live-inside-your-bot)
    

If you want to try it:

*   **UBDCC:** https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster
    
*   **UBDCC Dashboard:** https://github.com/oliver-zehentleitner/ubdcc-dashboard
    
*   **Telegram:** https://t.me/unicorndevs
    

* * *

I hope you found this tutorial informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!