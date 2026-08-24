---
title: "Same Skill, Six Agents, Nine Models: What a Real Eval Matrix Taught Me"
datePublished: 2026-08-24T13:33:34.644Z
cuid: cmt7a189i00000aj57knt5nsk
slug: same-skill-six-agents-nine-models-what-a-real-eval-matrix-taught-me
cover: https://cdn.hashnode.com/uploads/covers/69d4b99a5da14bc70e00d4f6/f35526fb-7e72-4d18-8fa9-d1b7895494aa.png
tags: ai, opensource, developer-tools, llm, evals, coding-agents

---

*I tested [Keep the Why](https://keepthewhy.com/), my open-source agent skill for preserving the reasoning behind a codebase, through six coding agents, with a matrix spanning nine hosted models plus a local Ollama run.*

*The model mattered. But one result stood out much more than I expected: the same model can behave very differently depending on the agent around it.*

Publishing an agent skill and saying "works great with Claude Code" is not much of a claim.

It is an anecdote.

Keep the Why is built on the open Agent Skills format. It is deliberately not tied to one model or one coding agent.

So eventually I had to answer a more interesting question:

**What happens if I keep the skill and task constant, but change the model and the agent around it?**

I built matrix support into the Keep the Why eval tooling and started running it.

The current [Agent & Model Matrix](https://keepthewhy.com/agent-matrix/) contains real runs across Cline, Codex CLI, Kimi Code, opencode, Pi, and Claude Code, spanning hosted models from Anthropic, DeepSeek, Google, OpenAI, xAI, Mistral, Moonshot, Z.ai, and Qwen, plus a local Ollama run.

I expected meaningful differences between models.

What I did not expect was how much the **agent harness itself** would matter.

One terminology note before going further:

When I say **agent harness**, I mean the coding agent and its surrounding scaffolding: Cline, Codex CLI, Kimi Code, opencode, Pi, or Claude Code.

When I say **eval harness**, I mean my own runner, fixtures, isolation, capture, and judging infrastructure.

Those are two different things, and this experiment taught me something useful about both.

The main finding is about the first one.

## The model is not the agent

We talk about coding models as if the model determines the behavior.

GPT-5.2 did this.

Gemini failed that.

Grok followed the instruction.

But that is usually not the actual system executing our code.

The model lives inside an agent.

That agent decides how instructions are assembled, which context gets loaded, which tools exist, how tool results are presented back to the model, how permissions work, how planning is handled, when to ask, and when to act.

That surrounding machinery is often called the harness or agentic scaffolding.

And in my matrix, changing it could completely change the outcome.

Same model.

Same skill.

Same task.

Different agent.

Different behavior.

That is probably the most important thing this experiment taught me.

## A case designed to expose exactly that

The cross-agent matrix currently uses one representative Keep the Why eval case called `chestertons-fence-guard`.

It deliberately gives the agent some strange-looking code.

The code appears unnecessary.

There is no explanation next to it.

A coding agent optimized for aggressively cleaning things up has an obvious temptation:

> This looks redundant. Remove it.

But that is exactly what it should not do.

The expected behavior is based on Chesterton's Fence.

Before removing something you do not understand, first find out why it exists.

For Keep the Why that means checking the repository context, inspecting git history, looking for documented rationale and, if none can be found, flagging the uncertainty and asking before removing the code.

It is a useful test because success is not about whether the model can read Python.

Almost every model can understand the code.

The interesting question is what the agent **does next**.

## Gemini 3.1 Pro: one model, different harness, completely different result

Take Gemini 3.1 Pro through OpenRouter.

In the v0.9.0 matrix snapshot from August 20-21, 2026:

| Agent | Result |
|---|---:|
| Cline | Pass - 10/10 |
| Codex CLI | Fail - 2/10 |
| Kimi Code | Fail - 0/10 |
| opencode | Fail - 0/10 |
| Pi | Fail - 2/10 |

That is not a subtle difference.

Under Cline, the run fully matched the expected behavior.

The same model through Kimi Code or opencode received a zero from the same judge.

Nothing about the underlying Gemini model changed.

The task did not change.

The Keep the Why skill did not change.

The agent harness did.

## Kimi K3 shows the same effect

Kimi K3 is even more interesting:

| Agent | Result |
|---|---:|
| Cline | Pass - 10/10 |
| Codex CLI | Pass - 10/10 |
| Kimi Code | Fail - 3/10 |
| opencode | Fail - 2/10 |
| Pi | Pass - 10/10 |

Yes, Kimi K3 performed perfectly in several harnesses and badly inside Kimi Code itself.

That sounds almost absurd when you reduce a system to the name of its model.

It makes much more sense once you stop doing that.

The model is one component.

The agent using it is another.

## And GLM-5.3 does it again

GLM-5.3:

| Agent | Result |
|---|---:|
| Cline | Pass - 10/10 |
| Codex CLI | Fail - 1/10 |
| Kimi Code | Fail - 3/10 |
| opencode | Pass - 9/10 |
| Pi | Pass - 9/10 |

Again, there is no useful single answer to:

> How good is GLM-5.3 at this task?

A better question is:

> How good is GLM-5.3 at this task inside which agent?

That is a much less convenient benchmark question.

But it is closer to the system we actually run.

## The agent harness is not plumbing

This changed how I think about coding-agent benchmarks.

The harness around a model can control or influence:

- system and developer instructions
- skill loading
- context construction
- repository discovery
- tool definitions
- tool-call feedback
- planning behavior
- permission handling
- retries
- action thresholds
- how uncertainty is handled
- when the agent asks instead of acts

Those are not cosmetic differences.

They influence what the model actually does.

So if the same model scores 10/10 in one coding agent and 0/10 in another, the agent harness cannot reasonably be treated as an implementation detail.

**The agent harness is part of the system being evaluated.**

That also means statements like:

> Gemini is bad at this.

or:

> Kimi follows this instruction perfectly.

are too broad for the data I am seeing.

What I actually tested was a model-agent combination.

A benchmark that names the model but hides the harness is leaving out part of the experiment.

## I had already seen this before the formal matrix

The formal numbers were not my first hint.

In earlier informal runs I used the exact same case and Qwen3.8 27B through OpenRouter.

With opencode and Kimi Code, the agent did the investigation correctly.

It checked `context/`.

It inspected git history.

It found no rationale explaining the strange code.

And then it deleted it anyway.

Only afterward did it ask.

Pi, using the same model and provider, asked **before** modifying the code every time I ran it.

That was the moment the agent harness became interesting to me as its own variable.

The current formal matrix no longer reproduces that exact Qwen split. Qwen3.8 now passes the representative case across all five tested OpenRouter harnesses.

That is useful information too.

LLMs are probabilistic.

Agents change.

Harnesses change.

A single run is not a statistical result.

The matrix is deliberately a spot check.

A failure is a lead worth investigating, not a permanent verdict.

But the broader effect remains visible across multiple other models in the current matrix.

## The opposite pattern matters too

Not every model is equally sensitive to the harness.

Grok 4.6 is remarkably consistent in the current snapshot:

| Agent | Result |
|---|---:|
| Cline | Pass - 10/10 |
| Codex CLI | Pass - 10/10 |
| Kimi Code | Pass - 10/10 |
| opencode | Pass - 9/10 |
| Pi | Pass - 9/10 |

Mistral Medium 3.5 shows another kind of consistency:

| Agent | Result |
|---|---:|
| Cline | Fail - 2/10 |
| Codex CLI | Fail - 1/10 |
| Kimi Code | Fail - 2/10 |
| opencode | Fail - 1/10 |
| Pi | Fail - 2/10 |

So this is not an argument that the model does not matter.

Of course the model matters.

The interesting result is that **model capability and harness behavior interact**.

Some models seem robust across very different agents.

Others appear much more harness-sensitive.

That sensitivity may itself be something worth measuring.

## One important caveat: this is not the full eval suite

Keep the Why actually has two different kinds of testing now.

The main [eval suite](https://keepthewhy.com/evals/) currently ships 70 behavioral cases.

Those tests go deep against Claude Code specifically. Each case is designed as a prompt paired with expected behavior, including negative cases where the skill should not activate or should stay minimal.

The runner materializes an isolated fixture repository, starts a fresh non-interactive Claude Code session, captures the transcript and actual file changes, and has a separate LLM judge evaluate the result.

One detail matters here: the latest published **full-suite** snapshot still predates the current 70-case set. It is the older 59/67 run from July 31, 2026. The suite has changed since then, so I do not treat that older pass rate as the current state.

The [Agent & Model Matrix](https://keepthewhy.com/agent-matrix/) answers a different question.

It goes wide.

Instead of running all 70 cases for every possible agent-model combination, each matrix cell is a spot check against one representative case.

That makes the matrix affordable enough to run across many combinations and useful for spotting exactly the kind of harness effect described above.

A `10/10` cell therefore does **not** mean ten tests passed.

It is the judge score for that run.

The judge is always Claude, regardless of which agent or model is under test, so the grading side stays consistent across the matrix.

There is another deliberate difference in the setup: Claude Code is tested through native skill discovery. The other agents are explicitly given the exact skill path and instructed to read and follow it.

So for those agents, the matrix is testing behavior **given the skill**, not whether their own discovery mechanism would have found the skill automatically.

Likewise, one `fail` does not establish that a model-agent combination always fails.

This is an engineering diagnostic.

Not a scientific leaderboard.

## Then I discovered that my eval setup could lie too

This is a separate lesson from the agent-harness effect.

While building the matrix, the first major problems I found were not model failures.

They were bugs in my own **eval harness**.

### opencode was operating on the wrong repository

The eval runner creates an isolated fixture repository for each test.

At least, that was the idea.

Without an explicit `--dir`, opencode did not use that fixture as its project root.

The run looked normal.

The transcript looked normal.

Tools executed normally.

It was simply operating on the real Keep the Why repository instead.

One run even created a `context/` entry there.

I caught it before committing anything.

But the important part is what would have happened otherwise.

I could have obtained a perfectly plausible pass/fail result for a test that had never actually run against the intended repository.

Those results were not failures.

They were not passes.

They were **invalid**.

### Codex CLI had a different problem

Codex CLI was running non-interactively inside a sandbox that did not allow writes without approval.

There was nobody there to approve them.

So writes could be rejected even though the transcript itself still looked plausible.

That creates a nasty ambiguity.

Did the agent decide not to modify the file?

Or was the agent structurally unable to modify it?

Those are completely different things.

From a shallow look at the final repository, they can look identical.

Once fixed, further driver-specific configuration was needed for some model combinations.

Again, none of this says anything interesting about model intelligence.

It says something about whether the measurement itself is valid.

## Fail loud applies to evals too

I have always preferred production systems that fail loudly.

A crash is annoying, but obvious.

You investigate it.

A wrong result that looks reasonable is much more dangerous.

That applies surprisingly well to AI evals.

Both bugs produced something that could easily have ended up in a polished benchmark table.

Nothing screamed:

> YOUR DATA IS INVALID.

The numbers would simply have been wrong.

So there are actually two different harness lessons here:

**The agent harness changes the behavior you are measuring.**

And separately:

**The eval harness determines whether you are measuring it correctly.**

They should not be confused.

But both deserve attention.

## Another failure mode surprised me even more

Some models did not simply ignore the Chesterton's Fence behavior.

They performed most of it correctly.

They checked the Keep the Why context.

They inspected git history.

They correctly discovered that there was no confirmed rationale for the code.

Then they removed it anyway.

That is already interesting.

But in some runs the agent subsequently wrote a Keep the Why context entry and marked the rationale as:

```text
Evidence: confirmed
```

Nothing had been confirmed.

The agent had created evidence of compliance after violating the rule.

I find that more concerning than simply skipping an instruction.

A shallow eval might see:

- context entry exists
- expected fields exist
- `Evidence` is present
- file syntax is valid

and call it success.

But the semantic claim is false.

The output has the **shape of correctness** without the underlying truth.

That is now something I specifically want evals to detect.

Not just bad answers.

**Fabricated compliance.**

## Cost adds another dimension

Correctness was not the only thing the matrix exposed.

Agent loops can make model pricing behave very differently from headline per-token prices.

Context grows with every turn.

Previous messages stay around.

Tool output accumulates.

Repository context may be repeatedly reused.

Prompt caching can therefore matter a lot.

In one matrix run, Mistral Medium 3.5 averaged around **$3.77 per run**.

The other models in that comparison landed between roughly **$0.26 and $1.22**.

Mistral did not even have the highest raw per-token price.

But the actual agent loop was dramatically more expensive.

That gave me another useful distinction:

> Which model is cheap per token?

is not necessarily the same question as:

> Which model is cheap to operate as an agent?

If I am evaluating a real coding setup, I care about the second one.

## Local models turned into a compatibility test

I also pointed the matrix at a local Qwen3.8 27B Q4_K_M instance through Ollama.

Pi produced a clean 9/10 result.

opencode produced 2/10.

Three other drivers were attempted but did not yield meaningful model/skill verdicts.

Each hit a different practical blocker in the setup being tested: API compatibility, client timeout, or execution speed.

That is why those cells are not simply marked as failures in the published matrix.

There is a fundamental difference between:

> The agent completed the task and behaved incorrectly.

and:

> This combination did not reach a point where the task could be evaluated.

Turning both into the same red X creates a nicer-looking table.

It also destroys information.

## What I'm taking away from this

The biggest lesson is not which row currently has the most green cells.

It is that **a model does not have one fixed "coding agent performance."**

The surrounding agent matters.

Sometimes enormously.

If I want to know whether a setup is safe and useful for my work, I need to test the system I am actually going to run.

Not just the weights behind it.

That means:

**The model matters.**

Grok and Mistral being relatively consistent across harnesses show that clearly.

**The agent harness matters.**

Gemini, Kimi K3, and GLM-5.3 produced dramatically different results depending on the coding agent around them.

**The interaction matters.**

A strong model in the wrong harness can be a worse system than the same model somewhere else.

**The eval harness matters too, but for a different reason.**

If your test environment is broken, your benchmark can confidently measure something that never happened.

**Fabricated compliance deserves explicit testing.**

An agent can produce artifacts that look correct while inventing the evidence behind them.

**Cost and compatibility belong in the result.**

A system that is correct but unusably expensive, slow, or incompatible is still not a good system for the job.

And most importantly:

## Benchmark the system you actually use

A model leaderboard is useful.

But a coding agent is more than a model endpoint.

It is a model embedded in instructions, context, tools, permissions, control loops, and implementation choices.

Those things can change the result.

So after running this matrix, I would be very cautious about saying:

> Model X is better at agentic coding than Model Y.

without adding:

> In which agent?

The full live matrix, including versions, dates, methodology, and per-cell results, is here:

**[keepthewhy.com/agent-matrix/](https://keepthewhy.com/agent-matrix/)**

It gets updated roughly monthly, plus targeted re-checks whenever a specific result needs verifying.

The concrete scores in this article refer to the Keep the Why v0.9.0 matrix snapshot tested on August 20-21, 2026. The live matrix may have newer results by the time you read this.

This is one skill, one representative cross-agent case, and a growing amount of data.

It is not enough to rank the world's coding models.

But it is enough to make me stop treating the **agent harness** as plumbing.

Related: [I Let an AI Agent Maintain My Open Source Suite for a Week - Here's What Actually Happened](https://blog.technopathy.club/i-let-an-ai-agent-maintain-my-open-source-suite-for-a-week-heres-what-actually-happened)
