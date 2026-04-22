---
title: "nailproxy.space: A Multi-Repository GitHub Malware Campaign"
datePublished: 2026-04-22T11:00:24.693Z
cuid: cmo9xymls006h2akbbex69jgw
slug: nailproxy-space-github-malware-campaign
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/61a3b717-1e78-4976-914b-f0f43cd2820a.png
tags: github, python, security, malware, threat-intelligence

---

> **Update (2026-04-22, 13:33):** I submitted this case to GitHub Support for campaign-level review. Ticket ID: **4313391**.

> **TL;DR:** What initially looked like a single fraudulent repository impersonating **UNICORN Binance WebSocket API** now appears to be part of a broader **multi-repository GitHub malware campaign**.  
> Across the currently confirmed set, the repositories share the same C2 infrastructure, the same staged Windows payload flow, the same deceptive repository framing, and the same manipulated-looking social proof patterns.  
> At the time of writing, I have **19 confirmed repositories** in scope. The list is likely incomplete.

This post is a follow-up to my earlier write-up on the fraudulent repository impersonating UNICORN Binance WebSocket API:

*   [Security Warning: Fraudulent GitHub Repository Impersonating UNICORN Binance WebSocket API](https://blog.technopathy.club/security-warning-fraudulent-github-repository-impersonating-unicorn-binance-websocket-api)
    

That first article focused on one repository. Further analysis shows that the UBWA-themed repository is very likely **one lure inside a broader campaign**.

## How This Started

The starting point was a repository using the **UNICORN-Binance-WebSocket-API** name while the legitimate project is maintained separately through the official project channels:

*   [Official GitHub repository: oliver-zehentleitner/unicorn-binance-websocket-api](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api)
    
*   [Official PyPI package: unicorn-binance-websocket-api](https://pypi.org/project/unicorn-binance-websocket-api/)
    

What initially made the fraudulent repository stand out was the mismatch between its visible development footprint and its public social proof. That led to a closer static review of the startup path, which in turn exposed staged Windows payload behavior.

From there, the obvious next question was:

> Is this an isolated repository, or part of something larger?

The answer now appears to be: **something larger**.

## Campaign Summary

Across the currently confirmed set, the repositories share several core properties:

*   the same decoded C2 host
    
*   the same session and sync paths
    
*   the same staged Windows payload execution flow
    
*   the same deceptive facade pattern
    
*   the same or highly similar `utils/` module structure
    
*   the same repeated commit choreography
    
*   the same manipulated-looking star and fork behavior
    

At the time of writing, I have **19 confirmed repositories** associated with this pattern.

## Shared Infrastructure

The strongest campaign signal is the infrastructure overlap.

Across all confirmed samples I reviewed, the per-repository XOR-decoding logic in `utils/compat.py` resolves to the same endpoint:

```text
https://api.nailproxy.space
```

The same repositories also use the same two application paths:

*   `POST /api/v1/auth/session`
    
*   `POST /api/v1/data/sync`
    

The visible code also contains the same DNS-bypass strategy and the same Windows-oriented fallback behavior.

## Deobfuscated Indicators

The following values are relevant from a defensive and incident-response perspective:

*   **C2 endpoint:** `https://api.nailproxy.space`
    
*   **Session path:** `/api/v1/auth/session`
    
*   **Sync path:** `/api/v1/data/sync`
    
*   **Hard-coded fallback IP:** `104.21.0.1`
    
*   **Additional IP present in pool:** `172.67.0.1`
    
*   **Transport fallback:** `curl.exe --resolve`
    
*   **Payload type:** Windows PE (`MZ`)
    
*   **Temp prefix:** `~DF`
    
*   **Staged extension:** `.exe`
    
*   **Launch flag:** `0x08000000` (`CREATE_NO_WINDOW`)
    

These indicators are included as defensive context only.

## Shared Execution Flow

The execution flow is consistent across the analyzed repositories.

In the UBWA-themed repository, the application entry point is decorated with `@ensure_env` in `main.py`, which means the environment bootstrap path executes immediately when the application starts.

A simplified version of the visible flow looks like this:

1.  Check operating system support and Python version.
    
2.  Check whether the architecture is `x64` or `x86`.
    
3.  Decode the remote endpoint from XOR-obfuscated byte arrays.
    
4.  Request a session object from `/api/v1/auth/session`.
    
5.  Sign the returned challenge using HMAC-SHA256.
    
6.  Request an encrypted blob from `/api/v1/data/sync`.
    
7.  Decrypt the blob with AES-GCM.
    
8.  Hand the result to `bootstrap.apply()` in a daemon thread.
    
9.  Stage the result as a Windows executable in temp.
    
10.  Launch it without a visible window.
     
11.  Delete the staged file afterward.
     

The same overall structure appears in the other confirmed samples, even where lure topic, README framing, or facade modules differ.

## Shared Dropper Components

One of the clearest signs that this is a campaign rather than unrelated copycats is the shared `utils/` structure.

The key modules are:

*   `utils/__init__.py`
    
*   `utils/bootstrap.py`
    
*   `utils/compat.py`
    
*   `utils/http.py`
    
*   `utils/integrity.py`
    

In the UBWA and TurnKey samples, four of these files are byte-identical:

*   `__init__.py`
    
*   `bootstrap.py`
    
*   `http.py`
    
*   `integrity.py`
    

Only `compat.py` differs, and that difference is narrow: the XOR key and encoded byte arrays vary per repository, but the decoded result still points to the same C2.

That strongly suggests a reusable generation pattern rather than hand-written independent tooling.

## Commit and Timing Pattern

The commit history also shows repetition.

Across multiple repositories, the commit pattern follows the same sequence:

1.  `Initial commit`
    
2.  `Script release`
    
3.  `Update README.md`
    
4.  `Add files via upload`
    

The important part is the last step.

That final upload is where the malicious `utils/` path appears.

In the two repositories under `gesine1541ro7`, both repos were created within minutes of each other, and both received the malicious upload on the same day only minutes apart.

That is not what normal open-source iteration looks like.

It looks like **batch preparation followed by staged weaponization**.

## Social-Proof Manipulation

The social-proof pattern is also highly unusual.

In the UBWA-themed repository:

*   **816 total stars**
    
*   **607 stars on 2026-03-17**
    
*   **204 stars on 2026-03-18**
    

That means **811 of 816 stars** arrived on two consecutive days, well after repository creation.

In the TurnKey sample:

*   **83 total stars**
    
*   **82 stars on a single day**
    

The fork pattern is equally suspicious.

The visible fork-owner names follow a repeated structure that looks machine-generated, and there is confirmed overlap between fork accounts across multiple campaign repositories.

That does not prove every single account is controlled by the same operator, but it is more than enough to treat the apparent popularity of these repositories as untrustworthy.

## Lure Diversity

Another notable feature of the campaign is the range of lure topics.

The currently confirmed repositories include themes such as:

*   Binance and crypto trading tools
    
*   Web3 airdrop or farming bots
    
*   GPT wrappers and jailbreak tools
    
*   OSINT tooling
    
*   fake CVE/security research tools
    
*   AI utility tools
    
*   cryptography-related libraries
    

That is important because it shows the campaign is not narrowly focused on one brand or one audience.

The pattern is broader:

> use plausible developer-adjacent or trader-adjacent bait, attach a polished facade, then reuse the same malware delivery architecture underneath.

## Confirmed Repository Set

At the time of writing, I have **19 confirmed repositories** in scope:

1.  `gesine1541ro7/UNICORN-Binance-WebSocket-API`
    
2.  `gesine1541ro7/TurnKey-Auto-Bot`
    
3.  `lucija8320nhung4/HacxGPT`
    
4.  `MarCmcbri1982/KawaiiGPT`
    
5.  `Kaleighc793/freqtrade-bot`
    
6.  `Janis174756/Binance-Futures-Trading-Bot`
    
7.  `lauraevz6y70/gnark-crypto`
    
8.  `Jamie3t1991/BeraChainTools`
    
9.  `FrankDavis236869/spyder-osint`
    
10.  `CrystALqsxvk39/Python-Bitcoin-Utils`
     
11.  `Courtneybake80/Polyseed-Monero`
     
12.  `courtneyb8345/pharos-automation-bot`
     
13.  `Della38840/Robinhood`
     
14.  `charlo1492charlo14928/openfi-bot`
     
15.  `Giuditta8/Uomi-Testnet`
     
16.  `Jessica74016/CVE-2025-8088`
     
17.  `fernandez81188studio/SORA2-Watermark-Remover`
     
18.  `Annehuqr0Craft96/Mitosis-Farm`
     
19.  `charlo1492charlo14928/NFT-Yield-Farming`
     

This list should be treated as a **confirmed minimum**, not a complete boundary.

## What Is Confirmed vs. What Is Likely

It is important to separate what is directly supported from what is still inference.

### Confirmed

*   multiple repositories decode to the same C2 host
    
*   the same session and sync paths recur
    
*   the same staged Windows payload flow recurs
    
*   the same `utils/` architecture recurs
    
*   the same repeated commit pattern recurs
    
*   the same manipulated-looking star and fork patterns recur
    
*   fork-account overlap exists across campaign repositories
    

### Likely, but still incomplete

*   the campaign is larger than the currently confirmed set
    
*   some account clusters are rotating identities used by the same operator or operator group
    
*   the visible star/fork activity is synthetic or purchased
    
*   the lure set is still expanding or can be regenerated on demand
    

That distinction matters. It keeps the write-up grounded.

## Pycache and Build Fingerprint

I also checked the committed `.pyc` files in the UBWA-themed repository.

They do **not** contain a hidden second-stage path that differs from the visible facade source. In other words, the malicious behavior is already present in the visible source path.

What is more interesting is the asymmetry:

*   committed `__pycache__` exists for facade modules
    
*   no equivalent committed cache set was present for the malware-bearing `utils/` path in the UBWA sample
    

That is not proof of workflow by itself, but it is consistent with a timeline in which the facade was developed or tested locally first, and the malware-bearing `utils/` path was introduced later.

## Why This Matters

This matters for three reasons.

### 1\. It is an ecosystem trust problem

This is not just about one repo name.

It is about abusing trust signals developers rely on:

*   recognizable topics
    
*   polished READMEs
    
*   stars and forks
    
*   plausible project structures
    
*   familiar library ecosystems
    

### 2\. It targets real user behavior

The repositories are designed around realistic behavior:

*   cloning code from GitHub
    
*   running `python main.py`
    
*   launching via `run.bat`
    
*   trusting a tool because it looks connected to a known project or topic
    

### 3\. The lure themes are operationally smart

The campaign does not depend on one niche. It spans trading, AI, crypto, automation, and developer tooling.

That is exactly how a scalable lure operation behaves.

## Recommended Response

If you ran one of these repositories on Windows, I would treat the host as potentially compromised.

Recommended next steps:

1.  isolate the system
    
2.  preserve evidence
    
3.  review endpoint protection / EDR / antivirus telemetry
    
4.  inspect recent process execution and temp-directory activity
    
5.  review outbound connections
    
6.  rotate potentially exposed credentials
    
7.  revoke and recreate exchange/API credentials where relevant
    

I would also strongly recommend reporting the relevant repositories and accounts to GitHub.

## Request for Independent Validation

The current confirmed set is useful, but likely incomplete.

If others want to independently validate or extend this set using only public repository metadata and static source review, that additional confirmation would be valuable.

I am **not** asking anyone to interact with the infrastructure, execute the code, or probe the C2.

Useful contributions would be things like:

*   confirming additional repositories that decode to the same C2
    
*   validating reuse of the same `utils/` architecture
    
*   checking for broader account overlap
    
*   identifying additional star/fork anomalies
    
*   correlating commit timing and upload choreography
    

## Final Note

The first fraudulent UBWA-themed repository was enough to justify a warning.

The broader pattern now justifies something stronger:

> this does not look like an isolated deceptive repository. It looks like a repeatable GitHub malware campaign using multiple lures, shared infrastructure, and manufactured trust signals.

Open source depends on trust.

Campaigns like this are dangerous not only because they deliver payloads, but because they systematically erode the assumptions people make about what looks legitimate on public platforms.