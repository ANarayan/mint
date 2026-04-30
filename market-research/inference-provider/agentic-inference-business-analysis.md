# Agentic Inference — Market, Buyer & Unit Economics for the Second Wedge

*A working analysis of agentic AI inference as the second product positioning for our hard-locked Qwen 3 chip. Where `batch-inference-business-analysis.md` covers archetype 1 (high-throughput batch), this document covers archetype 3: agentic workloads — code agents, research agents, customer-service agents, browser/computer-use agents, voice agents — that consume 10–100x more tokens per user-action than chat and represent the fastest-growing inference category in 2026. Sections that overlap with the batch analysis (compliance posture, US-DC choice, sales-team build-out) are referenced rather than repeated.*

> Companion to `batch-inference-business-analysis.md` (archetype 1) and `tokens-too-cheap-to-meter-thesis.md` (the cost-curve framing). Read those first; this document focuses on what's distinctive about agentic as a workload class and a market opportunity.

---

## 1. What "Agentic Inference" Actually Means

**Q: Define the workload tightly. What makes agentic inference different from batch and from chat?**

### The Operating Point

Agentic workloads sit between chat (interactive, latency-critical, low-token) and batch (offline, throughput-only, high-token). The defining properties:

- **Token-explosive per user-action.** A single user request ("fix this bug", "research this topic", "process this customer") triggers 100K–10M+ tokens of LLM consumption across many turns.
- **Multi-turn, stateful.** 5–100+ LLM calls per task. Each turn re-uses context built up in prior turns. KV cache is the load-bearing data structure.
- **Long context.** 50K–1M+ tokens of input per turn (full code repos, document corpora, tool-call traces, conversation histories).
- **Tool-using.** Agents call code, search, browse, write files, hit APIs. Each tool call is part of the loop.
- **Latency-tolerant per-task, latency-sensitive within-task.** Users wait minutes for an agent to finish — but per-token speed matters because faster tokens = faster turns = faster task completion.
- **Prefill-heavy.** Each turn re-reads context, so prefill (input processing) is often 60–90% of compute; decode (output generation) is the smaller fraction.

### What Counts as Agentic in 2026

Concrete workload categories ranked by current token volume:

| Workload | Tokens per task | Volume estimate (2025) |
|---|---|---|
| **Code agents** | 500K–5M | Largest category — Cursor/Anysphere, Cognition, Replit, Codeium, Cline |
| **Research agents** | 1M–10M | Perplexity, Glean, custom enterprise — smaller volume but growing fast |
| **Customer-service agents** | 100K–1M per conversation | Sierra, Decagon, Cresta, Ada — large volume, mid-size per-task |
| **Voice agents** | 50K–500K per call | Bland, Vapi, Retell, Synthflow — emerging |
| **Browser / computer-use agents** | 1M–20M per task | Anthropic Computer Use, OpenAI Operator — token-explosive but small user base today |
| **Sales / outbound agents** | 200K–2M per workflow | 11x.ai, Artisan, Regie.ai |

The unifying property: **agents burn tokens to produce thinking, planning, and tool-using output that wasn't economical at chat-class pricing.**

### Why Agentic Deserves Its Own Analysis

> An agent doesn't fit batch or chat. It needs both: long-context prefill (batch-like) plus per-stream latency (chat-like). The HW economics, the buyer set, the pricing model, and the GTM motion all sit between the two extremes.

Specifically:
- **Batch-only HW** (massive throughput, slow per-stream) doesn't serve agents because per-token latency is too high.
- **Chat-only HW** (low TTFT, small batch sizes) doesn't serve agents because token volume per task crushes per-call economics.
- **A hard-locked Qwen chip with 17k tokens/sec/user** is purpose-fit: extreme per-stream throughput plus the cost basis to absorb 10–100x per-task token volume.

---

## 2. Market Size — Today and Through 2030

**Q: How big is the agentic inference market today, and what's the credible 2030 number?**

### 2025 Baseline

Public and triangulated:
- **Total agentic inference spend (2025):** ~$5–15B globally. Composed of:
  - Frontier-API agentic spend (OpenAI o-series, Claude agents, Gemini agents): $3–8B.
  - Open-weight self-hosted agent backends (Cursor hybrid, Sierra hybrid, etc.): $1–3B.
  - Hyperscaler agent services (Bedrock Agents, Vertex AI Agent Builder, Azure AI Foundry): $1–4B.
- **Growth rate:** 200%+ YoY. Anthropic public statement: agents are the single fastest-growing category. Cursor reached $200M+ ARR in <18 months; Sierra valued at multi-billion.

### Why It Grows Faster Than Other Inference Categories

Three structural drivers:

1. **Token consumption per user-action is 10–100x chat.** As agentic UX becomes the default (vs. one-shot chat), per-user spend explodes proportionally.
2. **Agents are getting better fast.** Closed-frontier reasoning (o-series, Claude Opus reasoning) keeps unlocking new tasks; open-weight reasoning (Qwen3-reasoning, DeepSeek-R1) is closing the gap on most tasks. Better agents drive more deployment.
3. **Cost reduction is a flywheel.** Cheaper agent tokens → more agents per task → deeper recursive agent hierarchies → more aggressive deployment.

### 2030 Projection

| Scenario | 2030 Total Agentic | Driver |
|---|---|---|
| Conservative | $30B | Frontier dominates; open-weight stays specialty |
| Base | $80B | Open-weight catches up on reasoning; agents go mainstream in F500 |
| Aggressive (expand-the-pie) | $200–300B | Cost-curve unlock makes "agents on every interaction" economic |

