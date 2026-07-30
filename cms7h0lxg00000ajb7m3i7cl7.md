---
title: "Operation Endgame Disrupted 326 Servers. The StealC Backend I Reported Still Responds."
seoDescription: "Operation Endgame disrupted 326 servers and 142 domains. Three months after disclosure, the StealC malware routes I documented still respond."
datePublished: 2026-07-30T12:09:20.677Z
cuid: cms7h0lxg00000ajb7m3i7cl7
slug: operation-endgame-stealc-backend-still-responds
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/99b44f22-e7cd-4822-8ba6-cb7ff76eaf3b.png
tags: cloudflare, opensource, infosec, supplychain, malware-analysis, infostealer, stealc, threatintelligence, abusereporting, operationendgame

---

*TL;DR: GitHub removed all 19 typosquat repositories from the nailproxy.space campaign about a month after I reported them. On June 24, Europol and Microsoft announced a much larger Operation Endgame action against SocGholish, Amadey, and StealC infrastructure: 326 servers and 142 domains actioned, 27 million stolen credentials recovered, and more than EUR 41 million in criminal crypto assets restricted. On July 30, I checked the infrastructure from my own April disclosure again. The two documented malware routes still return the same route-specific* `405 Method Not Allowed` *response while a random control path returns* `404 Not Found`*. That is evidence that the known application routes remain registered. It is not proof that the complete malware workflow still functions. This post is about that distinction — and about what large takedown numbers do and do not tell us about one specific, previously reported backend.*

* * *

## A short recap

The discovery and technical breakdown are in two earlier posts:

*   [nailproxy.space: A Multi-Repository GitHub Malware Campaign](https://blog.technopathy.club/nailproxy-space-github-malware-campaign) — the typosquats, how I found them, and the initial disclosure.
    
*   [From a Coffee-in-Bed Google Search to a StealC-Linked Campaign](https://blog.technopathy.club/from-a-coffee-in-bed-google-search-to-a-stealc-linked-campaign-the-story-behind-nailproxy-space) — the full kill chain, the StealC v2 attribution, and the backend infrastructure.
    

The short version: a Python dropper was published across 19 fake GitHub repositories, including one impersonating my own UNICORN Binance WebSocket API. It called `api.nailproxy.space`, received an AES-GCM-encrypted Windows PE loader, and eventually dropped a 55 KB DLL attributed to StealC v2 with Chrome App-Bound Encryption bypass capability. The samples referenced `62.60.226.113:6673` and `spellmarketplace.club` in the exfiltration chain.

I also published the indicators outside my own blog. Eight campaign entries are available through my public [ThreatFox profile](https://threatfox.abuse.ch/user/12877/), including the domains, both documented API routes, the host and port, and the relevant payload hashes. The broader public dataset is available through my [AlienVault OTX profile](https://otx.alienvault.com/user/oliver-zehentleitner/pulses) and the dedicated [nailproxy.space OTX pulse](https://otx.alienvault.com/pulse/69f9ab354863232e7c0dcad1), which currently contains 40 indicators and 14 ATT&CK technique mappings.

Those timestamps matter. The indicators were documented publicly before the June disruption action.

* * *

## What GitHub removed

I reported the 19 repositories through GitHub Support on April 22, 2026, under ticket 4313391. Based on my own contemporaneous monitoring, they disappeared around May 20, roughly four weeks after the report. GitHub Trust & Safety formally confirmed a Terms-of-Service violation and that action had been taken on June 10.

The difference between those dates is worth noting: the observable removal and the formal case closure were about three weeks apart. An abuse-ticket close date is therefore not necessarily the same as the date the content actually disappeared.

On July 30, I checked all 19 repository URLs again through both the GitHub website and API. Every one of them returns `404`.

That closed the campaign's visible GitHub delivery front. It did not disable the hard-coded backend URL in any copy that had already been cloned, downloaded, cached, or mirrored elsewhere. Suspending the delivery domain would break that path once DNS caches expired. GitHub had no direct lever over it because the domain and hosting infrastructure were outside GitHub.

* * *

## What the backend checks actually show

On May 21, the day after I first observed the repository removal, I ran minimal, unauthenticated HTTP checks against the documented infrastructure. These were low-impact active probes: `GET`, `HEAD`, DNS resolution, and a bare TCP connection. I did not send the malware's authentication handshake or any payload.

The important result was the difference between the known application routes and a random control path:

```text
GET  https://api.nailproxy.space/                      -> 404 {"error":"not found"}
GET  https://api.nailproxy.space/api/v1/auth/session   -> 405 {"error":"method not allowed"}
GET  https://api.nailproxy.space/api/v1/data/sync      -> 405 {"error":"method not allowed"}
GET  https://api.nailproxy.space/foo/bar/baz           -> 404 {"error":"not found"}
```

I repeated the checks on July 30. Both `GET` and `HEAD` produced the same result on the two documented malware routes:

```text
GET  /api/v1/auth/session   -> 405
HEAD /api/v1/auth/session   -> 405
GET  /api/v1/data/sync      -> 405
HEAD /api/v1/data/sync      -> 405

GET  /foo/bar/baz           -> 404
HEAD /foo/bar/baz           -> 404
```

The narrow conclusion is defensible: the two known application routes remain registered and distinguishable from an arbitrary path. A generic parked domain or completely unrelated replacement service would normally not reproduce that exact route pattern.

The stronger conclusion is not defensible. A `405` response does not prove that a valid `POST` request would still deliver Stage 2, accept exfiltrated data, or complete the original malware workflow. I deliberately did not test that.

The root paths produced an additional result I had not noticed in the earlier draft:

```text
GET  https://spellmarketplace.club/        -> 404
HEAD https://spellmarketplace.club/        -> 200

GET  https://62.60.226.113:6673/           -> 404
HEAD https://62.60.226.113:6673/           -> 200

TCP  62.60.226.113:6673                    -> connection succeeded
```

`HEAD` should normally return the same status code as the corresponding `GET`, without the body. Here it does not. That could be a reverse-proxy quirk, special method handling, or some other implementation detail. From outside, I cannot tell. The root-path `200` responses are therefore not clean proof that the original exfiltration service is fully operational.

The malware-specific route pattern is the stronger signal. The root-path result is ambiguous.

* * *

## The monitoring was useful — but not continuous

I originally configured an automated daily monitor. In practice, it produced only 18 measurements across 69 days, with gaps of up to 20 days. That makes the dataset a series of snapshots, not proof of uninterrupted availability.

The log also contains more movement than my first draft acknowledged:

*   Around May 24–27, the delivery checks temporarily returned curl status `000`, meaning the request failed before an HTTP response was received. That may have been an outage at the target, a DNS or TLS problem, or a failure on the monitoring side. The log cannot distinguish between them.
    
*   `spellmarketplace.club` moved between `523`, `200`, and `000` during later checks.
    
*   Outside those recorded failures, the two documented delivery routes repeatedly returned the same `405` signature seen in April and May.
    

So I cannot claim that the infrastructure remained continuously available through the entire period or through every moment of Operation Endgame. I can say that after the operation, on July 30, the same documented malware routes were still present and responding in the same way.

That is a less dramatic statement. It is also the one the evidence supports.

* * *

## Then Operation Endgame happened

On June 24, Europol announced a new phase of [Operation Endgame](https://www.europol.europa.eu/media-press/newsroom/news/global-cyber-strike-disrupts-socgholish-amadey-and-stealc-malware-networks) targeting infrastructure associated with SocGholish, Amadey, and StealC. Europol described coordinated actions conducted “over the past two weeks”; it did not publish a precise June 15–19 action window.

For this specific action, Europol named law-enforcement participation from Canada, Denmark, Germany, the Netherlands, the United Kingdom, and the United States, coordinated with Europol and Eurojust. Its official private-partner list included Microsoft, Bitdefender, Shadowserver, Proofpoint, IBM X-Force, Infoblox, Spamhaus, Have I Been Pwned, and several others.

The published results were substantial:

*   **326 servers and 142 domains** actioned by law enforcement and private-sector partners across the combined SocGholish, Amadey, and StealC operation.
    
*   **27 million stolen login credentials** recovered.
    
*   More than **EUR 41 million / USD 47 million** in criminal crypto assets identified, flagged, and restricted from use.
    

In its own [technical report](https://www.microsoft.com/en-us/security/blog/2026/06/24/stealc-and-amadey-breaking-down-infostealers-and-the-cybercrime-services-that-deliver-them/), Microsoft's Digital Crimes Unit said it had identified **over 200 malicious Amadey and StealC command-and-control domains and IPs** and moved to shut them down through court orders, domain seizures, registrations, and provider notifications.

That wording matters. It is not “200 StealC servers.” It is a combined Amadey-and-StealC count of domains and IP addresses.

Microsoft also described using Copilot-assisted tooling to analyse binaries, extract configuration data, decrypt strings, identify hard-coded C2 infrastructure, and confirm C2 activity. That is an interesting technical detail, but the relevant point here is the scale: this was a large, coordinated, and technically sophisticated disruption effort.

Microsoft's IOC table publishes twelve example C2 URLs: seven attributed to StealC and five to Amadey. None of the public examples contain `nailproxy.space`, `spellmarketplace.club`, `62.60.226.113`, or the two main hashes from my disclosure.

That does **not** prove my indicators were absent from the full target set. Microsoft published examples, not a complete list. I have no basis for claiming that this infrastructure was overlooked, evaluated, or deliberately excluded.

What I can verify is the outcome visible from outside: several weeks after the announcement, the two documented `api.nailproxy.space` malware routes still return the same route-specific responses.

* * *

## This is not a takedown “gotcha”

Operation Endgame's published numbers are large and meaningful. Disrupting hundreds of servers and domains, recovering 27 million credentials, and restricting tens of millions of euros in criminal assets is real impact.

But aggregate success does not imply complete coverage.

“Hundreds of malicious systems were disrupted” and “this particular previously reported infrastructure still responds” can both be true. A press release describes the aggregate operation. It does not tell a maintainer whether one specific domain, IP, route, or hash from an earlier report was included.

My checks also cannot tell me *why* the infrastructure still responds. It may never have been on the target list. It may have been assessed and deprioritised. The routes may remain while the operational backend behind them has changed or become inert. Some part may have been rebuilt. From the outside, those possibilities are indistinguishable without stronger evidence.

The useful conclusion is smaller: the specific route pattern I disclosed in April was still observable after the operation.

* * *

## GitHub was probably only one delivery path

The investigation also found a sibling Stage-2 build with SHA-256 `155dc73761ebaab0e4f5c0e18cf09dbd5728ce61361db218a5727355ca8adc1a`.

It shared the same masquerade family and referenced the same exfiltration infrastructure, which makes a common operator or campaign plausible. It did not surface through the `nailproxy.space` delivery pivot, however, and its actual delivery channel remains unidentified.

That does not prove the other channel is still active, that it continues producing victims, or that the operator has an ongoing revenue stream. It does suggest that the GitHub repositories were not necessarily the only way this infrastructure was used.

Removing one delivery front can therefore be effective without removing every path connected to the same backend.

* * *

## Who still has an operational lever

Different organisations control different layers:

| Asset | Party with an operational lever | Realistic action |
| --- | --- | --- |
| `nailproxy.space` | Namecheap; registry escalation if appropriate | Domain hold or suspension, breaking DNS-based delivery |
| `spellmarketplace.club` | Dynadot; registry escalation if appropriate | Domain hold or suspension, breaking DNS resolution |
| Cloudflare proxy layer | Cloudflare abuse process | Forward the report and origin details to the website operator and hosting provider; action on Cloudflare-hosted services where applicable |
| `62.60.226.113:6673` | Femo IT Solutions / relevant upstream network | Block the service, null-route traffic, or take another network-level abuse action |
| RIPE NCC | No direct mitigation authority | Maintain registry and abuse-contact data; identify the responsible network operator |

Cloudflare's [published abuse approach](https://www.cloudflare.com/trust-hub/abuse-approach/) says that for its reverse-proxy and CDN services it normally forwards reports to the website operator and hosting provider, providing the origin IP to the hosting provider. That is different from routinely disclosing the origin IP directly to the reporter or independently removing content it does not host.

Likewise, RIPE NCC is not an operational upstream and cannot null-route the service. Its role is to allocate and register Internet number resources and provide the responsible network's abuse contact. The network operator is responsible for acting on the report.

Current RDAP data still shows Namecheap as registrar for `nailproxy.space`, Dynadot for `spellmarketplace.club`, Cloudflare nameservers on both domains, and Femo IT Solutions associated with the `62.60.226.0/24` network. Neither domain shows a suspension or hold status.

An additional historical detail is relevant but needs context: when I checked ThreatFox in April, it contained an independent StealC record from reporter `juroots` for a different URL path on the same IP, first seen on December 15, 2025, at confidence 50. That record has since aged out of ThreatFox's live API. It does not validate port `6673`, but it shows that the IP had appeared in independent StealC reporting before my campaign report.

* * *

## Six reports, one verifiable action

On May 21, I sent six targeted abuse reports in parallel:

*   Dynadot
    
*   Namecheap
    
*   Cloudflare Trust & Safety
    
*   Femo IT Solutions
    
*   CERT-Bund / BSI
    
*   GitHub Security Lab
    

Each report contained the relevant asset, minimal live-verification results, the full IOC set, public references, and one request scoped to what that recipient could realistically do. The Cloudflare report focused on routing the evidence and origin information to the appropriate hosting party. The Femo report requested action against the service on port `6673`, not the entire shared network.

As of July 30, I have no local record of a ticket number, acknowledgment, or human reply from Dynadot, Namecheap, Cloudflare, Femo IT Solutions, or CERT-Bund. GitHub Security Lab responded in relation to the existing GitHub Support case, but it has no authority over external registrars or hosting infrastructure.

The GitHub repository removal remains the only concrete action I can independently verify from this reporting cycle.

That statement is deliberately limited to my records. I cannot rule out internal handling, provider-to-provider communication, law-enforcement coordination, or an email that never reached the archive I checked.

* * *

## What I deliberately did not do

A few technically possible steps were intentionally left out:

*   **No authenticated malware requests.** I did not send the HMAC handshake or a valid `POST` request to `/api/v1/auth/session`. That would go beyond the minimal verification needed and raise unnecessary legal and ethical questions.
    
*   **No payload retrieval.** I already had the April samples. Triggering another delivery would add risk without materially improving the conclusion.
    
*   **No independent Cloudflare de-cloaking.** Historical-certificate pivots, subdomain enumeration, and similar techniques can expose an origin, but they can also publish useful information to the wrong audience. I left that step to the abuse process and parties with the appropriate authority.
    
*   **No additional IOC disclosure.** The relevant indicators were already public through the earlier posts, my ThreatFox submissions, and the OTX pulse.
    
*   **No contact with the operator.** This is criminal infrastructure, not a normal coordinated vulnerability disclosure with a vendor. Contact would create an OPSEC signal and unnecessary exposure.
    

* * *

## What this case is useful for

Three practical lessons survive regardless of whether this infrastructure eventually disappears.

### 1\. A `405`/`404` difference is useful, but bounded evidence

Comparing documented application paths with a random control path can show that known routes remain registered without sending the expected request body. It is stronger than checking only whether a hostname resolves or whether `/` returns something.

It still does not prove that the complete malicious workflow is operational. Route registration, payload delivery, authentication, storage, and exfiltration are separate claims requiring different evidence.

### 2\. Audit the monitoring, not just the target

My cron was configured daily. The evidence showed only 18 actual measurements over 69 days. The original draft overstated both the cadence and stability because I had not first measured the monitor itself.

The GET/HEAD mismatch on the root paths was another reminder: a convenient `200` can be a protocol or implementation quirk rather than the signal you think it is. Check multiple methods, use a control path, preserve raw output, and state exactly what the observation proves.

### 3\. Read disruption numbers as aggregates

“326 servers and 142 domains actioned” is a meaningful operational result. “Over 200 Amadey and StealC C2 domains and IPs identified” is meaningful too.

Neither number answers whether a particular indicator was included. For infrastructure you directly track, the only responsible approach is to verify the specific asset and describe the result narrowly.

In this case, the narrow result is simple: the 19 GitHub repositories are gone, while the two documented malware routes on `api.nailproxy.space` still respond with the same route-specific status pattern recorded after the original disclosure.

That is not eradication. It is not proof of a fully functioning C2 either.

It is the evidence I have.

* * *

## IOCs and public threat-intelligence records

The complete technical context remains in the [second post](https://blog.technopathy.club/from-a-coffee-in-bed-google-search-to-a-stealc-linked-campaign-the-story-behind-nailproxy-space).

Core indicators:

*   Delivery domain: `api.nailproxy.space`
    
*   Documented routes:
    
    *   `/api/v1/auth/session`
        
    *   `/api/v1/data/sync`
        
*   Exfiltration infrastructure: `62.60.226.113:6673`
    
*   Associated domain: `spellmarketplace.club`
    
*   Stage-2 SHA-256: `251037ceebfbacd419b663ebcf0e01ec80a2c46dbfc85f66492c8585b481fb8c`
    
*   Sibling Stage-2 SHA-256: `155dc73761ebaab0e4f5c0e18cf09dbd5728ce61361db218a5727355ca8adc1a`
    
*   Stage-3 StealC v2 DLL SHA-256: `c27590c766583599eac98ed3e20c54e49c792be409f126577e7475294affac1f`
    

Public records:

*   [My ThreatFox submissions](https://threatfox.abuse.ch/user/12877/)
    
*   [My AlienVault OTX pulses](https://otx.alienvault.com/user/oliver-zehentleitner/pulses)
    
*   [nailproxy.space OTX pulse — 40 indicators, 14 ATT&CK mappings](https://otx.alienvault.com/pulse/69f9ab354863232e7c0dcad1)
    

* * *

This post is part of the [Security Research series](https://blog.technopathy.club/series/security-research) and the third article in the nailproxy.space disclosure thread. See the [first post](https://blog.technopathy.club/nailproxy-space-github-malware-campaign) and the [second post](https://blog.technopathy.club/from-a-coffee-in-bed-google-search-to-a-stealc-linked-campaign-the-story-behind-nailproxy-space).

I build open-source tools, including [Keep the Why](https://keepthewhy.com), and maintain the [UNICORN Binance Suite](https://blog.technopathy.club/page/unicorn-binance-suite).

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [Mastodon](https://burningboard.net/@oliverzehentleitner), [X](https://x.com/unicorn_oz), and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/), or join [Telegram](https://t.me/unicorndevs) for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\\\_(ツ)\_/¯