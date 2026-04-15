---
title: "Create and Cancel Orders via WebSocket on Binance"
datePublished: 2026-04-15T09:32:21.025Z
cuid: cmnzuqf1b002z1qkrblkofb04
slug: create-and-cancel-orders-via-websocket-on-binance
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/04f19538-b770-4777-8ced-d4f87c1842c8.webp

---

# Create and Cancel Orders via WebSocket on Binance

*How to access the Binance API via WebSocket in Python*

*Until now it was only possible to receive data from Binance via WebSocket. To send requests to the Binance API — for example to create or cancel orders — you always had to use the slower REST API. This has changed now!* 🚀

Recently, [Binance Spot Testnet](https://testnet.binance.vision) and [Binance Spot](https://www.binance.com) added the ability to place orders, cancel orders, and handle other API requests via a WebSocket connection:

[Binance WebSocket API Documentation](https://binance-docs.github.io/apidocs/websocket_api/en/#change-log)

[UNICORN Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) already supports these new features in Python. The following steps will walk you through the setup:

1. [Binance Account — API key and API secret](#1-binance)
2. [Installation of requirements](#2-installation-of-requirements)
3. [Create a Python script and establish a WebSocket API connection](#3-create-a-python-script-and-establish-a-websocket-api-connection)
4. [Send requests and handle the responses](#4-send-requests-and-handle-the-responses)
5. [Available methods for API requests](#5-available-methods-for-api-requests)
6. [Further information](#6-further-information)

---

### 1. Binance

#### Account

To trade on Binance you need a user account. If you don't have one yet, you can [sign up here](https://www.binance.com).

#### API key and API secret

How to create an API Key/Secret pair is described in detail [here](https://technopathy.club/how-to-create-a-binance-api-key-and-api-secret-3bb5f47e360d).

---

### 2. Installation of requirements

Minimum requirement is a working **Python 3.8+** installation.

Install via pip:

```bash
pip install unicorn-binance-websocket-api
```

Or within an [Anaconda](https://www.anaconda.com/) environment via conda:

```bash
conda install -c conda-forge unicorn-binance-websocket-api
```

---

### 3. Create a Python script and establish a WebSocket API connection

First import `unicorn_binance_websocket_api`, define the callback function `handle_socket_message()` to print received data, and store an instance of [`BinanceWebSocketApiManager()`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager) in the variable `ubwa`.

To connect to the [testnet](https://testnet.binance.vision), pass `'binance.com-testnet'` to the `exchange` parameter.

With `process_stream_data=handle_socket_message` you pass a global callback function — all received API responses are forwarded to it by default.

To automatically convert JSON responses to a Python dictionary, configure `output_default="dict"`.

> ⚠️ *Code examples were not included in the exported source — please add them manually from the [original repository](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/tree/master/examples).*

---

### 4. Send requests and handle the responses

The following patterns are supported:

- Global async function
- Global callback function
- Stream specific async function
- Stream specific callback function
- Request specific callback function
- Save answer in variable
- Using the `stream_buffer`
- Multiple API streams

> ⚠️ *Code examples were not included in the exported source — please add them manually.*

**Example output:**

```json
{
  "id": "4db8f58ee69e-d597-fb26-8551-786707fa",
  "status": 200,
  "result": {
    "symbol": "BUSDUSDT",
    "orderId": 942394911,
    "orderListId": -1,
    "clientOrderId": "1",
    "transactTime": 1680911574690,
    "price": "1.00000000",
    "origQty": "15.00000000",
    "executedQty": "0.00000000",
    "cummulativeQuoteQty": "0.00000000",
    "status": "NEW",
    "timeInForce": "GTC",
    "type": "LIMIT",
    "side": "SELL",
    "workingTime": 1680911574690,
    "fills": [],
    "selfTradePreventionMode": "NONE"
  },
  "rateLimits": [
    {"rateLimitType": "ORDERS", "interval": "SECOND", "intervalNum": 10, "limit": 50, "count": 1},
    {"rateLimitType": "ORDERS", "interval": "DAY", "intervalNum": 1, "limit": 160000, "count": 20},
    {"rateLimitType": "REQUEST_WEIGHT", "interval": "MINUTE", "intervalNum": 1, "limit": 1200, "count": 4}
  ]
}
```

---

### 5. Available methods for API requests

- `ubwa.api.spot.cancel_order()`
- `ubwa.api.spot.create_order()`

> ⚠️ *Code examples were not included in the exported source — please add them manually.*

---

### 6. Further information

There are many other [functions to send requests to the Binance API](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.api.BinanceWebSocketApiApi).

If you find bugs or have suggestions, feel free to [open an issue on GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/issues).

Full [documentation for unicorn-binance-websocket-api](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/).

---

I hope you found this tutorial informative and enjoyable! Follow me on [GitHub](https://github.com/oliver-zehentleitner) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!