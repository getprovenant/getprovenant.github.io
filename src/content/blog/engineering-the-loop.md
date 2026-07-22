---
title: Engineering the Loop
description: How we built Provenant — 300,000+ lines of Rust — mostly with AI agents, and what that actually takes.
kicker: Engineering
standfirst: How we built Provenant — 300,000+ lines of Rust — mostly with AI agents, and what that actually takes.
date: 2026-05-22
---

Provenant is a license, copyright, and SBOM scanner — today more than 300,000 lines of Rust — reimplementing the detection behavior of a mature Python tool (ScanCode) that took a decade to build. A small human team built the bulk of it in about four months, mostly by directing AI agents, for around $20,000 in tokens.

Those numbers are easy to misread as "AI wrote our software for us." That's not what happened, and the gap between what people imagine and what it actually took is the interesting part.

Here's the one-line version of what we learned: **agents are a throughput engine, not a trust engine.** A capable agent will produce an enormous amount of plausible code very quickly. Almost none of that is trustworthy on arrival. The entire craft is building the system _around_ the agent that decides what to keep — and at scale, that system is a closed feedback loop. The senior engineering work shifted from _writing the code_ to _engineering the loop that writes the code._

## What "the loop" is

A loop is a closed system where the agent acts, observes the result, and corrects itself — without a human in every cycle. You start it, and it grinds toward a target you've defined in machine-checkable terms.

A loop that actually converges has three ingredients:

1. **Deterministic feedback.** The compiler, a golden-test suite, benchmark scripts — anything that returns a hard, unambiguous verdict. This is the load-bearing part. Rust helped enormously here: a strong type system and a strict compiler mean a large fraction of "plausible but wrong" gets rejected for free, before any test runs.
2. **Soft guidance.** Prompt-level direction on _how_ to approach the goal — the conventions, the gotchas, the preferred shape of a solution.
3. **Living documentation.** Notes the loop reads and writes, so knowledge persists across iterations instead of being re-derived (and re-hallucinated) every cycle.

Notably, we did **not** build an elaborate framework on top of the coding harness. The leverage came from the quality of the feedback, not the sophistication of the scaffolding.

## A loop that worked: license-detection convergence

License detection was the hardest surface, and the clearest example.

We started with an implementation that didn't pass. Then we introduced a golden-test suite — real files with known, correct detections — and wired the scan against it. Out of the box, after some setup and a first pass at performance, it already passed ~70% of goldens. Then we pointed a loop at the gap and let it run with minimal supervision.

What came out was almost a textbook convergence curve — roughly exponential, with a half-life of about four and a half days. The **first ~20% of commits fixed ~80% of the failing tests**; the long tail was the expensive part. When progress stalled, the highest-leverage move wasn't a cleverer prompt — it was swapping in a stronger model and letting the same loop keep running.

And convergence has a price you can read straight off the token bill. Reaching a passing state cost **tens of thousands of tokens for every line that ended up in the tree** — because the loop gets there by generating candidates and letting the tests throw most of them away. That ratio isn't waste; it _is_ the mechanism. You don't reason your way to a fixed point, you iterate to it, and the iteration is what you pay for.

## Why a loop beats a conversation, at scale

Two things surprised us.

**It fights sycophancy with facts.** In a chat, a model will happily agree that your broken idea is brilliant. A loop doesn't care what anyone thinks — the golden tests either pass or they don't. Hard ground truth is immune to flattery.

**Creativity — even hallucination — becomes an asset.** In a conversation, a confidently wrong answer is the cardinal sin. Inside a loop with ground truth, it's cheap fuel. Instead of carefully reasoning out the one true cause of a bug, the agent can _imagine several_ and check them all in parallel; one hit is enough, and the wrong guesses cost nothing because the tests catch them. This is especially powerful with cheaper, weaker models — the loop turns their volume into progress.

## What breaks

This is the part the hype skips. A running loop is not a fire-and-forget machine.

- **Drift.** What you don't legislate, the agent invents. Leave a gap in your constraints and it fills it — sometimes usefully, sometimes with sprawling files so large the agent then struggles to edit them.
- **Slow feedback on loop health.** The loop optimizes its target; it will not tell you when _it_ is sick. Noticing that a loop has quietly gone off the rails is the human's job, and the signal is often slow.
- **Stuck loops don't self-recover.** A task that's too hard produces an agent that resets its own progress, thrashes through compactions, or runs away. Left alone, it burns tokens going nowhere.
- **The 24/7 trap.** Once a loop works, you have a coding machine that runs around the clock — and wants to be maintained around the clock. It becomes an on-call problem: you check in at night and find it broke hours ago.

The real bottleneck, by the way, was rarely the model. It was local compute — building and running the code fast enough to keep the loop's iteration time short.

## What this does to the craft

If you squint, loop engineering turns writing software into an optimization problem: the loop's constraints are the cost function, and the agent is the optimizer. The license-detection curve is exactly that — an optimizer descending toward a target.

But it only optimizes what you can _define_. Everything you can express as a deterministic check, a loop will relentlessly improve. Everything you can't — the parts of "good code" nobody has ever managed to reduce to a metric — stays exactly as hard as it always was. The old problem didn't disappear; it just became the whole job.

So the work isn't eliminated. It's transformed — into prompting, code generation, test generation, integration, failure analysis, retries, context reloading, and iterative refinement until the behavior is right. It also _feels_ different: less like writing, more like running a main base and a dozen satellite bases at once, deciding where attention is scarce. Some days it's two prompts and a lot of watching.

> Spec-driven development is the waterfall of the agentic age. You don't define everything upfront and hand it down — you feel the product as it takes shape and keep cutting, more like shaping something out of a block of marble than pouring it into a mold.

Cheap iteration is what makes that possible: a suboptimal early decision is now often cheaper to fix later than to agonize over upfront.

None of this made software free or instant. The effort didn't vanish — its _unit_ changed, from person-months to tokens plus iteration time. What we actually bought with that trade was a roughly one-to-two-orders-of-magnitude compression of both cost and calendar time. That's not the "AI writes your app" fantasy. It's something more useful, and more demanding: a new kind of engineering, where the thing you build best is the loop.

<span class="foot-note">Provenant is open source under Apache-2.0. If you want to see what the loop produced: [github.com/getprovenant/provenant](https://github.com/getprovenant/provenant).</span>
