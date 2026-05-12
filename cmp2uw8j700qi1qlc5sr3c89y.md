---
title: "The Complete Binance Python API Guide (2026)"
seoDescription: "Python Binance API guide: REST, WebSockets, order books, trading, trailing stops, and cluster-scale DepthCache in 2026."
datePublished: 2026-05-12T16:39:53.405Z
cuid: cmp2uw8j700qi1qlc5sr3c89y
slug: the-complete-binance-python-api-guide-2026
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c6bb8757-7fef-4fc7-8ca9-2c4a0f6fb1f3.png
tags: websockets, api, python, crypto, cryptocurrency, binance, algorithmic-trading, trading-bot, trading-bot-development, order-book, depth-cache

---

# The Complete Binance Python API Guide (2026)

If you Google **"python binance"** in 2026, the first hits are `python-binance`, `binance-connector-python`, and `CCXT`. Useful tools, all of them. But once a Binance bot moves from *script* to *service*, the interesting question changes: not only *can this library call the endpoint?*, but *does it give me the operational model to keep REST, WebSocket streams, WebSocket API trading requests, order state, reconnects, depth caches, and failure handling under control?*

One library with a more operational focus still barely shows up in search results: the **UNICORN Binance Suite** (UBS). 2.8M+ downloads. 388+ public dependent projects. Several interlocking packages, all MIT, all maintained by one developer with a public name, public GitHub repos, and a Telegram you can actually message.

This guide is the cornerstone reference: what each tool does, why it matters, when to use which, and how the pieces fit together. With **verified, live code** — every output you see in this article was captured from a real call to `api.binance.com` while writing it.

> If you're already sold and just want to read code, skip to [Your First Binance REST Call in Python](#your-first-binance-rest-call-in-python).

* * *

## The Python-on-Binance Landscape in 2026

| Library | Strength | Weakness | Sweet spot |
| --- | --- | --- | --- |
| **python-binance** | Most popular, REST + WS in one package, familiar name | Less explicit lifecycle signaling and order-book trust-state handling than UBS; depth-cache behavior is harder to reason about | Existing projects, quick experiments, users who specifically want that ecosystem |
| **binance-connector-python** | Official, minimal, low overhead, supports HMAC/RSA/Ed25519 | "Naked" — no reconnect logic, no queue, no order-book management | Code that needs the official stamp, or that wraps Binance into something else |
| **CCXT** | Multi-exchange abstraction, huge surface area | Exchange abstraction is the point, but it naturally limits Binance-specific controls and operational detail | You run on three exchanges and want one interface |
| **UNICORN Binance Suite** | Binance-native and deep. REST, WebSocket streams, stream lifecycle signals, WebSocket API requests, reconnect, sequence validation, out-of-sync handling, multi-account, asyncio queue, multi-arch wheels, K8s-scale option | Manager-centric mental model takes ten minutes to internalize; Binance-only by design | Beginners who want sane defaults, production bots, 24/7 services, anything that should still be running next month without you watching |

**This guide is about the fourth row.** Not because the others are bad — `python-binance` got a lot of people started and `CCXT` solves a real problem nobody else solves — but because the gap between "I have a script" and "I have a service that runs for a year" is exactly where UBS lives. If you've never lost money to a silent WebSocket disconnect, an unhandled sequence gap, or a depth cache that quietly stopped reflecting reality — good. This article is for *before* that happens, not after.

> A friendly maintainer's note: I am the author of UBS. I'll cite community sources where I can, show you working code, and let you decide. If you spot anything that looks unfair, the comments and [Telegram](https://t.me/unicorndevs) are open.

* * *

## What UBS Actually Is

UBS is **not a single library** — it's a coordinated suite of six packages plus an optional Kubernetes-scale service. Each piece is its own PyPI package, its own GitHub repo, its own release cadence — but the interfaces line up so they compose without glue code.

| Package | Role |
| --- | --- |
| **unicorn-binance-rest-api** (UBRA) | REST client for public and private Binance endpoints |
| **unicorn-binance-websocket-api** (UBWA) | WebSocket streams, WebSocket API requests, user-data streams, reconnects, lifecycle signals |
| **unicorn-binance-local-depth-cache** (UBLDC) | Local synchronized order books with sequence validation, pruning, resync, and `DepthCacheOutOfSync` |
| **unicorn-binance-trailing-stop-loss** (UBTSL) | Trailing stop engine and CLI |
| **unicorn-fy** | Raw Binance payloads → normalized Python dictionaries |
| **ubdcc** (UBDCC) | Shared DepthCache service for local or Kubernetes-scale infrastructure |
| **unicorn-binance-suite** (meta) | Installs the suite components together |

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/fa74b3be-2493-4ad8-872c-a863f926b68e.png align="center")

* * *

## Install in One Line

```bash
pip install unicorn-binance-suite
```

That gives you the core UNICORN Binance Suite packages — UBRA, UBWA, UBLDC, UBTSL, and UnicornFy — in one install. UBDCC is different: it is the cluster service built on top of UBLDC, not a client library inside the suite. Want a single piece? Install just that one:

