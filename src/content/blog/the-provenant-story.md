---
title: The Provenant Story
description: Building a license and copyright scanner fast enough to run everywhere.
kicker: The story
standfirst: Building a license and copyright scanner fast enough to run everywhere.
---

Every organization that ships software has to answer the same question, sooner or later: _what's actually inside this, and what are we allowed to do with it?_ Licenses, copyrights, dependencies, the whole provenance of the code you're about to ship. It's unglamorous work, and it's load-bearing — get it wrong and you find out at the worst possible time.

**Provenant** exists to make answering that question fast, accurate, and trustworthy enough to do continuously. It's a license, copyright, package, and SBOM scanner that runs as a single static binary with no runtime to install, an embedded license index, and bounded, predictable resource use. Fast enough to put in every CI run, lean enough to embed in a long-running service, and precise enough to trust the output.

## What makes it different

- **Fast.** Native Rust, no interpreter, no warm-up — roughly **20× faster on average** across 254 recorded workloads (19.7× median _and_ geometric mean), scaling _up_ with size: about 9× on small trees, 37× on large 10k+-file repositories, over 70× on the largest we've measured.
- **Source-faithful.** Copyright and license text is reported the way it actually appears in your files — accented names are kept as written (`Björn`, not ASCII-folded to `Bjorn`), and statements aren't silently trimmed — so what Provenant reports is what's really there.
- **Monorepo-aware.** Workspace, reactor, and multiproject layouts are understood as such: nested sources and shared lockfiles are attributed to the package that actually owns them.
- **Low-noise.** It suppresses the false-positive classes that make scan output tedious to review — code and prose bleed — and treats bare-word license mentions as clues, not conclusions.
- **Trustworthy to ship.** Every release binary and container image carries SLSA build-provenance attestation, so you can verify what you're running.

## Why we built it

Accurate license detection is a solved problem in principle — but running it _at scale_ often isn't. Teams scanning millions of files a day, embedding a scan in every pull request, or trying to run detection inside a service keep hitting the same walls: slow runs, heavy resource use, a runtime to provision and babysit. Some organizations simply can't run the tooling they'd like to on the infrastructure they have.

We wanted the same quality of answer, delivered in a form you could drop into a pipeline or a service and forget about. That meant a compiled, static, embeddable tool with a license index baked in — and a relentless focus on making the common case fast without cutting corners on correctness.

## Standing on shoulders — on purpose, and out loud

Provenant didn't start from nothing, and we've never pretended otherwise. It is a **derivative of ScanCode Toolkit**, the tool built by nexB and the AboutCode community that, over more than a decade, defined what accurate open-source license and copyright detection should look like. Provenant ports that detection engine (Apache-2.0) into Rust and builds its embedded license index from ScanCode's license dataset (CC-BY-4.0).

We treat that lineage as part of the engineering, not a footnote:

- **nexB has been credited in our `NOTICE` since Provenant's first release** — including that ScanCode is a trademark of nexB Inc.
- **ScanCode-derived source files carry dual attribution headers** with a "Derived from ScanCode Toolkit" notice.
- **Every modification to the license dataset is documented** — each rule we change, remove, or add carries a justification.
- **Each scan records which license dataset it used** and how that dataset was modified.

You don't credit a project by name in your first release as Provenant if your goal is to hide it. Building openly on excellent open-source work, with credit, is the whole point — and it's why we make Provenant's own output more transparent about its provenance than most tools bother to be.

## How it was built

Here's the part people find hard to believe: most of Provenant's 300,000-plus lines of Rust were written by AI agents, directed by a small human team — the bulk of it in about four months. Not in a chatbot — inside tight feedback loops where golden tests judged detection quality against real-world corpora and benchmarks judged speed, so that hard data, not guesswork, drove every change. The agents supplied the throughput; the loop supplied the trust. That method turned out to be a story in its own right — we tell it in [Engineering the Loop](/blog/engineering-the-loop/).

## An open invitation

Provenant is open source under Apache-2.0. If you work in SBOM, compliance, or open-source governance, we'd genuinely like you to put it through its paces — and to tell us where it's wrong. File an issue, open a discussion, send us a repo that breaks it. Scrutiny from the compliance community is exactly what makes a tool like this trustworthy, and we're building it in the open for that reason.

```sh
brew install getprovenant/tap/provenant
provenant scan . --license --package --copyright --json-pp -
```

<span class="foot-note">Provenant is open source under Apache-2.0. Try it: [github.com/getprovenant/provenant](https://github.com/getprovenant/provenant).</span>
