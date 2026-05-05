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

In that time, I found or stumbled into:

1.  [a fraudulent GitHub repository impersonating UNICORN Binance WebSocket API](https://blog.technopathy.club/security-warning-fraudulent-github-repository-impersonating-unicorn-binance-websocket-api)
    
2.  [a broader `nailproxy.space` GitHub malware campaign](https://blog.technopathy.club/nailproxy-space-github-malware-campaign)
    
3.  [a StealC-linked delivery chain using fake open-source repositories](https://blog.technopathy.club/from-a-coffee-in-bed-google-search-to-a-stealc-linked-campaign-the-story-behind-nailproxy-space)
    
4.  [a fake job interview that tried to turn VSCode workspace trust into code execution](https://blog.technopathy.club/i-had-a-fake-job-interview-it-was-a-malware-delivery-chain)
    
5.  [a clean-but-dangerous PyPI typosquat targeting my Binance WebSocket library](https://blog.technopathy.club/the-pypi-package-was-clean-that-was-the-problem)
    

Different cases.

Same direction.

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

In December 2024, I reported a Binance API IP-whitelisting bypass through Bugcrowd.

The core issue was simple:

Binance API keys can be restricted to specific IP addresses.

That creates a strong security expectation:

> Even if credentials leak, they should be useless outside the trusted IP range.

That is the whole point of IP whitelisting.

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
    

![](https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/eeaa04b6-accd-4dde-8f60-560ed29c8c44.png align="center")

For automated trading systems, that is not harmless data.

In trading, visibility is value.

Open orders are value.

Positions are value.

Execution timing is value.

Strategy behavior is value.

A system that exposes that data outside the configured IP boundary leaks more than most users realize.

## Why I never accepted “Social Engineering” as the answer

The report was not about someone tricking a user into handing over a token.

It was about an architectural boundary mismatch.

If an API key is restricted to a set of IP addresses, then a credential derived from that API key should not become more portable than the parent credential.

That was the issue.

Bugcrowd / Binance treated the attack scenario as requiring Social Engineering.

I disagreed then.

I disagree now.

Because the relevant attack path was never:

> Please send me your `listenKey`.

The relevant attack path was:

> Trusted code running in a whitelisted environment can create or exfiltrate a `listenKey`, and the attacker can consume it outside that environment.

That is not a fantasy scenario.

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

The original Bugcrowd reports included scripts and video material demonstrating the flow. Later, I also published a public case study:

[When IP Whitelisting Isn't What It Seems: A Real-World Case Study from the Binance API](https://blog.technopathy.club/when-ip-whitelisting-isn-t-what-it-seems-a-real-world-case-study-from-the-binance-api)

I also have an old video from November 26, 2025 demonstrating that the issue was still reproducible at that time:

%[https://www.youtube.com/watch?v=y9dGtHLEBp8] 

That video has about 30 views.

Which is funny in a bitter way.

Because the finding was not subtle.

The user expectation around IP whitelisting was clear.

The architectural mismatch was clear.

The supply-chain threat model was clear.

The recommended fix was also clear:

*   enforce the same IP restrictions on `listenKey` as on the original API key
    
*   require proof of possession of the API secret when issuing a stream credential
    
*   treat stream credentials as sensitive account credentials, not harmless session strings
    

That was the whole point.

## Why I re-tested it now

The last thirteen days changed the context.

I found fake repositories impersonating my Binance tooling.

I found a broader GitHub malware campaign using open-source project identities as lures.

I walked into a fake job interview that tried to abuse VSCode workspace trust.

I found a PyPI package squat around my Binance WebSocket library.

This matters because all of those cases target the same place:

the developer environment.

And that is where Binance API keys live.

If hostile code reaches that environment, IP whitelisting becomes more important, not less.

The question becomes:

> If attacker-controlled code runs inside the trusted environment, can it create or steal a private stream credential that remains useful outside the whitelist?

That is the practical version of the original finding.

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

That is the important technical outcome.

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

It suggests the old `listenKey` model was not cleaned up uniformly across product lines.

The final state is still good for users:

*   Spot: retired
    
*   Margin: retired
    
*   Futures: live but now IP-enforced
    

From a security perspective, that is a real improvement.

## The bug is gone. The process is the story now.

I want to be very clear:

I am glad this is fixed.

Users are safer now.

That matters more than my ego.

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

## Why this matters for responsible disclosure

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
    
*   warnings that further response requests may affect the reporter’s accuracy score or ability to request review
    

That last point matters.

I was not spamming a bounty program with vague claims. I was trying to get a reproducible architectural issue reviewed correctly. The follow-up material explained the supply-chain scenario, the real-world leakage paths, and the trust-boundary mismatch.

Being warned about process penalties while the underlying behavior later disappears from production is exactly the kind of incentive failure this article is about.

That sends the wrong signal.

The wrong signal is:

> Do the work, hand over the finding, get dismissed, then maybe watch the issue disappear later.

That is a terrible incentive model.

Especially for the kind of people most likely to find architectural issues:

*   maintainers
    
*   infrastructure developers
    
*   SDK authors
    
*   bot developers
    
*   API integrators
    
*   security engineers
    
*   people who actually live inside the ecosystem
    

Those people are not scanning random forms for easy XSS.

They understand the system because they build around it every day.

If they stop trusting the disclosure process, the platform loses an early-warning system.

## The supply-chain findings make the original argument stronger

This is why the last thirteen days matter.

The original Bugcrowd response treated the issue as if exploitation depended on a user somehow handing over a `listenKey`.

But the modern developer threat model does not work that way.

In the last thirteen days alone, I documented or found:

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

## What a consistent model should look like

The correct model is simple:

> A derived credential must not be more portable than the credential that created it.

If the API key is IP-bound, the stream credential should be IP-bound.

If the parent credential normally relies on proof of possession through the API secret, the derived stream credential should not be issued through an unsigned API-key-only flow.

If users are encouraged to rely on IP whitelisting, the API should not create a second path around that boundary.

That was the fix I asked for.

And based on the May 5 re-test, that is essentially what Binance now appears to enforce for Futures `listenKey`.

So the technical direction is right.

The problem is the path it took to get there.

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

## To be clear: I still like Binance

I build and maintain open source infrastructure for the Binance ecosystem because I think the ecosystem matters.

Binance has built useful APIs.

The developer community around those APIs is real.

A lot of people depend on this infrastructure.

That is exactly why I care.

This is not about hating Binance.

It is about expecting better from a platform that sits at the center of a large automated-trading ecosystem.

Security bugs happen.

Architecture mistakes happen.

Legacy APIs happen.

Silent fixes also happen.

But if a report is good enough to lead to a fix, it should not be treated as if it was never a valid issue.

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

The system got safer.

The process did not.

* * *

I hope you found this informative and useful.

Follow me on [Binance Square](https://www.binance.com/en/square/profile/oliver-zehentleitner), [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\*(ツ)*/¯