```bash
pip install unicorn-binance-websocket-api
pip install unicorn-binance-rest-api
pip install unicorn-binance-local-depth-cache
pip install unicorn-binance-trailing-stop-loss
pip install unicorn-fy
pip install ubdcc
```

A note on a myth that still floats around: *"UBS is hard to install because it needs C build tools."* Not since multi-arch wheels (x86\_64, aarch64, arm64) were added. Where UBS uses native/Cython components, `pip install` resolves to a pre-built binary on common platforms. Cython where it pays off, plain Python where it doesn't.

You **do not need API keys** for anything in the *Market Data* sections below (REST tickers, WebSocket public streams, depth caches). For account operations and trailing stops, see [How to create a Binance API Key and API Secret](https://blog.technopathy.club/how-to-create-a-binance-api-key-and-api-secret).

* * *

## Your First Binance REST Call in Python

The most common starting question: **"How do I get the current price of BTC in Python?"** With UBRA:

```python
from unicorn_binance_rest_api import BinanceRestApiManager

ubra = BinanceRestApiManager(exchange="binance.com")

ticker = ubra.get_symbol_ticker(symbol="BTCUSDC")
print(ticker)

ubra.stop_manager()
```

**Live output** (captured from `api.binance.com` while writing this):

```python
{'symbol': 'BTCUSDC', 'price': '81250.60000000'}
```

24h statistics in one call:

```python
stats = ubra.get_ticker(symbol="BTCUSDC")
for k in ("lastPrice", "priceChangePercent", "highPrice", "lowPrice", "volume", "quoteVolume"):
    print(f"  {k}: {stats[k]}")
```

```plaintext
  lastPrice: 81250.60000000
  priceChangePercent: 0.394
  highPrice: 82137.26000000
  lowPrice: 80462.97000000
  volume: 11772.94006000
  quoteVolume: 957215665.33209680
```

Historical candles for backtesting or charts:

```python
klines = ubra.get_klines(symbol="BTCUSDC", interval="1h", limit=3)
for k in klines:
    print(f"  open={k[1]}  high={k[2]}  low={k[3]}  close={k[4]}  volume={k[5]}")
```

```plaintext
  open=81249.93000000  high=81294.42000000  low=81024.41000000  close=81058.80000000  volume=187.87319000
  open=81058.79000000  high=81303.99000000  low=81000.00000000  close=81240.01000000  volume=398.28206000
  open=81240.01000000  high=81283.80000000  low=81203.81000000  close=81250.60000000  volume=157.79464000
```

**Authentication for account/trading endpoints** is one extra constructor argument:

```python
ubra = BinanceRestApiManager(
    api_key="YOUR_API_KEY",
    api_secret="YOUR_API_SECRET",
    exchange="binance.com",
)
balance = ubra.get_account()  # private endpoint, requires signed request
```

UBRA supports **com, com-margin, com-isolated-margin, com-futures, us, and tr**, plus all matching testnets — switch with the `exchange` argument. A signed-order example for an OCO take-profit/stop-loss pattern lives in [Buy an Asset and Instantly Create a Take-Profit + Stop-Loss OCO Sell Order](https://blog.technopathy.club/buy-an-asset-and-instantly-create-a-take-profit-and-stop-loss-oco-sell-order-using-python-in-binance-isolated-margin).

* * *

## Your First WebSocket Stream in Python

REST polling is fine for one-off queries. For real-time price feeds you want WebSockets. UBWA's model is simple: **one Manager, many streams**. The receiving side can be as small as a few lines, or as explicit as a dedicated asyncio queue per stream.

The examples below intentionally follow the UBWA README style. Start simple, then choose the processing model that fits your bot.

### Option 1 — Pull from the stream buffer

The shortest possible pattern. No callbacks, no asyncio in your code. UBWA receives frames in the background; you pull the oldest item from the stream buffer when you are ready.

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager

ubwa = BinanceWebSocketApiManager(exchange="binance.com")
ubwa.create_stream(
    channels=["trade", "kline_1m"],
    markets=["btcusdc", "bnbbtc", "ethbtc"],
)

while True:
    oldest_data_from_stream_buffer = ubwa.pop_stream_data_from_stream_buffer()
    if oldest_data_from_stream_buffer:
        print(oldest_data_from_stream_buffer)
```

That is the easiest way to understand the flow: Binance pushes data asynchronously, UBWA stores it, and your code consumes it.

For normalized Python dictionaries instead of raw Binance payloads, request UnicornFy output on the stream:

```python
ubwa.create_stream(
    channels=["trade"],
    markets=["btcusdc"],
    output="UnicornFy",
)
```

When to use it: scripts, demos, notebooks, quick collectors. For long-running services, avoid a tight empty polling loop. Use a callback or an asyncio queue.

### Option 2 — Callback per received frame

You hand UBWA a normal function. Every received frame is passed to it.

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager


def process_new_receives(stream_data):
    print(str(stream_data))


ubwa = BinanceWebSocketApiManager(exchange="binance.com")
ubwa.create_stream(
    channels=["trade", "kline_1m"],
    markets=["btcusdc", "bnbbtc", "ethbtc"],
    process_stream_data=process_new_receives,
)
```

There is also an async callback variant:

```python
import asyncio
from unicorn_binance_websocket_api import BinanceWebSocketApiManager


async def process_new_receives(stream_data):
    print(stream_data)
    await asyncio.sleep(1)


ubwa = BinanceWebSocketApiManager(exchange="binance.com")
ubwa.create_stream(
    channels=["trade", "kline_1m"],
    markets=["btcusdc", "bnbbtc", "ethbtc"],
    process_stream_data_async=process_new_receives,
)
```

When to use it: clean event-style processing where each message can be handled independently.

### Option 3 — Await the stream data in an asyncio coroutine

This is the README-recommended pattern for processing stream data when you want explicit async handling. UBWA creates the stream and feeds an asyncio queue; your coroutine awaits data from that queue.

```python
import asyncio
from unicorn_binance_websocket_api import BinanceWebSocketApiManager


async def process_asyncio_queue(stream_id=None):
    print(f"Start processing data from stream '{ubwa.get_stream_label(stream_id)}':")
    while ubwa.is_stop_request(stream_id=stream_id) is False:
        data = await ubwa.get_stream_data_from_asyncio_queue(stream_id=stream_id)
        print(data)
        ubwa.asyncio_queue_task_done(stream_id=stream_id)


async def main():
    ubwa.create_stream(
        channels=["trade"],
        markets=["ethbtc", "btcusdc"],
        stream_label="TRADES",
        process_asyncio_queue=process_asyncio_queue,
    )
    while not ubwa.is_manager_stopping():
        await asyncio.sleep(1)


with BinanceWebSocketApiManager(exchange="binance.com") as ubwa:
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("Gracefully stopping ...")
```

What this gives you:

*   one explicit coroutine for the stream,
    
*   a real `await` on incoming data,
    
*   clean backpressure via `asyncio_queue_task_done()`,
    
*   a manager lifecycle that exits cleanly when used with `with`.
    

### Subscribe and unsubscribe without rebuilding the stream

UBWA can add or remove markets and channels at runtime:

```python
markets = ["engbtc", "zileth"]
channels = ["kline_5m", "kline_15m", "depth5"]

ubwa.subscribe_to_stream(stream_id=stream_id, channels=channels, markets=markets)
ubwa.unsubscribe_from_stream(stream_id=stream_id, markets=markets)
ubwa.unsubscribe_from_stream(stream_id=stream_id, channels=channels)
```

That matters in real bots. You do not want to tear down and recreate a socket every time your market universe changes.

### What's true for all receiving patterns

Regardless of which option you pick:

1.  **WebSocket receiving is asynchronous by nature.** You subscribe once; Binance pushes frames when they exist. Your job is to route and process them correctly.
    
2.  **UBWA owns the socket lifecycle.** Reconnects, listenKey renewal for user-data streams, ping/pong, and stream health happen below your strategy code.
    
3.  **UnicornFy is optional but useful.** Use `output="UnicornFy"` when you want readable dict keys instead of raw Binance event fields.
    
4.  **Receiving data is not the same as knowing the stream is healthy.** That is what `stream_signals` are for.
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/160c42a2-83c7-44db-a8c2-773c5f0ae58d.png align="center")

* * *

## Stream Signals: Know When Your Bot Is Blind

Receiving data is only half of a WebSocket client. The other half is knowing whether the stream itself is currently trustworthy.

A WebSocket is asynchronous by nature. Data arrives when Binance pushes it. Silence can mean "no trade happened", but it can also mean "your connection is gone", "the stream is reconnecting", "the first data frame has not arrived yet", or "this stream cannot be restored". For a trading bot, those states are not cosmetic. They decide whether indicators are still valid, whether a strategy should pause, whether missing data must be reloaded via REST, or whether open positions should be handled defensively.

UBWA exposes this through `stream_signals`. They tell your code about lifecycle changes in real time:

| Signal | Meaning |
| --- | --- |
| `CONNECT` | The stream connection was established. |
| `FIRST_RECEIVED_DATA` | The first data record arrived. This is the point where the stream is no longer just connected, but actually feeding data. |
| `DISCONNECT` | The stream disconnected. UBWA includes the last received data record if available, so your code has a recovery anchor. |
| `STOP` | The stream was stopped. |
| `STREAM_UNREPAIRABLE` | UBWA cannot restore the stream, for example because of invalid credentials or an exception in your own processing coroutine. |

That last distinction matters: **connected is not the same as usable**. A bot that subscribes to `btcusdc@depth` and immediately starts trading before the first data frame arrived is guessing. A bot that keeps calculating indicators after a disconnect is blind. `stream_signals` make those states explicit.

The callback version is usually the cleanest production pattern:

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager
import time


def process_stream_signals(signal_type=None, stream_id=None, data_record=None, error_msg=None):
    print(
        f"Received stream_signal for stream '{ubwa.get_stream_label(stream_id=stream_id)}': "
        f"{signal_type} - {stream_id} - {data_record} - {error_msg}"
    )


with BinanceWebSocketApiManager(process_stream_signals=process_stream_signals) as ubwa:
    ubwa.create_stream(channels="trade", markets="btcusdc", stream_label="TRADES")
    time.sleep(7)
```

For simpler scripts you can also enable the signal buffer and poll it, similar to normal stream data:

```python
ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    enable_stream_signal_buffer=True,
)

