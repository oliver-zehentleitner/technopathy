---
title: "Installation and Configuration of Dante on Debian/Ubuntu with `apt`"
datePublished: 2026-04-15T09:22:10.602Z
cuid: cmnzudc15002c1qkrea4pg0p7
slug: installation-and-configuration-of-dante-on-debian-ubuntu-with-apt
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/27a06642-444b-409a-9e85-95c420ff0c91.webp

---

*Instructions for installing and configuring a Dante SOCKS5 proxy on Debian/Ubuntu on the command line.*

The **following instructions are for Debian and Ubuntu** — for [CentOS, RedHat, AWS EC2 and other Linux distributions that use `yum`](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb) as a package manager please [follow these instructions](https://technopathy.club/installation-and-configuration-of-socks-proxy-danted-on-redhat-centos-aws-ec2-from-soure-code-f643a183cccb).

If you are still looking for a server for this project, I can recommend the **cx31** server for 4.51 EUR/month with 20TB traffic volume from the European provider [HETZNER CLOUD](https://www.hetzner.com).

---

### Step 1 — Install danted

Project homepage: [https://www.inet.no/dante/](https://www.inet.no/dante/)

Log into your Linux server and get root privileges:

```bash
sudo -i
```

Install with `apt`:

```bash
apt update
apt install dante-server
```

> **Info:** After installation dante does not work yet and will throw errors — it has not been configured yet!

Verify the installation:

```bash
danted -v
```

> **Info:** In Ubuntu and other distributions `danted` can also be called `sockd`.

Edit the config file with `nano` or `vi`:

```bash
nano /etc/danted.conf
```

or

```bash
vi /etc/danted.conf
```

Set the following configuration. Replace `1.2.3.4` with your client IP or use `0.0.0.0/0` as wildcard (**please consider your security concept!**):

```
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

Start danted:

```bash
systemctl start danted
```

Check the status:

```bash
systemctl status danted
```

If desired, enable autostart:

```bash
systemctl enable danted
```

You can now connect to the SOCKS5 proxy via the public server IP on port 1080.

If you have chosen a server from [HETZNER CLOUD](https://www.hetzner.com), here is a detailed [step by step Dante SOCKS5 Proxy installation guide](https://community.hetzner.com/tutorials/install-and-configure-danted-proxy-socks5) tailored to Hetzner servers.

---

### Step 2 — Test the SOCKS5 Proxy

There are many ways to test the new SOCKS5 proxy.

**Firefox:**
*Settings* → search for *proxy* → enter the SOCKS5 proxy address and port number. Open [https://ipchicken.com](https://ipchicken.com) and check the IP address.

**Putty:**
Open Putty and click on *Connection* → *Proxy* → enter the SOCKS5 proxy address and port number. Open a SSH connection.

**curl:**
This should return your public IP address:

```bash
curl -x socks5://<your_ip_server>:<your_danted_port> ifconfig.co
```

---

I appreciate constructive feedback that helps me improve the content of this article. Follow me on [GitHub](https://github.com/oliver-zehentleitner) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) for more updates and insights!

*Image source: [pixabay.com](https://pixabay.com)*