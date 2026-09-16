---
title: "picows in UNICORN Binance WebSocket API: Up to 2× the Throughput, Opt-In for Now"
datePublished: 2026-09-13T15:01:55.667Z
cuid: cmtzxzvwp00010agmep9c7vd0
slug: picows-in-unicorn-binance-websocket-api-up-to-2-the-throughput-opt-in-for-now
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/553d76c2-9de2-4b58-8288-c7a18dc83f49.png
tags: websockets, python, performance, opensource, benchmarking, binance, asyncio, unicorn-binance-websocket-api, picows

---

[UNICORN Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) (UBWA) has used [`websockets`](https://github.com/python-websockets/websockets) since day one.

Since UBWA 2.16.0, you can switch a manager instance to [`picows`](https://github.com/tarasko/picows), a Cython WebSocket implementation.

For typical Binance messages below ~1 KB, picows delivers roughly 1.7–2× the throughput with significantly lower CPU cost. At normal trading-bot message rates, however, you probably won't notice a difference.

That's why picows is opt-in for now.

## Why picows?

`websockets` is a very good pure-Python library. For a few hundred messages per second it is nowhere near the bottleneck.

But UBWA is also used for hundreds of `depth@100ms` subscriptions, multiplexed connections and local depth caches processing every diff. At those rates, per-message overhead starts to matter.

picows moves much of the WebSocket protocol handling into Cython.

It also provides a `picows.websockets` compatibility layer with the familiar API: `connect()`, `recv()`, `send()`, `close()` and matching exceptions.

That made the UBWA integration small.

## Switching libraries

One parameter:

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager

ubwa = BinanceWebSocketApiManager(
    exchange="binance.com"
)

ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    websocket_library="picows"
)
```

picows is optional:

```bash
pip install --upgrade "unicorn-binance-websocket-api[picows]>=2.16.0"
```

Streams, WebSocket API, userData streams, subscribe/unsubscribe, reconnects, signals and proxies use the same UBWA code path.

There is also no silent fallback.

If you select `"picows"` without installing it, UBWA raises `ImportError`. Unknown values raise `ValueError`.

If a bot says it is running on picows, it should actually be running on picows.

## Benchmark

The benchmark runs both libraries through the full UBWA stack against a local server sending Binance-shaped messages.

Python 3.13, websockets 16.0, picows 2.1.3, x86\_64 Linux, `output_default="raw_data"`, median of three runs:

| Scenario | ~msg size | websockets msgs/s | picows msgs/s | speedup | websockets CPU µs/msg | picows CPU µs/msg |
| --- | --- | --- | --- | --- | --- | --- |
| aggTrade | 0.2 KB | 201,912 | 403,316 | 2.00x | 5.1 | 2.5 |
| kline | 0.3 KB | 195,460 | 371,019 | 1.90x | 5.2 | 2.9 |
| depth20 | 1.0 KB | 153,187 | 259,960 | 1.70x | 6.8 | 4.1 |
| depth diff | 9.1 KB | 64,172 | 67,972 | 1.06x | 16.3 | 15.4 |
| !ticker@arr | 453.9 KB | 1,768 | 1,662 | 0.94x | 608.7 | 641.0 |
| multiplex mix | 0.2 KB | 180,406 | 334,188 | 1.85x | 5.7 | 3.2 |

The pattern is simple.

For small Binance messages, picows is clearly faster. Around 10 KB the difference mostly disappears. On the huge 450 KB `!ticker@arr` payload, picows even comes out a few percent behind. That row turned out to be an artifact of the benchmark, see the next section.

With `output_default="dict"` and JSON parsing included, the advantage for small messages is still around 1.4–1.7×.

Against live Binance at only a few hundred messages per second, there is effectively no difference. Both libraries spend most of their time waiting for data.

If that is your workload, switching gives you little.

If you push tens of thousands of messages per second through one process, it matters.

## The big-message row is a benchmark artifact

The `!ticker@arr` row is real and reproducible, not noise: a follow-up sweep from 9 KB to 900 KB with ten paired runs per size shows picows 4 to 12 percent behind websockets through UBWA for everything from 32 KB upwards, with a run-to-run spread of only a few percent. It is also an artifact of the benchmark, not of picows. Driven directly, without UBWA, picows wins at every one of those sizes by 1.6x to 2.1x. The difference is how the two libraries drain the socket. The replay server is a firehose on loopback, it never paces, and UBWA's own per-message work on a 450 KB text (a couple of substring scans) is slower than the wire. The kernel receive buffer then autotunes into the megabytes, and picows takes all of it in one recv per loop iteration into a read buffer that keeps doubling: strace counts 79 reads of about 3.4 MB for 600 messages. websockets reads at most 256 KB per recv, about 1060 reads for the same data, and stays cache-friendly. Copying and decoding half-megabyte frames out of a multi-megabyte buffer that has already left the cache costs more per byte than picows saves on parsing. The proof is a one-line change: capping `SO_RCVBUF` to 128 KB on the client socket flips the 450 KB result to picows 1.3x to 1.4x ahead, with nothing else touched. The benchmark script has a `--rcvbuf` option for that now, and the full tables are in [context/websocket-library.md](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/blob/master/context/websocket-library.md). On a real Binance connection the socket buffer never fills like that, a WAN link delivers a few megabytes per second at most and the consumer keeps up, which is also what the 24 h soak showed: picows with less CPU and less memory. Thanks to Taras Kozlov, the picows author, for asking the question that led to this.

## The benchmark found a UBWA bottleneck first

The first benchmark showed only about 1.4× improvement.

Profiling found why.

UBWA had 18 `logger.debug()` f-strings formatted for every message even when debug logging was disabled, seven lock acquisitions per message where one was enough, and heartbeat/stop checks running twice.

After removing that overhead:

*   websockets: **116k → 202k msgs/s**
    
*   picows: **163k → 403k msgs/s**
    

picows did not suddenly get faster.

UBWA stopped hiding its speed.

The details are documented in `context/stream-loop.md`.

## Why I did not use the native picows API

picows also has a native listener API based on `ws_connect()` and `WSListener`.

I benchmarked it too.

On 0.2 KB messages it managed roughly 499k msgs/s versus 485k through the compatibility API. On 9 KB messages the difference was around 13%.

Inside the full UBWA stack that would translate to less than 5% end-to-end.

In return, UBWA would need a second connection implementation of roughly 300–400 lines.

Not worth it.

UBWA stays on the compatibility API.

## Failure-path testing found a real bug

Performance benchmarks are easy. Failure handling is more interesting.

The test suite now runs both libraries through scenarios including:

*   server-side closes and reconnects
    
*   fragmented frames
    
*   450 KB payloads
    
*   messages above `max_size`
    
*   server pings
    
*   Unicode
    
*   rejected handshakes
    
*   WebSocket API round trips
    
*   keepalive timeouts
    

The rejected-handshake test found a real compatibility issue.

When Binance returns HTTP 429 or 404 during the WebSocket upgrade, UBWA reads the status from `InvalidStatus`.

`websockets` exposed `response.status_code`.

picows 2.1.x exposed `response.status`.

UBWA hit an `AttributeError`, the stream thread died and nothing useful was logged.

I reported it upstream as [tarasko/picows#108](https://github.com/tarasko/picows/issues/108).

picows 2.2.0 fixed it the next day.

## Proxy support got simpler too

picows 2.3.0 added native HTTP, HTTPS, SOCKS4 and SOCKS5 proxy support.

`websockets` has supported the same since 15.0.

UBWA previously handled SOCKS5 itself using PySocks and a blocking handshake inside the event loop.

That code is now gone.

UBWA simply passes the proxy URL to the selected WebSocket library:

```python
ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    proxy="socks5://user:pass@127.0.0.1:9050"
)

ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    proxy="http://127.0.0.1:3128"
)
```

Both libraries now support:

```text
http://
https://
socks4://
socks5://
```

The old `socks5_proxy_server` parameters still work and are converted internally.

Testing also found two UBWA issues in the old proxy path:

*   rejected SOCKS5 credentials could kill a stream thread without a useful log message
    
*   TLS certificate verification was not actually enabled on the proxy path despite the option defaulting to `True`
    

Both are fixed.

One difference remains: `websockets` currently does not URL-decode proxy credentials such as `p%40ss`, while python-socks/picows does. I reported that as [python-websockets/websockets#1761](https://github.com/python-websockets/websockets/issues/1761).

UBWA rejects affected credentials up front when using `websockets` instead of entering a reconnect loop.

The performance and soak tests below were run with picows 2.1.3. During integration, 2.2.0 fixed the handshake compatibility issue and 2.3.0 added native proxy support. That is why UBWA requires picows 2.3.0.

## 24 hours against live Binance

Local tests do not tell you what happens after hours of reconnects, traffic spikes and memory allocation.

So both libraries ran for 24 hours in parallel against `binance.com` with identical subscriptions.

Load:

*   `!ticker@arr`
    
*   `!miniTicker@arr`
    
*   aggTrade
    
*   trade
    
*   `depth20@100ms`
    
*   `kline_1m`
    
*   bookTicker
    
*   `depth@100ms`
    

across up to 50 USDT markets.

Host: 8 cores, 12 GB RAM, Python 3.13.5.

|  | picows | websockets |
| --- | --- | --- |
| Messages / data | 138.8 M / 49.9 GB | 137.9 M / 49.7 GB |
| Avg / peak msgs/s | 1,606 / 8,260 | 1,596 / 7,695 |
| RSS start → end | 62 → 126 MB | 63 → 149 MB |
| CPU avg | 9.5 % | 12.8 % |
| Reconnects (3 streams) | 2 / 80 / 2 | 2 / 88 / 2 |
| Reconnect duration | 5–6 s | 5–6 s |
| Max seconds without data | 5 s | 5 s |
| Errors / stalls / unrepairable streams | 0 / 0 / 0 | 0 / 0 / 0 |

**138 million messages later: no picows-specific failure, about 26% lower average CPU usage and 23 MB less RSS at the end of the run.**

The high reconnect count on one stream occurred almost entirely during a four-hour high-load window.

Both implementations disconnected in the same seconds with keepalive ping timeouts and recovered within five to six seconds.

That strongly points to a shared external or workload-related cause rather than either WebSocket implementation.

Memory rose during traffic peaks and then remained flat for the final hours despite further reconnects.

No sign of a reconnect leak.

## Why websockets is still the default

Because 138 million messages and a 24-hour soak are good evidence.

They are not the same as thousands of users running picows for months on different systems.

The compatibility layer is younger than `websockets`, and issue #108 already showed that small API differences can matter.

So for now:

**websockets remains the default. picows is opt-in.**

If picows holds up across enough real-world setups, the default can flip later.

The switch will remain either way.

## Try it

```bash
pip install --upgrade "unicorn-binance-websocket-api[picows]>=2.16.0"
```

```python
from unicorn_binance_websocket_api import BinanceWebSocketApiManager

ubwa = BinanceWebSocketApiManager(
    exchange="binance.com",
    websocket_library="picows"
)

ubwa.create_stream(
    ["aggTrade", "depth20@100ms"],
    ["btcusdt", "ethusdt", "solusdt"]
)
```

Run your workload with both libraries and compare CPU and memory usage.

If you have numbers, edge cases or failures, post them in [issue #477](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/issues/477).

That feedback will decide whether picows becomes the default.

The reasoning behind these decisions — including the native picows implementation I chose not to build — lives next to the code in `context/`, maintained with [Keep the Why](https://keepthewhy.com/).

* * *

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Bluesky](https://bsky.app/profile/o-zehentleitner.bsky.social), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