ubwa.create_stream(channels="trade", markets="btcusdc", stream_label="BTCUSDC_TRADES")

while True:
    signal = ubwa.pop_stream_signal_from_stream_signal_buffer()
    if signal:
        print(signal)
```

This is a small feature with huge operational impact. It turns WebSocket reliability from log-reading into code-level state. Your own application can know:

*   "I am connected, but not live yet."
    
*   "I received the first usable market-data frame."
    
*   "I just lost the stream and must stop trusting derived indicators."
    
*   "This stream is unrecoverable and needs human or strategy-level intervention."
    

This is also why UBWA fits so well underneath UBLDC and UBDCC. A depth cache does not only need bids and asks. It needs lifecycle truth. `CONNECT`, `FIRST_RECEIVED_DATA`, `DISCONNECT`, and `STREAM_UNREPAIRABLE` are the vocabulary that lets the next layer decide whether data is authoritative, stale, recovering, or unusable.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/4c062903-1c98-4402-bc31-c4117ea53c8d.png align="center")

* * *

## Trading Over WebSocket: Create and Cancel Orders Without REST

There is one Binance feature many Python guides still treat as an afterthought: the **Binance WebSocket API**.

This is not the same thing as a market-data WebSocket stream.

A market-data stream pushes events to you: trades, klines, depth updates, user-data events. The **WebSocket API** is a request/response API over a persistent WebSocket connection: place an order, cancel an order, query account state, and receive a correlated response later — without opening a new HTTP request for every action.

That last word matters: **later**.

WebSocket is asynchronous by nature. You send a request into an already-open connection and move on. At some later point Binance pushes a response frame back. Your code should not pretend that this is just REST with a different transport. The clean design is not "call function, block forever, hope the socket behaves." The clean design is:

1.  send a request with an ID,
    
2.  receive frames asynchronously,
    
3.  route the matching response to the right handler,
    
4.  keep the connection lifecycle independent from the strategy logic.
    

For slow scripts, REST is fine. For systems that already live on WebSockets, jumping back to REST for trading actions creates an awkward split:

*   market data arrives over WebSocket,
    
*   the strategy reacts in memory,
    
*   order placement jumps back to REST,
    
*   response handling follows a different path,
    
*   request IDs, retries, state reconciliation, and failure handling become your problem.
    

UBWA supports Binance WebSocket API requests directly through the same manager model used for streams.

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager

ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    api_key="YOUR_API_KEY",
    api_secret="YOUR_API_SECRET",
    output_default="dict",
)

api_stream_id = ubwa.create_stream(api=True)

ubwa.api.spot.create_order(
    stream_id=api_stream_id,
    symbol="BTCUSDC",
    side="BUY",
    order_type="LIMIT",
    time_in_force="GTC",
    quantity="0.001",
    price="50000",
)

# The response is pushed back by Binance and handled asynchronously.
```

