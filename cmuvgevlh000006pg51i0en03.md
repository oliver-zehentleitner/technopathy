---
title: "The Model Changed. My Skill Didn't. The Score Still Dropped."
datePublished: 2026-10-05T16:18:19.668Z
cuid: cmuvgevlh000006pg51i0en03
slug: the-model-changed-my-skill-didn-t-the-score-still-dropped
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/79fa2fb2-535e-4a3a-b40e-c8147afad134.png
tags: ai, testing, llm, evals, context-engineering, coding-agents, keepthewhy

---

My rule for evaluating an agent skill is deliberately asymmetric:

**Test the agent on the weakest model you intend to support. Choose the judge by measuring which model grades that task reliably.**

Those are two different jobs. For the agent under test I want the floor: if a skill only works because a stronger model silently compensates for vague instructions, I have not tested the skill. For the judge I want a steady measuring instrument, and how strong it has to be for that is something to measure, not to assume.

I arrived at this while building the eval suite for [Keep the Why](https://keepthewhy.com/), a repo-native project-memory skill for coding agents. The lesson applies to any setup where one model performs a task and another one grades it, because neither side of that measurement stays still. And if you do not record the instrument, a regression in the instrument looks exactly like a regression in your product.

## The day the score dropped without the skill changing

The [eval runner](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/tools/evals/README.md) runs each case in a fresh fixture project through a real coding-agent session, captures transcript and disk diff, applies deterministic checks where possible and sends the rest to an LLM judge. Agent and judge were both called through the `sonnet` alias.

For the 0.18.0 series the alias resolved to Claude Sonnet 5. Three full runs scored **100 · 100 · 98 of 101**. The median case took about 13 turns, 11 tool calls and 2,300 thinking tokens.

A few days later the 0.18.2 tag [measured](https://keepthewhy.com/evals/0.19.0/) **92 · 96 · 93 of 101**. The alias now resolved to Sonnet 5.5, for agent and judge. More telling than the pass count was the session shape: about 6 turns, 4 tool calls and 250 thinking tokens per case. Nothing in the repository explained a change that large. The instrument had changed underneath the test.

I could only see that because every run records more than a score: resolved model IDs, judge-prompt hash, CLI version and session-shape statistics. Without them the obvious reading would have been "0.18.2 regressed the skill", and the obvious response would have been to rewrite the skill. That would have been fixing the wrong thing.

**Compare instruments before comparing pass counts.** The same alias is not the same instrument.

## The floor itself moves

Not even the same resolved model ID is enough. Four days after the 0.17.0 series scored 87 · 88 · 87 of 88, essentially the same skill text on the same Sonnet 5 ID [measured 83 · 80 · 81](https://keepthewhy.com/evals/0.17.1/), with the same halved session shape. A counter-run on the previous CLI behaved the same way. The next morning the sessions were back to normal length and the series measured 87 · 87 · 88. I changed nothing in the skill for that evening's failures, and both series stayed in the history. Rerunning until green would have published the instrument's good day, not the behavior of the skill.

This is also where "test on the weakest model" needs a qualification. I used to say: if the skill works on the weakest model, upward only gets easier. That is too simple. Sonnet 5.5 is newer than Sonnet 5, but for this skill it was the weaker reader: fewer steps, referenced files opened far less often. Newer is not stronger for a given skill. So the principle is:

**Stabilize the skill on the weakest supported tier, then re-measure when the model inside that tier changes.**

One caveat: the "upward gets easier" half is a principle, not something these series measured. They run the agent on Sonnet only. A stronger model can fail differently, for example by doing more than it was asked to, which is one reason hard prohibitions are checked deterministically and not left to a judge.

## What the weaker reader exposed

The model change was not only noise. It exposed a structural weakness in the skill.

`SKILL.md` often named a rule and pointed to a reference file for the operational detail. That works as long as the agent opens the reference. Sonnet 5.5 often did not. In every failing `local-lint-auto` run the agent installed the linter into a virtual environment, which the referenced setup section forbids. None of those runs had opened that section. The one run that did, passed. Without the reference, the model fell back on a trained habit.

The rule that came out of it:

**If missing a rule can cause harm, its operative clause belongs in `SKILL.md`, not only behind a pointer.**

For procedures too long for a clause, the vague "see this reference" became a read-before trigger at the point of action. I rejected both extremes: always reading every reference multiplies the cost of every session, and moving everything into the main file bloats what is loaded on every activation.

The stronger reader had quietly compensated for that dependency. That is exactly why the agent under test belongs on the floor.

## Then the judge turned out to be the bigger source of noise

Not every failure belonged to the skill either. One deterministic check had a schema version hard-coded and failed on every later release. One fixture sat under a setting where the behavior it was supposed to test never occurs, and the judge split pass/fail on identical behavior. Two expectation texts demanded things the skill never requires. **Expectation texts are specs too:** if the case says "must" where the product says "may", the eval is wrong, and rewriting the product to satisfy it means training your implementation against a bug in the test.

The harness stores transcript and disk diff for every run, so the judge can be asked again about the exact same agent behavior without a new agent session. That helper, `regrade.py`, dates back to the drift evening above. After the switch to 5.5 it became the first step for every failure, because it separates agent variance from judge variance. On one regrade of 15 stored failures, the Sonnet 5.5 judge confirmed only 6 of its own verdicts.

**Regrade before reword.** If the verdict moves on fixed evidence, you have a measurement problem first.

Eventually I regraded a complete candidate series, 309 stored records, with Opus 5.5 as the judge:

| Judge | Run 1 | Run 2 | Run 3 |
|---|---|---|---|
| Sonnet 5.5 | 98 | 100 | 98 |
| Opus 5.5, same transcripts | 102 | 102 | 102 |

(of 103 cases each)

The interesting part is not that the number went up. Opus kept all 296 passes as passes. Of the 13 failures it confirmed 3 and overturned 10, among them a "fail" whose own reasoning called the behavior pass-like and a deduction for a write the skill explicitly requires.

That does not make "use the strongest model as judge" a rule. It shows something narrower: for this suite, on these transcripts, Opus was a much steadier instrument than Sonnet 5.5. The method is to compare candidate judges on the same stored evidence and take the cheapest one that is steady enough. Here that threshold landed on Opus. In another suite Sonnet may be enough, or a model from a different vendor may be better.

The judge prompt stayed untouched across the switch, same hash, only the model changed. I rejected the other tempting path, a judge prompt extended for every known failure form. As the project's own [rationale](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/context/evals.md) puts it:

> "The judge should be reliable and not become a product of its own."

One variable at a time is slower. It is also how you learn which layer was actually wrong.

## The rules I would carry into another eval project

1. **Record the instrument.** Resolved model IDs instead of aliases, judge-prompt hash, CLI version, session shape. A score without its instrument is worth much less than it looks.
2. **Regrade before reword,** and check the expectation text before either.
3. **No sentence for a single flip.** A new sentence goes into the skill only for a failure form seen twice. Nine sentences added for one-off flips bought no measurable improvement, grew the skill by 17 percent and caused regressions in neighboring cases.
4. **Deterministic guards for deterministic prohibitions.** "Do not write under `context/`" or "never put a secret on disk" is checked mechanically and tolerates zero violations.
5. **A series gate instead of a perfect run.** Three full runs and a repeatability rule say more than one lucky 100 percent.
6. **Keep the failed measurements.** A series that failed because the ruler changed can be more informative later than another clean one.

## Did the thing change, or the ruler?

The [0.20.0 series](https://keepthewhy.com/evals/0.20.0/), Sonnet 5.5 as agent and Opus 5.5 as judge, scored **103 · 104 · 103 of 104** and was the first to pass every line of the suite's release gate. That is a dated result, not the lesson. Case count, agent model, judge model and harness have all changed since the earlier series, so the raw numbers do not compare.

The lesson is the third thing between the system under test and the score: the measuring instrument. The model behind an alias, the CLI, the judge, its prompt, the fixtures, the checks. Any of them can move. So when an eval suddenly gets worse, I no longer start with "what should I change in the prompt?" but with "did the thing being measured change, or did the ruler?" That question has already kept me from "fixing" correct behavior several times.

**Be pessimistic about the agent. Measure the judge. Record the instrument.**

## Further reading and raw material

- [Keep the Why eval suite and full run history](https://keepthewhy.com/evals/)
- [Eval runner](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/tools/evals/README.md)
- [Project rationale behind the eval methodology](https://github.com/oliver-zehentleitner/keep-the-why/blob/main/context/evals.md)
- [PR #556: fixes after the Sonnet 5.5 failure forms](https://github.com/oliver-zehentleitner/keep-the-why/pull/556)
- [PR #582: expectation-text calibration](https://github.com/oliver-zehentleitner/keep-the-why/pull/582)
- [PR #609: judge regrading and the switch to Opus](https://github.com/oliver-zehentleitner/keep-the-why/pull/609)

---

I hope you found this informative and useful.

Follow me on [GitHub](https://github.com/oliver-zehentleitner), Bluesky, Mastodon, X, and LinkedIn, or join Telegram for updates on my latest publications. Constructive feedback is always appreciated.

Thank you for reading, and happy coding! ¯\_(ツ)_/¯