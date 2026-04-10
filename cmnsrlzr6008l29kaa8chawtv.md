---
title: "Binance Websocket via SOCKS5"
datePublished: 2026-04-10T10:30:32.518Z
cuid: cmnsrlzr6008l29kaa8chawtv
slug: binance-websocket-via-socks5
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/a6608ac2-e339-430a-801e-d74e4f7e3a56.webp

---

***US servers unfortunately can no longer connect to*** [***binance.com***](https://www.binance.com) ***(geoblocking).***

> ***HTTP 451 error “Service unavailable from a restricted location…”***

This article is about the **Binance Websocket API**, if you want to [redirect a REST connection to Binance via a SOCKS5 proxy](https://technopathy.club/how-to-connect-to-binance-com-rest-api-using-python-via-a-socks5-proxy-638dbbecacfd), please [read this article](https://technopathy.club/how-to-connect-to-binance-com-rest-api-using-python-via-a-socks5-proxy-638dbbecacfd).

A SOCKS5 server is a type of proxy server that routes network traffic between a client and a server. It allows clients to bypass internet restrictions and access restricted content by using a different IP address.

I will explain in this article **how to create a SOCKS5 proxy server using Linux and how to configure the websocket connection in Python to access the Binance API through the SOCKS5 proxy**.

For this solution you need a simple virtual cloud Linux server such as a 4.51 EUR server with 20TB traffic volume included from [HETZNER CLOUD](https://www.hetzner.com), which supports the required traffic volume for the SOCKS5 proxy, and you need a working Python 3.8+ environment for the Python script!

> We would like to explicitly point out that in our opinion US citizens are exclusively authorized to trade on Binance.US and that this restriction must not be circumvented!

> The purpose of supporting a SOCKS5 proxy in the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite) and its modules is to allow non-US citizens to use US services. For example, Github actions with UBS will not work without a SOCKS5 proxy, as they will inevitably run on servers in the US and be blocked by Binance.com. Moreover, it also seems justified that traders, data scientists and companies from the US analyze binance.com market data — as long as they do not trade there.

### **1\. Step — Set up the** SOCKS5 **service**

There are countless ways to create a SOCKS5 proxy:

*   **danted SOCKS server**
    
*   tinyproxy
    
*   openssh
    
*   and many more …
    

The **following instructions are for Debian and Ubuntu** — for [CentOS, RedHat, AWS EC2 and other Linux distributions that use *\`yum\`*](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb) as a package manager please [follow these instructions](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb).

#### danted SOCKS server

Project homepage: [https://www.inet.no/dante/](https://www.inet.no/dante/)

*   Log into your Linux server where you want to install the SOCKS5 proxy and get root privileges:  
`sudo -i`
    
*   Install with `apt`:  
`apt update`  
`apt install dante-server`

**Info**: After the installation dante does not work and still throws errors because it has not been configured yet!
    
*   Test the installation with:  
    `danted -v`  
    **Info**: In Ubuntu and other distributions the `danted` can also be called `sockd`.
    
*   Now edit the configfile of danted with `nano` or `vi`:  
    `nano /etc/danted.conf`  
    or  
    `vi /etc/danted.conf`  
    and set the following configuration, replace the IP “*1.2.3.4*” with your client IP of the Python script or use “*0.0.0.0/0*" as wildcard (**please consider your security concept!**):
    

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
    `$ systemctl start danted`
    
*   Check the status:  
    `systemctl status danted`
    
*   If desired create an autostart for danted:  
    `systemctl enable danted`
    

If you have chosen a server from [HETZNER CLOUD](https://www.hetzner.com), here is a detailed [step by step Dante SOCKS5 Proxy installation guide](https://community.hetzner.com/tutorials/install-and-configure-danted-proxy-socks5) tailored to Hetzner servers.

### 2\. Step — Test the SOCKS5 Proxy

Before you start testing, open a stream of the logfile on your server console, so you can watch all of Dante’s actions:  
`tail -f /var/log/danted.log`

There are many ways to test the new SOCKS5 proxy:

**Firefox**:  
“*Settings*” -> search for “*proxy*” -> enter the SOCKS5 proxy address and port number. Open [https://ipchicken.com](https://ipchicken.com) and check the IP address.

**Putty**:  
Open Putty and click on “*Connection*” -> “*Proxy*” -> enter the SOCKS5 proxy address and port number. Open a SSH connection.

**curl**:  
This should return your public IP address:  
`curl -x socks5://<your_ip_server>:<your_danted_port> ifconfig.co`

Or with the [**python script**](https://gist.github.com/oliver-zehentleitner/74f6c5a461b01b5249c44e335ccf4e88#file-ubwa_socks5_proxy-py) we create next.

### 3\. Step — The Python script

For the websocket connection to the Binance API we use the [UNICORN Binance Websocket API](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api). It directly allows the use of a SOCKS5 proxy and is configured via the parameters `socks5_proxy_server`, `socks5_proxy_user`, `socks5_proxy_pass` and `socks5_proxy_ssl_verification`.

> ***Please note that the proxy support is only cleanly supported from unicorn-binance-websocket-api 2.0.0 and higher.***

Install/upgrade the dependencies:  
`$ python3 -m pip install unicorn_binance_websocket_api`  
Info: [https://pypi.org/project/unicorn-binance-websocket-api](https://pypi.org/project/unicorn-binance-websocket-api/)

[Save or copy this script](https://gist.github.com/oliver-zehentleitner/74f6c5a461b01b5249c44e335ccf4e88) to your system and replace “*1.2.3.4*” in `_socks5_proxy_` with the IP address of your Socks5 Proxy server.

Start the file:  
`python3 ubwa\_socks5\_proxy.py`

You can find the full **documentation for unicorn-binance-websocket-api** here: [https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api](https://oliver-zehentleitner.github.io/unicorn-binance-websocket-api/)

* * *

I hope you found this tutorial informative and enjoyable!

Thank you for reading, and happy coding!

Image source: [https://pixabay.com](https://pixabay.com)