That is the natural WebSocket way: send the request, keep the socket alive, process the response when it arrives.

For scripts, demos, tests, or migration from REST-style code, UBWA can also wait for the matching response and return it directly:

```python
response = ubwa.api.spot.create_order(
    stream_id=api_stream_id,
    symbol="BTCUSDC",
    side="BUY",
    order_type="LIMIT",
    time_in_force="GTC",
    quantity="0.001",
    price="50000",
    return_response=True,
)

print(response)
```

Canceling an order follows the same model:

```python
ubwa.api.spot.cancel_order(
    stream_id=api_stream_id,
    symbol="BTCUSDC",
    order_id=123456789,
)
```

The important part is not only that order placement and cancellation work over WebSocket. The important part is **how responses can be processed**.

UBWA lets you handle WebSocket API responses in several ways:

*   global callback,
    
*   stream-specific callback,
    
*   request-specific callback,
    
*   async handler,
    
*   blocking `return_response=True`,
    
*   or the normal stream buffer.
    

That makes the same feature usable in a notebook, a CLI tool, a long-running bot, or a service with dedicated request routing.

A small but useful detail: `stream_id=api_stream_id` is only required when more than one WebSocket API stream is active. If there is exactly one WebSocket API stream, UBWA can use that stream automatically. In simple examples I still pass the `stream_id` explicitly because it makes the routing visible.

The real advantage shows up in multi-account or multi-key setups: you can run several WebSocket API streams with different `api_key` / `api_secret` pairs and then explicitly route a request to the stream that belongs to the right account.

