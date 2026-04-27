---
title: "Your Binance Order Book Is Wrong — Here's Why"
datePublished: 2026-04-16T09:59:33.040Z
cuid: cmo1b58z1000s2dnne84pejby
slug: your-binance-order-book-is-wrong-here-s-why
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/07e9bd38-b649-464c-9856-97f8af518874.jpg
tags: algorithms, python, software-engineering, cryptocurrency, trading, binance, data-engineering

---

If you maintain a local Binance order book, there is a good chance it contains price levels that no longer exist on the exchange. Not because your code has a bug — but because Binance's documentation tells you to build it in a way that guarantees silent data corruption over time.

I am not talking about reconnect logic, sequence gaps, or the well-known initial-snapshot race condition. Those are documented and most libraries handle them. I am talking about a class of stale entries that accumulate silently in your depth cache because Binance never tells you they should be gone.

This came up while I was maintaining [UNICORN Binance Local Depth Cache (UBLDC)](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache). A user reported unbounded growth in their order book — and when I investigated, it turned out to be a gap in Binance's own synchronization spec, not a library bug. The fix was straightforward, but it is not in Binance's documentation, and I have not seen it handled in any other library.

* * *

## How a depth cache is supposed to work

This is Binance's official algorithm, documented in ["How to manage a local order book correctly"](https://binance-docs.github.io/apidocs/spot/en/#how-to-manage-a-local-order-book-correctly). Quick refresher for people who have not built one:

1.  You open a WebSocket stream for diff updates (`@depth`).
    
2.  You request a REST snapshot of the current order book.
    
3.  You discard any updates whose sequence number is older than the snapshot.
    
4.  You apply the remaining updates to the snapshot in order.
    
5.  From now on, every diff update arrives in real time and you mutate your local copy.
    

A diff update is a list of price levels. For each level you receive a price and a quantity. If the quantity is `0`, the level is gone — you delete it from your book. Otherwise you set the quantity for that price level.

That is the whole protocol. Snapshot, apply diffs, handle the `0`\-quantity delete event. Done.

Except it is not done.

* * *

## The part Binance does not tell you

Binance's depth snapshots cover the **top 1000 price levels** on each side. The diff stream sends you updates for any level that changes — but only for levels that are currently in the top 1000.

The moment a price level drops out of the top 1000 because more aggressive activity pushes it down, Binance stops sending updates for that level. Not just stops — **never sends another update again**, including no `0`\-quantity delete event.

If you followed the documented protocol, that level stays in your local copy forever. Your bid book still claims there is meaningful resting size at a price that has not been part of the real order book for hours, days, weeks — however long your process has been running.

These are what I call **orphaned entries**. Ghost orders. Levels that exist only in your memory, with no representation on the actual exchange.

* * *

## Why this is easy to miss

A few reasons this stays invisible long enough to ship into production:

**It does not break anything obvious.** Your top-of-book is fine because the top-of-book gets updated most aggressively. If you only ever read `asks[0]` and `bids[0]`, you will never notice. The corruption builds up at the edges of the book where you stop looking.

**The numbers look plausible.** A stale ask at $63,247.50 with 0.4 BTC quantity is a perfectly reasonable-looking number. Nothing about it screams "I am a fossil from three hours ago." A volume aggregator summing quantities will return larger numbers than reality, but it will not return obviously wrong numbers.

**Spot-checking against the REST snapshot does not catch it.** If you periodically pull a fresh snapshot and compare, you will see differences — but you will attribute them to the snapshot being a different moment in time. That is plausible. The actual cause — that levels have silently fallen off your stream — is not an obvious hypothesis.

**Documented behavior matches buggy behavior.** Binance's documentation describes how to apply diff updates. It does not warn you that a level that disappears from the top 1000 will never be corrected. So if you followed the docs to the letter, your code looks correct against the spec. The spec is just incomplete.

* * *

## How this was found

A user [reported](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache/issues/45) that their asks and bids lists were growing without any bound. Starting at around 1000 entries, the count kept climbing — slowly but monotonically. After 15 minutes, already noticeably above 1000. After hours, thousands of levels on each side with no churn at the bottom of the book.

At first glance it looked like a library bug. But when I dug into it, I realized the library was doing exactly what Binance's documentation says to do. The problem was upstream — in the spec itself.

I went back to the Binance documentation. There was no mention of bottom-of-book eviction. I tested it: created a cache, waited for active levels to roll off the top 1000, and watched whether any correction arrived for a known stale level. Nothing. The level was orphaned, and Binance had no mechanism to tell the client about it.

That was the moment it stopped being "there might be a bug in the apply-diff logic" and became "Binance's documented protocol is incomplete, and every implementation that follows it strictly is accumulating ghost entries."

* * *

## The fix

The fix is conceptually simple: **prune everything beyond the top 1000 levels**.

