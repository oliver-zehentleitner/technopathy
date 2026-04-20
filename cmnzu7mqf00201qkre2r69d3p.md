---
title: "Restful Binance Requests in Python with UNICORN Binance REST API"
datePublished: 2026-04-15T09:17:44.537Z
cuid: cmnzu7mqf00201qkre2r69d3p
slug: restful-binance-requests-in-python-with-unicorn-binance-rest-api
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/6246be15-0b28-47cb-a160-56bcbaef8399.webp

---

*In this article I will show you an easy way to get started with the Binance REST API using Python.*

**We use the** `unicorn-binance-rest-api` **package:**

*   PyPI: [https://pypi.org/project/unicorn-binance-rest-api](https://pypi.org/project/unicorn-binance-rest-api)
    
*   Documentation: [https://oliver-zehentleitner.github.io/unicorn-binance-rest-api](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api)
    
*   GitHub: [https://github.com/oliver-zehentleitner/unicorn-binance-rest-api](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api)
    

* * *

### Installation

Install via pip:

```bash
pip install unicorn-binance-rest-api
```

Or via conda:

```bash
conda install -c conda-forge unicorn-binance-rest-api
```

Alternatively download the [latest release](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api/releases) or clone the repository to run the [examples](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api/tree/master/examples) locally:

```bash
git clone git@github.com:oliver-zehentleitner/unicorn-binance-rest-api.git
```

* * *

### How to use BinanceRestApiManager?

Import [`BinanceRestApiManager`](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api/unicorn_binance_rest_api.html#unicorn_binance_rest_api.manager.BinanceRestApiManager) and create an instance:

```python
from unicorn_binance_rest_api.manager import BinanceRestApiManager

ubra = BinanceRestApiManager(exchange="binance.com")
```

Configure logging — use `DEBUG` instead of `INFO` for a very verbose mode:

```python
import logging
import os

logging.getLogger("unicorn_binance_rest_api")
logging.basicConfig(level=logging.INFO,
                    filename=os.path.basename(__file__) + '.log',
                    format="{asctime} [{levelname:8}] {process} {thread} {module}: {message}",
                    style="{")
```

Now you can execute various methods via the `ubra` instance:

```python
# Get market depth
depth = ubra.get_order_book(symbol='BNBBTC')
print(f"depth: {depth}")

# Get all symbol prices
prices = ubra.get_all_tickers()
print(f"prices: {prices}")

# Get used weight
used_weight = ubra.get_used_weight()
print(f"weight: {used_weight}")

# Retrieve 30-minute klines for the last month of 2021
klines_30m = ubra.get_historical_klines("BTCUSDT", "30m", "1 Dec, 2021", "1 Jan, 2022")
print(f"klines_30m:\r\n{klines_30m}")
```

For private data, provide API credentials when creating the instance:

```python
api_key = "aaa"
api_secret = "bbb"

ubra = BinanceRestApiManager(api_key=api_key,
                             api_secret=api_secret,
                             exchange="binance.com")
```

Get private data:

```python
# Get account info
account = ubra.get_account()
print(f"account: {account}")

# Get all orders
all_orders = ubra.get_all_orders(symbol='BTCUSDT', limit=10)
print(f"all_orders: {all_orders}")

# Get open orders
open_orders = ubra.get_open_orders(symbol='BTCUSDT')
print(f"open_orders: {open_orders}")
```

A full overview of all available methods can be found in the [documentation](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api).

* * *

I hope you found this tutorial informative and enjoyable! 

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!

* * *

*Image source:* [*pixabay.com*](https://pixabay.com)