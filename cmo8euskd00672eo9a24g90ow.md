---
title: "Buy an Asset and instantly create a Take Profit and Stop Loss OCO Sell Order using Python in Binance Isolated Margin"
datePublished: 2026-04-21T09:17:46.911Z
cuid: cmo8euskd00672eo9a24g90ow
slug: buy-an-asset-and-instantly-create-a-take-profit-and-stop-loss-oco-sell-order-using-python-in-binance-isolated-margin
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/80d23f07-2913-4ea7-bebe-3c49733a54d4.png
tags: python, binance, oco-orders, unicorn-binance-rest-api

---

> Automate your Binance trading strategy by creating OCO sell orders with Python in isolated margin accounts.

There is always the justified desire to buy an asset and at the same time to create a take profit and a stop loss order. In this way, if a profit is made, the asset is sold and any risks are minimized.

This strategy consists of two orders: a BUY and an OCO SELL order.

OCO stands for "[One Cancells the Other](https://academy.binance.com/en/glossary/oco-order)" and creates both a sell order for Take Profit and one for Stop Loss:

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/7f0a621d-882c-4a43-82c9-edcdbc40839c.png align="center")

In this tutorial we will create a Market Buy Order for Binance Isolated Margin in a Python Script and buy some BTC with 15 USDT. We will take the purchased amount of BTC and use it to create an OCO Sell Order that includes both Take Profit and Stop Loss (LIMIT).

In order for the code from this example to work for you, you need to transfer at least 15 USDT to your [Isolated Margin account on Binance](https://www.binance.com/en/support/faq/how-to-use-the-isolated-margin-mode-on-binance-2da03a8901de4754ab98800a3a92fdd4) and you need an [API key/secret pair from Binance.com](https://blog.technopathy.club/how-to-create-a-binance-api-key-and-api-secret).

Ok, here we go :)

First, make sure that the latest [UNICORN Binance REST API](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api) version is installed:

```bash
$ python3 -m pip install unicorn-binance-rest-api --upgrade
```

Download this script and enter your API key/secret pair:

%[https://gist.github.com/oliver-zehentleitner/be1c4880b79db7596cf40b7d169b72f6] 

The comments in the script should help you understand the code.

Watch this video to see how I start the script and what happens on Binance:

%[https://youtu.be/iuwoweIFJL4] 

* * *

I hope you found this tutorial informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!