**Anchor for the deck: $80–150B addressable agentic inference market by 2030.** This is *bigger* than the batch market ($45B base case in `batch-inference-business-analysis.md` §2). Agentic is the higher-growth inference category.

### What's Addressable to Us Specifically

Our hard-locked Qwen 3 chip can serve the 60–70% of agent workloads where "open-weight is good enough":
- File reads, simple tool use, mechanical agent steps (most coding-agent token volume).
- Customer service routine handling (most CS-agent token volume).
- Document processing within an agent loop.
- Voice agent transcription and routine response generation.

We don't compete (year 1) on:
- Complex multi-step reasoning that requires frontier (the "hard 30%").
- Highly novel multimodal agents.
- Anything that requires Anthropic / OpenAI brand trust.

**Year-1 addressable share of $80B 2030 market: ~50–60% = $40–50B.** Realistic capture rate at maturity: 10–20% = $4–10B agentic ARR. **Stacked on top of batch ARR.**

---

## 3. The Buyers — Who Actually Pays for Agentic Inference

**Q: Under hard-lock + Qwen + US-DC, who buys agentic inference at our price point?**

### How Agentic Reshapes the Buyer Set

Compared to batch (`batch-inference-business-analysis.md` §3), agentic introduces a new wedge tier and shifts priorities:

| Buyer tier | Batch fit | Agentic fit | Why the difference |
|---|---|---|---|
| **G — AI agent product companies (NEW)** | n/a | **Year 1 wedge** | Inference COGS is their largest variable cost; pre-IPO margin pressure forces lowest-cost provider |
| **C — Vertical AI platforms** | Year 1 wedge | **Year 1 wedge** | Same dynamic; some Tier C are also Tier G (Harvey, Hebbia, Glean run agents) |
| **D — US single-workload enterprise** | Year 2 | **Year 1–2** | F500 internal agents are real and growing; one big segment per agent type |
| **B — Hyperscaler marketplace** | Year 2 | **Year 2** | Hyperscaler agent marketplaces (Bedrock Agents, Vertex Agent Builder, AI Foundry) need cheap-token backends |
| **A — Foundation labs** | Largely out | **Niche** | Labs running agentic experiments may use us for non-frontier-required portions |
| **E — Sovereign** | Year 3+ | Year 3+ | Same as batch (international DC required) |
| **F — Federal** | Year 3+ via Gemma | Year 3+ via Gemma | Same as batch (Gemma SKU required) |

The big new tier: **Tier G — AI Agent Product Companies.** This is the wedge for agentic.

### Tier G — AI Agent Product Companies (the agentic wedge)

**Why they're the wedge:**
- **Inference is their dominant COGS line.** Cursor/Anysphere reportedly ~70% of OpEx is inference. Sierra similar. They will switch providers in a week to save 50%+.
- **Most are venture-funded; cost pressure ahead of fundraise/IPO.** "Path to gross margin" is the #1 board topic post-Series C.
- **Technically sophisticated.** Bottoms-up engineering eval; they will benchmark our endpoint within 24 hours of access.
- **Public reference value compounds.** Cursor moves to us → every other coding-agent company evaluates within a month.
- **Already running open-weight for parts of their stack.** Most use a model mix (Sonnet for hard reasoning, Haiku/Qwen for mechanical steps). We replace the mechanical-steps backend, not the reasoning backend.

**Names that matter:**
- **Coding agents:** Cursor (Anysphere), Cognition (Devin), Replit, Codeium, Aider, Cline, Continue.
- **Customer-service agents:** Sierra, Decagon, Cresta, Ada, Forethought, Ema.
- **Voice agents:** Bland, Vapi, Retell, Synthflow.
- **Sales / outbound agents:** 11x.ai, Artisan, Regie.ai.
- **Browser / computer agents:** various early-stage.
- **General-purpose agent platforms:** Crew.ai, LangGraph (LangChain), AutoGen-based platforms.

**Spend bracket:**
- Top tier (Cursor, Sierra, Cognition, Decagon): $50–200M annual inference COGS each.
- Mid tier (Replit, Codeium, Cresta): $10–50M annual.
- Emerging (voice, browser agents): $1–10M annual.

**Total Tier G addressable today: ~$3–8B in agentic inference; growing 200%+ YoY.**

**Contract size:** $1–25M ACV at signature, scaling to $20–100M+ at maturity.
**Sales cycle:** 4–10 weeks (faster than batch Tier C — cost pressure is more acute, technical evaluation is faster, decision authority is concentrated at CTO).

### Other Tiers (Brief)

**Tier D — US enterprise internal agents (year 1–2):** every F500 has an "internal copilot" or "agent" project. Cost ceiling of cloud APIs is the binding constraint on scaling. We sell through hyperscaler marketplaces. Cycle 6–12 months. Persona similar to batch §9 Tier D, but the internal sponsor is typically Chief Digital Officer or VP AI rather than VP Data.

**Tier B — Hyperscaler agent-platform backends (year 2):** AWS Bedrock Agents, Azure AI Foundry, GCP Vertex AI Agent Builder. These platforms host agent applications and need cheap-token backends. We list as "lowest-cost agent inference provider on [marketplace]." Same motion as batch §9 Tier B with agent-specific positioning.