To stay fair: this is no longer a UBS-only checkbox. `python-binance` documents [API requests via WebSockets](https://python-binance.readthedocs.io/en/latest/websockets.html), Binance's official `binance-sdk-spot` describes itself as supporting REST API, WebSocket API, and WebSocket Streams, and CCXT/CCXT Pro has exchange-dependent WebSocket support. The difference is the operational model: in UBWA, WebSocket streams, WebSocket API requests, reconnect handling, callbacks, stream buffers, async queues, and request routing all live inside one Binance-native manager.

A full walkthrough lives here: [Create and Cancel Orders via WebSocket on Binance](https://blog.technopathy.club/create-and-cancel-orders-via-websocket-on-binance).

This is where UBS becomes more than "REST client plus WebSocket client". REST, market streams, user-data streams, WebSocket API trading requests, response routing, and reconnect handling fit into one mental model.

* * *

## A Local Order Book Without the Pain (UBLDC)

If your strategy reads the order book more than a few times per second, REST polling is a dead end: every call is a round trip across the internet, you will hit rate limits, and the data is stale by the time you parse it. The right answer is a **local depth cache** — Binance pushes diffs over WebSocket, you keep a synchronized copy in memory.

The naive way to do this is on the third page of Binance's docs. The right way is what UBLDC does: create the cache, read asks and bids, and let the manager handle synchronization, pruning, and resync logic.

```python
import time
from unicorn_binance_local_depth_cache import BinanceLocalDepthCacheManager, DepthCacheOutOfSync

ubldc = BinanceLocalDepthCacheManager(exchange="binance.com", depth_cache_update_interval=100)
ubldc.create_depthcache("BTCUSDC")

# Wait for the initial REST snapshot + WebSocket diff synchronization.
while ubldc.is_depth_cache_synchronized("BTCUSDC") is False:
    time.sleep(0.1)

try:
    asks = ubldc.get_asks(market="BTCUSDC", limit_count=5)
    bids = ubldc.get_bids(market="BTCUSDC", limit_count=5)

    print("Top 5 asks:")
    for price, qty in asks:
        print(f"  {price}  qty={qty}")

    print("Top 5 bids:")
    for price, qty in bids:
        print(f"  {price}  qty={qty}")

except DepthCacheOutOfSync:
    print("BTCUSDC depth cache is currently out of sync — skip this decision cycle.")

ubldc.stop_manager()
```

That exception is the important contract.

A local order book can temporarily be unusable: initial snapshot still loading, sequence gap detected, reconnect in progress, resync running. UBLDC does **not** silently pretend that stale memory is still authoritative. If you access the book while it is out of sync, it can raise `DepthCacheOutOfSync`. Your strategy can then pause, skip this tick, reduce risk, or wait for the cache to recover.

### Reading the book the way strategies actually need it

Most strategies don't want "the full book." They want **the first N levels** or **enough depth to fill X quote-units of volume**. UBLDC's `get_asks` / `get_bids` accept both forms:

```python
# First 10 levels on each side
asks = ubldc.get_asks("BTCUSDC", limit_count=10)
bids = ubldc.get_bids("BTCUSDC", limit_count=10)

# All levels until cumulative volume crosses 300 000, e.g. USDT for a USDT pair
asks = ubldc.get_asks("BTCUSDC", threshold_volume=300000)
bids = ubldc.get_bids("BTCUSDC", threshold_volume=300000)
```

`limit_count=N` trims the result to the top N levels by price. `threshold_volume=X` walks the book outward until the cumulative quote volume crosses X and stops there — exactly what you need to estimate "how far would I move the price if I market-bought X USDT of BTC right now?" without slicing the full book yourself.

### `DepthCacheOutOfSync` is the production boundary

You do **not** have to call `is_depth_cache_synchronized()` before every read. That is useful for dashboards, readiness checks, or explicit control flow. The stronger production pattern is to treat reads as authoritative only if they succeed, and to handle `DepthCacheOutOfSync` where your strategy consumes the book:

```python
from unicorn_binance_local_depth_cache import DepthCacheOutOfSync

try:
    asks = ubldc.get_asks("BTCUSDC", limit_count=10)
    bids = ubldc.get_bids("BTCUSDC", limit_count=10)
except DepthCacheOutOfSync:
    logger.warning("BTCUSDC depth cache out of sync — skipping decision cycle")
    return

# If execution reaches this point, the strategy has a usable book snapshot.
run_strategy(asks=asks, bids=bids)
```

That distinction matters. A boolean pre-check can be stale by the time you act on it. The exception is raised at the access boundary, exactly where trust is needed.

`is_depth_cache_synchronized("BTCUSDC")` still has a place:

```python
if not ubldc.is_depth_cache_synchronized("BTCUSDC"):
    logger.info("BTCUSDC still initializing or resyncing")
```

Use it for observability and readiness. Use `DepthCacheOutOfSync` for correctness.

### Sidebar — Binance's docs are incomplete about depth caches

This is the part most libraries get subtly wrong, and most users never notice until their P&L starts drifting.

Binance's published depth-cache algorithm is **incomplete** in one specific way: when a price level falls out of the top-1000, Binance stops sending updates for it — but **never sends a "delete" event** for it either. A library that follows the documentation blindly will accumulate orphaned levels forever. Your "order book" turns into a museum of orders that no longer exist. Strategies that key off the depth profile slowly start trading against ghosts.

UBLDC actively prunes out-of-scope levels. I ran a 25-hour side-by-side experiment to quantify the rot: a naive cache built strictly to spec versus the UBLDC-style pruned cache, fed by the same WebSocket stream. After 9 hours the naive cache was at **~34% bid-match / ~45% ask-match** against a fresh REST snapshot, with 22 000 stale levels still in memory. The pruned cache held a steady ~1050 levels at 90–97% match for the full run.

Full write-up with charts and raw data: [**Your Binance DepthCache Is Rotting — Here's the Proof in 25 Hours**](https://blog.technopathy.club/your-binance-depthcache-is-rotting-here-s-the-proof-in-25-hours). If you build your own depth cache or use a library that doesn't handle this, please at least read the comparison chart.

UBLDC additionally:

*   validates `U` / `u` sequence numbers on every update and resynchronizes on gap detection,
    
*   raises `DepthCacheOutOfSync` instead of silently serving stale data while a cache is unusable,
    
*   buffers WebSocket events during the initial REST snapshot load,
    
*   removes orphaned out-of-scope levels beyond the top-1000 corridor,
    
*   runs many caches in one Manager.
    

[![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/5fb40777-678e-49ba-abb6-f67b7de458ba.png align="center")](https://oliver-zehentleitner.github.io/binance-depthcache-forensics/comparison.html)

* * *

## Trailing Stop Loss from the CLI or Python (UBTSL)

Risk management is the second-most-asked Binance-Python topic after "how do I get the price?"

A trailing stop looks simple from the outside: follow the market while it moves in your favor, keep moving the stop behind it, and exit when the market reverses far enough. In practice, the annoying parts are all operational:

*   where to store API keys,
    
*   how to test connectivity before risking a live order,
    
*   how to run the trailing engine from a terminal,
    
*   how to reuse predefined profiles,
    
*   and how to integrate the same logic into Python code when the CLI is not enough.
    

That is what **UNICORN Binance Trailing Stop Loss** (UBTSL) is for.

### CLI workflow

The CLI is the fastest way to use UBTSL directly from a terminal.

```sh
# add Binance API key and secret
ubtsl --createconfigini
ubtsl --openconfigini

# test connectivity
ubtsl --test binance-connectivity

# start trailing
ubtsl -m BTCUSDC -n trail --stoplosslimit 1% -e binance.com

# use a profile
ubtsl --createprofilesini
ubtsl --openprofilesini
ubtsl --profile BTCUSDC_SELL --stoplosslimit 1.5%

# cancel all open orders on the configured account/exchange
ubtsl --profile BTCUSDC_SELL --cancelopenorders

# help
ubtsl -h
```

The normal flow is:

1.  create the config file,
    
2.  add your Binance API key and secret,
    
3.  test connectivity,
    
4.  start a trailing stop directly or through a named profile.
    

Profiles are useful once you repeatedly trade the same market or strategy pattern. Instead of passing every detail on the command line each time, you define the profile once and then override only the values you want to change, such as `--stoplosslimit`.

The example above starts a trailing stop on `RENDERUSDC` with a 1% stop-loss limit on `binance.com`. The profile example starts the predefined `BTCUSDC_SELL` setup and overrides the stop-loss limit to 1.5%.

`--cancelopenorders` is a practical cleanup command when you intentionally want to cancel all currently open orders on the configured account/exchange before starting fresh. Use it deliberately — it does exactly what the name says.

### Python integration

UBTSL can also be used from Python when the trailing stop should be part of a larger bot or service. The exact parameters depend on the trade direction, market, profile, and execution mode you want to use, so the official example should be treated as the reference implementation:

[example\_binance\_trailing\_stop\_loss.py](https://github.com/oliver-zehentleitner/unicorn-binance-trailing-stop-loss/blob/master/example_binance_trailing_stop_loss.py)

The important architectural point is this: UBTSL is not just a small formula that calculates a moving stop price. It is an engine around Binance execution state. It tracks the market, updates the stop logic, reacts to partial fills and finished orders, and gives you callbacks for operational handling.

A minimal SDK integration usually follows this shape:

```python
import asyncio
from unicorn_binance_trailing_stop_loss.manager import BinanceTrailingStopLossManager


def callback_error(error_msg=None):
    print(f"ERROR: {error_msg}")


def callback_finished(msg=None):
    print(f"FINISHED: {msg}")


def callback_partially_filled(msg=None):
    print(f"PARTIALLY FILLED: {msg}")


async def main():
    with BinanceTrailingStopLossManager(
        api_key="YOUR_API_KEY",
        api_secret="YOUR_API_SECRET",
        market="BTCUSDC",
        stop_loss_limit="1.5%",
        callback_error=callback_error,
        callback_finished=callback_finished,
        callback_partially_filled=callback_partially_filled,
    ) as ubtsl:
        while not ubtsl.is_manager_stopping():
            await asyncio.sleep(1)


try:
    asyncio.run(main())
except KeyboardInterrupt:
    print("Gracefully stopping ...")
```

For a production bot, I would not paste this blindly and call it done. I would start from the official example, wire the callbacks into your own logging/alerting, and make sure the account, market, order side, and stop-loss behavior match your intended execution model.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/b55858b6-a185-4b59-95ed-2a8c8b7cf443.jpg align="center")

* * *

## Stop Parsing Raw Binance JSON by Hand (UnicornFy)

Binance's raw WebSocket frames are compact and cryptic. Here's a kline update straight off the wire:

```json
{"stream":"btcusdc@kline_1m","data":{"e":"kline","E":1778563770000,"s":"BTCUSDC","k":{"t":1778563740000,"T":1778563799999,"s":"BTCUSDC","i":"1m","o":"81251.35","c":"81292.61","h":"81293.00","l":"81250.61","v":"1.42442","n":50,"x":false,"q":"115789.12","V":"0.81","Q":"65872.55","B":"0"}}}
```

Single-letter keys. Nested envelopes. Easy to mis-parse, hard to read at a glance. UnicornFy turns that into:

```python
from unicorn_fy.unicorn_fy import UnicornFy
parsed = UnicornFy().binance_com_websocket(raw)
```

```python
{
  "stream_type": "btcusdc@kline_1m",
  "event_type": "kline",
  "event_time": 1778563770000,
  "symbol": "BTCUSDC",
  "kline": {
    "kline_start_time": 1778563740000,
    "kline_close_time": 1778563799999,
    "symbol": "BTCUSDC",
    "interval": "1m",
    "open_price": "81251.35",
    "close_price": "81292.61",
    "high_price": "81293.00",
    "low_price": "81250.61",
    "base_volume": "1.42442",
    "number_of_trades": 50,
    "is_closed": false,
    "quote": "115789.12",
    "taker_by_base_asset_volume": "0.81",
    "taker_by_quote_asset_volume": "65872.55"
  }
}
```

You usually don't call UnicornFy directly — pass `output_default="UnicornFy"` to UBWA and every frame that lands in your buffer is already normalized.

* * *

## Scaling Beyond a Single Process: UBDCC

A single UBLDC process can handle many markets. The moment you need **redundancy** or **multiple consumers**, the order book should stop living inside one bot process.

UBDCC turns UBLDC into a shared service. You run the cluster once, create DepthCaches there, and every bot, dashboard, or service reads the same synchronized order-book source over HTTP. Locally, the REST API listens on port `42081`; in Kubernetes, it is exposed through the `ubdcc-restapi` service, usually on port `80` behind a LoadBalancer.

Quick local start:

```bash
pip install ubdcc
ubdcc start
```

Create redundant DepthCaches:

```bash
curl -X POST 'http://127.0.0.1:42081/create_depthcaches' \
  -H 'Content-Type: application/json' \
  -d '{"exchange": "binance.com", "markets": ["BTCUSDC", "ETHUSDT", "BNBUSDC"], "desired_quantity": 2}'
```

Query the order book via REST:

```bash
curl 'http://127.0.0.1:42081/get_asks?exchange=binance.com&market=BTCUSDC&limit_count=5'
curl 'http://127.0.0.1:42081/get_bids?exchange=binance.com&market=BTCUSDC&limit_count=5'
```

The important point is the URL shape:

```text
/get_asks?exchange=binance.com&market=BTCUSDC&limit_count=5
/get_bids?exchange=binance.com&market=BTCUSDC&threshold_volume=100000
```

UBDCC exposes synchronized order-book data through explicit REST endpoints such as `/get_asks` and `/get_bids`. For normal consumers, the response stays focused on the requested book side. When you need operational details, `debug=true` adds routing and timing metadata.

```bash
curl 'http://127.0.0.1:42081/get_asks?exchange=binance.com&market=BTCUSDC&limit_count=2&debug=true'
```

UBDCC consists of three component types:

*   **mgmt** on `42080`: cluster state and DepthCache distribution,
    
*   **restapi** on `42081` locally / `80` in Kubernetes: the endpoint your clients call,
    
*   **DCN** processes from `42082` upward: the workers running the actual UBLDC DepthCaches.
    

Your client calls `restapi`. It routes reads to the responsible DCN and management operations to `mgmt`.

Two posts dive into UBDCC in detail:

*   [UBDCC Deep-Dive: Building a Trust Layer for Binance Order Books](https://blog.technopathy.club/ubdcc-deep-dive-building-a-trust-layer-for-binance-order-books) — architecture, why it exists, what problem it solves.
    
*   [From `pip install` to a Redundant Binance Order Book Cluster — UBDCC + Dashboard Quickstart](https://blog.technopathy.club/from-pip-install-to-a-redundant-binance-order-book-cluster-ubdcc-dashboard-quickstart) — a working cluster in minutes.
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/662b37c2-98e0-471d-85ca-283f16e5fb9c.png align="center")

* * *

## What "Production-Grade" Actually Means

Most articles compare libraries on **what they do**. The ones that decide whether your bot survives a year compare on **what they handle when things go wrong**. Here's the practical checklist that separates a weekend project from a service:

| Concern | python-binance | binance-connector / official SDKs | CCXT | UBS |
| --- | --- | --- | --- | --- |
| Automatic WebSocket reconnect | Partial | Low-level / DIY | Partial | Yes, managed |
| Stream lifecycle signals | Minimal / DIY | DIY | Mixed | **Yes — CONNECT, FIRST\_RECEIVED\_DATA, DISCONNECT, STOP, STREAM\_UNREPAIRABLE** |
| WebSocket API trading requests | Recent support | Yes, official SDK direction | Exchange-dependent / Pro | **Yes — integrated into UBWA manager** |
| Request-specific WS API callbacks | Limited | SDK-style / DIY routing | Abstraction-dependent | **Yes** |
| Multiple WS API streams | Limited | DIY | Abstraction-dependent | **Yes — via stream IDs / labels** |
| WebSocket sequence-gap detection (depth) | No | N/A | No | Yes (validates `U`/`u`) |
| Explicit out-of-sync error surface | No | N/A | No | **Yes —** `DepthCacheOutOfSync` |
| Orphaned depth-level pruning | No | N/A | No | **Yes** |
| User-data listenKey auto-renew | Yes (less robust) | DIY | N/A | Yes, robust |
| Subscribe at runtime without reconnect | No | N/A | No | **Yes** |
| Native asyncio `await` queue | No (busy-wait) | N/A | Mixed | Yes |
| Multi-account in one client | DIY | DIY | DIY | Yes |
| Native/Cython components with multi-arch wheels | N/A / mostly pure Python | N/A / mostly pure Python | N/A / pure Python | **Yes — x86\_64, aarch64, arm64** |
| Logging quality | Minimal | Minimal | Minimal | Exceptional — *the* reason `binance-trade-bot` referenced UBWA |
| Test coverage (WS client) | Light | Light | Mixed | 54 unit tests in UBWA alone — the reference standard across the suite |
| Cluster-scale option | No | No | No | UBDCC |

This is not only for large desks. Beginners benefit from stable defaults too. A small bot is not better because its WebSocket handling is fragile, and `python-binance` is not automatically simpler just because it ranks first. The practical reason to look at UBS is that it makes many failure modes explicit before they become your problem.

* * *

### Trust and installation notes

UNICORN Binance Suite is MIT-licensed open source. Install from PyPI or the official GitHub repositories under `oliver-zehentleitner`.

There have been fraudulent repositories impersonating UBWA with malware payloads, so avoid random forks, ZIP downloads, or similarly named projects. The official package names are listed above.

* * *

## Where to Go from Here

**Build something:**

*   Stream market data → Kafka, ready for downstream processing: [Passing Binance Market Data to Apache Kafka in Python with aiokafka](https://blog.technopathy.club/passing-binance-market-data-to-apache-kafka-in-python-with-aiokafka)
    
*   Atomic OCO take-profit + stop-loss: [Buy an Asset and Instantly Create a Take-Profit + Stop-Loss OCO Sell Order](https://blog.technopathy.club/buy-an-asset-and-instantly-create-a-take-profit-and-stop-loss-oco-sell-order-using-python-in-binance-isolated-margin)
    
*   Run a redundant order-book cluster: [UBDCC + Dashboard Quickstart](https://blog.technopathy.club/from-pip-install-to-a-redundant-binance-order-book-cluster-ubdcc-dashboard-quickstart)
    

**Understand the internals:**

*   Why a "correct" depth cache rots if you follow the spec: [Your Binance DepthCache Is Rotting](https://blog.technopathy.club/your-binance-depthcache-is-rotting-here-s-the-proof-in-25-hours)
    
*   What UBDCC actually does, architecturally: [UBDCC Deep-Dive](https://blog.technopathy.club/ubdcc-deep-dive-building-a-trust-layer-for-binance-order-books)
    
*   The Binance API security model and where it leaks: [Binance Fixed the IP Whitelist Gap. The Disclosure Process Is Still Broken](https://blog.technopathy.club/binance-fixed-the-ip-whitelist-gap-the-disclosure-process-is-still-broken)
    

**Docs and source:**

*   UBWA: [docs](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api) · [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api)
    
*   UBRA: [docs](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api) · [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api)
    
*   UBLDC: [docs](https://oliver-zehentleitner.github.io/unicorn-binance-local-depth-cache) · [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-local-depth-cache)
    
*   UBTSL: [docs](https://oliver-zehentleitner.github.io/unicorn-binance-trailing-stop-loss) · [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-trailing-stop-loss)
    
*   UnicornFy: [docs](https://oliver-zehentleitner.github.io/unicorn-fy) · [GitHub](https://github.com/oliver-zehentleitner/unicorn-fy)
    
*   UBDCC: [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-depth-cache-cluster)
    
*   Suite meta-package: [GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-suite)
    

**Talk to humans:**

*   Telegram: [**t.me/unicorndevs**](https://t.me/unicorndevs) — the answer to most questions is one message away.
    
*   GitHub Discussions on any of the repos above.
    

* * *

## Closing

`python-binance`, the official Binance SDKs, and `CCXT` are useful tools. They exist for real reasons. But UBS is built around a different premise: a Binance bot should not only be able to call endpoints — it should understand stream lifecycle, reconnects, order-book trust, out-of-sync states, WebSocket API routing, and failure surfaces from the beginning.

That is useful for production systems, but it is also useful for beginners. Starting small does not mean starting fragile. A stable library is not overkill just because your first script has 50 lines.

If you are building anything Binance-specific in Python, try the suite once. The install is one line. The list of things you no longer have to reinvent is the rest of this article.

Either way: name the failure modes before you ship. The libraries that make those states visible are the ones you want under your code.

* * *

*This guide will be expanded into a series of deep-dives — WebSocket API request routing, WebSocket reconnect internals,* `stream_signals`*, the* `high_performance` *flag, OCO order patterns, UBDCC cluster architecture, and more. Subscribe to the* [*UNICORN Binance Suite*](https://blog.technopathy.club/series/unicorn-binance-suite) *series to catch each one as it lands.*

* * *

I hope you found this informative and useful.

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