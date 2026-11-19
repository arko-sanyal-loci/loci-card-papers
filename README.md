<div align="center">
<img alt="Billo Tech" src="https://raw.githubusercontent.com/arko-sanyal-loci/LOCI-MEMORY-ENGINE/main/brand/billo-tech-mark.svg" width="72">

# loci-card-papers
</div>

The published research behind **LOCI** (a heat-tiered, goal-conditioned memory architecture for
LLM agents) and **CARD** (Category-Adaptive Recall Depth, a retrieval scheme built on top of it).
Nine papers, published on [Technical Disclosure Commons](https://www.tdcommons.org/dpubs_series)
as defensive publications (one exception noted below), all by Arko Sanyal. This repo is a
publications archive — the code that implements these ideas lives in
[`LOCI-MEMORY-ENGINE`](https://github.com/arko-sanyal-loci/LOCI-MEMORY-ENGINE), the single
canonical engine repo (memory core, RAG pipeline, and the multi-agent coordination bus under
`coordination_bus/`).

## Reading order

The papers build on each other. If you're new to this, read in numeric order — later papers
assume the vocabulary and mechanisms of earlier ones (especially 07 → 08 → 09, which is one
continuous architecture, and 01 ↔ 02, which are companion papers on the same closed loop).

## Papers

| # | Title | What it contributes |
|---|---|---|
| [01](papers/01_Forecasting-Affective-State-LOCI-r+e.pdf) | Forecasting Affective State from Interaction Streams (LOCI r+e) | A closed loop that forecasts a person's 24-hour mood from the interaction stream alone — no self-report, no labels — beating a carry-forward baseline (48%) and a memoryless LLM (61%) at 74% accuracy. Introduces `MSympPrev` (a bounded, fast-decaying mood estimate derived from prediction error, not labels), a confidence gate (θ=0.82) below which the system abstains from acting on its own forecast, and a result showing that *trajectory* — not a single snapshot — is what separates a crisis from someone venting. |
| [02](papers/02_Reflection-on-Action-LOCI-e+r.pdf) | Reflection-on-Action: Label-Free Learning from Behavioral Outcomes (LOCI e+r) | The companion to 01: a mechanism that adapts purely from what a person does next, with no labels, rewards, or ratings — using how *surprising* that behavior was as the only learning signal. States the formal update rule, pre-registers a held-out non-circular experiment before running it, and argues (as a finding, not a caveat) that the same objective-agnostic mechanism becomes a manipulation engine if pointed at engagement instead of a defended wellbeing objective. |
| [03](papers/03_Separate-Fast-Decaying-Memory-Spectrum-LOCI.pdf) | A Separate, Fast-Decaying Memory Spectrum for a System's Own Thoughts, with Reflection-Gated Crystallization | Argues an agent's own reasoning traces need a different store than its observations about the world: a fast-decaying "thought spectrum" (`h(t) = h₀·e^(−λt)·(1+refs)`, ~4-hour half-life) that only becomes a permanent fact when a *later, externally-produced outcome* confirms it — an anti-hallucination gate by construction, since the model never gets to promote its own belief into ground truth. |
| [04](papers/04_Category-Adaptive-Recall-Depth-CARD.pdf) | Category-Adaptive Recall Depth (CARD) for Heat-Tiered Memory Systems, with Experimental Validation on LongMemEval-500 | The retrieval scheme this project's CARD implementation is named after: retrieval depth K and ranking strategy are conditioned on a query-category signal (information extraction, knowledge update, multi-session reasoning, temporal reasoning), each with its own (K, strategy) pair. Validated on the LongMemEval-500 external benchmark at 96.6% overall accuracy, ~14-15 points above reported GPT-4 performance, entirely on local hardware. |
| [05](papers/05_LOCI-Heat-Decaying-Goal-Conditioned-Memory-Architecture.pdf) | LOCI: A Heat-Decaying, Goal-Conditioned Memory Architecture for Long-Horizon AI Agents | The architecture paper: every fact carries a heat score that decays over time by tier (critical facts survive 30 days, routine facts 3 days, nothing is ever deleted), a circuit breaker that halts before re-processing an identical goal in a loop, and the eight-stream taxonomy (`qsymprev`, `isymprev`, `dsymprev`, `card`, `psymprev`, `ssymprev`, `msymprev`, `fsymprev`) this project's schema is organized around. Also documents the "Imagination Engine" (novel-connection synthesis between high-heat, semantically distant facts) and reports 96.6% accuracy / +18.2pp over zero-memory GPT-4 on the same LongMemEval benchmark. |
| [06](papers/06_Focused-Retention-LOCI.pdf) | Focused Retention: Goal-Conditioned Switched Decay for LOCI | Extends the heat-decay model with goal-conditioned switched decay (GCSD): a fact bound to an active goal decays at a slow "hold" rate (`λ_hold`, ~14-day half-life) while the goal is open, and switches to a fast "free" rate (`λ_free`, ~2-hour half-life) once the goal resolves — with a five-outcome goal taxonomy (fulfilled / relinquished / nullified / abandoned / reclamped) distinguishing "the goal is done" from "the goal is merely suspended." |
| [07](papers/07_LOCI-Infinity-Memory-Middleware.pdf) | LOCI ∞: Infinity Memory Middleware | Introduces the `isymprev`/`qsymprev` dual-stream split (narrative/informational vs. exact/quantifiable) this project's schema uses directly, along with the heat-propagation increments (direct access +1.0, 1-hop neighbor +0.5, 2-hop +0.25) and storage-tier thresholds (HOT > 1.5, WARM > 0.5, else COLD) that `loci-memory-engine`'s `heat.py` implements. |
| [08](papers/08_LOCI-Persistent-AI-Memory-Engine.pdf) | LOCI: A Persistent AI Memory Engine | Adds versioned exact-fact storage: a `supersedes`/`valid_from`/`valid_until` chain so a fact's history is queryable ("what did we believe as of time T"), source-trust weighting (measured > paper > user-stated, roughly), and an asymmetric conflict gate — a higher-trust source needs less confidence to overturn an existing fact than a lower-trust one does. |
| [09](papers/09_Thermal-Reasoning-Substrate.pdf) | Thermal Reasoning Substrate: A Heat-Driven Shared Memory Architecture for Parallel LLM Inference | A preprint (not TD Commons) proposing a shared, quantized entity heat-map that parallel reasoning branches read and write to instead of hard-blocking on each other's completion — replacing "wait for branch A to finish" with "wait for branch A's heat/value signal to cross a threshold." Appendix B's dual-stream SQL schema (`isymprev`/`qsymprev`, with the exact decay/increment/tier constants) is the literal base this project's `schema.sql` is transcribed from, and its `preview_query()`/`process_turn()` two-call integration pattern is the intended interface for any LLM to use the memory engine. |

## Provenance

Papers 01–08 are Technical Disclosure Commons defensive publications (TD Commons, a Diamond
Open Access venue for prior-art disclosures — publishing here places the described techniques in
the public domain and blocks anyone, including the author, from later patenting them). Each PDF's
own header carries its TD Commons article number and DOI. Paper 09 is an independent preprint,
not a TD Commons disclosure.

## Relationship to the code

This repo is documentation, not implementation. For the actual, running system built from these
ideas — with an explicit map of which paper contributed which mechanism to which module — see
[`LOCI-MEMORY-ENGINE`](https://github.com/arko-sanyal-loci/LOCI-MEMORY-ENGINE). The original
paper-to-module map (`loci-engine/INDEX.md`) and design spec (`docs/superpowers/specs/`) are kept
for reference in the archived
[`loci-memory-engine-1`](https://github.com/arko-sanyal-loci/loci-memory-engine-1).

## Citing and license

Cite a paper as: Sanyal, Arko. "<title>". Technical Disclosure Commons, (<date>).
`https://www.tdcommons.org/dpubs_series/<number>`. Verified numbers for all nine are in the
engine's [`PAPERS.md`](https://github.com/arko-sanyal-loci/LOCI-MEMORY-ENGINE/blob/main/PAPERS.md).

Papers 01–08 are CC BY 4.0, as stated on each PDF. Paper 09 states no license. See
[`LICENSE.md`](LICENSE.md).

<sub>Billo Tech · Arko Sanyal</sub>