**Tier C overlap with G:** some vertical AI platforms (Harvey, Hebbia, Glean) are *also* agent product companies. We sell them on both batch and agentic capacity simultaneously — natural cross-sell.

**Tier A, E, F:** same as batch — labs largely out, sovereign year 3+ via international DC, federal year 3+ via Gemma SKU.

### Wedge Recommendation

> **Land 2–3 Tier G AI agent product companies as wedge customers in months 0–18 (pre-ship via design-partner agreements). Cursor/Anysphere is the marquee target; Sierra-class CS agents and Cognition-class coding agents round out the design-partner cohort. Use them as public reference to unlock Tier D enterprise internal-agent deployments and Tier B agent-marketplace listings in year 2.**

---

## 4. Competitive Landscape — Why Agentic Isn't Already Won

**Q: Agentic inference is the fastest-growing inference category. Why is there room for a new specialized provider?**

### The Five Cohorts and Why Each Falls Short

**1. Closed-frontier providers (OpenAI, Anthropic, Google).**
- **Pricing:** Claude Opus 4.5 $15/$75; Claude Sonnet 4.5 $3/$15; GPT-5 ~$3/$10; Gemini 2.5 Pro $2.50/$10.
- **Why limited on cost:** running closed-weight models on H100/B200/TPU. Cost basis includes NVIDIA/Google silicon plus model R&D amortization. **Cannot price agentic inference below ~$1/M output without losing money** even at Haiku-class.
- **Their moat is reasoning quality and tool-integration ecosystem**, not cost. They will own the "hard 30%" of agent tasks for years.

**2. Open-weight inference clouds (Together, Fireworks, DeepInfra) on agentic.**
- **Pricing:** Qwen-3-72B at $0.30–0.90/M output. Llama-4 similar. DeepSeek-V3.1 at $0.27/M.
- **Why limited:** running on NVIDIA H100/B200. Same cost ceiling as everyone else on GPUs. Cannot get below ~$0.20/M output regardless of optimization.
- **They are our natural marketplace partners** — they have engineering relationships with Tier G companies but can't compete on cost.

**3. Specialized inference HW (Cerebras, Groq, SambaNova, Etched).**
- **Cerebras WSE-3:** Strong per-stream throughput but $2M+ per system. Cost-per-token at agentic volumes uneconomic.
- **Groq LPU:** Optimized for chat-class TTFT. Bad fit for prefill-heavy agentic workloads.
- **Etched Sohu:** Transformer ASIC, model-flexible. Closest analog architecturally but doesn't hard-lock to a specific model — leaves perf/W on the table.
- **Gap:** none have packaged "agentic capacity at fleet scale on a hard-locked open-weight model with multi-turn KV cache management."

**4. Hyperscaler agent platforms (AWS Bedrock Agents, Azure AI Foundry, GCP Vertex AI Agent Builder).**
- Reselling frontier APIs at small markups, plus first-party silicon (Trainium, TPU, Maia).
- **Cost structure inherits from NVIDIA + frontier vendor.** Same floor.
- **They own the customer relationship.** Distribution moat — but they're our marketplace partner, not direct competitor.

**5. Self-hosted on commodity GPUs.**
- Big agent companies (Cursor reportedly hybrid; Sierra at scale) run their own H100 clusters with vLLM or SGLang for the open-weight portion.
- At scale: ~$0.20–0.30/M output for Qwen-3-72B class.
- **Structural ceiling:** still NVIDIA. Same chip, same memory, same power.

### The Whitespace

> Nobody offers **first-party agentic capacity at fleet scale, on co-designed hard-locked silicon, with serving-stack optimizations specifically tuned for multi-turn long-context agent loops.** That's the gap.

The closest historical analog: AWS to enterprise SaaS in 2010–2015. When AWS made compute cheap enough, the agent / AI-application layer moved off self-hosting onto hyperscaler infra. We are AWS-for-agents-running-on-Qwen — the cost-curve enabler.

---

## 5. Why Specialized HW Wins on Agentic (The Architectural Argument)

**Q: Walk through, from first principles, why hard-locked Qwen beats NVIDIA on agentic — and how durable that lead is.**

### Agentic Inference Is a Memory + Prefill Problem

For a Qwen-3-72B-class agent loop with 200K–1M-token context:
- **Decode (output) compute:** ~140 GFLOPS per token at FP8. Memory-bandwidth-bound, not compute-bound.
- **Prefill (input) compute:** ~140 TFLOPS per 1M-token prefill. Compute-bound but parallelizable.
- **KV cache:** 200K context = ~30 GB per stream at FP8. 1M context = ~150 GB.
- **KV cache reuse across turns:** if turn 2 reuses turn 1's prefix, KV must persist across the agent session. **This is the hard problem.**

An H100 SXM has 989 TFLOPS (FP16) and 80 GB HBM3. The chip is **compute-overprovisioned for decode and memory-underprovisioned for KV cache** in the agentic context. You spend HBM on weights (140 GB for 72B FP8) and have little left for KV cache; multi-stream long-context agents OOM fast.

### What a Hard-Locked Agentic Chip Optimizes

Architectural levers:

