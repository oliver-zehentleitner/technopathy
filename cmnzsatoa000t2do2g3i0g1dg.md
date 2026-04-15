---
title: "Installation and Configuration of SOCKS Proxy Danted on Redhat/CentOS/AWS EC2 from Source Code"
datePublished: 2026-04-15T08:24:14.268Z
cuid: cmnzsatoa000t2do2g3i0g1dg
slug: installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-source-code
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/a056790f-19f6-4837-8f11-cf44ffb43f96.webp

---

The goal of this tutorial is to describe the installation and configuration of the SOCKS5 Proxy [Dante](https://www.inet.no/dante/) on a CentOS, Redhat, AWS EC2 or similar Linux distribution using the `yum` package manager to end up with **a working SOCKS5 proxy**.

For **[Debian and Ubuntu](https://technopathy.club/installation-and-configuration-of-dante-on-debian-ubuntu-with-apt-ebce7410e7d2)** please follow [these instructions](https://technopathy.club/installation-and-configuration-of-dante-on-debian-ubuntu-with-apt-ebce7410e7d2).

If you are still looking for a server for this project, I can recommend the **cx31** server for 4.51 EUR/month with 20TB traffic volume from the European provider [HETZNER CLOUD](https://www.hetzner.com).

Log into your server, access root privileges and run a system update:

```bash
sudo -i
yum update -y
```

---

### Step 1 — Installation

The dante-server package is not available in the Amazon Linux package repository by default. Instead, build it from source following these steps.

Install the build tools and dependencies:

```bash
yum groupinstall "Development Tools"
yum install libevent-devel
```

Download the Dante source code:

```bash
curl -O https://www.inet.no/dante/files/dante-1.4.2.tar.gz
```

Extract the source code:

```bash
tar xvzf dante-1.4.2.tar.gz
cd dante-1.4.2
```

Configure and compile dante-server:

```bash
./configure
make
make install
```

Verify the installation:

```bash
sockd -v
```

---

### Step 2 — Configuration

Copy the `sockd.conf` file to the appropriate location:

```bash
cp /home/ec2-user/dante-1.4.2/example/sockd.conf /etc/sockd.conf
```

To find the IP address of the `eth0` interface run:

```bash
ip add show eth0
```

**Example output:**

```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 06:72:6b:5d:09:dd brd ff:ff:ff:ff:ff:ff
    inet 172.1.2.3/20 brd 172.1.2.255 scope global dynamic eth0
    valid_lft 3451sec preferred_lft 3451sec
    inet6 fe80::472:6bff:fe5d:9dd/64 scope link
    valid_lft forever preferred_lft forever
```

**Replace `172.1.2.3` with your server IP and `1.2.3.4` with your client IP from where you want to connect.**

Edit the config file:

```bash
vi /etc/sockd.conf
```

```
internal: 0.0.0.0 port = 1080
external: 172.1.2.3
logoutput: stderr
logoutput: /var/log/danted.log
clientmethod: none
socksmethod: none
user.privileged: root
user.notprivileged: nobody

client pass {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: error connect disconnect
}
client block {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: connect error
}
socks pass {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: error connect disconnect
}
socks block {
    from: 1.2.3.4/32 to: 0.0.0.0/0
    log: connect error
}
```

The system service file is not included, so create one:

```bash
vi /etc/systemd/system/sockd.service
```

```ini
[Unit]
Description=SOCKS5 Proxy Server
After=network.target

[Service]
ExecStart=/usr/local/sbin/sockd -D
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Reload the system configuration and start the SOCKS5 proxy:

```bash
systemctl daemon-reload
systemctl start sockd.service
systemctl enable sockd.service
```

Check the status to confirm it's running:

```bash
systemctl status sockd
```

If everything worked, you now have a working SOCKS5 proxy server that only accepts connections from IP `1.2.3.4` and forwards to any destination address.

Additionally you can [enable proper authentication](https://www.inet.no/dante/doc/latest/config/auth.html) — more info can be found [here](https://www.inet.no/dante/doc/latest/config/auth.html).

---

I appreciate constructive feedback that helps me improve the content of this article. Follow me on [GitHub](https://github.com/oliver-zehentleitner) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) for more updates and insights!

*Image source: [pixabay.com](https://pixabay.com)*