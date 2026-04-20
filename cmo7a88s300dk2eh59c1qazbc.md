---
title: "When IP Whitelisting Isn't What It Seems: A Real-World Case Study from the Binance API"
datePublished: 2026-04-20T14:20:30.201Z
cuid: cmo7a88s300dk2eh59c1qazbc
slug: when-ip-whitelisting-isn-t-what-it-seems-a-real-world-case-study-from-the-binance-api
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e43e64f5-e5a5-4d97-8b19-46e6fcbe0057.webp
tags: python, binance, api-security

---

A case study on how Binance's listenKey design bypasses IP whitelisting, why Bugcrowd dismissed it, and what this teaches us about API security in 2025.

* * *

In 2024, I discovered an unexpected API behavior in **Binance**, the world's largest crypto exchange.

I reported the issue twice through their official bug bounty program on **Bugcrowd** (*Submission ID 3897aec7–373d-46b2-b544–29bba9b04a0b from 09 Dec 2024 and b33df044–8db1–446f-9613–3d498067a995 from 18 Dec 2024*) — both times the report was closed as:

*   **"Not security relevant"**
    
*   **"Not applicable"**
    
*   Essentially framed as **"social engineering"**
    

This article contains **zero exploit code**, **zero harmful details**, **zero endpoints**, and is purely an **architectural case study** — safe, abstract and intended for DevSecOps engineers, API designers, and security researchers. Anyone with a Binance account who knows a little bit of programming or can ask an AI can verify what I am claiming for themselves!

