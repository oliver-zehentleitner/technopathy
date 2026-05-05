---
title: "Binance Fixed the IP Whitelist Gap. The Disclosure Process Is Still Broken."
datePublished: 2026-05-05T16:55:27.440Z
cuid: cmosvdakr00fg2flr1g634dhv
slug: binance-fixed-the-ip-whitelist-gap-the-disclosure-process-is-still-broken
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/231ac2dc-3310-405a-8360-0782eed7d850.png
tags: github, opensource, malware, binance, pypi, bugbounty, api-security, bugcrowd, supply-chain, disclosure

---

I wanted to re-open an old Binance API security issue.

Not because I enjoy re-litigating old reports.

Because the last thirteen days made the threat model painfully concrete.

I found or stumbled into fake GitHub repositories, a broader `nailproxy.space` malware campaign, a StealC-linked delivery chain, a fake job interview abusing VSCode workspace trust, and a PyPI typosquat targeting my Binance WebSocket library.

Different cases. Same direction.

Attackers are trying to get code executed exactly where API keys live: developer machines, bot servers, CI jobs, dependency trees, IDE workspaces, and trading infrastructure.

That matters because I reported a Binance API trust-boundary issue around IP whitelisting and `listenKey` in December 2024.

So I went back and re-tested it.

The result surprised me.

The vulnerability appears to be fixed.

That is good.

Really good.

But the way we got here is not good.

Because the same issue was previously rejected as “Social Engineering,” marked “Not Applicable,” not rewarded, not acknowledged — and now the behavior I reported is gone.

The finding was not rewardable enough to acknowledge.

But apparently it was technical enough to fix.

That is the part I cannot ignore.

## The original issue

Binance API keys can be restricted to specific IP addresses.

That creates a strong security expectation:

> Even if credentials leak, they should be useless outside the trusted IP range.

That is the point of IP whitelisting.

But the old `listenKey` model for private user data streams did not follow that boundary consistently.

A `listenKey` could be created from a whitelisted system using only the API key — no API secret, no request signature, no proof of possession.

The resulting `listenKey` could then be used from outside the configured IP whitelist to consume private user data streams.

No trading permission.

No withdrawal permission.

No direct account takeover.

But full real-time visibility into sensitive account activity:

*   balances
    
*   orders
    
*   executions
    
*   positions
    
*   liquidation-relevant state
    
*   timing
    
*   strategy behavior
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/e86206e5-0b36-4479-9736-b1e15701f3df.png align="center")

For automated trading systems, that is not harmless data.

In trading, visibility is value.

Open orders are value. Positions are value. Execution timing is value. Strategy behavior is value.

A system that exposes that data outside the configured IP boundary leaks more than most users realize.

The rule should be simple:

> A derived credential must not be more portable than the credential that created it.

That was the issue.

## Why “Social Engineering” was the wrong answer

The report was not about someone tricking a user into handing over a token.

It was about an architectural boundary mismatch.

Bugcrowd / Binance treated the attack scenario as requiring Social Engineering.

I disagreed then.

I disagree now.

Because the relevant attack path was never:

> Please send me your `listenKey`.

The relevant attack path was:

> Trusted code running in a whitelisted environment can create or exfiltrate a `listenKey`, and the attacker can consume it outside that environment.

That is how supply-chain compromise works.

A malicious dependency does not ask nicely.

A compromised bot does not ask the user to forward a token.

An infected package does not need a phishing form.

A malicious IDE workspace does not need the user to understand what it runs.

It executes in the place the user already trusts.

That is exactly why IP whitelisting exists.

And that is exactly why derived stream credentials should inherit the same boundary.

## The old proof was real

This was not only an argument.

I had proof.

The original Bugcrowd reports included scripts and video material demonstrating the flow. Later, I also published the public case study:

