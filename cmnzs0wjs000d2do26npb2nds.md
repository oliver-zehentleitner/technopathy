---
title: "How to Connect to binance.com REST API using Python via a SOCKS5 Proxy"
datePublished: 2026-04-15T08:16:31.440Z
cuid: cmnzs0wjs000d2do26npb2nds
slug: how-to-connect-to-binance-com-rest-api-using-python-via-a-socks5-proxy
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/a53e7ada-8ee1-4d48-80ef-3e292cd70cd7.webp

---

***US servers unfortunately can no longer connect to*** [***binance.com***](https://www.binance.com) ***(geoblocking).***

> **unicorn\_binance\_rest\_api.exceptions.BinanceAPIException: APIError(code=0): Service unavailable from a restricted location according to 'b. Eligibility' in** [**https://www.binance.com/en/terms**](https://www.binance.com/en/terms)**. Please contact customer service if you believe you received this message in error.**

This article is about the **Binance REST API**. If you want to [redirect a websocket connection to Binance via a SOCKS5 proxy](https://blog.technopathy.club/binance-websocket-via-socks5), please read [this article](https://blog.technopathy.club/binance-websocket-via-socks5).

A SOCKS5 server is a type of proxy server that routes network traffic between a client and a server. It allows clients to bypass internet restrictions and access restricted content by using a different IP address.

I will explain in this article **how to create a SOCKS5 proxy server using Linux and how to configure the connection in Python to access the Binance REST API through the SOCKS5 proxy**.

For this solution you need a simple virtual cloud Linux server such as a 4.51 EUR server with 20TB traffic volume included from [HETZNER CLOUD](https://www.hetzner.com), which supports the required traffic volume for the SOCKS5 proxy, and you need a working Python 3.8+ environment for the Python script!

> We would like to explicitly point out that in our opinion US citizens are exclusively authorized to trade on Binance.US and that this restriction must not be circumvented!

> The purpose of supporting a SOCKS5 proxy in the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite) and its modules is to allow non-US citizens to use US services. For example, GitHub Actions with UBS will not work without a SOCKS5 proxy, as they will inevitably run on servers in the US and be blocked by Binance.com. Moreover, it also seems justified that traders, data scientists and companies from the US analyze binance.com market data — as long as they do not trade there.

* * *

### Step 1 — Set up the SOCKS5 service

There are countless ways to create a SOCKS5 proxy:

*   **danted SOCKS server**
    
*   tinyproxy
    
*   openssh
    
*   and many more …
    

The **following instructions are for Debian and Ubuntu** — for [CentOS, RedHat, AWS EC2 and other Linux distributions that use `yum`](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb) as a package manager please [follow these instructions](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb).

#### danted SOCKS server

Project homepage: [https://www.inet.no/dante/](https://www.inet.no/dante/)

*   Log into your Linux server where you want to install the SOCKS5 proxy and get root privileges:
    

```bash
sudo -i
```

*   Install with `apt`:
    

```bash
apt update
apt install dante-server
```

> **Info:** After the installation dante does not work and still throws errors because it has not been configured yet!

*   Test the installation with:
    

```bash
danted -v
```

> **Info:** In Ubuntu and other distributions `danted` can also be called `sockd`.

*   Now edit the config file of danted with `nano` or `vi`:
    

```bash
nano /etc/danted.conf
```

or

```bash
vi /etc/danted.conf
```

Set the following configuration, replacing the IP `1.2.3.4` with your client IP of the Python script or use `0.0.0.0/0` as wildcard (**please consider your security concept!**):

```plaintext
logoutput: stderr
logoutput: /var/log/danted.log
internal: eth0 port = 1080
external: eth0
socksmethod: username none #rfc931
client pass {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: connect disconnect error
}
socks pass {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: connect disconnect error
}
```

*   Start danted:
    

```bash
systemctl start danted
```

*   Check the status:
    

```bash
systemctl status danted
```

*   If desired, create an autostart for danted:
    

```bash
systemctl enable danted
```

If you have chosen a server from [HETZNER CLOUD](https://www.hetzner.com), here is a detailed [step by step Dante SOCKS5 Proxy installation guide](https://community.hetzner.com/tutorials/install-and-configure-danted-proxy-socks5) tailored to Hetzner servers.

* * *

### Step 2 — Test the SOCKS5 Proxy

Before you start testing, open a stream of the logfile on your server console, so you can watch all of Dante's actions:

```bash
tail -f /var/log/danted.log
```

There are many ways to test the new SOCKS5 proxy:

\*\*Firefox:\*\**Settings* → search for *proxy* → enter the SOCKS5 proxy address and port number. Open [https://ipchicken.com](https://ipchicken.com) and check the IP address.

**Putty:** Open Putty and click on *Connection* → *Proxy* → enter the SOCKS5 proxy address and port number. Open a SSH connection.

**curl:** This should return your public IP address:

```bash
curl -x socks5://<your_ip_server>:<your_danted_port> ifconfig.co
```

Or with the [Python script](https://gist.github.com/oliver-zehentleitner/d55aa1ee03b7959e285c467e07eab2a4#file-ubra_socks5_proxy-py) we create in the next step.

* * *

### Step 3 — The Python script

For the REST API connection to Binance we use the [UNICORN Binance REST API](https://github.com/oliver-zehentleitner/unicorn-binance-rest-api). It directly allows the use of a SOCKS5 proxy and is configured via the parameters `socks5_proxy_server`, `socks5_proxy_user`, `socks5_proxy_pass` and `socks5_proxy_ssl_verification`.

> **Please note that proxy support is only cleanly supported from unicorn-binance-rest-api 2.7.0 and higher.**

Install/upgrade the dependencies:

```bash
python3 -m pip install unicorn_binance_rest_api
```

More info: [https://pypi.org/project/unicorn-binance-rest-api](https://pypi.org/project/unicorn-binance-rest-api)

[Save](https://gist.github.com/oliver-zehentleitner/d55aa1ee03b7959e285c467e07eab2a4) or copy this script to your system and replace `1.2.3.4` in `socks5_proxy` with the IP address of your SOCKS5 proxy server.

[Download this code from GitHub](https://gist.github.com/oliver-zehentleitner/d55aa1ee03b7959e285c467e07eab2a4/archive/81d94f546da381366b99d8855f05c06487283518.zip)

Start the file:

```bash
python3 ubra_socks5_proxy.py
```

Full **documentation for unicorn-binance-rest-api**: [https://oliver-zehentleitner.github.io/unicorn-binance-rest-api](https://oliver-zehentleitner.github.io/unicorn-binance-rest-api)

* * *

I hope you found this tutorial informative and enjoyable! 

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!

* * *

*Image source:* [*pixabay.com*](https://pixabay.com)