1. **Weights on-die (SRAM or embedded).** Frees all HBM for KV cache. Critical for long-context multi-turn.
2. **Specialized KV cache memory hierarchy.** Fast SRAM for hot KV; HBM for warm; DRAM for cold. Multi-tier KV management is the agentic killer feature.
3. **Prefix-sharing across turns.** When turn N reuses turn N-1's prefix, only the delta needs prefill. Hard-lock chip can implement this in silicon. NVIDIA does it in software with overhead.
4. **Per-stream throughput at scale.** 17k tokens/sec/user means a 1000-token agent response completes in 60ms. A 10-turn agent task with 200K-token aggregate output completes in ~12 seconds. **That's 10x faster than what's possible on H100.**
5. **Power tuned for sustained per-stream throughput**, not peak interactive bursts.

NVIDIA can't easily make these trades because **the same chip has to win training too.** Training demands flexible memory + max FLOPS. Hopper, Blackwell, Rubin are all training-capable compromises that leave money on the table for prefill-heavy agentic workloads.

### Durability of the Lead

3–5 year window before NVIDIA Rubin (2028–2029) closes most of the perf/W gap with custom inference SKUs. But the *architectural* advantages of hard-lock (weights on-die, model-specific KV management) require giving up training flexibility — a trade NVIDIA structurally cannot make.

> **Hard-lock + weights on-die is a permanent architectural advantage on Qwen specifically. NVIDIA can close perf/W; they cannot replicate the architectural fit without taping out a model-specific chip themselves — which they won't, because their business is breadth.**

### Concrete Performance Targets for Agentic

| Metric | NVIDIA H100 (today) | Required from us | Stretch |
|---|---|---|---|
| Tokens/sec/user on Qwen-3-72B w/ 200K context | 30–60 | **17,000** (per user spec) | — |
| KV cache supported per stream | 30–80 GB (max ~3 long streams) | 150+ GB (32+ streams) | 1M context, 64+ streams |
| Prefix-cache hit latency reduction | 0–30% | 80%+ | 95%+ |
| End-to-end agent task latency (10-turn, 5M tokens total) | 2–5 minutes | 10–30 seconds | <10 seconds |

`[Numbers TBD pending internal HW validation; calibrated against the user-supplied 17k tok/sec/user baseline plus agentic-specific assumptions.]`

---

## 6. Unit Economics — Does This Print Money on Agentic?

**Q: Re-do the §6 cost / pricing math for agentic. How does it differ from batch?**

### Cost Stack: Same Chip, Slight Workload Adjustment

The cost stack from `batch-inference-business-analysis.md` §6 applies — same chip, same DC, same compliance posture. Cost basis: **$0.016/M output tokens at the mid-scenario (17 tok/sec/W).**

**One adjustment for agentic:** prefill-heavy workloads use compute differently from batch. Effective utilization may be 5–15% lower because of KV cache management overhead and turn-boundary stalls. Adjusted agentic cost basis: **~$0.018–0.020/M output, $0.008–0.010/M input.**

### Competitive Pricing Landscape (2026) — Agentic Tier

| Provider / Model | Output $/M (list) | Output $/M (batch) | Notes |
|---|---|---|---|
| **Closed-frontier reasoning** | | | |
| Claude Opus 4.5 | $75 | $37.50 | Hardest reasoning agents only |
| Claude Sonnet 4.5 | $15 | $7.50 | Default for serious agent backends |
| GPT-5 | ~$10 | ~$5 | OpenAI agent default |
| Gemini 2.5 Pro | $10 | $5 | Google agent default |
| **Closed cheap tier** | | | |
| Claude Haiku 4.5 | $5 | $2.50 | Mechanical agent steps |
| GPT-5-mini | $0.60 | $0.30 | Volume tier |
| Gemini 2.5 Flash | $0.40 | $0.20 | Volume tier |
| **Open-weight Qwen-3-72B (agent backend)** | | | |
| Together / Fireworks | $0.88 / $0.90 | $0.44 / $0.45 | Premium DX |
| **DeepInfra** | **$0.40** | **$0.20** | **Cheapest open-weight** |
| Self-hosted on H100 | n/a | $0.20–0.30 | Cursor / Sierra hybrid |

**The agent customer's effective blended price today** (mix of Sonnet for reasoning + Haiku/Qwen for mechanical) typically lands around $1–3/M output across the full agent loop. **Our pricing target: $0.05–0.15/M output for the open-weight portion (60–70% of token volume).**

### Margin Sensitivity — Agentic

| Output price | Margin @ $0.018 cost (agentic mid) |
|---|---|
| $0.30 (retail) | 94% |
| $0.20 (match Gemini Flash) | 91% |
| $0.15 (clear undercut) | 88% |
| $0.10 (Tier G anchor) | 82% |
| $0.06 (demolish) | 70% |
| $0.04 (Tier B wholesale) | 55% |
| $0.03 | 40% |
| $0.02 | 10% |

Same conclusion as batch: **every realistic price tier prints money.** Slightly tighter margins than batch because of agentic prefill overhead, but still healthy.

### Recommended Agentic Pricing Card

| Tier | Input $/M | Output $/M | Margin @ $0.018 cost |
|---|---|---|---|
| Retail / on-demand | $0.10 | $0.20 | 91% |
| Tier G reserved (1-yr) | $0.05 | $0.10 | 82% |
| Tier G dedicated (3-yr) | $0.03 | $0.06 | 70% |
| Tier D enterprise via marketplace | $0.04 | $0.08 | 78% |
| Tier B hyperscaler wholesale | $0.02 | $0.04 | 55% |

