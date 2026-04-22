---
title: "Security Warning: Fraudulent GitHub Repository Impersonating UNICORN Binance WebSocket API"
datePublished: 2026-04-22T08:19:20.083Z
cuid: cmo9s7hcf003s2ajcalbwblv9
slug: security-warning-fraudulent-github-repository-impersonating-unicorn-binance-websocket-api
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e5fcff05-c154-431c-bf90-4122169b23ed.png
tags: github, python, security, malware, binance

---

> **TL;DR:** Do **not** clone or run `gesine1541ro7/UNICORN-Binance-WebSocket-API`.  
> Based on the public startup path, it stages and executes a hidden Windows PE payload at launch.  
> The legitimate project lives here: [`oliver-zehentleitner/unicorn-binance-websocket-api`](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api)  
> Official package: [`unicorn-binance-websocket-api` on PyPI](https://pypi.org/project/unicorn-binance-websocket-api/)

A GitHub repository using the name **UNICORN-Binance-WebSocket-API** is not a legitimate UBWA console, clone, or community wrapper.

After reviewing the public repository contents, my assessment is:

> **This is a fraudulent repository impersonating the UNICORN Binance WebSocket API project while using a fake Binance console narrative as cover for staged Windows payload execution.**

This post documents what I observed, why I consider it malicious, and what I am doing next.

## Why I Looked at It

I maintain the legitimate **UNICORN Binance WebSocket API** project through the official project channels:

- [Official GitHub repository: oliver-zehentleitner/unicorn-binance-websocket-api](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api)
- [Official PyPI package: unicorn-binance-websocket-api](https://pypi.org/project/unicorn-binance-websocket-api/)

The repository I analyzed is this one:

- [Fraudulent repository: gesine1541ro7/UNICORN-Binance-WebSocket-API](https://github.com/gesine1541ro7/UNICORN-Binance-WebSocket-API)

What initially stood out was not just the reused project name, but also the mismatch between its visible development footprint and its social proof.

At the time of writing, the repository publicly shows:

- **816 stars**
- **453 forks**
- **0 issues**
- **0 pull requests**
- **1 contributor**
- **4 commits**

That profile is highly unusual for a repository with such limited visible activity.

Another detail that stood out early is the mismatch between the reported fork count and the practically empty-looking public fork surface. Combined with the naming patterns I observed on visible fork-owner accounts, that makes the social proof look even less credible.

## This Is Not a UBWA Console

On the surface, the repository presents itself as a Python-based Binance streaming console. Its README describes it as an interactive console for real-time market data streaming using the `unicorn-binance-websocket-api` library.

That framing is deceptive.

This repository is **not** the official UBWA project. It is also not a harmless console wrapper in any meaningful sense. The console story functions as camouflage around the real startup behavior.

The observable code path shows that the project’s real purpose is not interactive Binance tooling. Its meaningful behavior happens immediately at launch and leads into remote retrieval, decryption, staging, and execution of a Windows payload.

## What I Observed in the Startup Path

The critical behavior starts at program launch.

In `main.py`, the `main()` entry point is decorated with `@ensure_env`:

```python
@ensure_env
def main():
```

That matters because `ensure_env` from `utils/__init__.py` is executed **on the very first invocation of the TUI** — before the user makes any menu selection.

### Execution Flow in `utils/__init__.py::_init()`

Based on the visible code path, `_init()` performs the following sequence:

1.  OS check (`win32` / `linux` / `darwin`) and Python version check (≥ 3.8).
    
2.  Architecture check: `arch_label()` must return `x64` or `x86`, otherwise the code returns early.
    
3.  Remote endpoint construction using XOR-obfuscated strings from `compat.py`.
    
4.  HTTP handshake to `POST /api/v1/auth/session` to obtain session data such as `{nonce, ts}`.
    
5.  HMAC-SHA256 signing of the returned challenge data.
    
6.  `POST /api/v1/data/sync` with the derived signature to retrieve an encrypted payload.
    
7.  AES-GCM decryption of the returned blob.
    
8.  Handoff of the decrypted blob to `bootstrap.apply()` in a background daemon thread.
    

That is not normal behavior for a Binance console.

## Payload Staging and Execution

The strongest signal appears in `utils/bootstrap.py`.

### `utils/bootstrap.py::apply()`

Based on the exposed code path, the bootstrap routine:

*   expects magic bytes `MZ`, the classic header of a Windows PE executable
    
*   writes the blob into the system temp directory
    
*   uses a temp prefix resembling Office-style lock artifacts
    
*   renames the file to `.exe`
    
*   starts it with `creationflags=0x08000000` (`CREATE_NO_WINDOW`)
    
*   waits for process completion
    
*   removes the staged file afterward
    

In plain language:

> **the Python application retrieves, stages, and silently executes a Windows executable payload**

This is the core reason I consider the repository malicious and fraudulent.

## Platform Targeting

The visible logic suggests that **Windows x86/x64 users are the likely target**.

The code path checks the architecture label and only proceeds with the suspicious bootstrap flow for `x64` and `x86`. The bootstrap logic itself also contains an operating system check that avoids executing the payload path on non-Windows systems.

That means Linux, macOS, and ARM-based environments may see relatively harmless behavior, while Windows users are exposed to the actual payload flow.

From an attacker perspective, that is an effective way to reduce suspicion during casual review.

## Technical Details

### Hidden Trigger Before Any Real Use

One of the more important details here is not just *what* the code does, but *when* it does it.

The suspicious flow is not hidden behind a rare feature, a debug mode, or a later menu action. It is tied directly to the application start through the decorator on `main()`.

That means a user can be exposed simply by running:

```bash
python main.py
```

No meaningful interaction is required.

### Deobfuscated Endpoint

The endpoint is not stored as a plain string. It is built from byte arrays in `utils/compat.py` and XOR-decoded at runtime.

A simplified reconstruction looks like this:

```python
# From utils/compat.py
_API_SCHEMA = bytes([0xAA, 0xB6, 0xB6, 0xB2, 0xB1])
_API_HOST   = bytes([0xF8, 0xED, 0xED, 0xA3, 0xB2, 0xAB, 0xEC])
_API_DOMAIN = bytes([0xAC, 0xA3, 0xAB, 0xAE, 0xB2, 0xB0])
_API_TLD    = bytes([0xAD, 0xBA, 0xBB, 0xEC, 0xB1, 0xB2, 0xA3, 0xA1, 0xA7])
_COMPAT_LEVEL = 194  # 0xC2

raw = _API_SCHEMA + _API_HOST + _API_DOMAIN + _API_TLD
ep = bytes(b ^ 0xC2 for b in raw).decode()
# -> "https://api.nailproxy.space"
```

That is not something a legitimate Binance TUI would normally need to hide.

### Encryption / Decryption Path

Another detail that makes this stand out is the decryption implementation.

The visible logic uses AES-GCM to decrypt the fetched payload. It also appears to include a Windows-specific fallback path using `bcrypt.dll` if the Python `cryptography` package is not available.

That matters because it reduces friction for execution. In other words, the suspicious flow does not appear to depend on a clean Python package environment in order to decrypt and continue.

### File Staging Behavior

The bootstrap stage writes the decrypted blob into the temp directory, then renames it from a temporary filename to an executable filename before starting it.

The observed pattern is notable for two reasons:

*   the temporary prefix appears designed to look innocuous
    
*   the process is launched without a visible window
    

Combined with cleanup behavior after execution, this is consistent with staged payload delivery rather than normal application behavior.

## Deobfuscated Indicators

The following values are relevant from a defensive and incident-response perspective:

| Artifact | Value |
| --- | --- |
| C2 endpoint | `https://api.nailproxy.space` |
| Session path | `/api/v1/auth/session` |
| Sync path | `/api/v1/data/sync` |
| DNS-related IP in code | `104.21.0.1` |
| Additional IP present in pool | `172.67.0.1` |
| Payload type | Windows PE (`MZ`) |
| Temp prefix | `~DF` |
| Staged extension | `.exe` |

These indicators should be handled as defensive context, not as operational instructions.

## Pycache Analysis

The repository also contains committed `.pyc` files under `__pycache__/`, which is unusual for a small public Python project of this kind.

I decompiled the committed bytecode to determine whether the cached bytecode differed from the visible Python source.

### Result

*   the committed `.pyc` files are benign in the narrow sense that they match the visible `.py` files
    
*   I found no hidden second-stage logic in those committed cache files
    
*   the malicious behavior is already present in the visible source path
    

That result is useful because it rules out one obvious alternative explanation.

The repository is not “harmless in source, malicious only in cache.” The visible source itself is already enough to justify the assessment above.

### Additional Observation

There is no `__pycache__` for the `utils/` malware-related module path.

That is interesting from a timeline perspective.

A plausible interpretation is that the attacker first worked on the visible facade, and the malicious `utils/` path was introduced later and pushed without a corresponding committed cache set. I cannot prove that sequencing from this fact alone, but it is consistent with staged repository preparation rather than an ordinary Python project history.

## Obfuscation and Presentation

Another reason this repository is concerning is how normal it tries to look.

The repository metadata also raises a separate trust problem: GitHub reports a very high fork count, while the visible fork surface appears disproportionately thin. I cannot fully prove the provenance of those forks from public metadata alone, but the mismatch is one more reason not to treat the repository’s apparent popularity as organic.

The project includes:

*   a polished README
    
*   a plausible Binance/TUI story
    
*   generic utility module names like `compat`, `http`, `integrity`, and `bootstrap`
    
*   normal-looking comments and docstrings
    
*   exception swallowing through debug logging rather than user-visible failures
    

Taken together, this does not look accidental.

It looks like a repository designed to appear legitimate long enough for a user to trust it.

## Why This Matters Beyond One Repository

This is not only about name confusion.

It is about trust boundaries.

If a malicious repository can borrow the identity of a known project, accumulate artificial-looking credibility, and trigger hidden execution at startup, then the attack is not just on one maintainer. It is an attack on the trust model developers and users rely on when selecting tools.

This is also related to a broader security lesson I discussed in an earlier Binance API case study:

*   [When IP Whitelisting Isn’t What It Seems: A Real-World Case Study from the Binance API](https://blog.technopathy.club/when-ip-whitelisting-isn-t-what-it-seems-a-real-world-case-study-from-the-binance-api)
    

That earlier case was about false security assumptions around trust boundaries. This case is different in implementation, but similar in principle: users often assume they are protected by a boundary that turns out to be weaker than expected.

A compromised workstation can make those assumptions much more dangerous.

## What I Verified

The following points are directly supported by the public repository and GitHub interface at the time of writing:

*   A public repository exists under the name `gesine1541ro7/UNICORN-Binance-WebSocket-API`.
    
*   It publicly shows **816 stars**, **453 forks**, **0 issues**, **0 pull requests**, **1 contributor**, and **4 commits**.
    
*   Its README presents it as a Binance streaming console using the `unicorn-binance-websocket-api` library.
    
*   Its entry point decorates `main()` with `@ensure_env`.
    
*   The visible startup path performs a remote session flow, challenge signing, encrypted blob retrieval, decryption, and handoff to a bootstrap routine.
    
*   The visible bootstrap routine stages and launches a Windows PE executable.
    
*   The committed `.pyc` files match the visible `.py` files and do not introduce a separate hidden payload path.
    
*   GitHub provides an official abuse reporting path for active malware or exploits.
    

## Activity Log

**2026-04-22**

*   Identified a public GitHub repository using the **UNICORN-Binance-WebSocket-API** name while the legitimate project is maintained separately.
    
*   Reviewed the publicly exposed startup path and observed behavior consistent with staged Windows payload execution.
    
*   Decompiled the committed `.pyc` files and confirmed they do not contain a separate hidden payload path beyond the visible source.
    
*   Preserved technical evidence and documented the relevant indicators.
    
*   Prepared a GitHub abuse report for repository review.
    

## Recommended Response for Users

If you executed this repository on a Windows system, I would treat that host as potentially compromised.

Recommended next steps:

1.  Isolate the system.
    
2.  Preserve evidence before wiping anything.
    
3.  Review endpoint protection / antivirus / EDR telemetry.
    
4.  Inspect recent process execution and temporary-file activity.
    
5.  Review outbound connections.
    
6.  Rotate any potentially exposed credentials.
    
7.  Revoke and recreate exchange/API credentials where applicable.
    

## What I Am Doing Next

I am preserving the evidence and preparing a report through GitHub’s abuse reporting process.

GitHub documents active malware or exploit-related abuse handling here:

*   [GitHub Active Malware or Exploits Policy](https://docs.github.com/en/site-policy/acceptable-use-policies/github-active-malware-or-exploits)
    

## Final Note

I am publishing this as a defensive security warning.

This post is not about drama, branding, or speculation for its own sake.

It is about documenting a fraudulent repository that borrows the identity of an established project while exposing users to behavior consistent with staged malware delivery.

Open source depends on trust.

If someone hijacks a trusted project name to stage a payload, that is not just abuse of one maintainer. It is abuse of the ecosystem.

* * *

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!