[When IP Whitelisting Isn't What It Seems: A Real-World Case Study from the Binance API](https://blog.technopathy.club/when-ip-whitelisting-isn-t-what-it-seems-a-real-world-case-study-from-the-binance-api)

I also have an old video from November 26, 2025 demonstrating that the issue was still reproducible at that time:

%[https://www.youtube.com/watch?v=y9dGtHLEBp8] 

That video has about 30 views.

Which is funny in a bitter way.

Because the finding was not subtle.

The user expectation around IP whitelisting was clear. The architectural mismatch was clear. The supply-chain threat model was clear.

The recommended fix was also clear:

*   enforce the same IP restrictions on `listenKey` as on the original API key
    
*   require proof of possession of the API secret when issuing a stream credential
    
*   treat stream credentials as sensitive account credentials, not harmless session strings
    

That was the whole point.

## Why I re-tested it now

The last thirteen days changed the context.

I documented or found:

*   [fake GitHub repositories impersonating my Binance tooling](https://blog.technopathy.club/security-warning-fraudulent-github-repository-impersonating-unicorn-binance-websocket-api)
    
*   [a broader malware campaign using open-source project names as lures](https://blog.technopathy.club/nailproxy-space-github-malware-campaign)
    
*   [a StealC-linked delivery chain](https://blog.technopathy.club/from-a-coffee-in-bed-google-search-to-a-stealc-linked-campaign-the-story-behind-nailproxy-space)
    
*   [a fake job interview abusing developer-tool trust](https://blog.technopathy.club/i-had-a-fake-job-interview-it-was-a-malware-delivery-chain)
    
*   [a PyPI package squat around my Binance WebSocket library](https://blog.technopathy.club/the-pypi-package-was-clean-that-was-the-problem)
    

Those are not theoretical attack paths.

They target exactly the environments where API credentials live.

That was my point in 2024.

It is even more obvious in 2026.

Supply-chain attacks do not require the user to forward a token.

They require trusted code to run in a trusted place.

That is the whole problem.

So I re-tested the current state on May 5, 2026.

## The re-test setup

The test used one Binance API key with full permissions except withdrawals.

The key was restricted to one specific Telekom Austria IPv4 address.

I tested two source-IP states:

*   a whitelisted Telekom Austria home IPv4
    
*   non-whitelisted Mullvad WireGuard exits
    

The probes were pure REST `POST` calls to the relevant `listenKey` endpoints using only the `X-MBX-APIKEY` header.

No trades.

No signed actions for the `listenKey` probes.

No account interaction beyond testing whether the stream credential could be created.

The API key was handled via environment variable, redacted in logs, and rotated after the test.

## What I found

Short version:

> Binance fixed it.

More precisely:

| Product line | Current state on May 5, 2026 |
| --- | --- |
| Spot | old `listenKey` endpoint retired |
| Cross-Margin | old `listenKey` endpoint retired |
| Isolated-Margin | old `listenKey` endpoint retired |
| USDⓈ-M Futures | `listenKey` still exists, but IP whitelist is enforced |
| COIN-M Futures | `listenKey` still exists, but IP whitelist is enforced |
| Binance US | not tested |
| Binance TR | not tested |

Spot and Margin did not merely start enforcing the old model.

They moved away from it.

Futures still has `listenKey`, but the old bypass no longer reproduced.

From the whitelisted IP:

```text
POST /fapi/v1/listenKey  -> 200 OK
POST /dapi/v1/listenKey  -> 200 OK
```

From a non-whitelisted Mullvad IP:

```text
POST /fapi/v1/listenKey  -> 401 / -2015
POST /dapi/v1/listenKey  -> 401 / -2015
```

The error message included the live request IP.

That matters.

It strongly suggests Binance is now checking the request source IP against the API key whitelist for the Futures `listenKey` endpoints, just as it does for signed private endpoints.

I also cross-checked that this was not simply Binance blocking Mullvad.

Public no-auth Futures endpoints worked from the same Mullvad exit.

Signed private endpoints failed with the same `-2015` IP-whitelist error.

Changing the Mullvad exit changed the reflected request IP in the error message.

That is exactly what enforced IP whitelisting looks like.

## The lifecycle details are interesting

Spot now returns:

```text
410 Gone
```

for the old Spot user data stream endpoint.

That is a deliberate lifecycle signal.

It means: this endpoint used to exist, and it is gone.

Margin behaves differently.

Cross-Margin and Isolated-Margin returned generic `404 Not Found` responses through the `sapi` routing layer.

So Binance appears to have retired Spot more explicitly, while Margin was simply removed from routing.

That is not the central issue, but it is interesting.

The final state is still good for users:

*   Spot: retired
    
*   Margin: retired
    
*   Futures: live but now IP-enforced
    

From a security perspective, that is a real improvement.

## The bug is gone. The process is the story now.

I want to be very clear:

I am glad this is fixed.

Users are safer now.

That matters.

If a compromised API key can no longer be used from an arbitrary IP to obtain a Futures `listenKey`, that is good.

If Spot and Margin moved away from the old model entirely, that is good.

If Binance quietly closed a trust-boundary gap, that is good.

But it does not erase the disclosure problem.

Because the report was not accepted as a valid security issue.

It was closed as Not Applicable.

It was repeatedly framed as Social Engineering.

And now the behavior it described is gone.

That is the uncomfortable part.

The finding was not rewardable enough to acknowledge.

But apparently it was technical enough to fix.

## The process punished persistence

Responsible disclosure is not only about bugs.

It is about incentives.

If a researcher reports a real architectural issue, provides proof, explains supply-chain impact, provides real-world examples, and the issue later disappears from production, then the process should be able to say:

> Yes, this was useful.

It does not always need to mean a big bounty.

It does not always need to mean a CVE.

It does not always need to mean a public incident report.

But it should not result in:

*   rejection as Not Applicable
    
*   repeated misclassification
    
*   no acknowledgement
    
*   no reward
    
*   no clear public fix note
    
*   no correction of the original assessment
    
*   procedural pressure against further review requests
    

There is another part of this that should not be softened.

After I pushed back and asked for a proper review, the response did not become more technical.

It became more procedural.

I was warned that further response requests without “additional information” could affect my accuracy score or even my ability to create response requests.

That matters.

Because I was not submitting vague noise.

I had provided a reproducible issue, PoC material, real-world leakage examples, supply-chain scenarios, and a clear explanation of why this was not Social Engineering.

And now the behavior I reported appears to be gone.

That is not just disappointing.

That is a broken incentive structure.

A disclosure process should not make the researcher feel like the risky action is continuing to explain the bug.

## Why this matters

The people most likely to find architectural issues are often the people who live inside the ecosystem:

*   maintainers
    
*   infrastructure developers
    
*   SDK authors
    
*   bot developers
    
*   API integrators
    
*   security engineers
    
*   power users
    

Those people are not scanning random forms for easy XSS.

They understand the system because they build around it every day.

If they stop trusting the disclosure process, the platform loses an early-warning system.

And that is exactly the wrong outcome.

Especially for ecosystems where automated trading, API credentials, third-party libraries, and supply-chain risk are tightly connected.

## What I want from Binance

I do not want to pretend the fix does not matter.

It does.

I also do not want to pretend the process was acceptable.

It was not.

A fair outcome would have been simple:

*   acknowledge the trust-boundary question
    
*   confirm reproducibility
    
*   classify the finding honestly
    
*   explain product-line limitations if legacy compatibility made fixing hard
    
*   give a clear fix timeline or documentation note
    
*   recognize the report, even if no bounty was paid
    

That would already have been enough.

Instead, the public result is this:

A reported architectural issue was rejected as Social Engineering.

The same architectural behavior is now gone.

And the person who reported it had to discover that by re-testing it independently.

That is not how a healthy disclosure process should feel.

To be clear: I still like Binance.

I build and maintain open source infrastructure for the Binance ecosystem because I think the ecosystem matters.

Binance has built useful APIs.

The developer community around those APIs is real.

A lot of people depend on this infrastructure.

That is exactly why I care.

This is not about hating Binance.

It is about expecting better from a platform that sits at the center of a large automated-trading ecosystem.

## The uncomfortable conclusion

I started this follow-up expecting to show that the old issue was still open.

That would have been easy.

Instead, I found something more interesting.

The bug appears to be gone.

The security boundary is better now.

Users are safer.

Good.

But the disclosure process still failed.

The last thirteen days made the original threat model practical.

The May 5 re-test showed that Binance has apparently moved in the direction the report argued for.

Those two facts belong together.

The technical fix deserves credit.

The process deserves criticism.

Because responsible disclosure should not work like this:

> Report the issue.
> 
> Get told it is not an issue.
> 
> Watch it silently get fixed.
> 
> Receive no acknowledgement.

That is not sustainable.

And it is not fair.

The technical system got safer.

The disclosure process did not.

That should worry every platform that depends on external researchers.

* * *

I hope you found this informative and useful.

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\*(ツ)*/¯