Why agentic pricing skews slightly higher than batch on the same chip: **agent customers value per-stream throughput (17k tok/sec/user) more than batch customers do.** They pay a premium for fast agent task completion even though the underlying $/M cost is the same.

### The Per-Task Pricing Flip — The Headline Slide

The most useful framing for the Tier G pitch isn't $/M tokens — it's **$/agent-task.** A typical 5M-token agent task (4M input + 1M output, e.g. a complex coding task or research synthesis):

| Provider | Input $/M | Output $/M | Cost per 5M-token agent task |
|---|---|---|---|
| Claude Sonnet 4.5 (list) | $3 | $15 | **$27** |
| Claude Sonnet 4.5 (batch) | $1.50 | $7.50 | $13.50 |
| Claude Haiku 4.5 (list) | $1 | $5 | $9 |
| GPT-5-mini (batch) | $0.13 | $0.30 | $0.82 |
| Gemini 2.5 Flash (batch) | $0.15 | $0.20 | $0.80 |
| Qwen-3-72B DeepInfra (batch) | $0.10 | $0.20 | $0.60 |
| **Us (Tier G anchor)** | **$0.05** | **$0.10** | **$0.30** |
| **Us (Tier G dedicated)** | **$0.03** | **$0.06** | **$0.18** |
| **Us (cost basis)** | **$0.009** | **$0.018** | **$0.054** |

> **An agent task that costs $27 on Claude Sonnet costs $0.30 on us. That's 90x cheaper.** Customers don't switch for 30% — they switch for 30x. This is the headline.

A Cursor-class agent company doing 1B coding tasks/year goes from $27B/yr at Sonnet (impossible) to $300M/yr at our anchor — and at our cost basis, $54M/yr. That's the difference between a financially nonviable product and a healthy gross-margin business.

---

## 7. SKU Strategy — Same Chip or Separate?

**Q: Does agentic require its own tape-out, or does the Qwen-3 batch chip serve both?**

### The Case for Same Chip (Year 1)

- Same model (Qwen-3-72B) — no SKU difference.
- 17k tokens/sec/user spec is excellent for agentic per-stream throughput.
- KV cache management is largely a software/scheduler problem (we tune the runtime per workload).
- One SKU = simpler operations, faster time-to-market, lower NRE risk.
- **No reason to split silicon spend in year 1** when both workloads run profitably on the same chip.

### The Case for a Separate Agentic SKU (Year 3+)

- **Bigger KV cache memory hierarchy** — agentic needs 5–10x more KV memory than batch. A dedicated "Qwen-Agent" SKU could have 200+ GB SRAM and 500+ GB HBM dedicated to KV.
- **Prefill-prioritized compute layout** — agentic is prefill-heavy; batch is decode-heavy. Different optimal datapaths.
- **Reasoning-model variant** — Qwen3-235B-A22B (reasoning) is meaningfully better at agent tasks than Qwen3-72B (chat). A separate chip locked to a reasoning Qwen variant might be the right move.
- **Different power envelope** — agentic prefill bursts higher than batch sustained throughput.

### Recommendation

> **Year 1: same chip serves both batch and agentic, with software/scheduler tuned per workload class.** Cost basis is the same ($0.016–0.020/M); pricing differs by product positioning (Tier G anchor at $0.10/M output vs. Tier C batch anchor at $0.08/M).

> **Year 3+ (parallel with Qwen-N refresh tape-out): tape out a dedicated "Qwen-Agent" SKU optimized for prefill-heavy long-context multi-turn workloads, locked to whichever Qwen reasoning variant wins by then.** This is the second-product-line bet.

This phasing means we don't bet $300M on agentic NRE before validating market fit. Tier G design partners (Cursor / Sierra / Cognition / Decagon / Bland) on the year-1 batch chip validate demand. Year-3 NRE is committed against contracted revenue from year 1–2.

### What This Means for the Pitch

> **We sell the same silicon to two distinct buyer sets at two different price tiers, doubling our addressable market without doubling our R&D burn. Year 3 we tape out a dedicated agentic SKU funded by year 1–2 revenue from both segments.**

That's the leverage story for Series C/D investors.

---

## 8. GTM Sequencing — The First 60 Months for Agentic

**Q: What does the agentic GTM sequencing look like alongside the batch sequencing?**

### Phase 1 — Months 0–18: Pre-Ship (Tier G Design Partners)

Run parallel to batch design partners:
- **2–3 Tier G design-partner agreements.** Selected from Cursor/Anysphere, Sierra, Cognition, Decagon, Replit, Bland, Codeium. No cash up front; small refundable deposit ($250K–$1M); guaranteed agentic-tier capacity allocation; co-marketing rights at GA.
- **Workload trace exchange:** each Tier G partner shares an agent workload trace; we run on simulator and demonstrate $/M-token + per-stream throughput.
- **Agent-specific compliance work:** beyond batch's SOC 2 / ISO 27001, we add tool-call audit logging, agent-action traceability, sandbox execution environment. These are agent-specific compliance asks.

### Phase 2 — Months 18–30: First Ship, Tier G Live

