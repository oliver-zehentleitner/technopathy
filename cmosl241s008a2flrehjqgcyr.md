---
title: "The PyPI Package Was Clean. That Was the Problem."
datePublished: 2026-05-05T12:06:49.605Z
cuid: cmosl241s008a2flrehjqgcyr
slug: the-pypi-package-was-clean-that-was-the-problem
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/1a74c14d-cc12-49d7-a245-a20150b1b387.png
tags: python, opensource, security, malware, pypi, supply-chain, typosquatting

---

Someone uploaded a package to PyPI called:

```text
unicorn-binance-websocket-api-onion
```

I do not maintain this package.

There is no official Tor, onion, proxy, or privacy-focused variant of UNICORN Binance WebSocket API.

The official package is:

```text
unicorn-binance-websocket-api
```

At first glance, the `-onion` suffix looks plausible enough to be mistaken for a legitimate variant.

That is exactly the problem.

And there is another reason this caught my attention immediately:

the same PyPI account, `onionj`, also maintains `pybotnet`, a package whose own description presents it as a framework for building remote control, botnet, trojan, or backdoor functionality.

So this was not just a random suffix collision.

It was a plausible package-name squat around my Binance trading library, published by an account that also publishes remote-control / botnet tooling, while keeping old LUCIT author metadata that makes the package look more official than it is.

I analyzed the package statically.

The version I reviewed does not contain malware. No obvious payload. No stage-2 download. No `exec`. No `eval`. No base64-decoded loader. No hidden exfiltration logic.

It is basically a near-identical fork of my upstream `1.46.2` release.

And I am reporting it anyway.

Because a clean typosquat can still be a supply-chain problem.

## Why I looked at it

