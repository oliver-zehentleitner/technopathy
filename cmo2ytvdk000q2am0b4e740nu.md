---
title: "I Let an AI Agent Maintain My Open Source Suite for a Week — Here's What Actually Happened"
datePublished: 2026-04-17T13:50:19.163Z
cuid: cmo2ytvdk000q2am0b4e740nu
slug: i-let-an-ai-agent-maintain-my-open-source-suite-for-a-week-here-s-what-actually-happened
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f4a8d926-3d2e-438d-b973-868cc9001d85.png
tags: artificial-intelligence, python, opensource, devops, software-engineering

---

I maintain the [UNICORN Binance Suite](https://github.com/oliver-zehentleitner/unicorn-binance-suite): 7 Python repositories, 2.8M+ PyPI downloads, 388+ dependent projects, and production usage in algorithmic trading systems running 24/7 against real money.

WebSocket streams, REST APIs, local order books, trailing stops, Kubernetes depth cache clusters — the kind of infrastructure that normally has a team behind it.

It doesn’t. It has me.

Last week I gave [Claude Code](https://claude.com/claude-code) commit access via a dedicated GitHub account  
([@oliver-zehentleitner-aigent](https://github.com/oliver-zehentleitner-aigent)) and pointed it at the backlog.

**One week later:**

Work that would have taken me weeks to months was done.

Notably:

*   500+ files changed
    
*   100+ PRs created
    
*   7 repositories fully synchronized and updated
    
*   entire UBS stack released and brought up to current state
    
*   all known bug reports closed
    
*   ~€100 cost
    

This is not hype. This is a field report.

* * *

## The Backlog

Years of solo-maintainer entropy:

*   LUCIT branding everywhere (7 repos, hundreds of files)
    
*   Custom LSOSL license blocking contributors
    
*   Python 3.7 baseline (in 2026)
    
*   A core repo with no release for 4 years
    
*   CI pipelines that barely validated anything
    
*   Dead services, dead links, dead integrations
    

Individually trivial. Collectively unmaintainable.

That’s the key failure mode: not complexity, but accumulation.

* * *

## The Setup (Guardrails First)

Before letting an agent touch production code, I built constraints.

### Dedicated Git Identity

Everything runs through a separate account:  
[@oliver-zehentleitner-aigent](https://github.com/oliver-zehentleitner-aigent)

Full transparency:

*   every commit is traceable
    
*   no hidden automation
    
*   no impersonation
    

* * *

### CLAUDE.md — Project Constitution

Each repo defines strict rules:

*   communication style (German chat, English code)
    
*   forbidden actions (e.g. versioning scripts, direct commits to merged PRs)
    
*   architectural context
    

It helps. It does not guarantee compliance. Drift still happens.

* * *

### AGENTS.md + TASKS.md

*   `AGENTS.md`: architecture + conventions
    
*   `TASKS.md`: living backlog per repo
    
*   `UBS-TASKS.md`: cross-repo coordination
    

* * *

### Fork + PR Workflow

No direct pushes.

Everything: fork → branch → PR → review → merge

This is non-negotiable for production-grade OSS.

* * *

## What the Agent Actually Did

We executed bottom-up through the dependency stack:

UnicornFy → UBRA → UBWA → UBLDC → UBDCC → UBTSL → UBS

* * *

### Day 1–2: Foundation Repos

*   Replace all LUCIT branding
    
*   Switch LSOSL → MIT
    
*   Raise Python baseline to 3.9–3.14
    
*   Expand CI across versions
    
*   Fix documentation and metadata consistency
    

* * *

### Day 3: UBTSL (the worst one)

No release in 4 years. Broken startup due to licensing dependency.

The agent:

*   removed entire licensing system
    
*   fixed CI (geo-blocked endpoints, artifact merging bug)
    
*   repaired PyPI wheel publishing
    
*   resolved 3-year-old PR conflicts
    

* * *

### Day 4–5: Suite-wide Cleanup

*   removed dead Gitter integrations
    
*   fixed issue templates across repos
    
*   normalized exchange lists
    
*   removed deprecated endpoints
    

* * *

### Day 5: README Rewrite

*   install-first onboarding
    
*   metrics-driven introduction
    
*   architecture diagram
    
*   comparison table
    
*   copy-paste examples
    

* * *

### Day 6: AI-Native Documentation

Introduced `llms.txt` across all repos.

Meta-layer shift: AI writing docs for other AIs.

* * *

### Day 7: Releases, Stabilization & Full Stack Sync

This is where things became more interesting than planned.

During execution the agent identified **two independent race conditions** in core streaming and order-handling paths. These were subtle timing issues that had not surfaced in production yet, but were structurally real and reproducible under load.

After fixing them, we did not just ship isolated patches — we executed a full stack alignment:

*   UBTSL 1.3.0 released (first release in 4 years)
    
*   UBS 2.1.0 released
    
*   all dependent repositories updated and republished
    
*   full UBS ecosystem brought to a consistent, current version state
    
*   all known bug reports across the stack closed
    

Net effect: the entire suite is now not just “maintained”, but structurally consistent again.

* * *

## Cost Model

*   Claude Pro Max: ~€100/month
    
*   Time: ~1 week active steering
    

The real cost is not money. It is decision bandwidth.

* * *

## The Workflow That Worked

1.  Analyze state together
    
2.  Agree on direction
    
3.  Define scope
    
4.  Execute
    
5.  Review and correct
    

* * *

## What Worked

*   Cross-repo pattern replication
    
*   CI + packaging bug discovery
    
*   Merge conflict resolution
    
*   Parallel PR execution
    
*   Documentation iteration speed
    
*   Structural bug detection in production-adjacent code paths
    

* * *

## What Didn’t

*   Context limits
    
*   Premature execution
    
*   Tone issues in marketing copy
    
*   Occasional hallucinated assumptions
    
*   Git sync conflicts
    

* * *

## Operating Model

The agent behaves like a very fast staff engineer:

*   never bored
    
*   never inconsistent
    
*   never skipping files
    
*   still needs direction
    

* * *

## The Real Shift

Most OSS maintenance is not engineering.

It is:

*   version updates
    
*   CI fixes
    
*   link rot
    
*   dependency drift
    
*   template maintenance
    

AI agents are extremely good at this.

* * *

## Outcome

The suite is now:

*   consistently MIT licensed
    
*   Python 3.9–3.14
    
*   CI-clean across repos
    
*   structurally unified
    
*   fully up to date across the entire stack
    
*   all known bug reports resolved
    
*   properly documented
    
*   partially AI-native (`llms.txt`)
    
*   significantly more stable under real-world load
    

* * *

## What’s Next

*   UBRA v3 architecture rewrite
    
*   deeper feature-level agent collaboration
    
*   defining the boundary of autonomy
    

* * *

*This article was drafted with assistance from an AI agent, iterated through review loops, and finalized by me.*

* * *

*UNICORN Binance Suite — production-grade Python infrastructure for Binance trading systems.*

* * *

I hope you found this tutorial informative and enjoyable! 

Follow me on [GitHub](https://github.com/oliver-zehentleitner), [X](https://x.com/unicorn_oz) and [LinkedIn](https://www.linkedin.com/in/oliver-zehentleitner/) to stay updated on my latest releases. Your constructive feedback is always appreciated!