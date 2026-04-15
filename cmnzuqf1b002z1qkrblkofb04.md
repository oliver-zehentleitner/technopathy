---
title: "Create and Cancel Orders via WebSocket on Binance"
datePublished: 2026-04-15T09:32:21.025Z
cuid: cmnzuqf1b002z1qkrblkofb04
slug: create-and-cancel-orders-via-websocket-on-binance
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/04f19538-b770-4777-8ced-d4f87c1842c8.webp

---

*Until now it was only possible to receive data from Binance via WebSocket. To send requests to the Binance API, for example to create or cancel orders, you always had to use the slower REST API. This has changed now!* 🚀

Recently, the [Binance Spot Testnet](https://testnet.binance.vision) and [Binance Spot](https://www.binance.com) have added the ability to place orders, cancel orders, and handle other API requests via a WebSocket connection:

[Binance WebSocket API Documentation](https://binance-docs.github.io/apidocs/websocket_api/en/#change-log)

In Python, [UNICORN Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) already supports the new features to send API requests to Binance via WebSocket. To do this, we will go through the following steps:

1. **Binance** 
    — Account 
    — API key and API secret
2. **Installation of requirements** 
    — PIP 
    — Conda
3. **Create a Python script and establish a WebSocket API connection to Binance**
4. **Send requests and handle the responses** 
    — Global async function 
    — Global callback function 
    — Stream specific async function 
    — Stream specific callback function 
    — Request specific callback function 
    — Save answer in variable — Using the `stream_buffer` 
    — Multiple API Streams
5. **Available methods for API requests** 
    — `ubwa.api.spot.cancel_order()` 
    — `ubwa.api.spot.create_order()`
6. **Further information**

---

### 1. Binance

#### Account

To be able to trade on Binance you need a user account. If you don't have one yet, you can sign up via my [referral link](https://www.binance.com/en/activity/referral-entry/CPA?fromActivityPage=true&ref=CPA_008DXU7CWB). With this we both get 100 USDT cashback vouchers for a 50 USD deposit.

#### API key and API secret

How to create an API Key/Secret pair you can read in detail [here](https://technopathy.club/how-to-create-a-binance-api-key-and-api-secret-3bb5f47e360d).

---

### 2. Installation of requirements

**Minimum requirement** is a working [**Python**](https://www.python.org/) **3.8+** installation. To use the [UNICORN Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) in your Python script, you need to install it on the command line using one of the two commands:

#### With [PIP](https://pypi.org/project/unicorn-binance-websocket-api/):

This is the default way and should always work. It contains the plain source code and optimized [Cython](https://cython.org/) and [PyPy](https://www.pypy.org/) Wheels:

```bash
pip install unicorn-binance-websocket-api
```

#### Or with [Conda](https://anaconda.org/conda-forge/unicorn-binance-websocket-api):

This is only possible within an [Anaconda](https://www.anaconda.com/) environment:

```bash
conda install -c conda-forge unicorn-binance-websocket-api
```

---

### 3. Create a Python script and establish a WebSocket API connection to Binance

First we import `unicorn_binance_websocket_api`, define the callback function `handle_socket_message()` to print the received data and then store an instance of [`BinanceWebSocketApiManager()`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager) in the variable `ubwa`.

If you want to connect `BinanceWebSocketApiManager()` to the [testnet](https://testnet.binance.vision), you need to pass the string `'binance.com-testnet'` to the `exchange` parameter.

With `process_stream_data=handle_socket_message` we pass `BinanceWebSocketApiManager()` a global callback function, to this function all received responses of the API are passed by default.

By default, all API requests return a string with a JSON structure. To automatically convert the JSON structure to a Python dictionary, we configure `output_default="dict"`.

%[]
<!-- gist: Initiate UNICORN Binance WebSocket API -->

Next we create the API stream, for this it is important to set the parameter `api` to `True`, otherwise a connection to another WebSocket endpoint would be established, where the API requests would not work. Please make sure that you use a valid API key/secret pair and consider the IP whitelist restrictions when testing!

%[]
<!-- gist: Create a Binance WebSocket API stream -->

Now all the prerequisites are met for us to send and process requests to the Binance API.

---

### 4. Send requests and handle the responses

> **Note:** According to this scheme all methods of `ubwa.api` can be called and executed.

Using the `ubwa.api` object we can now send requests to the Binance API, e.g. a connection test with the `ubwa.api.spot.ping()` method.

**There are several ways to handle the API requests:**

- Global async function
- Global callback function
- Stream specific async function
- Stream specific callback function
- Request specific callback function
- Save answer in variable
- Using the `stream_buffer`
- Multiple API Streams

#### Global async function

When receiving the response from Binance, the function `handle_socket_message()` is executed which we passed when initiating `BinanceWebSocketApiManager()`, and this is where you can hook your code.

In our example, we simply output the received data:

%[]
<!-- gist: Test connectivity to the Binance WebSocket API -->

**Output:**

```
Received data:
{"id":"1cc8812a0c6b-08aa-6098-2742-ac0cedc8","status":200,"result":{},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":2}]}
```

I will use the example with `ubwa.api.spot.ping()` to introduce the other methods how you can receive and process the received data.

#### Global callback function

When receiving the response from Binance, the function `handle_socket_message()` is executed which we passed when initiating `BinanceWebSocketApiManager()`, and this is where you can hook your code.

In our example, we simply output the received data:

%[]
<!-- gist: Test connectivity to the Binance WebSocket API (global callback) -->

**Output:**

```
Received data:
{"id":"1cc8812a0c6b-08aa-6098-2742-ac0cedc8","status":200,"result":{},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":2}]}
```

#### Stream specific callback function

It is possible to pass a callback function to [`ubwa.create_stream()`](https://unicorn-binance-websocket-api.docs.lucit.tech/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager.create_stream) as well. Then, for receiving responses from this stream, **the stream specific callback function is used instead of the global callback function** we passed to `BinanceWebSocketApiManager()`.

%[]
<!-- gist: Create a Binance WebSocket API stream with a stream specific callback function -->

Now we can run ping again and this time we would use the callback function `handle_stream_message()` instead of `handle_socket_message()`.

%[]
<!-- gist: Test connectivity to the Binance WebSocket API (stream specific) -->

**Output:**

```
Received stream data:
{"id":"1cc8812a0c6b-08aa-6098-2742-ac0cedc8","status":200,"result":{},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":2}]}
```

#### Request specific callback function

Since not all types of data are treated the same way, there is also the possibility to process responses of a specific request with a specific callback function.

%[]
<!-- gist: Test connectivity with the Binance WebSocket API and let a specific callback function handle the response -->

**Output:**

```
Received ping response:
{"id":"1cc8812a0c6b-08aa-6098-2742-ac0cedc8","status":200,"result":{},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":2}]}
```

#### Save answer in variable

There is also the possibility to let the called function wait until the response to the request has arrived and then store it directly into a variable. The disadvantage of this method is that **the function becomes blocking**, whereby several requests can **only be processed sequentially**.

%[]
<!-- gist: Test connectivity with the Binance WebSocket API and take the response into a variable -->

**Output:**

```
Awaited ping response:
{"id":"1cc8812a0c6b-08aa-6098-2742-ac0cedc8","status":200,"result":{},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":2}]}
```

#### Using the `stream_buffer`

A completely different approach than the callback functions is the [`stream_buffer`](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/wiki/%60stream_buffer%60). This is the standard way of `BinanceWebSocketApiManager()` to process received data, here all received data is stored in a [`deque()`](https://docs.python.org/3/library/collections.html#collections.deque) stack and can be used as FIFO as well as LIFO stack to read the received data in a loop.

The main advantage of this solution is the **desynchronization between receiving and processing** the new data.

**What does this mean?** [UBWA](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) receives the new data in an AsyncIO event loop and processes them asynchronously. This means that the data is not received and processed sequentially, but that the callback functions within the event loop are started in parallel for each data record received, which can easily push your system to its limits during peak load times, e.g. if you receive more data faster than you can store in your database over a longer period for speed reasons.

Because each received record is simply stored in the `stream_buffer`, the processing is immediately finished for UBWA. You can easily access the new data in the `stream_buffer` in another thread or better another process and adjust the processing time to your circumstances. Additionally the `stream_buffer` can be monitored by you and offers [further options](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/wiki/%60stream_buffer%60).

As already mentioned the `stream_buffer` is the standard method of UBWA and is simply overwritten by setting the callback functions. So if you want to use it, do not pass a callback function in the class initialization of `BinanceWebSocketApiManager()` or anywhere else.

One way to use callback functions everywhere and still use the `stream_buffer` in a special situation is to explicitly pass [`ubwa.add_to_stream_buffer`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager.add_to_stream_buffer) as callback function.

Of course it is also possible to use different `stream_buffer`, how to do that is described in the [wiki](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/wiki/%60stream_buffer%60).

I now choose a simple example with only one global `stream_buffer`. If you have now simply omitted passing the callback functions, the global `stream_buffer` is activated by default and your stream will write its data there.

Hereby you get access to the oldest record in the stack:

%[]
<!-- gist: Get the oldest record from the UBWA stream_buffer -->

> **Note:** If the `stream_buffer` is empty, [`ubwa.pop_stream_data_from_stream_buffer()`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager.pop_stream_data_from_stream_buffer) returns the boolean value `False`!

For example, if you have problems with your database, you can catch the exception of the database connection in a `try` block and use [`ubwa.add_to_stream_buffer()`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager.add_to_stream_buffer) in the `except` block to save the record back into the `stream_buffer` and simply retrieve it later. Unfortunately, this messes up the chronological sorting! 🤨

%[]
<!-- gist: Example of how to pass a record back to the stream_buffer on a processing error -->

#### Multiple API streams

If there is more than one API stream (with `api=True`), the methods must be told which stream to use. The `stream_id` is used to uniquely identify the streams and is returned by [`ubwa.create_stream()`](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.manager.BinanceWebSocketApiManager.create_stream), alternatively you can also work with a `stream_label`. For this to work, a `stream_label` must also be defined when creating the stream with `ubwa.create_stream()`.

**Example with `stream_id`:**

%[]
<!-- gist: Send API requests with multiple WebSocket API streams (stream_id) -->

**Example with `stream_label`:**

%[]
<!-- gist: Send API requests with multiple WebSocket API streams (stream_label) -->

---

### 5. Available methods for API requests

> **Note:** Actions executed via WebSocket are subject to the same filters and rate restrictions as when executed via the REST API, and are also functionally equivalent: they provide the same functionality, accept the same parameters, and return the same status and error codes.

Each API request has its own list of accepted and mandatory parameters, gives different responses, and is subject to different request [limits](https://developers.binance.com/docs/binance-trading-api/websocket_api#general-information-on-rate-limits). To understand what is happening in the background, it is always advisable to also study the official documentation of [Unicorn Binance WebSocket API](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/) and the [Binance WebSocket API](https://developers.binance.com/docs/binance-trading-api/websocket_api).

- `ubwa.api.spot.cancel_order()`
- `ubwa.api.spot.create_order()`

#### Cancel open orders

Cancel all open orders on a symbol, including OCO orders.

*Docs: [UBWA](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/), [Binance](https://developers.binance.com/docs/binance-trading-api/websocket_api#cancel-open-orders-trade)*

If you cancel an order that is a part of an OCO pair, the entire OCO is canceled.

%[]
<!-- gist: Cancel an order via Binance WebSocket API (cancel open orders) -->

**Output:**

```json
{"id":"d37149a608c4-7951-fbbb-33f6-93d68e2d","status":200,"result":{"symbol":"BUSDUSDT","origClientOrderId":"1","orderId":942396461,"orderListId":-1,"clientOrderId":"rkAnxL9oEZr7wzKaCMiAuQ","price":"1.00000000","origQty":"15.00000000","executedQty":"0.00000000","cummulativeQuoteQty":"0.00000000","status":"CANCELED","timeInForce":"GTC","type":"LIMIT","side":"SELL","selfTradePreventionMode":"NONE"},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":3}]}
```

#### Cancel an order

Cancel an active order.

*Docs: [UBWA](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/), [Binance](https://developers.binance.com/docs/binance-trading-api/websocket_api#cancel-order-trade)*

If you cancel an order that is a part of an OCO pair, the entire OCO is canceled.

%[]
<!-- gist: Cancel an order via Binance WebSocket API -->

**Output:**

```json
{"id":"d37149a608c4-7951-fbbb-33f6-93d68e2d","status":200,"result":{"symbol":"BUSDUSDT","origClientOrderId":"1","orderId":942396461,"orderListId":-1,"clientOrderId":"rkAnxL9oEZr7wzKaCMiAuQ","price":"1.00000000","origQty":"15.00000000","executedQty":"0.00000000","cummulativeQuoteQty":"0.00000000","status":"CANCELED","timeInForce":"GTC","type":"LIMIT","side":"SELL","selfTradePreventionMode":"NONE"},"rateLimits":[{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":3}]}
```

#### Create an order

Create a new order.

*Docs: [UBWA](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/), [Binance](https://developers.binance.com/docs/binance-trading-api/websocket_api#place-new-order-trade)*

`ubwa.api.spot.create_order()` automatically creates a `client_order_id` and returns it. This way you can always uniquely identify the order in further steps.

%[]
<!-- gist: Create an order via Binance WebSocket API -->

**Output:**

```json
{"id":"4db8f58ee69e-d597-fb26-8551-786707fa","status":200,"result":{"symbol":"BUSDUSDT","orderId":942394911,"orderListId":-1,"clientOrderId":"1","transactTime":1680911574690,"price":"1.00000000","origQty":"15.00000000","executedQty":"0.00000000","cummulativeQuoteQty":"0.00000000","status":"NEW","timeInForce":"GTC","type":"LIMIT","side":"SELL","workingTime":1680911574690,"fills":[],"selfTradePreventionMode":"NONE"},"rateLimits":[{"rateLimitType":"ORDERS","interval":"SECOND","intervalNum":10,"limit":50,"count":1},{"rateLimitType":"ORDERS","interval":"DAY","intervalNum":1,"limit":160000,"count":20},{"rateLimitType":"REQUEST_WEIGHT","interval":"MINUTE","intervalNum":1,"limit":1200,"count":4}]}
```

---

### 6. Further information

There are many other [functions to send requests to the Binance API](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/unicorn_binance_websocket_api.html#unicorn_binance_websocket_api.api.BinanceWebSocketApiApi).

If you find bugs or have suggestions for improving the API implementation, you can [open an issue via GitHub](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/issues).

For more information please read the [documentation for unicorn-binance-websocket-api](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/).

---

I hope you found this tutorial informative and enjoyable! Follow me on [GitHub](https://github.com/oliver-zehentleitner) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!