I started checking project-name collisions around the UNICORN Binance Suite more actively after the recent [`nailproxy.space` GitHub impersonation campaign](https://blog.technopathy.club/nailproxy-space-github-malware-campaign).

That campaign abused recognizable open source project names, including mine, to lure developers into running malicious code.

I also published a separate [security warning about a fraudulent GitHub repository impersonating UNICORN Binance WebSocket API](https://blog.technopathy.club/security-warning-fraudulent-github-repository-impersonating-unicorn-binance-websocket-api).

That one was already weaponized.

This PyPI case is different.

It is not a live malware dropper.

But it belongs to the same family of supply-chain hygiene problems:

someone is using a recognizable project name to put adversary-controlled code into a developer package index.

That alone is enough reason to look closely.

## The package

The package I analyzed:

| Field | Value |
| --- | --- |
| Package | `unicorn-binance-websocket-api-onion` |
| Version | `1.46.2` |
| Release status | Only release observed |
| Upload time | `2025-05-27 14:45:12 UTC` |
| Wheel SHA256 | `65147181cff1a672652c843f85ec354c1b71f97ac1249381e067401ad196c6b8` |
| Sdist SHA256 | `4a7691898951f9eaee30f705d9a875def5b017f595a6a0c2e0d1e462868ce92a` |
| Wheel size | `71,744 B` |
| Sdist size | `81,966 B` |
| Declared author | `LUCIT Systems and Development <info@lucit.tech>` |
| Declared homepage | `https://github.com/onionj/unicorn-binance-websocket-api` |
| Homepage status at time of review | `404 Not Found` |
| PyPI maintainer | `onionj` |
| Real uploader from package artifacts | `onionj` / `onionj98@gmail.com` |

The declared author is the first obvious problem.

`LUCIT Systems and Development <info@lucit.tech>` is historical metadata from my own project. It is not the uploader’s identity. It makes the package look official at first glance.

It is not.

PyPI itself shows `onionj` as the maintainer of this project, while the unverified metadata still lists `LUCIT Systems and Development` as the author and keeps multiple old LUCIT / UNICORN Binance WebSocket API project links.

The declared homepage also points to:

```text
https://github.com/onionj/unicorn-binance-websocket-api
```

At the time of review, that URL returned `404 Not Found`.

So the project page says, in effect:

*   maintained by `onionj`,
    
*   authored by `LUCIT Systems and Development`,
    
*   linked to an unavailable GitHub repository,
    
*   packaged under a plausible `-onion` suffix,
    
*   and installed into the official import namespace.
    

That is exactly the kind of metadata confusion that package-name squats benefit from.

## Recent download footprint

Recent PyPI / Linehaul download stats at the time of review:

| Package | Last day | Last week | Last month |
| --- | --- | --- | --- |
| `unicorn-binance-websocket-api` | 292 | 2,331 | 14,854 |
| `unicorn-binance-websocket-api-onion` | 0 | 3 | 7 |
| `pybotnet` | 5 | 31 | 168 |

The squat is still small.

That is the entire reason this is worth catching now.

There is no large installed userbase yet to harm.

## Methodology

I treated the package as hostile until proven otherwise.

I did not run it.

I did not install it.

I did not import it.

I only downloaded the artifacts and inspected them passively:

```bash
curl -sS -o pkg.whl  https://files.pythonhosted.org/packages/.../*.whl
curl -sS -o pkg.tar.gz https://files.pythonhosted.org/packages/.../*.tar.gz
sha256sum pkg.whl pkg.tar.gz

python3 -m zipfile -l pkg.whl
tar -tzf pkg.tar.gz

python3 -m zipfile -e pkg.whl extracted/whl/
tar -xzf pkg.tar.gz -C extracted/
chmod -R a-x extracted/
```

Then I compared the contents against my official upstream `1.46.2` tag.

That is enough to answer the important question:

what code would a user get if they installed this package?

## What is inside

The package installs into the same Python import namespace as the official package:

```text
unicorn_binance_websocket_api/
```

That means a user can run:

```bash
pip install unicorn-binance-websocket-api-onion
```

and then write:

```python
import unicorn_binance_websocket_api
```

At the import-path level, the `-onion` suffix disappears.

That is not a small detail.

The distribution name is different, but the Python module name is the same. This makes the package a drop-in shadow of the official one.

If a future version becomes malicious, the import statement would not reveal anything unusual.

## The diff

The package is almost byte-identical to my official `1.46.2` release.

Most files are unchanged.

| File | Status |
| --- | --- |
| `__init__.py` | byte-identical |
| `api.py` | byte-identical |
| `connection.py` | byte-identical |
| `connection_settings.py` | byte-identical |
| `exceptions.py` | byte-identical |
| `manager.py` | **1-line change** |
| `restclient.py` | byte-identical |
| `restserver.py` | byte-identical |
| `sockets.py` | byte-identical |
| `README.md` | byte-identical |
| `LICENSE` | byte-identical |
| `setup.py` | 3-line change |

The relevant difference in `manager.py` is only this:

```diff
@@ -229,7 +229,7 @@
                  socks5_proxy_pass: Optional[str] = None,
                  socks5_proxy_ssl_verification: Optional[bool] = True,):
         threading.Thread.__init__(self)
-        self.name = "unicorn-binance-websocket-api"
+        self.name = "unicorn-binance-websocket-api-onion"
         self.version = "1.46.2"
```

That is the entire functional delta in `manager.py`: a name string.

The `setup.py` changes are also minimal:

```diff
-     name='unicorn-binance-websocket-api',
+     name='unicorn-binance-websocket-api-onion',

-     url="https://github.com/LUCIT-Systems-and-Development/unicorn-binance-websocket-api",
+     url="https://github.com/onionj/unicorn-binance-websocket-api",

-     install_requires=['colorama', 'requests', 'websocket-client', 'websockets==10.4', 'flask_restful',
+     install_requires=['colorama', 'requests', 'websocket-client', 'websockets>=10.4', 'flask_restful',
```

So the package was renamed, the project URL was changed, and one dependency pin was loosened.

The author metadata was not changed.

That is the part I do not like.

## What I did not find

I searched for the usual static indicators:

*   `exec(`
    
*   `eval(`
    
*   `compile(`
    
*   `__import__`
    
*   `base64.b64decode`
    
*   `marshal.loads`
    
*   `pickle.loads`
    
*   `subprocess`
    
*   `os.system`
    
*   `os.popen`
    
*   suspicious `requests.get` / `requests.post`
    
*   Telegram, Discord, webhook, `.onion`, `tor2web`
    
*   wallet paths
    
*   browser cookie paths
    
*   SSH or cloud credential paths
    
*   obvious C2 strings
    
*   simple obfuscation patterns
    

I did not find a malicious payload.

The hits I did find were legitimate pre-existing code from the original project: Binance request signing, GitHub version checks, platform/user-agent handling, and normal library logic.

So the verdict for this version is clear:

`unicorn-binance-websocket-api-onion` **version** `1.46.2` **is not malware.**

That does not make it acceptable.

## Why this is still a problem

The short version:

this is a clean package today, but it has the structure of a dormant supply-chain slot.

There are four separate issues here.

### 1\. Identity spoofing

The package claims:

```text
LUCIT Systems and Development <info@lucit.tech>
```

That identity does not belong to the uploader.

Users looking at package metadata may reasonably believe this is connected to the original project history.

It is not.

That is misleading even if the code is clean.

### 2\. Namespace shadowing

The package name is different, but the import namespace is the same as the official package:

```python
import unicorn_binance_websocket_api
```

That means the package behaves like a drop-in replacement.

There is no obvious runtime clue that the code came from a suffixed, unofficial distribution.

That is exactly the setup a future malicious update would exploit.

### 3\. The name is plausible

The suffix `-onion` is not random.

It sounds like it could be a Tor/onion-routing variant.

There are legitimate Python projects and extras around SOCKS, Tor, proxies, and privacy routing. A developer searching for such functionality could plausibly think this package is an official variant.

It is not.

### 4\. The maintainer profile makes the dormant-package risk concrete

This is the part that makes the package more concerning.

The PyPI account `onionj` maintains exactly two projects at the time of writing:

*   `unicorn-binance-websocket-api-onion`
    
*   `pybotnet`
    

That connection is public on the PyPI user profile.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/3f8c8083-1f9b-4ef4-bb9d-194da06589c6.png align="center")

`pybotnet` describes itself as:

> A Python framework for building remote control, botnet, trojan or backdoor with Telegram or other control panels

Its own feature list includes a Telegram control panel, reverse shell, file upload/download, remote Python code execution, screenshots, keylogger functionality, DoS, scheduling, and custom scripts.

I am not claiming that `pybotnet` is illegal malware.

I am also not claiming that `unicorn-binance-websocket-api-onion` is malicious in the version I reviewed.

But this combination matters.

The same account that publishes remote-control / botnet / trojan / backdoor tooling also uploaded a near-identical fork of my Binance trading library, under a plausible suffix, into the same import namespace, while keeping historical LUCIT author metadata that makes it look more official than it is.

That is not proof of an active attack.

But it is a very uncomfortable dormant supply-chain setup.

A clean package can still be a staging point:

1.  publish a harmless fork,
    
2.  use a plausible name,
    
3.  keep the official import namespace,
    
4.  accumulate a few downloads,
    
5.  wait until it lands in a script, CI job, Dockerfile, or AI-generated install instruction,
    
6.  weaponize a later release.
    

The right time to remove that setup is before step six.

## Clean does not mean harmless

This is the important part.

Security teams often look for malware.

That makes sense.

But package-index abuse is not always malware on day one.

Sometimes the first step is just occupying a name.

Or copying a brand.

Or shadowing an import namespace.

Or making a package look official enough that one user, one developer, one CI pipeline, or one AI-generated install instruction picks it up.

Once that happens, the uploader does not need to win the whole ecosystem.

They only need one useful installation.

This is why clean typosquats matter.

They are not harmless just because the current artifact has no payload.

They are infrastructure.

## How this relates to the GitHub impersonation campaign

The recent GitHub impersonation campaign was different.

That one was already weaponized.

It used fake repositories impersonating known open source projects and delivered malware through a Python dropper.

This PyPI package is not that.

But the trust primitive is the same:

the user relies on a recognizable project name.

On GitHub, that trust was abused through repository impersonation.

On PyPI, it is abused through package-name similarity, metadata spoofing, and import namespace shadowing.

Different vector.

Same weak point.

Developer trust.

## Why this matters for Binance developers

UNICORN Binance WebSocket API is used by developers who connect trading systems, bots, dashboards, and infrastructure to Binance.

That makes package identity more sensitive than it may look at first glance.

A malicious package in this space does not need to steal money directly to cause serious damage.

It could steal environment variables.

It could read API keys.

It could exfiltrate account metadata.

It could observe trading infrastructure.

It could tamper with stream handling.

It could simply prepare a developer machine for a second-stage compromise.

This is exactly why I care about these cases early.

A clean squat today can become a dangerous dependency tomorrow.

And when the affected ecosystem is automated trading, “tomorrow” is too late.

## Recommendations for users

If you use UNICORN Binance WebSocket API, the official package is:

```bash
pip install unicorn-binance-websocket-api
```

Not:

```bash
pip install unicorn-binance-websocket-api-onion
```

There is no official `-onion` package.

If you find `unicorn-binance-websocket-api-onion` in an environment, remove it and replace it with the official distribution.

Also check how it got there.

Look in:

*   `requirements.txt`
    
*   `pyproject.toml`
    
*   `setup.py`
    
*   Dockerfiles
    
*   CI workflows
    
*   deployment scripts
    
*   cached virtual environments
    
*   internal package mirrors
    

For any package, not just mine, check:

*   the exact distribution name
    
*   the project URL
    
*   the author metadata
    
*   the upload history
    
*   whether the import namespace matches a different known project
    
*   whether the package has a plausible-but-unofficial suffix like `-secure`, `-pro`, `-fast`, `-onion`, `-tor`, `-utils`, or `-plus`
    

These checks take seconds.

They prevent ugly surprises.

## Recommendation for PyPI

I am reporting this package.

The removal argument is not that the current artifact contains malware.

The removal argument is:

*   misleading project name
    
*   spoofed historical author metadata
    
*   same import namespace as the official package
    
*   no legitimate reason to exist under this naming pattern
    
*   concrete dormant-package risk
    

The installed footprint appears to be small.

That makes now the right time to remove it.

Removing a dormant squat before it is used is much less disruptive than removing a weaponized package after it has landed in production environments.

## Recommendation for maintainers

Run name-collision sweeps for your own projects.

Search PyPI for your package names plus common suffixes:

```text
-onion
-tor
-secure
-pro
-fast
-utils
-plus
-client
-api
```

If something looks suspicious:

1.  Download the artifacts.
    
2.  Do not install them.
    
3.  Do not import them.
    
4.  Inspect the wheel and sdist statically.
    
5.  Compare against your own release.
    
6.  Record hashes.
    
7.  Check metadata.
    
8.  Report with a concise technical summary.
    

You do not need a huge malware lab for this kind of first-pass triage.

A clean diff, hashes, and a clear explanation are already useful.

## Closing note

The interesting thing about this case is not the malware.

There is none in the version I reviewed.

The interesting thing is the setup:

a spoofed brand, a plausible suffix, the official import namespace, old author metadata, and a maintainer profile that makes the dormant slot uncomfortable.

That is enough.

A supply-chain attack does not start when the payload executes.

It often starts earlier, when trust is quietly acquired.

That is why I am reporting this package now.

Not because it is malware today.

Because it should not be allowed to become useful tomorrow.

* * *

I hope you found this informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding! ¯\_(ツ)\_/¯