# Batch Inference as the Wedge — Market, Buyer & Unit Economics for a Specialized Inference Datacenter

*A working analysis of high-throughput, latency-tolerant inference as the first datacenter we build. Who pays for it, what they pay today, what proprietary HW + open-weight models change, and where the unit economics go from "interesting" to "uncontestable."*

> Companion to `inference-provider-thesis.md` (forthcoming), which lays out the broader "shaped capacity" thesis — that the inference market splits into 3–4 distinct workload archetypes, each demanding a co-designed HW/SW stack. This document zooms in on **archetype 1: high-throughput batch** — the wedge with the fastest path to a named-customer pipeline and the cleanest unit-economics story.

---

## 1. What "High-Throughput Batch Inference" Actually Means

**Q: Before we size the market, what workload are we even talking about? Define the archetype tightly enough that we know what we're building for and what we're not.**

### The Operating Point

Batch inference is defined by what the workload *doesn't* care about more than what it does:

- **Time-to-first-token (TTFT) doesn't matter.** Jobs are submitted now and consumed minutes-to-hours later.
- **Inter-token latency doesn't matter.** No human is watching tokens stream.
- **Interactivity doesn't matter.** Single-shot, fire-and-forget.
- **Throughput per dollar and per watt is everything.** $/M tokens at a defined SLA (typically 1hr–24hr completion) is the only KPI.

That asymmetry is what makes the workload architecturally distinct from chat or coding-assistant inference. A chip and serving stack that ignores TTFT can run **batch sizes 10–100x larger**, pipeline prefill and decode across many requests, and saturate memory bandwidth in ways an interactive system never can.

### What Counts as Batch in 2026

Concrete workload categories, ranked by current token volume:

| Workload | Typical shape | Volume estimate |
|---|---|---|
| **Document understanding / extraction** | 5K–50K input → 1K–5K structured output | Largest enterprise category |
| **Embeddings at scale** | Re-embedding corpora, vector DB refresh | Massive but priced separately |
| **Synthetic data generation** | 1K input → 4K–16K output, per AI lab | Lab-driven, ~$1B+ today |
| **Bulk classification / moderation** | Short input, classification head | Hyperscaler-internal, hard to size |
| **Bulk summarization** | CRM notes, support tickets, transcripts | Mid-market enterprise |
| **Code analysis at repo-scale** | Static analysis, security scanning | Emerging |
| **ETL with LLMs** | Snowflake/Databricks-style transforms | Just starting; could be huge |
| **Re-ranking pipelines** | Search relevance refresh | Cost-sensitive, growing fast |

The unifying property: **the customer is willing to wait minutes-to-hours in exchange for 50–95% lower $/token.**

### Why This Workload Deserves Its Own Datacenter

> A datacenter optimized for $0.50/M-token batch inference looks nothing like one optimized for 50ms-TTFT chat. The HW topology, memory hierarchy, scheduler, and even the building's power and cooling profile diverge.

Specifically:
- **Batch HW can sustain 100% utilization** because there's a backlog. Interactive HW must keep headroom for traffic spikes; effective utilization is 30–60%.
- **Batch HW can run hot.** Sustained max-throughput thermal design vs. peaky interactive thermal design.
- **Batch HW can be larger and slower per chip** because memory bandwidth and capacity dominate, not clock speed.
- **The scheduler is a different product.** Batch needs job queueing, fair-share, deadline scheduling, retry semantics. Interactive needs per-request load balancing.

The economic implication: **a purpose-built batch DC has 2–5x better effective $/M-token than a general-purpose GPU DC running the same chips, before any HW differentiation.** Layer in specialized HW and the gap widens to 5–20x at the bottom of the curve.

---

## 2. Market Size — Today and Through 2030

**Q: Is this a real market, or are we sizing a future hope? How big is batch inference revenue *today*, and what's the credible 2030 number?**

### 2025 Baseline

Public and triangulated numbers:

- **Total AI inference spend (2025):** ~$15–25B globally. Includes API revenue (OpenAI ~$5B inference, Anthropic ~$3B, Google ~$2B, others), inference clouds (Together ~$200M ARR, Fireworks ~$150M, DeepInfra, Replicate, Modal, etc.), and self-hosted GPU spend amortized.
- **Batch share:** ~15–25% of API inference. OpenAI Batch API and Anthropic Batch (both 50%-off realtime, 24h SLA) are the visible tip; far larger volume runs through self-hosted batch on AWS/Azure/GCP.
- **Implied 2025 batch market:** **~$3–6B**, of which ~$1–2B is "addressable by a third party" (the rest is captive on hyperscaler-internal workloads).

### Why It Grows Faster Than the Headline

Three forcing functions push batch share *up* over the next 4 years:

1. **Token volume per user-action is 10–100x interactive.** A single agent run reads 50 documents and writes 5 — that's 500K tokens for one user action. Most of those tokens don't need to stream live. Anything not on the critical interactive path migrates to batch.
2. **Open-weight models close the quality gap on batch-shaped workloads first.** Document extraction, classification, and structured-output tasks are the workloads where Llama 4 / Qwen 3 / DeepSeek-V3 are *already* at parity with GPT-4-class. The "frontier premium" doesn't apply.
3. **Enterprise deployment is hitting scale phase.** 2024 was POCs (interactive demos). 2026+ is "process every document, every record, every transaction" — and that's batch-shaped.

### 2030 Projection

Conservative-to-aggressive bracket:

| Scenario | 2030 Total Inference | Batch Share | Batch Market | Driver |
|---|---|---|---|---|
| Conservative | $80B | 25% | $20B | Linear extrapolation |
| Base | $150B | 30% | $45B | Agent + ETL workloads scale |
| Aggressive | $300B | 35% | $105B | LLM-native ETL replaces traditional data pipelines |