[UBLDC](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache) does this in `_clear_orphaned_depthcache_items()`. After processing a depth update, the cache is sorted by price (descending for bids, ascending for asks) and anything past position 1000 is deleted.

```python
def _clear_orphaned_depthcache_items(self, market, side, limit_count=1000):
    reverse = (side == "bids")
    sorted_items = [
        [price, float(quantity)]
        for price, quantity
        in list(self.depth_caches[market][side].items())
    ]
    orphaned_items = sorted(
        sorted_items, key=itemgetter(0), reverse=reverse
    )[limit_count:]
    for item in orphaned_items:
        del self.depth_caches[market][side][str(item[0])]
```

The reasoning:

*   Anything Binance still cares about will be inside the top 1000 and will keep getting diff updates.
    
*   Anything outside the top 1000 is, by Binance's own definition, not part of the snapshot you would get if you re-pulled right now.
    
*   So evicting it is correct: it brings your local copy into the same consistency window Binance guarantees.
    

The cost is one sort per update per side. For 1000 entries on a hot pair this is microseconds. UBLDC absorbs it inside the WebSocket event loop with no measurable impact on throughput.

* * *

## Why this matters even if you "only need top of book"

**Memory growth is real.** A long-lived process accumulating thousands of dead price levels per market per side, across hundreds of markets, becomes a real memory cost. Not a leak in the technical sense, but unbounded growth that nothing in Binance's protocol will stop.

**Volume-based queries break.** If you ever ask "what is the total bid volume below this price" or "what price level holds the next $1M of asks", stale entries are polluting your answer. Market depth charts will be subtly wrong in ways that compound the longer the process runs.

**Book-shape logic breaks.** Anything that compares against book width, density, or the relationship between price levels is reading partly from a fossil record. Strategies that look at order book shape — common in market-making and liquidity-provision — will be making decisions based on data that is half real and half archaeological.

* * *

## What to do about it

You have three options, depending on what you are building:

**1\. Prune yourself.** Periodically take the lowest-priced bids and highest-priced asks beyond the depth you care about and remove them. Six lines of Python. Works with any library.

**2\. Use UBLDC in your Python project.** [UNICORN Binance Local Depth Cache](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache) handles this plus reconnect logic, sequence validation, multi-market management, and automatic resync on gap detection. MIT licensed, 220K+ PyPI downloads.

**3\. Use UBDCC if you want it as a service.** This is the option I would recommend if you are not married to a specific language or if you run multiple bots. [UNICORN DepthCache Cluster](https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster) runs UBLDC as a background service and exposes order book data via REST API — accessible from Python, Node.js, Go, Rust, or anything that speaks HTTP:

```bash
pip install ubdcc
ubdcc start
```

That gives you a local REST API where any process can query consistent, pruned, auto-recovering order book data. No WebSocket management, no orphaned entries, no initialization wait on restart. Your bots connect to the service; the service handles the hard parts.

Either way, **do not rely on the Binance diff stream alone** to keep your book consistent past the active edge. The protocol does not guarantee what most documentation implies.

If you want the architectural reasoning behind running this as shared infrastructure instead of embedding it inside every bot, I wrote that up here:
[Why Your Binance Order Book Should Not Live Inside Your Bot](https://blog.technopathy.club/why-your-binance-order-book-should-not-live-inside-your-bot)

And if you just want to try the cluster directly, start here:
[From pip install to a Redundant Binance Order Book Cluster — UBDCC + Dashboard Quickstart](https://blog.technopathy.club/from-pip-install-to-a-redundant-binance-order-book-cluster-ubdcc-dashboard-quickstart)

* * *

## Why I wrote this

Since fixing this, I have seen the same question come up in different forms — *"why does my depth cache use so much memory after a few hours?"*, *"why is my book deeper than the REST snapshot?"*, *"why are there bids at prices that haven't been seen in days?"* I would rather link to one explanation than retype it.

This is the kind of finding that does not have a CVSS score and will not get a Binance changelog entry. It is a gap between "what the docs say" and "what actually happens at the protocol level", and the only way it gets fixed in the wider ecosystem is by being written down with code.

* * *

[*UNICORN Binance Local Depth Cache*](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache) *— MIT, 220K+ downloads · The fix is in* `manager.py`*, function* `_clear_orphaned_depthcache_items()`*.*

[*UNICORN DepthCache Cluster*](https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster) *— Consistent order book data as a REST service.* `pip install ubdcc && ubdcc start`*. Any language, any number of clients.*

Suggested reading path:

[Why Binance order books silently go wrong](https://blog.technopathy.club/your-binance-order-book-is-wrong-here-s-why)
[Why the order book should not live inside your bot](https://blog.technopathy.club/why-your-binance-order-book-should-not-live-inside-your-bot)
[How to run UBDCC locally in minutes](https://blog.technopathy.club/from-pip-install-to-a-redundant-binance-order-book-cluster-ubdcc-dashboard-quickstart)

* * *

I hope you found this tutorial informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!