- 2–3 Tier G design-partner customers in production.
- Per-stream throughput demonstrated publicly: "Cursor's agent loop completes a 5M-token task in 30 seconds on us vs. 5 minutes on Sonnet."
- 5–10 additional Tier G logos signed (network effect from initial references).
- First Tier D enterprise internal-agent deployment via hyperscaler marketplace.
- **Agentic ARR by month 30: $20–60M** (on top of batch's $25–75M).

### Phase 3 — Months 30–42: Scale + Marketplace + Qwen-Agent Tape-Out

- 15–25 Tier G customers; 3–5 Tier D enterprise agent deployments live.
- Hyperscaler agent-marketplace listings (AWS Bedrock Agents, Azure AI Foundry, GCP Vertex AI Agent Builder).
- **Qwen-Agent SKU tape-out begins** (~month 30, ships ~month 54).
- **Agentic ARR by month 42: $150–400M.**

### Phase 4 — Months 42–60: Multi-SKU Agentic + International

- Qwen-Agent SKU ships (~month 54). Reasoning-tuned, prefill-optimized, multi-turn KV hierarchy.
- International DC online; Tier E sovereign agent deployments emerge (sovereigns building national agent platforms).
- Federal agent platforms via Gemma-Agent variant tape-out (year 4+).
- **Agentic ARR by month 60: $800M–$1.5B** (on top of batch ARR of $1–2B).

### Combined ARR Path (Batch + Agentic)

| Month | Batch ARR | Agentic ARR | Combined |
|---|---|---|---|
| 30 | $25–75M | $20–60M | $45–135M |
| 42 | $200–500M | $150–400M | $350M–$900M |
| 60 | $1–2B | $0.8–1.5B | **$1.8–3.5B** |

> **Adding agentic doubles the company's revenue ramp without proportionally adding capex** (same silicon, same DC, marginal compliance investment). This is the "two products one chip" leverage that justifies a higher Series C/D valuation than batch-only would.

---

## 9. Buyer Persona — Tier G AI Agent Product Companies

**Q: Tier G is the new wedge. Who are they, what do they buy today, and what flips them?**

(Tier C, D, B, E, F personas largely overlap with batch — see `batch-inference-business-analysis.md` §9. This section focuses on Tier G.)

**The buyer.** CTO at sub-200-person companies (Cursor/Anysphere CTO, Sierra CTO, Cognition CTO); VP Eng or Head of ML Infrastructure at 200–500-person; CFO involved at scale. Reports to CEO at startups, CTO at growth-stage. Decision room: 1–3 for technical eval, expanding to 5+ at $10M+ ACV.

**What they buy today.**
- **Closed-frontier APIs for "hard" agent steps:** Claude Sonnet 4.5, Claude Opus 4.5, GPT-5, Gemini 2.5 Pro. ~30% of token volume but ~70% of inference COGS.
- **Open-weight inference clouds for "easy" steps:** Together, Fireworks, DeepInfra hosting Qwen-3 / Llama-4. ~70% of token volume, ~30% of COGS.
- **Self-hosted at scale:** Cursor reportedly hybrid; Sierra similar at maturity.

**KPIs they're measured on.**
- **Inference COGS as % of revenue.** Pre-IPO target: <50%. Post-IPO: <30%. Currently 60–80% at most agent companies.
- **Per-task inference cost.** Cursor cares about $/coding-task; Sierra cares about $/CS-conversation.
- **Agent-task latency p95.** Faster agents = better UX = retention.
- **Gross margin.** Inference is the main margin lever.

**Budget authority.**
- CTO: up to $5M/yr.
- CEO/CFO: $5–25M.
- Board: >$25M.

**Trigger to switch.**
1. **Fundraise prep.** Every Series C/D deck has a "path to gross margin" slide. Inference COGS is the single biggest lever.
2. **Specific anchor customer contract** that's marginal at current COGS.
3. **A competitor launches at lower price** ($5/seat vs. $20/seat coding agent — driven by inference cost).
4. **API rate limits or capacity constraints.** Frontier providers throttle them at peak.
5. **Latency complaints from users.** Faster agents need faster inference.

**What they hate.**
- Frontier API price unpredictability. Quarterly price changes wreck financial planning.
- "Mixture of model" complexity — tracking which calls go to Sonnet vs. Haiku vs. Qwen across a 50-step agent loop.
- Being locked into Anthropic / OpenAI for capacity.
- Slow per-stream throughput on commodity GPUs.

**Why hard-lock to Qwen doesn't scare them.**
- They already use Qwen for parts of their agent loop.
- They keep Sonnet/Opus for hard reasoning; we replace the mechanical-step backend.
- Single-model lock simplifies their LLM-routing logic (one less variable in the routing table).

**How to reach them.**
- **Founder-led outbound to CTO.** The agent-company CTO ecosystem is small (~200 people globally); 1 hour of LinkedIn outreach yields 5 meetings.
- **Public benchmark posts:** "Cursor's agent loop runs 10x faster on our endpoint."
- **AI Engineer events:** AI Engineer Summit, AI Engineer World's Fair, LangChain Interrupt.
- **VC warm intros:** Sequoia / a16z / Founders Fund portfolio overlap with most Tier G companies.

**The pitch.**
> "Cut the open-weight portion of your agent's inference COGS by 5–10x. Native Qwen-3 hard-locked silicon, 17k tokens/sec/user (10x faster than commodity GPU), KV-cache reuse across turns. OpenAI-compatible endpoint — drop in for the calls that don't need Sonnet. US data residency, SOC 2 + agent-action audit logging. Migrate one agent route in a week."

**Reference customer they trust.** Another named agent company. Cursor moves to us → Cognition evaluates within a month. The agent-company ecosystem is dense and reference-driven.

---

## 10. Sales Cycle Mechanics — Tier G

**Q: How does the Tier G agent-company sales cycle differ from Tier C batch?**

### Tier G Cycle (Target: 4–10 weeks)

| # | Stage | Duration | Activity | Gate |
|---|---|---|---|---|
| 0 | Source | — | Founder-led outbound or inbound benchmark | CTO agrees to test |
| 1 | Self-serve eval | 1 wk | Sign up, run their agent loop on our endpoint | Per-task latency + cost validated |
| 2 | A/B pilot | 1–3 wks | Route 10% of their agent traffic through us, A/B vs incumbent | ROI + UX validated |
| 3 | Contract | 1–2 wks | MSA, SLA (with per-stream throughput floor), pricing tier | Signed |
| 4 | Migration | 1–4 wks | Cutover non-frontier agent calls to us | Live |

**Total: 4–10 weeks. Median ~6 weeks.**

**Why it's faster than Tier C batch:** Tier G companies have inference COGS as their #1 OpEx pressure. They make decisions faster. The technical evaluation is straightforward (run their agent against our endpoint, measure $/task and per-stream throughput).

**Where it breaks:**
- A/B pilot doesn't show clear UX win. Per-stream throughput claim is the most important demo to land — if our 10x latency advantage doesn't materialize on their workload, the deal dies at Stage 2.
- OpenAI-compatible API has subtle incompatibilities with their LLM-routing logic (function-calling formats, tool-call schemas).
- They're mid-fundraise and don't want to switch infra mid-process.

**What compresses it:**
- A peer agent-company already migrated. The first Cursor-class win unlocks every other coding agent.
- We can A/B at zero cost during pilot (their existing infra runs in parallel).
- A particularly aggressive incumbent price hike from Anthropic / OpenAI in the same quarter.

(Tier C, D, B, E, F sales cycles for agentic largely match batch — see `batch-inference-business-analysis.md` §10.)

---

## 11. Pricing & Contract Structure — Agentic

(Largely same as batch — see `batch-inference-business-analysis.md` §11. Agentic-specific differences below.)

### Agentic-Specific Pricing Adjustments

- **Per-stream throughput SLA.** Tier G customers care about this; batch customers don't. We commit to 15K+ tokens/sec/user at p95 in the SLA. Critical differentiator.
- **Per-task pricing tier (optional).** For some Tier G customers, $/agent-task is more intuitive than $/M tokens. Offer optional per-task pricing for standard agent shapes ($/coding-task, $/CS-conversation, $/voice-call). This is a sales tool, not a margin lever — keep underlying $/M-token pricing as the source-of-truth.
- **Reasoning-model burst capacity.** For Tier G customers on the year-3+ Qwen-Agent reasoning SKU: separate pricing tier for reasoning calls vs. mechanical calls.

### Agentic Contract Terms

- **Commitment by token volume + per-stream throughput.** Standard reserved + an SLA on per-stream tok/sec.
- **No exclusivity asks from us.** Don't restrict Tier G customers from keeping Sonnet for hard reasoning. We're additive to their stack, not replacement. The "only the mechanical 70%" framing is critical to closing fast.

---

## 12. Sales Team & Org — Agentic Additions

(Largely overlaps with batch — see `batch-inference-business-analysis.md` §12. Agentic-specific additions below.)

### Year 1 Additions

- **1 Forward-Deployed Engineer dedicated to Tier G.** Profile: ex-OpenAI Solutions, ex-Anthropic Solutions, ex-Anyscale, ex-LangChain, ex-Cursor / ex-Sierra. Knows agent loops cold.
- **No separate sales hire in year 1.** CEO + the FDE close the design partners.

### Year 2 Additions

- **1 Director of Agent Customer Success.** Profile: ex-Cursor, ex-Sierra, ex-Replit. Owns Tier G renewal and expansion.
- **2 AEs specifically for Tier G.** Profile: ex-Snowflake AI specialists, ex-Anthropic enterprise sales, ex-Together.

### Year 3+

Standard scaling — see batch §12.

---

## 13. What Compounds — Agentic-Specific Moats

(In addition to the five compounding assets in `batch-inference-business-analysis.md` §13.)

**6. Agent-trace dataset.** Every Tier G customer's traffic generates anonymized agent traces (with consent in the design-partner agreement). This dataset is uniquely valuable for tuning the next-gen agent SKU — nobody else (not even Anthropic / OpenAI) sees the same volume of *real* agent loops across customers and verticals.

**7. Tool-call ecosystem integration.** Once Cursor / Sierra / Cognition integrate us, their custom tool-use logic (their MCP servers, their function-calling schemas) is co-tuned with our endpoint. Switching back to a generic provider = 6–12 months of re-integration.

**8. Per-stream-throughput benchmark moat.** Once we publish "10x faster agent loops," that becomes the table-stakes claim every competitor has to match. We hold the lead on this metric for 3–5 years.

---

## 14. Risks Specific to Agentic

(In addition to risks in `batch-inference-business-analysis.md` §14.)

**1. Closed-frontier reasoning keeps pulling ahead.**
- Indicator: Claude Opus 5 / GPT-6 reasoning closes more agent tasks that previously could split between frontier and open-weight.
- Mitigation: agentic addressable share shrinks from 60–70% to 30–50% — still a big market, but the upside narrows.

**2. Agent product companies vertical-integrate.**
- Indicator: Cursor / Sierra build their own inference (CoreWeave-class) or get acquired by a hyperscaler.
- Mitigation: lock multi-year reserved-capacity contracts before Series D-stage agent companies build their own. The marquee Tier G customer list (Cursor especially) is a 24-month window.

**3. Tool-call ecosystem fragmentation.**
- Indicator: MCP / function-calling diverges across providers; locks customers into specific ecosystems.
- Mitigation: support every major tool-call format (OpenAI, Anthropic, MCP, Google) at the API layer.

**4. Agent latency expectations drop.**
- Indicator: by 2028, users expect agent tasks to complete in <1 second (current: 30 sec — 5 min).
- Mitigation: per-stream throughput is our headline metric; we already lead by 10x. Stay there. Qwen-Agent SKU tape-out raises the bar further.

**5. Open-weight reasoning models stall.**
- Indicator: Qwen 4 ships without a reasoning variant; DeepSeek-R3 underperforms.
- Mitigation: Qwen-Agent SKU pivots to whichever open-weight reasoning model is best at year-3 tape-out time (Llama-Reasoning, DeepSeek-R3+, etc.).

---

## 15. Roadmap — Layered onto Batch

(Layered onto the batch roadmap from `batch-inference-business-analysis.md` §15.)

### Month 0–18: Tape-Out + Tier G Design Partners (Pre-Ship)

- 2–3 Tier G design partners signed (Cursor-class, Sierra-class, Cognition-class).
- Agent-specific software stack built (KV-cache management, tool-call API, agent observability).
- Agent-action audit logging in compliance stack (additional to SOC 2 / ISO 27001).

### Month 18–30: First Ship, Agentic Goes Live

- 2–3 Tier G customers in production.
- Public benchmark: "10x faster agent loops on Qwen-3 hard-locked chip."
- 5–10 additional Tier G logos.
- First Tier D enterprise internal-agent deployment.

### Month 30–42: Scale + Qwen-Agent Tape-Out

- 15–25 Tier G customers; hyperscaler agent-marketplace listings.
- Qwen-Agent SKU tape-out begins (~month 30). Optimized for prefill + KV-cache hierarchy + reasoning-model variant.

### Month 42–60: Qwen-Agent Ships + International + Federal

- Qwen-Agent SKU ships (~month 54).
- 30+ Tier G customers; 10+ Tier D enterprises.
- International + federal agent variants seeded.

---

## 16. The One-Slide Pitch (Agentic)

> **Agentic AI is the fastest-growing inference category — 200%+ YoY, $5–15B in 2025, projected $80–150B by 2030. Each user-action consumes 10–100x more tokens than chat. Today, agent companies (Cursor, Sierra, Cognition, Decagon-class) burn 60–80% of OpEx on inference, paying $25 per 5M-token agent task at frontier prices. On our hard-locked Qwen-3 chip the same task costs $0.30 — 90x cheaper, 10x faster per stream. Wedge buyer: Tier G AI agent product companies signing up in 4–10 weeks because inference COGS is their #1 financial pressure ahead of fundraise. Year 1: Cursor-class design partners on the year-1 batch chip. Year 2: $150–400M agentic ARR on top of batch's. Year 3: tape out dedicated Qwen-Agent reasoning SKU funded by year 1–2 revenue from both segments. Combined batch + agentic ARR: $1.8–3.5B by month 60. Same silicon, two products, double the addressable market.**

---

## Open Questions / Next Steps

- **Validate Tier G pipeline.** Top 8 named Tier G targets ranked by (a) annual inference COGS, (b) cost-pressure signals (recent fundraise, public margin commentary), (c) warm-intro availability. Cursor / Anysphere, Sierra, Cognition, Replit, Codeium, Decagon, Bland, Cresta is the obvious 8 — validate with 2–3 hours of network mapping.
- **Per-stream throughput proof point.** Need a single-rack demo running an agent loop end-to-end at the 17k tokens/sec/user spec, ideally on a real Tier G customer's workload. **This is the most important pre-launch artifact.** No deal closes without it.
- **Agent-trace dataset rights.** Negotiate trace-sharing terms with design partners up front. Anonymized agent traces are uniquely valuable; getting the rights at design-partner stage is much easier than retrofitting later. Add to standard design-partner agreement template.
- **MCP / function-calling support architecture.** Standardize on which tool-call formats we support natively in the OpenAI-compatible layer. OpenAI function-calling is table stakes; Anthropic Computer Use / MCP is increasingly important; Google function-calling for Vertex agent platform.
- **Per-task pricing modeling.** Some Tier G customers want $/agent-task pricing (predictable, easy to forecast). Build the conversion logic from $/M tokens → $/task for standard agent shapes (coding task, CS conversation, voice call, browser task).
- **Qwen-Agent SKU spec.** Long-context KV cache hierarchy, reasoning-variant model selection (Qwen3-235B-A22B-reasoning vs. successor), prefill-optimized layout. Spec freeze by month 18; tape-out month 30; ship month 54.
- **The "Cursor whale" question.** Cursor is the marquee Tier G target and also the most likely to vertical-integrate (build their own inference like CoreWeave). Strategy: lock them on a 3-year reserved contract with reservation deposit before they raise their next round.
- **Closed-frontier reasoning competitive watch.** Quarterly assessment of Claude Opus / GPT-X reasoning vs. Qwen reasoning on agentic-task benchmarks. If the gap widens vs. closes, our agentic addressable share shifts — and the Qwen-Agent SKU spec needs to react.

*Document compiled 2026-04-29.*
