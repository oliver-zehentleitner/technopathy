---
title: "How to Download Klines from Binance using Python?"
datePublished: 2026-04-10T10:06:37.830Z
cuid: cmnsqr8qt006u29ka5fb0huo4
slug: how-to-download-klines-from-binance-using-python
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/41cf0f0e-6103-4ef7-b39e-9fc6104c9e88.jpg

---

If you’re looking to download data from [Binance](https://www.binance.com) using [Python](https://www.python.org) (3.9+), there are two ways to do it: via REST API or via WebSocket API.

REST API is a simple and easy-to-use method that allows you to download historical data from [Binance](https://www.binance.com), but also to trigger functions such as creating a new order. You can use [Unicorn Binance REST API](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api), a sub module of the [Unicorn Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite), to make REST API calls and retrieve data in a JSON format.

WebSocket API, on the other hand, is a more advanced method that allows you to stream real-time data from Binance. You can use [Unicorn Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api), another sub module of the [Unicorn Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite), to subscribe to WebSocket streams and retrieve data in a JSON format.

[Unicorn Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite) is a comprehensive [Python](https://www.python.org) library for interacting with the Binance API. It consists of several sub modules, including [Unicorn Binance REST API](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api) and [Unicorn Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api), as well as other sub modules for trading, order management, and more.

To get started with [Unicorn Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite), you’ll need to install it using [pip](https://pypi.org/project/unicorn-binance-suite/):

`$ pip install unicorn-binance-suite`

Or with [conda](https://anaconda.org/conda-forge/unicorn-binance-suite):

`$ conda install -c conda-forge unicorn-binance-suite`

Once you have it installed, you can import the sub modules you need and start making API calls:

```plaintext
from unicorn\_binance\_rest_api.manager import BinanceRestApiManager  
  
ubra = BinanceRestApiManager(exchange='binance.com')  
  
klines = ubra.get_klines(symbol='BTCUSDT', interval='1h')  
print(klines) 
```

This example uses the [UNICORN Binance REST API](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api) to download klines data for the BTCUSDT trading pair with a 1-hour interval.

Similarly, you can use the [Unicorn Binance WebSocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) to subscribe to WebSocket streams and retrieve real-time data:

```plaintext
from unicorn_binance_websocket_api import BinanceWebSocketApiManager  
  
ubwa = BinanceWebSocketApiManager(exchange="binance.com")  
ubwa.create_stream(channels='kline_1h', markets='BTCUSDT')  
  
while True:  
    klines = ubwa.pop_stream_data_from_stream_buffer()  
    if klines:  
        print(klines)
```

This example creates a WebSocket stream for klines data with a 1-hour interval for the BTCUSDT trading pair.

When using the [Unicorn Binance Websocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api) to download data from Binance, it’s important to keep in mind that the first data packet you receive will always be `{"result": null, "id": 1}`. This is Binance's way of acknowledging that your subscription request was successful. After that, you’ll start receiving the actual data packets that you subscribed to.

In summary, if you’re looking to download data from Binance using Python, you can use either the REST API or the WebSocket API. The [Unicorn Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite) provides a comprehensive Python library for interacting with the Binance API, with sub modules for both REST and WebSocket APIs, making it easy to get started with downloading data from Binance.

To connect to other Binance exchanges just change the `exchange` string:  
[Binance](https://www.binance.com): `binance.com`  
[Binance Testnet](https://testnet.binance.vision/): `binance.com-testnet`  
[Binance Margin](https://www.binance.com): `binance.com-margin`  
[Binance Margin Testnet](https://testnet.binance.vision/): `binance.com-margin-testnet`  
[Binance Isolated Margin](https://www.binance.com): `binance.com-isolated_margin`  
[Binance Isolated Margin Testnet](https://testnet.binance.vision/): `binance.com-isolated_margin-testnet` 
[Binance USD-M Futures](https://www.binance.com): `binance.com-futures`  
[Binance USD-M Futures Testnet](https://testnet.binancefuture.com): `binance.com-futures-testnet`  
[Binance Coin-M Futures](https://www.binance.com): `binance.com-coin_futures`  
[Binance US](https://www.binance.us): `binance.us`  
[Binance TR](https://www.trbinance.com): `trbinance.com`

You can find the documentations here:  
[https://oliver-zehentleitner.github.io/unicorn-binance-rest-api/](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api/)  
[https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/)

I hope you found this tutorial informative and enjoyable! 

Thank you for reading, and happy coding!

Image source: [https://pixabay.com](https://pixabay.com)