%[https://youtu.be/y9dGtHLEBp8?si=nuKtzWajhIApdfEg] 

Watch this 5-min live demo first — then read the full technical breakdown below.

#### Why publish it?

Because local inconsistencies in trust boundaries matter. And because after ~11 months, the underlying behavior still exists. And because Bugcrowd, from my perspective, did not fulfill its role as a neutral and fair mediator in this case — which is the very purpose bug bounty platforms are supposed to serve for researchers.

*   [What Users Expect From IP Whitelisting](#what-users-expect-from-ip-whitelisting)
    
*   [The Real Model: API Keys vs. listenKeys](#the-real-model-api-keys-vs-listenkeys)
    
*   [Critical Detail: You Can Obtain the listenKey WITHOUT the Secret](#critical-detail-you-can-obtain-the-listenkey-without-the-secret)
    
*   [The Core Security Mismatch: Whitelisting Protects the API Key, Not the listenKey](#the-core-security-mismatch-whitelisting-protects-the-api-key-not-the-listenkey)
    
*   [A supply chain attack needs zero user interaction](#a-supply-chain-attack-needs-zero-user-interaction)
    
*   [Why This Isn't "Just Social Engineering"](#why-this-isnt-just-social-engineering)
    
*   [Real-World Impact (Without Sensationalism)](#real-world-impact-without-sensationalism)
    
*   [Bugcrowd's Handling — A Systemic Problem](#bugcrowds-handling--a-systemic-problem)
    
*   [What a Fair Process Would Have Looked Like](#what-a-fair-process-would-have-looked-like)
    
*   [Lessons for DevSecOps & API Designers](#lessons-for-devsecops--api-designers)
    
*   [Closing Thoughts](#closing-thoughts)
    

* * *

### What Users Expect From IP Whitelisting

Binance allows API keys to be restricted to specific IP addresses.

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/ff80d653-a760-446d-bd76-2eff2c372a50.png align="center")

This gives users a very strong expectation:

> ***"Even if my API key leaks, nobody can use it unless they are on my whitelisted IPs."***

That belief is the *entire purpose* of whitelisting.

It makes developers comfortable embedding an API key inside:

*   Trading bots
    
*   Cloud workloads
    
*   Docker containers
    
*   CI/CD tools
    
*   Third-party libraries
    
*   Older servers
    
*   Desktop applications
    

Because the assumption is:

> ***"My key is useless anywhere else."***

This assumption is reasonable. But — it is wrong.

* * *

### The Real Model: API Keys vs. listenKeys

Binance's API architecture includes:

#### Primary credentials

*   `apiKey`
    
*   `secretKey` (for HMAC signatures)
    

#### Secondary credential

*   **listenKey** (used for user data streams)
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/c1dd644c-f551-4aba-833f-184be775295e.png align="center")

This is not a payload vulnerability — it is an architectural trust-boundary flaw.

User data streams show **high-value real-time trading telemetry**, including:

*   Open orders
    
*   Order executions
    
*   Stop losses
    
*   Trailing stops
    
*   Liquidation-relevant positions
    
*   Balance changes
    
*   Strategy characteristics
    
*   Timing of order placement
    

This is *not harmless information*. It exposes the inner workings of a trading strategy.

* * *

### Critical Detail: You Can Obtain the listenKey WITHOUT the Secret

This is the first architectural red flag.

Binance only requires the API key to issue a listenKey — the secret is not needed. This diverges from the usual "proof of possession" pattern in API design.

* * *

### The Core Security Mismatch: Whitelisting Protects the API Key, Not the listenKey

This is the point that matters most.

#### IP whitelisting fully protects the primary `apiKey`.

But the **listenKey is NOT IP-restricted.** Not partially. Not conditionally. Not at all.

#### Therefore:

1.  **The API key is IP-bound.**
    
2.  **The listenKey is NOT IP-bound.**
    
3.  The listenKey can be obtained using only the API key.
    
4.  Developers believe they are protected by whitelisting — but they aren't.
    

This creates a **false sense of security**, and a **broken mental model**:

> *"My key is locked to my infrastructure."*

> ***…but a secondary stream that exposes my entire trading activity is not.***

This is not a hack. This is not an exploit. This is a **trust boundary inconsistency**.

* * *

### Why This Isn't "Just Social Engineering"

Binance and Bugcrowd classified this as:

*   "Social engineering"
    
*   "Not security relevant"
    

This framing is technically inaccurate.

Here's why:

#### A supply chain attack needs zero user interaction

Any compromised:

*   Python library
    
*   NPM module
    
*   Docker image
    
*   Browser extension
    
*   Trading bot wrapper
    
*   Cloud agent
    

…can silently:

1.  Request a listenKey (no secret required)
    
2.  Extract it from memory
    
3.  Exfiltrate it
    
4.  Access your user streams from *any* IP
    

No phishing. No manipulation. No fake login. No mistake by the user.

This is entirely **machine-side**, not **human-side**.

Social engineering requires human manipulation; this issue requires none.

Calling this "social engineering" is simply wrong.

#### A Sign of This False Sense of Security

This broken security assumption is also visible in the real world. Over the years, multiple users have publicly posted their listenKeys on GitHub — something nobody would ever do with an API key. Examples:

[https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/issues/98#issuecomment-682278783](https://github.com/oliver-zehentleitner/unicorn-binance-websocket-api/issues/98#issuecomment-682278783)

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/60056ccd-c361-406f-8af4-a8d8643816a3.png align="center")

[https://github.com/binance/binance-connector-python/issues/99](https://github.com/binance/binance-connector-python/issues/99)

![]( align="center")

When users misunderstand what is protected, they behave accordingly — and that's how real-world incidents happen.

People do this because they believe the listenKey is protected by the same IP whitelisting rules as the API key. It isn't — and that misconception is exactly why this architectural flaw matters.

* * *

### Real-World Impact (Without Sensationalism)

listenKeys do **not** grant trading or withdrawal privileges.

But they grant something extremely valuable:

#### Market-moving intelligence

*   Open orders
    
*   Stop-losses
    
*   Take-profit triggers
    
*   Liquidation thresholds
    
*   Wallet exposure
    
*   Detailed order execution flow
    
*   Balance movements
    
*   Position sizing
    
*   Leverage usage
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/9c82e683-aeca-4ebe-bba7-14809e827e8c.png align="center")

With enough listenKeys from many accounts, an attacker could:

1.  Front-run traders
    
2.  Hunt stops
    
3.  Trigger liquidations
    
4.  Detect whale movement patterns
    
5.  Understand strategy timing
    
6.  Analyze market sentiment by user position flow
    

This is the type of data for which hedge funds pay millions.

Yet with whitelisting enabled, users naturally believe this information is protected.

It isn't.

* * *

### Bugcrowd's Handling — A Systemic Problem

I submitted the issue twice, both times with professional detail on:

*   Architecture
    
*   Threat model
    
*   Python prototype
    
*   Timelines
    
*   Diagrams
    
*   Real user stream examples (safe & anonymized)
    
*   Demo video
    

The responses were:

*   "Not applicable"
    
*   "Not security relevant"
    
*   "Social engineering"
    

**No questions.No technical discussion.No escalation to senior security staff.No clarification.No interest.No attempt to understand the architecture.**

This is not a Binance-only problem. This is a **bug bounty ecosystem problem**:

*   Business logic vulnerabilities fall through the cracks.
    
*   Architectural flaws don't fit classic CVSS scoring.
    
*   Whitelisting is treated as a UX feature, not a security boundary.
    
*   Vendor customers get overly defensive.
    
*   Bugcrowd triage tends to favor the vendor.
    

This case is a textbook example.

But the triage process didn't reflect the actual technical depth of the issue.

#### A Note on Bugcrowd's Process Handling

When I submitted additional clarification through Bugcrowd's "Request for Response" mechanism, the reply I received did not address the technical points raised. Instead, it reiterated the original classification without engaging with the architectural arguments behind the issue. The response even included a reminder about potential penalties for requesting further clarification, despite the fact that no new technical review had taken place.

This experience reinforced a broader problem I've observed in the bug bounty ecosystem: **Platform triage processes are often optimized for clear, conventional vulnerabilities, but they struggle with architectural or trust-boundary issues that don't fit neatly into predefined categories.**

In this specific case, Bugcrowd did not provide the neutral and technically grounded mediation that researchers rely on — especially when a report involves design-level inconsistencies rather than classic "payload exploits." This is not about blame; it simply shows that current triage workflows are not well-equipped for complex, multi-layered security findings.

* * *

### What a Fair Process Would Have Looked Like

For findings of this kind, a fair bug bounty process usually includes:

*   A technical review
    
*   Follow-up questions
    
*   A classification discussion
    
*   A clear explanation of the vendor's reasoning
    
*   And at least a form of researcher recognition
    

These are standard expectations across reputable bounty programs — especially for architectural issues that require multi-day analysis, cross-checking, prototyping, and documentation.

My work included:

*   Several days of API investigation
    
*   Multiple reproductions
    
*   Python demonstrations
    
*   Clean and safe reporting
    
*   Detailed architectural reasoning
    
*   Real-world demonstration of the trust boundary mismatch
    

Across most platforms, this type of research is acknowledged regardless of payout decisions.

#### In my case, however, none of this happened:

*   No technical discussion
    
*   No meaningful review
    
*   No recognition
    
*   No acknowledgement of the work invested
    

#### This is why the case matters:

Not because of financial expectations — but because the process failed to deliver what bug bounty platforms are fundamentally meant to provide: **fairness, neutrality, and respect for substantial research effort.**

* * *

### Lessons for DevSecOps & API Designers

#### 1\. Secondary tokens must inherit all constraints of the primary.

No exceptions.

#### 2\. Proof of possession matters.

Never issue a token without confirming ownership of its parent credential.

#### 3\. IP whitelisting must be consistent.

If one flow bypasses it, the guarantee breaks.

#### 4\. Users' mental models matter as much as code.

When expectations don't match reality, security collapses.

#### 5\. Bug bounties must evolve beyond "classic bugs."

Architecture is security. Design is security. Assumptions are security.

* * *

### Closing Thoughts

The Binance listenKey issue isn't a catastrophic vulnerability. It won't drain wallets, freeze accounts, or cause instant chaos.

But it *is* a powerful example of:

*   Inconsistent trust boundaries
    
*   Misleading security assumptions
    
*   Risk of supply-chain leakage
    
*   Architectural flaws not fitting bounty templates
    

And it shows how easily deep research can be miscategorized and dismissed.

My goal in publishing this is simple:

> ***To spark a conversation on how we evaluate architectural security issues — especially in systems trusted with billions of dollars.***

If you work in DevSecOps, crypto infrastructure, or API security, I would genuinely value your perspective.

* * *

I hope you found this tutorial informative and enjoyable!

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!

Thank you for reading, and happy coding!