**Anchor for the deck: $40–80B addressable batch market by 2030.** Of that, roughly half is captive-to-hyperscalers (we won't get it). The other half — ~$20–40B — is the contestable wedge where a specialized DC can play.

### The Latent Market Argument

> The most underrated number in this thesis is the workload that *doesn't exist yet because tokens are too expensive*.

At today's batch pricing (~$0.30–1.00/M tokens for 70B-class), a Fortune 500 with 100M customer records and 10K tokens of LLM enrichment per record is staring at a **$300M–$1B project cost**. So they don't do it. At $0.05/M tokens, that project costs $5M and gets greenlit. **The latent demand curve is steep.** A 5–10x cost reduction unlocks 10–50x volume. This is the argument for going past "we win share" to "we expand the pie."

Historical analogy: AWS S3 going from $0.15/GB-month (2006) to $0.023/GB-month (2025) didn't just take share from on-prem storage — it created entire categories of data product (data lakes, observability, ML training) that were uneconomic at the old price.

---

## 3. The Buyers — Who Actually Pays for This (Under Qwen + US-DC)

**Q: Under hard-lock to Qwen 3 and a US-resident first datacenter, who actually buys this — and how does that differ from a model-agnostic batch DC?**

### How Hard-Lock + Qwen + US-DC Reshapes the Buyer Set

The architectural decisions in §7 reshape the buyer set significantly from a generic "shaped capacity" framing. Some tiers we'd serve in a model-agnostic, multi-DC world drop out; others move to the front.

| Buyer tier | Original (model-agnostic) fit | Fit under Qwen + US-DC | Why the change |
|---|---|---|---|
| **C — Vertical AI platforms** | Year 2 supplemental | **Year 1 wedge** | Cost-pressured, Qwen-tolerant, US-friendly, fast cycles |
| **D — US single-workload enterprise** | Year 2–3 | **Year 2** | Many F500 are Qwen-tolerant for non-frontier workloads if data stays in US |
| **B — Hyperscaler marketplace** | Year 1–2 strategic wholesale | Year 2 marketplace listing only | Hard-lock = "one model on the menu," not "strategic capacity" |
| **E — Sovereign / international** | Year 2–3 | **Year 3+** | US-DC choice forces deferral; international DC required |
| **F — US federal / defense / strict-F500** | Year 2–3 | **Year 3+, via Gemma SKU** | Chinese-origin model is structurally off-limits regardless of DC location |
| **A — Foundation model labs** | Original wedge | **Largely out** | Labs run their own custom checkpoints; cannot use a Qwen-locked chip |

### Tier C — Vertical AI Platforms (the new wedge)

**Why they're the wedge under Qwen + US-DC:**
- Cost-pressured — every Series C+ deck has a "path to gross margin" slide.
- Qwen-tolerant — venture-funded AI startups don't have "no Chinese AI" procurement bans.
- Already standardize on popular open-weight models — many use Qwen 3 today via Together / Fireworks / DeepInfra.
- US-HQ themselves — US-DC is a feature, not a problem.
- Sales cycle 4–12 weeks — fastest path to revenue and reference customers.

**Names:** Harvey (legal, mixed model), Hebbia (finance, multi-model), Glean (enterprise search), Sierra (CX agents), Decagon, Cresta, Sema4 (healthcare), Bland AI / Vapi (voice agents), Cursor / Anysphere (parts of agent loop), Perplexity (re-ranking + extraction), Sourcegraph (code analysis), Notion AI.

**2025 batch spend per leader:** $5–50M; top-3 (Harvey, Hebbia, Sierra-class) approaching $100M.
**Contract size:** $500K–$10M ACV at signature; $5–25M at scale.
**Sales cycle:** 4–12 weeks.

**Why they matter:** highest growth rate, most cost-sensitive, willing to switch, technically sophisticated, public reference value compounds.

### Tier D — US Single-Workload Enterprise (year 2)

**Why they fit:** Many F500 have batch use cases (claims processing, contract review, document extraction, content moderation) where Qwen 3 is "good enough" and cost is the binding constraint. If the data stays in the US (which it does), the "no Chinese AI" objection is partially defused — the live concerns are model behavior and procurement policy, not data flow.

**Names that are reachable:** Bloomberg (financial documents), Stripe (fraud + content classification), JPMorgan (contract review at scale — but watch policy), Walmart (catalog enrichment), Allstate / Lemonade (claims), Pfizer / AbbVie (molecule + patent screening), AT&T (call analysis), Salesforce (Agentforce backend).

**Sales cycle:** 6–12 months. Mostly via hyperscaler marketplace using the customer's committed AWS/Azure/GCP spend (this is how 60%+ of enterprise AI spend gets routed in 2026).
**Contract size:** $1–10M ACV.

### Tier B — Hyperscaler Marketplace (downgraded to year 2)

**Why the original framing collapses:** "Wholesale us your strategic batch tier" was a model-agnostic pitch. Under hard-lock to Qwen, the pitch shrinks to "list us as the lowest-cost Qwen-3 endpoint on your marketplace." That is a *much* weaker deal:

- We compete on the same marketplace, on the same model, against Together / Fireworks / DeepInfra.
- Hyperscaler margin is thinner (commodity model + API).
- We don't get the "fill the batch tier gap" strategic value — they already have Qwen via 5+ partners.

**What still works:** an *exclusive* lowest-cost-Qwen-batch listing on a single cloud, in exchange for a multi-year commit and joint marketing. AWS Marketplace / Azure Marketplace / GCP Marketplace listings are 3–6 month cycles, not 12–18 months — fast but small.

**Sales cycle:** 3–6 months for a marketplace listing (not the original 12–18 months for strategic wholesale).
**Contract size:** revenue-share, $500K–$5M annual to us, much larger gross transacted volume on their marketplace.

### Tier E — Sovereign + International (year 3+, requires international DC)

**Why deferred:** sovereign deals (UAE G42, Saudi HUMAIN, India Yotta, EU GAIA-X members) require **in-country compute**. US-DC choice means we don't have it in year 1. Asian-Pacific sovereigns *might* accept US-resident infra under a multi-region contract structure, but the canonical sovereign deal is local.

**Year 3 plan:** with year-1 Qwen revenue, fund a second DC outside the US (Singapore, UAE, or Ireland depending on which sovereign deal is closest to close). Sovereigns are the natural Qwen buyer once local-DC requirement is met — Qwen's Apache-2.0 license + non-US origin is actually *easier* for non-aligned sovereigns to accept than Llama or Gemma.

### Tier F — US Federal / Defense / Strict-F500 (year 3+, requires Gemma SKU)

**Why deferred:** ~20–30% of US enterprise spend is in segments with explicit "no Chinese AI" policies regardless of where the model runs (federal, defense primes, most banks, healthcare under HHS/HIPAA security review, parts of telco). Year-1 Qwen SKU does not address them.

**Year 3 plan:** Gemma SKU (or Llama SKU if Gemma quality lags) opens this segment. Tape-out at month 18, ship month 36, first US-federal close month 40+.

### Tier A — Foundation Model Labs (largely out)

Labs run their own custom-trained weights. They cannot use a chip locked to Qwen public weights. **Possible niche:** distillation pipelines, eval, synthetic-data generation where a generic-quality public model is acceptable. Anthropic and OpenAI reportedly use Llama-class for some such pipelines and would substitute Qwen if economics demanded. But this is a small workload, not a wedge — and several labs are actively avoiding Chinese-origin weights for governance reasons even when the workload is legal.

### Wedge Recommendation

> **Land 2–3 vertical AI platform whales (Tier C) as wedge customers in months 0–18 (pre-ship via design-partner agreements). Use them as the public reference to unlock Tier D US single-workload enterprise in year 2 and Tier B hyperscaler marketplace listings in year 2–3. Tier E sovereign and Tier F federal-via-Gemma are year-3 problems funded by Tier C/D revenue.**

This is a **fundamentally different sequencing** than the original "land a foundation lab as the wedge" plan. The change is forced by hard-lock + US-DC: the lab pitch dies, sovereigns defer, and vertical AI startups become the cleanest path to first revenue and first reference customers.

---

## 4. Competitive Landscape — Why Isn't This Already Won?

**Q: Inference is the most-funded category in AI infra. Why is there room for a new specialized batch player?**

### The Five Cohorts and Why Each Falls Short on Batch

**1. Frontier API Batch tiers (OpenAI Batch, Anthropic Batch).**
- **Pricing:** 50% off realtime. GPT-4o batch: $1.25/M in, $5/M out. Claude Haiku batch: $0.40/M in, $2/M out.
- **Why limited:** running closed-weight frontier models on H100/B200. Cost structure is set by NVIDIA + frontier-model R&D amortization. No path to <$0.10/M tokens.
- **Their moat is model quality, not cost.** They charge for intelligence, not throughput.

**2. Open-weight inference clouds (Together, Fireworks, DeepInfra, Replicate, Anyscale, Modal).**
- **Pricing:** Llama 70B at $0.40–0.88/M tokens. Llama 405B at $3–5/M.
- **Why limited:** running open weights on H100/B200 in colos. They're software differentiated (great DX, fast cold-start, observability) but their cost floor is set by NVIDIA hardware. They cannot get below ~$0.20/M tokens on 70B-class models without a HW change.
- **They are the *natural acquirer* of our capacity** — they have the customers, we have the cost structure.

**3. Specialized inference HW (Groq, Cerebras, SambaNova, Etched, Tenstorrent).**
- **Groq (LPU):** Optimized for low-latency interactive. Batch is an afterthought. Wrong workload.
- **Cerebras (WSE-3):** Single-wafer chip. Strong on batch in theory; in practice their commercial focus is interactive ("fastest inference in the world"). $2–3M per system; capex-heavy.
- **SambaNova (RDU):** Enterprise on-prem focus. Sells appliances, not capacity.
- **Etched (Sohu):** Transformer-only ASIC. 500K tokens/sec on Llama 70B claimed. Promising but unshipped at scale; transformer-lock-in is a long-term risk if architectures shift.
- **The gap:** none of them have packaged "batch capacity as a service at sovereign-DC scale." They sell chips or appliances.

**4. Hyperscaler batch (AWS Bedrock Batch, Azure OpenAI Batch, GCP Vertex Batch).**
- Reselling frontier APIs at small markups, plus their own first-party models.
- Cost structure inherits from NVIDIA + the frontier vendor. Same floor as #1.
- **But: they own the customer.** Distribution moat is real.

**5. Self-hosted on commodity GPUs.**
- Big enterprises run their own H100 clusters with vLLM / SGLang / TensorRT-LLM.
- At scale and high utilization (>70%), they hit ~$0.10–0.30/M tokens for 70B-class.
- **The structural ceiling:** they're still buying NVIDIA. Same chip, same power, same memory bandwidth. They beat the API providers on margin, not on physics.

### The Whitespace

> Nobody is offering **first-party batch capacity at sovereign-DC scale, on co-designed HW, served as a fully managed token API or reserved-capacity contract**. That's the gap.

The closest analog historically: AWS in 2006. Storage and compute existed; what was missing was a clean "pay for capacity, don't build it" abstraction at hyperscaler scale and pricing. The first-mover advantage compounds because every reference customer makes the next sale easier and every TWh of contracted capacity makes the next datacenter cheaper to finance.

---

## 5. Why Specialized HW Wins on Batch (The Architectural Argument)

**Q: Walk me through, from first principles, why proprietary HW beats NVIDIA on batch — and how durable that lead is.**

### Batch Inference Is a Memory Problem, Not a Compute Problem

For a 70B-class model running batch inference at high batch sizes:
- **Compute:** ~140 TFLOPS per token at FP8.
- **Memory bandwidth needed:** ~140 GB/s per token-stream (read full weights once per token for decode).
- **Memory capacity:** 70–140GB for weights + KV cache that scales with batch × context.

An H100 SXM has 989 TFLOPS (FP16) and 3.35 TB/s HBM bandwidth. The chip is **massively compute-overprovisioned** for batch inference. You're paying for FLOPS you can't use because the bottleneck is HBM bandwidth feeding decode.

### What a Co-Designed Batch Chip Optimizes

The architectural levers that matter:
1. **More HBM (or better SRAM hierarchy)** — directly increases batch size and KV cache capacity.
2. **Less compute** — strip the training-grade FLOPS you're not using; spend the silicon on memory and interconnect.
3. **Better data movement** — KV cache offload, prefix caching across requests, persistent state.
4. **Power envelope tuned for sustained throughput** — not peak interactive.
5. **Larger die or wafer-scale** — memory bandwidth aggregates across the chip.

NVIDIA can't easily make this trade-off because **the same chip has to win training too.** Training needs the FLOPS. So Hopper, Blackwell, and Rubin are all compute-heavy compromises that leave money on the table for batch.

### Durability of the Lead

The honest answer: **a 3–5 year window before NVIDIA closes most of it via Rubin and Rubin Ultra (likely 2027–2029 ship dates).** But:
- During that window we contract reserved capacity at our pricing, locking in 5–7 year revenue.
- We don't need to beat NVIDIA forever — we need to compound enough revenue + datacenter footprint + customer contracts that we're acquired or IPO before the window closes.
- After the window, we compete on operations, distribution, and contracts — not silicon.

> **The HW is a 5-year wedge. The DC + contracts + serving stack is the 20-year business.**

### What Concrete Performance Targets Need to Be True

For the thesis to hold, the proprietary HW needs to deliver, at the very least:

| Metric | NVIDIA H100 (today) | Required from us | Stretch |
|---|---|---|---|
| Tokens/sec/W on Llama-70B batch | ~3–5 | 15–25 | 50+ |
| Tokens/sec/$ of capex | 1× | 3–5× | 10× |
| Effective $/M tokens at 80% util | $0.20–0.30 | $0.05–0.08 | $0.02 |

`[TBD: insert actual numbers from internal hardware spec — these targets are placeholders calibrated against published Cerebras, Groq, and Etched claims plus first-principles HBM-bandwidth math.]`

---

## 6. Unit Economics — Does This Print Money?

**Q: Build the bottoms-up cost model. What does a fully loaded batch token cost on our infra at realistic utilization, and what's the gross margin profile?**

### Cost Stack per 1M Tokens (Qwen-3-72B-class output)

Anchored on the proprietary HW spec: **17,000 tokens/sec per concurrent user-stream** on Qwen-3-72B (hard-locked). Three perf-per-watt scenarios bracket the cost basis:

| Perf-per-watt | Power per stream | Streams per MW | Total throughput / MW | Annual tokens / MW (80% util) | Cost basis ($/M) |
|---|---|---|---|---|---|
| 10 tok/sec/W (conservative) | 1.7 kW | 588 | 10 M tok/sec | 252 T | **$0.027** |
| **17 tok/sec/W (§5 mid-target)** | **1.0 kW** | **1,000** | **17 M tok/sec** | **428 T** | **$0.016** |
| 25 tok/sec/W (§5 stretch) | 0.68 kW | 1,470 | 25 M tok/sec | 630 T | **$0.011** |

Annual cost per 1MW deployment, 3-yr amortization, US Tier-3 colo:

| Cost component | Annual ($K) | $/M tokens (mid scenario) | Notes |
|---|---|---|---|
| HW capex (3yr amortized) | $4,000 | $0.0093 | $12M HW capex per MW; `[TBD: confirm]` |
| Power (1MW × 8760h × $0.07/kWh × PUE 1.3) | $800 | $0.0019 | Wholesale power region |
| Cooling/colo/space | $400 | $0.0009 | $400/kW/yr typical |
| Bandwidth + networking | $200 | $0.0005 | |
| Operations, monitoring, on-call | $500 | $0.0012 | 3 SREs + tooling |
| Software stack (compiler, scheduler, serving) amortized | $1,000 | $0.0023 | R&D allocation |
| **Total cost / M tokens** | **$6,900** | **$0.016** | mid-scenario; ranges $0.011–$0.027 |

> **The headline number for the deck: $0.016/M output tokens at the mid-scenario perf-per-watt — roughly 12x below the cheapest Qwen-3-72B provider in market today (DeepInfra at $0.20/M batch).**

`[All numbers TBD pending internal HW spec validation — ranges calibrated against the user-supplied 17k-tok/sec/user baseline plus §5 perf-per-watt targets.]`

### Competitive Pricing Landscape (Q2 2026)

What does the market actually charge for Qwen-3-72B-class output today? Batch tier (~50% off list):

| Provider / Model | Output $/M (list) | Output $/M (batch) | Notes |
|---|---|---|---|
| **Open-weight Qwen-3-72B hosting** | | | |
| Together AI | $0.88 | $0.44 | Premium DX |
| Fireworks | $0.90 | $0.45 | Premium DX |
| Hyperbolic | $0.50 | $0.25 | Aggressive new entrant |
| **DeepInfra** | **$0.40** | **$0.20** | **Cheapest open-weight Qwen provider** |
| Self-hosted on H100 (high util) | n/a | $0.20–0.30 | F500 internal only |
| **Cheap closed-source comparable** | | | |
| GPT-5-mini | ~$0.60 | ~$0.30 | OpenAI volume tier |
| **Gemini 2.5 Flash** | **~$0.40** | **~$0.20** | **Cheapest closed-source comparable** |
| DeepSeek-V3.1 | $0.27 | $0.27 | Already at-cost; Chinese provider |
| Claude Haiku 4.5 | $5.00 | $2.50 | Higher quality, doesn't compete on price |

**Floor of the comparable market today: ~$0.20/M output tokens batch.** Median competitor: $0.30–0.45/M. Input pricing roughly half of output.

> Closed-source pricing is compressing. Best estimate: Gemini Flash batch at **$0.10–0.12/M output by 2028**. We must price against where Gemini will be in 24 months, not today.

### What Price Must We Hit — Three Thresholds

| Threshold | Target output price | Strategic meaning |
|---|---|---|
| **Match** | $0.20/M | "Same price, better SLA + US data residency + audited weights" |
| **Undercut** (30% below floor) | $0.14/M | "Meaningfully cheaper than DeepInfra and Gemini Flash" |
| **Demolish** (70% below floor) | $0.06/M | "5x cheaper than the cheapest comparable. Headline-grabbing." |

Year-1 wedge (Tier C reserved): target the **Undercut tier ($0.10–0.15/M output)** — clearly switch-worthy without burning money. Tier B marketplace listing: target the **Demolish tier ($0.06–0.08/M output)** — only viable if cost is truly best-in-class.

### Margin Sensitivity Across Cost Scenarios

Plugging the new cost basis into the price grid:

| Output price | Margin @ $0.027 cost (conservative) | Margin @ $0.016 cost (mid) | Margin @ $0.011 cost (stretch) |
|---|---|---|---|
| $0.30 (retail / on-demand) | 91% | **95%** | **96%** |
| $0.20 (match Gemini batch) | 87% | **92%** | **95%** |
| $0.15 (clear undercut) | 82% | **89%** | **93%** |
| $0.10 (anchor reserved) | 73% | **84%** | **89%** |
| $0.06 (demolish, Tier B) | 55% | **73%** | **82%** |
| $0.04 (loss-leader) | 33% | **60%** | **73%** |
| $0.03 (sub-DeepSeek) | 10% | **47%** | **63%** |
| $0.02 (extreme floor) | -35% | 20% | **45%** |

**Every realistic price tier prints money even in the conservative scenario.** No "make-or-break" threshold worry — at 17k tokens/sec/user, the cost basis is 2–5x below the $0.05/M threshold the previous sensitivity analysis flagged.

### The Strategic Choice: Skim vs. Expand-the-Pie

With this much pricing latitude, the question shifts from "can we charge enough to survive?" to "should we leave money on the table?"

| Strategy | Anchor price | Margin | Logic |
|---|---|---|---|
| **Skim the cream** | $0.10–0.15/M | 84–89% | Capture the existing market's spend at high margin; cap revenue at current TAM |
| **Expand the pie** | $0.04–0.06/M | 60–73% | Lower margin per token but unlock 10–50x volume — workloads uneconomic at $0.20/M become viable at $0.05/M |

For a venture-backed startup, **expand-the-pie wins** — the mission statement ("Aristotle-level intelligence in every desk, every device") only works at sub-$0.10/M, and the latent batch demand curve is steep below that price.

> The "tokens too cheap to meter" framing is achievable, not aspirational, with this cost structure. That is the deck's headline.

### Recommended Pricing Card

| Customer tier | Input $/M | Output $/M | Margin @ $0.016 cost (mid) |
|---|---|---|---|
| Retail / on-demand | $0.10 | $0.20 | 92% |
| Tier C reserved (1-yr) | $0.04 | $0.08 | 80% |
| Tier C dedicated (3-yr) | $0.025 | $0.05 | 68% |
| Tier D enterprise (via marketplace) | $0.03 | $0.06 | 73% |
| Tier B marketplace wholesale | $0.015 | $0.03 | 47% |
| Sovereign (year 3+) | negotiated | $0.15–0.30 | 89–95% |
| Federal via Gemma (year 3+) | negotiated | $0.15–0.40 | 89–96% |

**Blended target at maturity: 75–82% gross margin** (60% revenue from Tier C/D, 25% from Tier B marketplace, 15% from year-3 segments). This is best-in-class infra (Snowflake ~67%, CoreWeave ~75%, Cloudflare ~78%).

### Three Things to Flag in the Deck

1. **Closed-source pricing is a moving target — but the cost gap protects us.** Gemini Flash 2.5 → 3.0 → 3.5 will compress (likely $0.10–0.12/M output by 2028, ~40% below today). At our $0.016/M cost basis, we still maintain 80%+ margin even if our anchor price tracks Gemini down to $0.08/M.
2. **DeepSeek's "at-cost" pricing flips from threat to narrative gift.** DeepSeek-V3.1 prices output at $0.27/M with no batch discount because they're at cost. At $0.016/M cost, **we sit comfortably under DeepSeek profitably** — and "cheaper than the cheapest Chinese provider, on US infrastructure, with audited weights" is a clean customer-acquisition pitch.
3. **Input pricing is leverage on document-heavy batch.** For 10:1 input:output workloads, input cost dominates total $/transaction. We can be very aggressive on input ($0.015–0.04/M batch input) while keeping output prices at the table-stakes-undercut tier.

### Capacity & Capital

For the deck, the rule of thumb to memorize:

> **One MW of our infra = ~$12M capex = ~500B tokens/year = ~$25M revenue/year at $0.05/M token = ~30%+ EBITDA margin from year 1.**

Scale path:
- **Year 1:** 1–3 MW (single colo, single customer pilot). $30–80M ARR run-rate.
- **Year 2:** 10–25 MW (multi-customer, 2–3 sites). $250–600M ARR.
- **Year 3:** 50–150 MW (full DC + 1 international site). $1.2–4B ARR.
- **Year 5:** 500MW+ (multi-DC, sovereign deployments). $10B+ ARR plausible.

**Capex requirements:** $12M/MW × 100MW = $1.2B for the full year-3 buildout. Financeable as: 50% project debt against contracted revenue (CoreWeave model), 30% equity, 20% strategic capex partnerships (sovereigns, hyperscalers).

---

## 7. The Qwen-First Architecture — Hard-Lock, US Datacenter, Phased SKU Roadmap

**Q: The HW is hard-locked at tape-out, the first SKU targets Qwen, and the first DC sits in the US. What does the architectural and strategic picture actually look like? What are we optimizing for, what are we giving up, and what's the multi-SKU roadmap?**

### The Hard-Lock Reality

The proprietary HW is **hard-locked**: weights baked into silicon at tape-out. New model = new chip. Operating constants:

- Tape-out cycle: ~18–24 months from spec freeze to volume ship.
- NRE per SKU: ~$100–300M (silicon + verification + first-pass yield).
- Asset life: 3 years minimum to amortize NRE; 4–5 years to make a margin.
- Once shipped, the chip's economic value is bounded by the locked model's quality lifespan.

**Hard-lock is the most extreme form of "shaped capacity."** It is the strongest possible architectural commitment to a specific operating point — and it is defensible against NVIDIA precisely because NVIDIA cannot make the trade. NVIDIA needs every chip to win training too. We don't.

### The Three-Way Model Choice (Why Qwen)

| Candidate | Quality (2026) | Volume | License | Geo fit | 3-yr lifespan |
|---|---|---|---|---|---|
| **Qwen 3** | Top-2 open | #1–2 globally | Apache 2.0 | Global ex-restricted-US | Highest predictability |
| Gemma 3 | Mid-tier | Lower | Custom permissive | US/EU clean | Moderate (Google deprioritization risk) |
| Kimi K2 | Strong long-context | Mid Asia | Mixed | Asia | Lowest (small player) |

**Decision: Qwen 3 first.**

1. **Largest TAM** — ~3x Gemma, ~10x Kimi. More tokens flow through Qwen than any other open-weight model on HuggingFace.
2. **Best quality** — Qwen 3 is competitive with Llama 4 at every param count we'd target. Chip economics scale with token utility.
3. **Best license** — Apache 2.0 means zero licensing friction with any customer or jurisdiction. Uniquely portable for a chip-with-baked-weights.
4. **Most predictable roadmap** — Alibaba has shipped Qwen 1 → 1.5 → 2 → 2.5 → 3 over three years on consistent Apache-2.0. Lowest "model abandoned" risk of the three.

**Gemma SKU is a year-3 strategic option**, taped out once Qwen revenue funds the NRE. If Gemma 3/4 fails to close the quality gap to Llama-class, the "American option" SKU pivots to Llama instead. **Kimi tape-out is opt-in** — only justifies the NRE if a $50M+ long-context whale pre-commits.

### The US-DC Choice

**First datacenter in the United States** — despite the chip embedding Chinese-origin weights.

What it gets us:
- **US fundraising stays clean.** US-HQ + US-DC is the easy case for tier-1 US VCs (Sequoia, a16z, Founders Fund, Khosla).
- **Best power markets, talent pool, capital access.** Texas, Phoenix, Ohio, Northern Virginia all viable.
- **US data residency for US customers** — addresses the most common "no Chinese AI" objection (data flow, not model origin).
- **Chip never crosses a border** — sidesteps export-control on the silicon itself.

What it gives up:
- **Sovereign deals requiring local DC.** UAE, Saudi, India, EU sovereigns need in-country compute. Year 3+ via international DC build, not year 1.
- **US federal / defense / strict-China-banned F500** (~20–30% of US enterprise market) — structurally off-limits because the model itself is Chinese-origin, regardless of DC location. Year 3+ via Gemma SKU.

**The new wedge becomes US AI startups + global customers willing to use US infrastructure.** Sovereign and federal are year-3 problems funded by year-1 revenue. (See §8–§9 for the rebuilt GTM.)

### Compliance & Reputation Architecture

Selling Qwen-on-US-DC to US enterprise customers requires the **"American company, American datacenter, American power, American jobs — running independently-audited open-weight Qwen"** framing. Operational requirements:

- **3rd-party model audit.** Trail of Bits / NCC Group / academic red-team on Qwen weights. Published before commercial launch. Updated per Qwen-N refresh.
- **Compliance stack at GA:** SOC 2 Type II, ISO 27001, HIPAA-eligible, CCPA-compliant.
- **No customer data ever leaves the US.** Technical attestation + audit trail.
- **Open-source the serving wrapper** where it doesn't compromise IP. Reduces "black box Chinese AI" perception.
- **US-citizen-only ops team** for the year-1 DC. Defensible against later federal expansion.

This compliance posture is itself a moat. Competitors hosting Qwen via Together / Fireworks / DeepInfra haven't done this work and can't claim the posture. **The compliance gap is a real differentiator, not a checkbox.**

### Multi-SKU Roadmap

| SKU | Tape-out | Ship | Strategic role |
|---|---|---|---|
| Qwen-3-baked (gen 1) | Month 0 | Month 18 | Wedge SKU — global TAM, Apache 2.0, best quality |
| Qwen-N-baked (refresh) | Month 12 | Month 30 | Catch the next Qwen generation; protect installed base |
| Gemma-3-baked (US-procurement) | Month 18 | Month 36 | Open US federal / defense / strict-F500 segment |
| Optional: Llama-N-baked | Month 24 | Month 42 | Replace Gemma if quality gap persists |
| Optional: Kimi-baked | TBD | TBD | Niche long-context — only if $50M+ pre-commit |

### What Hard-Lock + Qwen + US-DC Buys You vs. Costs You

| Buys | Costs |
|---|---|
| Best-possible perf/W on Qwen — silicon makes zero compromise for other models | One model per chip; flexibility goes to zero |
| Strongest defensible $/M-tokens advantage on Qwen specifically (3–5x cheaper than GPU-hosted Qwen) | Stranded capital if Qwen 3 obsolesced inside the 3-year amortization |
| Architectural simplicity (one model = one compiler, one scheduler optimization target) | NRE per SKU = barrier to multi-model strategy |
| Differentiated against NVIDIA general-purpose silicon (we are not pretending to be flexible) | Investors must underwrite model-lifespan risk, not just market-size risk |
| US-HQ + US-DC = clean US fundraising and US tech-startup market access | Sovereign + federal segments deferred to year 3 |

> **The wager: Qwen-3-class capability is durable enough for a 3-year asset life. We hedge with a Qwen-N refresh tape-out at month 12 and a parallel Gemma SKU for year-3 US procurement re-entry.**

---

## 8. GTM Sequencing — The First 60 Months

**Q: Hard-lock means first chips ship at month 18. What do months 0–18 look like (no revenue, full burn), and what does the staged sequencing look like through month 60?**

> Sections 9–12 unpack each pillar of the GTM motion in operational detail: §9 the buyer personas, §10 the per-tier sales cycle mechanics, §11 the pricing menu and contract structure, §12 the sales team and channel build-out. This section is the chapter view.

### The Sequencing

The hard-lock timeline reshapes everything. There is no revenue in year 1 — first chips ship at month 18. Year 1 is design partners and reservations, not invoices.

**Phase 1 — Months 0–18: Pre-Ship (Design Partners + Reservations).**
- **Tape-out underway.** Spec freeze month 0; first silicon back month 12; volume ship month 18.
- **2–3 Tier C design-partner agreements.** Selected from Harvey, Hebbia, Glean, Sierra, Decagon, Cresta, Sema4, Bland AI, Sourcegraph. No cash up front; small refundable deposit ($250K–$1M); guaranteed early capacity allocation; co-marketing rights at GA. Goal: signed customers we can name on slide 1 of every fundraise.
- **1 Tier D enterprise reservation.** A single F500 with a clean Qwen-3-shaped batch use case (Bloomberg-class document processing, Walmart-class catalog enrichment, or insurance claims processing). $5–25M deposit against year-2 capacity.
- **DC site selected, power & colo signed.** Texas / Phoenix / Ohio are the realistic candidates. Power lead time means start month 0, energize by month 15.
- **Compliance stack built.** SOC 2 Type II (12 months), ISO 27001 (9 months), 3rd-party Qwen audit (6 months). Stack live before commercial GA.
- **Revenue: $0. Burn: ~$50–100M.** This is what Series A+B fund.

**Phase 2 — Months 18–30: First Ship, First Revenue.**
- Volume silicon shipping; first MW of DC online by month 18–20.
- 2–3 design-partner customers go live; co-marketing case studies published.
- Tier C self-serve onboarding live; 5–10 additional vertical AI customers signed.
- First Tier D enterprise customer in production via hyperscaler marketplace.
- Begin Qwen-N refresh tape-out (month ~24, anticipating ship at month 42).
- **Revenue: $25–75M ARR by month 30.**

**Phase 3 — Months 30–42: Scale + Hyperscaler Marketplace + Gemma Tape-Out.**
- 15–25 Tier C customers; 3–5 Tier D enterprises live.
- Marketplace listing live on AWS Marketplace + Azure Marketplace (and/or GCP).
- Gemma SKU tape-out begins (month 36 spec freeze, ship month 54).
- Second US DC site evaluation (capacity scaling).
- Qwen-N refresh ships month ~42.
- **Revenue: $200–500M ARR by month 42.**

**Phase 4 — Months 42–60: Multi-SKU + International + Federal-Adjacent.**
- Gemma SKU ships (~month 54); first US federal / defense / strict-F500 sales begin.
- International DC site selected (Singapore, UAE, or Ireland depending on which sovereign deal is closest).
- First Tier E sovereign deal in committed contract phase.
- 30+ Tier C customers; 10+ Tier D enterprises; 1 anchor sovereign.
- **Revenue: $1–2B ARR by month 60.**

### The Sales Motion (per tier, summarized — full mechanics in §10)

**Tier C (vertical AI):** Bottoms-up technical evaluation, founder-led for the first 5 logos, then VP-Sales-led. Self-serve onboarding is critical post-month-18. **Cycle: 4–12 weeks.**

**Tier D (US single-workload enterprise):** Through hyperscaler marketplace using customer's committed AWS/Azure/GCP spend. Multi-threaded but procurement-light because they're not "buying us," they're spending committed cloud dollars. **Cycle: 6–12 months.**

**Tier B (hyperscaler marketplace):** BD-led, single-threaded relative to original framing because we're listing one model, not negotiating strategic capacity. **Cycle: 3–6 months for marketplace listing.**

**Tier E (sovereign) and Tier F (federal/defense):** Year-3 motions, need dedicated SVP-level relationships and (for F) the Gemma SKU.

### What the Pricing Looks Like

Calibrated against the §6 cost basis ($0.016/M output, mid-scenario). Strategy: expand-the-pie pricing to unlock latent demand, not skim-the-cream.

| Customer Tier | Output $/M | Volume commit | Margin |
|---|---|---|---|
| Tier C on-demand | $0.20 | None | 92% |
| Tier C reserved (anchor, 1-yr) | $0.08 | 50–500B tokens/mo | 80% |
| Tier C dedicated (3-yr) | $0.05 | Hardware reserve | 68% |
| Tier D enterprise (via marketplace) | $0.06 | 100B–1T tokens/mo | 73% |
| Tier B marketplace listing | $0.03 wholesale | Revenue share | 47% |
| Tier E sovereign (year 3+) | $0.15–0.30 | Multi-year capacity | 89–95% (capex-offset) |
| Tier F federal via Gemma (year 3+) | $0.15–0.40 | Multi-year reserved | 89–96% |

Blended at maturity: **75–82% gross margin**, in line with or exceeding best-in-class infra businesses (Snowflake ~67%, CoreWeave reserved book ~75%, Cloudflare ~78%). See §11 for full pricing menu and contract terms.

---

## 9. Buyer Personas — Who Actually Signs the Check

**Q: For each tier of buyer, who is the actual decision-maker, what are they measured on, what are they buying today, and what flips them to switch?**

§3 mapped the buyer tiers structurally under hard-lock + Qwen + US-DC. This section is the operational sales playbook — title, KPIs, budget, trigger, and the exact pitch for each, ordered by year-1 fit.

### Tier C — Vertical AI Platforms (the wedge)

**The buyer.** CTO (sub-100-person), VP Eng or Head of ML Infrastructure (100–500-person), CFO involved at scale. Reports to CEO at startups, CTO at growth-stage. Decision room: 1–3 for technical eval, expands to 5+ for commercial close at $5M+ ACV.

**What they buy today.**
- Together AI, Fireworks, DeepInfra, Anyscale, Modal, Replicate — Qwen 3 specifically is hosted on all of these at $0.30–0.80/M tokens.
- OpenAI / Anthropic for frontier-required tasks (kept, not displaced).
- Some self-host on H100 reserves at $50M+ ARR (Harvey, Hebbia, Sierra reportedly).

**KPIs.**
- COGS as % of revenue (the LLM inference line).
- Gross margin (target 70%+ at most growth-stage AI companies pre-IPO).
- Customer LTV / payback period.
- Inference latency p95/p99.

**Budget authority.**
- CTO can sign up to $1M/yr.
- CEO/CFO involvement at $1–10M.
- Board for >$10M.

**Trigger to switch.**
1. Preparing for fundraise where margins matter (most common — every Series C+ deck has a "path to gross margin" slide).
2. Specific anchor customer contract that needs better economics.
3. Frustration with API rate limits, capacity, or shared-tenant noisy neighbors.
4. Compliance need (HIPAA, SOC 2, FedRAMP) current provider doesn't meet.
5. New product line (agents, batch features) with a different cost shape.

**What they hate.**
- Inconsistent latency on shared infra.
- Surprise rate limits.
- Volume commits with weak SLA teeth.
- Slow support / no dedicated solutions engineer at the scale they're at.

**Why hard-lock to Qwen doesn't scare them.**
- They're already running Qwen 3 via a third-party for parts of their workload.
- Single-model lock is fine when the model is good enough and the price is 5x better.
- They can keep frontier-required calls on OpenAI/Anthropic; we're just the cheaper Qwen endpoint.

**How to reach them.**
- Bottoms-up: engineers find you via a "compare Qwen-3 batch providers" benchmark blog, run a test, escalate to CTO.
- AI engineering events: AI Engineer Summit, MLOps World, LangChain Interrupt, AI Engineer World's Fair.
- Public engineering blog from a peer ("how Harvey cut Qwen inference COGS 70%").
- Founder-led outbound to their CTO / VP Eng — the AI startup ecosystem is small enough that direct outreach works.

**The pitch.**
> "Cut your Qwen-served inference COGS 50–70% with a 1-day integration. Native Qwen 3 — no quantization tradeoffs, no shared-tenant noisy neighbors. OpenAI-compatible endpoint. US data residency, SOC 2 + HIPAA in scope, 3rd-party-audited weights. Reserved capacity for predictable cost. Migrate one workload in a week."

**Reference customer they trust.** Another named vertical AI platform (peer in adjacent vertical works), or a public engineering blog post from one of theirs.

---

### Tier D — US Single-Workload Enterprise (year 2)

**The buyer.** VP of AI / Head of Data + VP of Infrastructure + CISO + Procurement + sometimes a line-of-business sponsor. Reports to CTO, Chief Data Officer, or Chief Digital Officer. Decision room: 5–8 people including procurement.

**What they buy today.**
- AWS Bedrock Batch, Azure OpenAI Batch, GCP Vertex Batch — most common.
- Some self-host on H100 reserves with internal MLOps teams.
- Some direct contracts with OpenAI / Anthropic for marquee workloads.

**KPIs.**
- Project-level ROI (each AI project has its own business case).
- COGS as % of revenue for AI-augmented products.
- Vendor consolidation (CIO mandate to reduce sprawl).
- Security / compliance posture.

**Budget authority.**
- VP-level: $500K–$2M.
- SVP: $2M–$10M.
- C-level (CTO/CIO/CDO): $10M+.

**Trigger to switch.**
1. Specific batch use case hitting cost ceiling on cloud APIs (claims processing at insurers; contract review at banks; document extraction at pharma; catalog enrichment at retail).
2. Internal mandate to reduce inference COGS, often after Q1/Q2 board review.
3. New product or business line requiring batch processing at scale.
4. Capacity / throughput limits hit on incumbent provider.

**What they hate.**
- Per-token pricing volatility.
- Lack of capacity guarantees.
- Unclear data residency or model-origin disclosures.
- Non-negotiable SLAs from the hyperscaler.

**Why hard-lock to Qwen is acceptable for many — and a dealbreaker for some.**
- Tolerant: tech-forward F500 (Bloomberg, Stripe, Walmart, Shopify, retail/insurance/CPG) where the security review focuses on data residency (US-DC solves it) and the workload is non-frontier (Qwen suffices).
- Dealbreaker: banks under OCC/Fed scrutiny, healthcare under HHS, defense, federal — see Tier F.

**How to reach them.**
- Through hyperscaler marketplace (using their committed AWS/Azure/GCP spend — the dominant motion for enterprise AI procurement).
- Via SI partners at scale (Accenture, Deloitte, Slalom).
- Direct enterprise sales team (year 2+; 5–10 enterprise AEs minimum).

**The pitch.**
> "Cut your batch inference cost 5x on [specific Qwen-served workload]. US data residency, SOC 2 + HIPAA in scope, 3rd-party-audited Qwen weights. Buy via your AWS/Azure/GCP marketplace using committed spend. Pilot in 60 days, full deployment in 90."

**Reference customer they trust.** Another named F500 in their vertical, a hyperscaler endorsement, or a Tier C platform that *they* are themselves a customer of.

---

### Tier B — Hyperscaler Marketplace (downgraded to year 2)

**The buyer.** Director of AI Marketplace partnerships at AWS Marketplace / Azure Marketplace / GCP Marketplace. *Not* the strategic AI BD team — that's the conversation that doesn't apply under hard-lock. Decision room: 3–5 people (BD + technical PM + legal).

**What they have today.** Qwen 3 is already listed on every major marketplace via Together / Fireworks / DeepInfra / Anyscale / Modal. We'd be one more Qwen endpoint among many.

**KPIs.**
- Marketplace listing breadth.
- Customer acquisition through marketplace.
- Partner ecosystem health (active partners, transacted volume).

**Budget authority.** Marketplace deals are revenue-share, not capex. Sign-off at director-level for standard listings; SVP-level for exclusive arrangements.

**Trigger to sign.**
1. Customer demand for cheaper Qwen-batch endpoint that competitor providers can't match.
2. We offer exclusivity on a specific Qwen-batch tier in exchange for joint marketing.
3. Their first-party silicon (Trainium / TPU / Maia) doesn't run Qwen efficiently.

**The pitch.**
> "List us as the lowest-cost Qwen-3-batch endpoint on your marketplace. Revenue share 70/30 in your favor. We bring our own US DC and an OpenAI-compatible API. Optional exclusivity on the lowest-cost tier in exchange for marketing co-op."

**Reference.** Another marketplace's existing partnership with us, or named Tier C/D customers transacting through marketplace.

> **This is a marketing channel, not a strategic deal.** The original §3 framing of strategic capacity wholesale at $50M+ multi-year commit does not survive hard-lock.

---

### Tier E — Sovereign + International (year 3+, requires international DC)

Buyer is a government agency head, sovereign-cloud-company CEO (G42's Peng Xiao, HUMAIN executives, Yotta India leadership, GAIA-X consortium members), or partner SI. Cycle 18–36 months, often capex co-investment. Year-1 US-DC choice precludes most of these; need a second DC outside the US (Singapore, UAE, Ireland) before opening this segment.

**Why Qwen is actually a strong fit for non-aligned sovereigns:** Apache 2.0 license, large permissive community, non-US origin (which some sovereigns prefer over Llama or Gemma). Gulf states, India, parts of Southeast Asia have stated preferences for "non-US-controlled AI infrastructure." Once we have the international DC, sovereigns become high-margin natural buyers of the Qwen SKU. Worth seeding contact relationships in year 2 ahead of international DC build-out.

---

### Tier F — US Federal / Defense / Strict-F500 (year 3+, requires Gemma SKU)

Off-limits in years 1–2 because Chinese-origin model is structurally incompatible with their procurement policies. Year 3+ via Gemma SKU. Buyer is the procurement officer at a federal agency, defense prime, bank under OCC/Fed scrutiny, or healthcare system under HHS review. Sales cycle 12–24 months. Worth seeding contact relationships in year 2 ahead of Gemma SKU ship.

---

### Tier A — Foundation Model Labs (largely out)

Labs run their own custom-trained checkpoints. They cannot use a chip with Qwen weights baked in. **Possible niche:** distillation pipelines, eval, synthetic-data generation where a generic-quality public model is acceptable. Anthropic and OpenAI reportedly use Llama-class for some such pipelines and would substitute Qwen if economics demanded — but this is small-volume, the buyer is not central, and several labs have governance reasons to avoid Chinese-origin weights even when legally permitted. **Not a wedge segment under hard-lock.**

---

## 10. Sales Cycle Mechanics — Stage by Stage

**Q: From first meeting to first dollar of revenue, what does the cycle actually look like for each tier? Where does it break, and what compresses it?**

### Tier C: Vertical AI Cycle (the wedge — Target 4–12 weeks)

| # | Stage | Duration | Activity | Gate |
|---|---|---|---|---|
| 0 | Source | — | Inbound benchmark / blog / event / founder outbound | Engineering team agrees to test |
| 1 | Self-serve eval | 1–2 wks | Sign up, OpenAI-compatible endpoint, run their Qwen workload | Perf + cost validated by engineering |
| 2 | Pilot | 2–4 wks | Production workload at small scale | CTO sign-off |
| 3 | Contract | 1–4 wks | MSA, SLA, pricing tier | Signed |
| 4 | Migration | 1–4 wks | Cutover from Together / Fireworks / DeepInfra | Live |

**Total: 5–14 weeks. Median ~8 weeks.**

**Pre-ship variant (months 0–18, before silicon ships).** This cycle becomes a "design-partner agreement" — slightly longer (~12–16 weeks) because there's no live infrastructure to evaluate. Concludes with a signed reservation + small refundable deposit, not a paid contract. Goal: 2–3 named design-partner logos for slide 1 of every fundraise.

**Why it's fast post-ship.** AI startups are CTO-led, technically sophisticated, cost-pressured, and don't have procurement gates until $1M ACV. The product sells itself if perf and integration are real and Qwen is already running in their stack.

**Where it breaks:**
- Self-serve onboarding doesn't work in <30 minutes.
- Migration tooling absent (need OpenAI-compatible API + workload-replay shim).
- SLA terms weaker than incumbent.
- Their existing Qwen integration (via Together / Fireworks) is "good enough" — they need a cost trigger to switch.

### Tier D: US Single-Workload Enterprise (Target 6–12 months)

| # | Stage | Duration | Activity | Gate |
|---|---|---|---|---|
| 0 | Source | — | Hyperscaler marketplace surfacing, SI partner intro, direct outbound | VP-level meeting agreed |
| 1 | Discovery | 4–6 wks | Workload deep-dive, cost benchmark, security pre-read | Internal sponsor identified |
| 2 | Procurement / security | 8–16 wks | InfoSec review, SOC 2 / HIPAA validation, 3rd-party Qwen audit reviewed, vendor onboarding | Security clears, vendor approved |
| 3 | POC | 6–10 wks | Production-shaped pilot on a single workload | ROI validated |
| 4 | Commercial close | 4–8 wks | Pricing, term, SLA via marketplace contract vehicle | Marketplace order placed |
| 5 | Production ramp | 6–12 wks | Migration, integration with internal MLOps, observability | Live |

**Total: 28–52 weeks. Median ~38 weeks (~9 months).**

**Where it breaks:**
- **Stage 2 — security review on Chinese-origin model.** Some F500 will flat-no this regardless of US-DC. Mitigation: pre-publish 3rd-party Qwen audit; pre-empt the meeting with the security architecture doc.
- **Stage 4 — procurement unfamiliarity.** Mitigation: marketplace listing makes us a known SKU, not a new vendor onboarding.

**What compresses it:**
- Pre-listing on AWS / Azure / GCP marketplace (cuts procurement 4–8 weeks).
- Existing relationship at VP-of-AI level.
- A Tier C reference customer the buyer already knows.

### Tier B: Hyperscaler Marketplace Listing (Target 3–6 months)

| # | Stage | Duration | Activity | Gate |
|---|---|---|---|---|
| 0 | Source | — | BD intro via partner program | Marketplace partner team agrees to listing review |
| 1 | Technical integration | 2–4 wks | Marketplace-spec API, OpenAI-compatible endpoint | Integration passes their bar |
| 2 | Commercial framework | 4–8 wks | Revenue share, pricing model, support SLA | Term sheet signed |
| 3 | Compliance review | 4–8 wks | Their security / legal review of Qwen audit + DC | Pass |
| 4 | Listing go-live | 2–4 wks | Marketplace listing live | First customer transacts |

**Total: 12–24 weeks. Median ~16 weeks (~4 months).**

**Why this is so much shorter than the original "strategic capacity" cycle:** under hard-lock we're listing one model among many, not negotiating strategic wholesale. The cloud's BD team treats us as another marketplace partner, not a strategic co-investment.

**Where it breaks:** compliance team flagging Chinese-origin model. Same mitigation as Tier D — pre-published 3rd-party audit, US-DC, US-citizen ops.

### Tiers E and F: 18–36 months — year 3+

Tier E (sovereign / international) blocked until international DC. Tier F (US federal / defense / strict-F500) blocked until Gemma SKU. Both are year-3 motions; seed contact relationships in year 2 ahead of unlock.

### Tier A: Foundation Model Labs

Not pursued under hard-lock — labs run their own custom-trained checkpoints. Edge cases (lab using public-weights distillation pipelines) handled opportunistically via Tier C self-serve.

---

## 11. Pricing, Contracts & Commercial Terms

**Q: What's the pricing menu? What commitment + SLA structure? What's negotiable, what isn't?**

### The Pricing Menu

Pricing tiers, calibrated against the $0.016/M cost basis derived in §6 (mid-scenario at 17k tok/sec/user × 17 tok/sec/W). Strategy: expand-the-pie (aggressive pricing to unlock latent demand), not skim-the-cream:

| Tier | Pricing model | Target buyer | Output $/M | Input $/M | Commit | Margin target |
|---|---|---|---|---|---|---|
| On-demand | Per-token | Tier C long-tail, dev | $0.20 | $0.10 | None | 92% |
| Reserved (1-yr) | Throughput commit (tokens/sec floor, $/M billed) | Tier C anchor | $0.08 | $0.04 | 1 yr | 80% |
| Dedicated (3-yr) | Hardware reserve (per-rack, per-MW) | Tier C / D scaled | $0.05 | $0.025 | 3 yr | 68% |
| Enterprise via marketplace | Marketplace order via cloud commit | Tier D | $0.06 | $0.03 | 1–3 yr | 73% |
| Hyperscaler wholesale | Revenue share | Tier B marketplace | $0.03 | $0.015 | Marketplace term | 47% |
| Sovereign / co-built | Capex co-investment, profit share | Tier E (year 3+) | $0.15–0.30 (negotiated) | negotiated | 5–10 yr | 89–95% (capex offset) |
| Federal via Gemma | Reserved on Gemma SKU | Tier F (year 3+) | $0.15–0.40 (negotiated) | negotiated | 3–5 yr | 89–96% |

**Anchor headlines for the deck:** $0.20/M on-demand (vs. Gemini Flash batch at $0.20 — match price, win on Apache-2.0 license + US-DC + audited weights). $0.08/M reserved (vs. DeepInfra batch at $0.20 — clear 60% undercut at 80% margin). $0.03/M wholesale (sub-DeepSeek pricing; profitable; no competitor can match).

### Commitment Structure

- **1-yr reserved:** Default for first-pilot conversion. Volume commit (e.g., 100B tokens/month). 30% discount vs. on-demand.
- **3-yr reserved:** Standard for production. 50% discount vs. on-demand.
- **5-yr reserved:** Strategic deals, hyperscaler wholesale. 60% discount vs. on-demand.
- **Take-or-pay** on all reserved tiers: 80% of committed volume billed regardless of consumption. Standard cloud-infra term.

### SLA Structure

- **Throughput SLA:** Guaranteed tokens/sec floor at p95 of the billing period.
- **Job completion SLA:** 24h, 12h, 4h, 1h tiers. 24h is default; lower tiers priced 1.5–4x.
- **Uptime:** 99.5% on shared, 99.9% on dedicated, 99.95% on sovereign. Industry standard.
- **Quality:** No SLA on output quality (model-dependent, not infra-dependent). We commit to running the customer's exact model checkpoint, no quantization unless they approve in writing.

### Penalty / Termination Clauses

- **Service credits** for SLA misses: 10–25% of monthly invoice, scaling with miss severity.
- **Termination for cause:** SLA misses for 3 consecutive months allow customer to exit reserved without penalty.
- **Termination for convenience:** Available with 90-day notice + 50% buyout of remaining commit.
- **Non-compete:** None on either side. We don't restrict what models customer runs; they don't restrict what other customers we serve. (Important: don't accept exclusivity even from a marquee lab — it kneecaps the rest of the book.)

### Expansion Mechanics

- **Overage:** Above committed volume, billed at on-demand rate (incentive to upsize commit).
- **Auto-upgrade trigger:** 3 months of overage > 25% of commit prompts an upsize conversation.
- **Multi-region:** First region included; additional regions priced as separate dedicated capacity.
- **New models:** Customer can swap models within their reserved capacity at no additional cost (as long as the new model fits the hardware envelope).

### Margin Profile by Pricing Tier

The blend works because high-margin on-demand subsidizes lower-margin sovereign / hyperscaler-wholesale deals while letting us stay aggressive on the wedge pricing for labs.

> **Target blended gross margin at scale: 70–75%**, with 90%+ on the long-tail self-serve and 50–60% on the largest strategic deals.

That blend is best-in-class for infra (Snowflake ~67%, CoreWeave reserved book ~75%, Cloudflare ~78%).

---

## 12. Sales Team & Org Build-Out

**Q: Who do you hire when, what does ramp look like, and what's the comp structure?**

### Year 1 (Months 0–18, Pre-Ship): Founder-Led, Tier C Design Partners

**Hires:**
- **CEO leads design-partner sales.** Single-threaded on 2–3 Tier C wedge customers (Harvey, Hebbia, Glean, Sierra, Decagon, Cresta, Sema4, Sourcegraph). 60–70% of CEO time.
- **1 Sales Engineer / Forward-Deployed Engineer.** Handles workload integration, design-partner benchmarking on simulator. Profile: ex-Anyscale, ex-Together, ex-Anthropic Solutions, ex-Databricks AI specialist.
- **1 Head of Compliance & Trust** (month 6–9). Owns SOC 2, ISO 27001, 3rd-party Qwen audit, and the security-architecture-doc that pre-empts every Tier D security review. This is unusually senior for year 1, but the "Chinese model on US infra" framing forces it.
- **1 BD/partnerships lead (month 12).** Starts seeding hyperscaler marketplace conversations for year 2 listing.

**No quota carriers yet.** Founder-led design-partner motion is the right motion when there's no shipping product. Hiring a VP Sales before silicon ships is wasted comp.

**Year 1 revenue target: $0** (correctly zero — first chips ship month 18). Goal is 2–3 signed design-partner agreements with refundable deposits, plus 1 Tier D enterprise reservation contract.

### Year 2 (Months 18–30, Post-Ship): First VP Sales + Tier C/D Build-Out

**Hires (in order):**
1. **VP Sales (month 15–18, hires before chips ship).** Profile: ex-Snowflake / Databricks infrastructure sales, ex-CoreWeave, ex-Anyscale, ex-AWS Marketplace BD. Must have closed multi-million-dollar reserved-capacity or marketplace-listing deals. Reports to CEO.
2. **3–4 Enterprise AEs (month 18–24).** Each owns 5–10 named Tier C + Tier D accounts. Profile: ex-cloud / ex-data infra (Snowflake, Databricks, MongoDB, Confluent, HashiCorp).
3. **2–3 SEs (month 18–24).** Each pairs with 1–2 AEs.
4. **1 BD lead for hyperscaler marketplace listings** (already in seat from year 1).
5. **1 Customer Success lead (month 24).** Owns renewal + expansion across the design-partner book.

**Year 2 revenue target:** $25–75M ARR run-rate by month 30.

### Year 3 (Months 30–42): Scale to 10–15 Quota Carriers + Year-3 Segment Prep

**Hires:**
- 5–7 more enterprise AEs across US verticals.
- 4–6 SEs.
- 1 Director of Sales Ops (forecasting, comp design, CRM hygiene).
- 1 Director of Customer Success.
- 1 Director of Marketing.
- **1 Sovereign-relations lead (month 30–36)** — seeds Tier E ahead of international DC. Profile: ex-G42, ex-Palantir, ex-government, ex-Booz Allen.
- **1 Federal / regulated-enterprise lead (month 36)** — seeds Tier F ahead of Gemma SKU ship.

**Year 3 revenue target:** $200–500M ARR run-rate by month 42.

### Year 4+ (Months 42–60): International + Federal + Multi-SKU

Add 8–10 more AEs across new geographies (UAE/KSA, EU, Singapore) and federal-cleared roles. **Year 4 revenue target:** $1–2B ARR run-rate by month 60.

### Comp Structure

Standard infra-sales:

- **AE:** 50/50 base/variable, OTE $300–500K, accelerators above quota. Quota: $5–8M ARR/yr at full ramp; $15–30M for hyperscaler-focused AEs.
- **VP Sales:** OTE $400–700K, equity heavy, comp tied to contracted ARR.
- **SE:** 70/30 base/variable, paid on AE quota attainment.
- **BD:** 60/40 base/variable, paid on contracted ARR + strategic deal milestones.
- **Spiff structure:** First-deal accelerators for marquee logos (lab, hyperscaler, sovereign). Pay 2x on the first 3 deals in each segment to incentivize the hard early wins.

### Channel Strategy: Direct, Partner, or Both

**Year 1–2: Direct only.** Founder-led + small enterprise team. Hyperscaler partnerships are a sales channel, not a reseller relationship — we keep customer relationship and the contract.

**Year 3+: Add SI / consulting partners.** Accenture, Deloitte, Slalom for Fortune 500. Localized SIs for sovereign markets (Presight in UAE, Atos in EU, TCS/Infosys in India).

> **Never give up customer ownership.** This is the AWS lesson: customer relationship is the moat. Channel partners drive deals; we own the contract and the renewal.

---

## 13. What Compounds — The Moats Beyond the Chip

**Q: The HW lead is 3–5 years. What compounds during that window so we have a business after NVIDIA closes the gap?**

### Five Compounding Assets

**1. Contracted capacity book.**
Every reserved-capacity deal is a 3–7 year revenue annuity that survives even if the next-gen NVIDIA chip is cheaper to operate. CoreWeave's contracted backlog ($X0B as of 2025) is the model — contracted revenue is the single most powerful financing instrument in this category.

**2. Datacenter footprint and power agreements.**
Power-purchase agreements (PPAs) at scale take 24–36 months. Once we own the power and the site, that's a moat against new entrants regardless of chip generation. **The DC is a 20-year asset, the chip is a 5-year asset.**

**3. Serving stack + scheduler + observability.**
The software stack to extract 80%+ utilization from our HW (continuous batching, KV-cache management, prefix sharing across customers, deadline scheduling, fair-share, multi-tenant isolation) is **2–3 years of focused R&D**. NVIDIA's stack (TensorRT-LLM, NIMs) is built for general-purpose; ours is built for batch-at-scale. Even on commodity HW post-window, the SW alone is differentiated.

**4. Customer-specific optimization.**
Each large customer's workload (Anthropic's synthetic data shape, Bloomberg's document shape, Pfizer's molecule-screening shape) gets co-tuned with our compiler and scheduler. After 18 months of co-engineering, switching cost is 6–12 months of re-integration. **Switching costs compound silently.**

**5. Sovereign and regulatory certifications.**
SOC 2 Type II, FedRAMP Moderate/High, ISO 27001, in-country data residency for GDPR/UAE/Saudi/India. These take 12–24 months each. Once stacked, they're a permission slip that new entrants don't have.

> **The HW wins the first three years. The contracted book + DC footprint + SW stack + certs win the next twenty.**

---

## 14. Risks and Kill-Shots

**Q: Name them honestly. What's the most likely path where this doesn't work, and what's the leading indicator?**

### Tier 1 Risks (named on slide 12 of the deck)

**1. NVIDIA Rubin closes the perf/W gap faster than expected.**
- Indicator: Rubin specs leak by mid-2026 showing >2x perf/W vs. Blackwell, with custom inference-mode SKUs.
- Mitigation: Lock 5-year reserved-capacity contracts before Rubin ships.

**2. Frontier API prices collapse.**
- Indicator: GPT-5 mini batch at <$0.20/M tokens.
- Mitigation: Stay below frontier batch pricing by 3–5x at all times. Maintain open-weight distillation as the cheaper alternative.

**3. Hyperscaler batch tiers commoditize.**
- Indicator: AWS announces "Bedrock Batch Reserved" at <$0.10/M tokens for Llama-70B-class.
- Mitigation: Be the back-end *for* AWS Bedrock Batch (Tier B sale). Symbiosis, not competition.

**4. Qwen 3 obsolesced before chip pays back.**
- Indicator: Qwen 4 ships at substantially better quality inside the year-1-to-year-3 window; customers migrate workloads off Qwen 3.
- Mitigation: Qwen-N refresh tape-out at month 12 (ships month 30) — keeps the SKU on the current Qwen generation. Architecturally hardest-hitting risk under hard-lock; needs explicit Series B/C disclosure.

**5. Open-weight model momentum stalls in the West.**
- Indicator: Meta deprioritizes Llama; Qwen/DeepSeek face deeper US restrictions; Mistral falters; Western enterprise retreats to closed APIs.
- Mitigation: Asia-Pacific markets are likely to retain open-weight momentum even if US slows; international DC (year 3) becomes a hedge.

**6. "No Chinese AI" wave hits enterprise faster than expected.**
- Indicator: A high-profile incident (e.g., a security flaw in any Chinese model) triggers F500 procurement to blanket-ban Chinese-origin models — including Qwen on US-DC.
- Mitigation: Accelerate Gemma SKU tape-out; aggressive 3rd-party audit publication; "American DC + American audit" framing as the table-stakes defense; year-1 wedge (Tier C startups) is least exposed to this wave.

**7. US export-control rule tightens to capture chip-with-Chinese-weights.**
- Indicator: BIS / Commerce updates 3A090 or 4A005 to specifically address embedded foreign AI models; or a new ECCN created.
- Mitigation: Engage export-controls counsel pre-tape-out; design HW for re-flashable weights as an emergency fallback if regs force it.

**8. Workloads shift to on-device (cannibalizing batch DC demand).**
- Indicator: Apple/Qualcomm ship NPUs that handle 30B-class models at watt-hours per task.
- Mitigation: Workstation/edge thesis hedge — the company should be diversified across DC, workstation, and edge under one local-AI umbrella. (See `workstation/gpu-workstation-thesis-and-vertical-selection.md`.)

### Tier 2 Risks

**9. Capital scarcity in down-cycle.** $1B+ buildout requires equity + debt markets to stay open. Pre-revenue years 0–18 are especially exposed. Hedge: stage gates, large early rounds (Series A+B at $150–250M to fund through ship), strong design-partner reservations as derisk.
**10. Talent.** Inference HW + SW is a tight talent market. NVIDIA, AMD, Google TPU, AWS Trainium, Apple Silicon are all hiring. Hedge: anchor team in geographies (Toronto, Tel Aviv, London) where competition is less acute.
**11. Power constraints.** 10MW+ DC sites have 18–36-month energization timelines. Hedge: start power conversations at month 0; consider Texas-stranded-gas or behind-the-meter sites; modular DC designs.
**12. Regulatory shock.** AI safety regs, energy quotas, state-level "Chinese AI" laws. Hedge: monitor state legislative activity; pre-position the audit + framing; California / Texas / NY are the watch states.

### What Makes This Not the SGI Story

The historical analog people will worry about: **specialized silicon companies always lose to general-purpose silicon** (SGI to commodity x86, Sun to commodity Linux, Ageia to NVIDIA).

The counter: **we're not selling silicon. We're selling capacity.** Customers don't care what's in the box — they care about $/M tokens and SLA. As long as we own the DC and the contracts, the silicon is an internal cost-of-goods question, not a product-category question. If our chip stops winning in 2030, we replace it with whatever wins next inside the same DC, behind the same API, on the same contracts. **The customer doesn't notice.**

This is the AWS-vs-Sun lesson, applied correctly.

---

## 15. The 18 / 30 / 42 / 60-Month Roadmap

**Q: What does the financing/proof-point ladder look like? What needs to be true at each milestone to unlock the next round?**

### Month 0–18: Tape-Out + Design Partners (Pre-Ship)

**Goals:**
- Spec freeze month 0; first silicon back month 12; volume ship month 18.
- DC site selected, power & colo signed (Texas / Phoenix / Ohio candidates).
- 2–3 Tier C design-partner agreements signed (Harvey-class, Hebbia-class).
- 1 Tier D enterprise reservation signed ($5–25M deposit).
- Compliance stack live before GA: SOC 2 Type II, ISO 27001, 3rd-party Qwen audit published.
- **Revenue: $0. Burn: ~$50–100M.**

**Capital:** Series A + B, ~$150–250M total. Funds NRE + DC site preparation + key hires + compliance build.

### Month 18–30: First Ship + First Revenue + 10MW

**Goals:**
- Volume silicon shipping; first MW–10MW DC online.
- 2–3 design-partner customers in production; co-marketing case studies published.
- 5–10 additional Tier C customers signed.
- First Tier D enterprise live via hyperscaler marketplace.
- Qwen-N refresh tape-out begins.
- **Revenue: $25–75M ARR by month 30.**

**Capital:** Series C, ~$300–500M equity + $100–200M project debt against contracted revenue.

### Month 30–42: Scale + Marketplace + Gemma Tape-Out

**Goals:**
- 10–25MW across 2 sites.
- AWS Marketplace + Azure Marketplace listings live.
- 15–25 Tier C customers; 3–5 Tier D enterprises.
- Gemma SKU tape-out begins (month 36).
- Qwen-N refresh ships (~month 42).
- **Revenue: $200–500M ARR by month 42.**

**Capital:** Series D, ~$500M–$1B equity + $500M–$1B project debt.

### Month 42–60: Multi-SKU + International + Federal-Adjacent

**Goals:**
- Gemma SKU ships (~month 54); first US federal / regulated-F500 sales.
- International DC online (Singapore / UAE / Ireland) — opens Tier E sovereign.
- 100MW+ contracted capacity.
- 30+ Tier C customers; 10+ Tier D enterprises; 1 anchor sovereign; 1–2 federal-adjacent F500.
- **Revenue: $1–2B ARR by month 60.**

**Capital:** Series E or pre-IPO; ~$1B+ equity + $2B+ project debt. At this scale, financeable like CoreWeave (contracted revenue is collateral).

### Milestones That Unlock Each Round

| Round | Cap. | Unlock metric |
|---|---|---|
| Series A | $75–125M | Tape-out spec frozen + 2 LOI design partners + DC site MOU |
| Series B | $100–150M | First silicon back from fab + 2–3 signed design partners + compliance stack on track |
| Series C | $300–500M | First 1MW shipping + first paying customers + $10M+ ARR |
| Series D | $500M–$1B | $200M+ ARR contracted + marketplace listings live + Qwen-N refresh tape-out underway |
| Series E / Pre-IPO | $1B+ | $1B+ ARR contracted + Gemma SKU shipping + international DC online + first sovereign deal |

---

## 16. The One-Slide Pitch (Working Draft)

> **We're building the first US datacenter where silicon is hard-locked to a single open-weight model — Qwen 3 first, weights baked in at tape-out, 17,000 tokens/sec per user. The result is **$0.016/M output cost** — roughly 12x below the cheapest Qwen provider in market today, and 6x below DeepSeek's at-cost pricing. We don't undercut. We make tokens too cheap to meter. At $0.08/M anchor pricing we run 80% gross margin while unlocking workloads that are uneconomic today (semantic enrichment of every CRM record, batch agents on every customer, full-corpus re-embedding nightly). Wedge buyer: US AI startups cutting inference COGS for fundraise. Year 2: F500 single-workload via hyperscaler marketplaces. Year 3: Gemma SKU for US federal, international DC for sovereign. Path to $25–75M ARR at month 30 (first revenue, post-ship), $200–500M at 42, $1–2B at 60. The HW is the 5-year wedge; the contracted book + DC footprint + compliance stack + multi-SKU roadmap is the 20-year business.**

---

## Open Questions / Next Steps

### Architecture & SKU

- **Internal HW spec stake-down.** Replace the `[TBD]` placeholders in §5 and §6 with real numbers from the silicon team — Qwen-3-specific tokens/sec/W, tokens/sec/$, $/M tokens at 80% utilization. The unit-economics model is everything; needs to be defensible to a Tier C CTO in week 1.
- **Qwen version & param-size selection.** Which Qwen 3 variant gets baked in first? Qwen3-32B (mainstream, fast), Qwen3-72B (top quality), Qwen3-MoE (efficiency)? Tradeoff: higher param = higher quality per token but lower tokens/sec/chip. Pick the variant with the largest 3-year token-volume forecast.
- **Qwen-N refresh trigger.** What's the rule for taping out the next-gen Qwen chip? Quality delta? Volume signal? Customer ask? Lock the trigger now to avoid year-2 paralysis.
- **Gemma vs. Llama for the year-3 US-procurement SKU.** Decision deferred to month 18, but the leading indicator is whether Gemma 3 closes the quality gap. Set up a quarterly bench review.
- **Kimi go/no-go.** Tape-out only on $50M+ pre-commit from a long-context whale. Identify the 3 most-likely whales now (national archives, global law firms, pharma patent processors).

### Compliance & Geopolitical

- **3rd-party Qwen audit scope & vendor.** Trail of Bits vs. NCC Group vs. academic team. Scope: prompt injection, weight integrity, refusal patterns on US-relevant adversarial prompts, data-exfiltration vectors. Publish before commercial GA.
- **US export-control classification of the chip.** Engage export-controls counsel early — the chip-with-baked-Chinese-weights category is novel. Likely 3A090 dual-use, possibly 4A005. Get a written opinion before tape-out, not after.
- **Procurement-policy mapping for Tier D buyers.** For top 50 F500, document which have explicit "no Chinese AI" policies and which are silent. The silent ones are the year-2 wedge. Banks and healthcare under federal-adjacent regulators are likely no-go regardless of DC.
- **The "American DC, Chinese model" narrative test.** Run the framing past 3 friendly comms / policy advisors before fundraising. If the narrative breaks under scrutiny, the year-1 GTM does too.

### GTM Operations

- **Tier C design-partner targeting list.** Top 12 vertical AI platforms ranked by (a) Qwen 3 spend today, (b) cost-pressure signals (recent fundraise, public margin commentary), (c) warm-intro availability. Harvey, Hebbia, Glean, Sierra, Decagon, Cresta, Sema4, Bland AI, Sourcegraph, Cursor, Notion AI, Perplexity are the obvious 12.
- **Workload trace request playbook for design partners.** Standard NDA + trace spec + sim-results template, repeatable in <2 weeks per partner.
- **DC site shortlist.** 3 candidates for the year-1 1–10MW deployment: Texas (cheap power, slow energization), Phoenix (good power, water risk), Ohio (rising AI hub, available capacity). Rank by power price × time-to-energization × talent proximity.
- **Hyperscaler marketplace prioritization.** Of AWS / Azure / GCP, which has the structural incentive to feature a Qwen-3 lowest-cost endpoint? Hypothesis: GCP first (Anthropic-aligned, less Trainium-loyalty), then Azure, then AWS. Validate via 3 BD conversations.
- **VP Sales hiring spec.** Profile: ex-Snowflake / Databricks / CoreWeave / Anyscale / AWS Marketplace BD. Target list of 30 candidates. Hire month 15–18 (before chips ship), recruiting starts month 12.
- **Pricing-card stress test.** Run the §11 pricing menu past 3 friendly Tier C CTOs (Harvey-class). Their pushback is the highest-signal pre-launch input.
- **SLA legal review.** §11 termination + service-credit + data-handling clauses need an infra-experienced legal counsel draft before any term sheet (Cooley, WSGR, Gunderson).

### Companion docs to write next

- **Archetype 3 (agentic inference).** Long-context / multi-turn agent workload as the next-SKU candidate after Qwen-N refresh. May or may not justify a separate hard-lock SKU.
- **Bridge to the workstation thesis.** Where does the local-AI / desktop-GPU side of the business connect to the inference DC? Same company narrative or two products?
- **Model-selection deep-dive.** Standalone analysis of Qwen vs. Gemma vs. Kimi vs. Llama vs. DeepSeek as hard-lock candidates, with explicit lifespan-risk modeling per model.

*Document compiled 2026-04-29. Major restructure to reflect hard-lock + Qwen + US-DC architecture (was originally drafted under model-agnostic + lab-wedge assumptions).*
