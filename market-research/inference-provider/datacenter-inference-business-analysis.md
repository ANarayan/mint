# Datacenter Inference as a Business — Selling Tokens on Proprietary Silicon

*A business analysis for the proprietary-chip datacenter inference play: who buys inference tokens today, where they buy from, what the proprietary chip's cost basis enables, which workloads are the most promising wedges, and what the unit economics and competitive position look like against incumbents (frontier API, hyperscaler resale, inference-as-a-service, on-prem) — through the 2026–2030 window.*

> Companion to `../llm-inference-workloads/llm-inference-workloads-taxonomy.md` (workload taxonomy) and `../llm-inference-workloads/accelerator-workload-fit-analysis.md` (accelerator-workload fit + cost basis). This document takes those as given and builds the business case for the *datacenter inference* archetype — selling tokens on the proprietary chip at scale.

> Adjacent to `batch-inference-business-analysis.md` (archetype 1), `agentic-inference-business-analysis.md` (archetype 3), and `tokens-too-cheap-to-meter-thesis.md` (cost-curve framing). Where those archetype docs choose a workload class and build a business around it, this one starts from the chip and asks: given that cost basis, which slice of the inference market is most lucrative to enter and how?

---

## 1. The Inference Market — Segments and Buyers

**Q: How big is the inference market in 2026, and how does it segment?**

The aggregate enterprise + consumer LLM inference spend in 2026 is on the order of **$50–70B globally**, with five structurally distinct segments that buy through different channels, accept different latency / quality / sovereignty tradeoffs, and pay different per-token prices. The proprietary-chip business needs to choose which segments to enter, in what order, and through which channel.

### Segment A — Frontier API direct (OpenAI / Anthropic / Google)

> Anchor: ~$25–40B in 2026. The largest segment by revenue, dominated by three vendors selling premium-quality reasoning at premium prices.

**Who they are.** OpenAI (~$15–18B annualized as of mid-2026), Anthropic (~$5–8B), Google Gemini API (~$3–5B). Sells GPT-5 / GPT-5.5, Claude Opus 4.7 / Sonnet 4.6 / Haiku 4.5, Gemini 2.5 / 3 Pro / 3 Flash directly through their APIs.

**Buyers.**
- **App-layer AI startups** that depend on frontier reasoning quality: Cursor, Anysphere ($1.2B ARR), Sierra ($1B+), Decagon, Glean, Notion AI, Perplexity Pro, Hebbia, Harvey, Jasper, Writer.
- **F500 R&D / innovation budgets** experimenting with frontier-only capabilities (multi-doc legal review, deep research, advanced multimodal).
- **Per-seat assistant platforms** (Microsoft Copilot 365 routing portion of traffic to OpenAI direct; Slack AI, Zoom AI Companion, Salesforce Einstein partial reroutes).
- **Top-5 frontier labs themselves** (synthetic data generation, evaluation, internal R&D).

**What they buy.** Premium-priced tokens with implicit frontier-reasoning quality bundle. Pay $1.25–$25/Mtok output depending on tier, usually with prompt caching at 90% discount on cached input.

### Segment B — Hyperscaler resale (Bedrock, Vertex, Azure OpenAI)

> Anchor: ~$15–25B in 2026. Same models as Segment A, sold inside the customer's existing cloud contract for VPC, IAM, audit, and procurement reasons.

**Who they are.** AWS Bedrock (multi-vendor: Anthropic, Meta, Cohere, AI21, Amazon Nova, Mistral); Google Vertex AI (Gemini, Anthropic, Llama, Mistral); Azure OpenAI (GPT-5, GPT-5.5, OpenAI legacy).

**Buyers.**
- **Regulated F500** that won't ship customer data to OpenAI/Anthropic direct: banks (JPMC, Wells, Citi), insurance (Allianz, Zurich, Liberty), healthcare (Anthem, Humana, Memorial Sloan Kettering), pharma (Pfizer, Merck), defense (Lockheed, Northrop, Raytheon — via GovCloud), government (DoD JWCC contracts, federal civilian agencies).
- **F500 lakehouse customers** running inference inside Bedrock Batch / Vertex Batch / Databricks FM Serving for ETL, support-ticket digesting, document understanding.
- **Workspace customers** routing M365 Copilot / Google Workspace AI / Salesforce traffic through their existing cloud contract.

**What they buy.** Same frontier list price (or slight markup) plus a "compliance/procurement tax" — but the buyer values the VPC-native, in-region, audit-logged, IAM-integrated delivery enough to pay it. Bedrock Batch at 50% off list is the dominant SKU for high-volume regulated workloads.

### Segment C — Inference-as-a-Service / GPU cloud

> Anchor: ~$3–6B in 2026 and growing at 100%+ YoY. The "open-source serving" tier — competitive on $/Mtok and TTFT.

**Who they are.** Together AI ($300–500M ARR estimate), Fireworks ($150–250M), Groq (custom-silicon sub-segment, $300M+ tokenization), Cerebras (similar), Lambda Labs Inference ($200M+), DeepInfra (cost-leader), Modal (serverless), Baseten (model-serving), Replicate (model marketplace), Anyscale (Ray-based), RunPod (raw GPU rental). Plus white-label / aggregator OpenRouter ($50M ARR on 5% take rate, implying ~$1B+ routed token spend).

**Buyers.**
- **OS-model power users** wanting Llama 3.3 70B / Qwen3-72B / DeepSeek-V3 / Mistral Large at 10× lower $/Mtok than frontier API.
- **Voice-agent platforms** (Vapi, Retell, Bland) demanding sub-300ms TTFT — Groq and Cerebras own this slice.
- **Companion / roleplay apps** running uncensored open-weight models (Character.AI alternative long tail, Janitor, Spicychat — high pass-through to OpenRouter Chinese models).
- **Cost-conscious app-layer startups** that don't need frontier (RAG over enterprise corpus, content moderation, classification, batch ETL).
- **Embedding / re-ranking workloads** routing through DeepInfra / Cloudflare Workers AI for sub-cent-per-Mtok serving.

**What they buy.** OS-model tokens at $0.05–$1/Mtok (typical). The buyer chose this segment because the workload doesn't justify frontier reasoning OR latency requirements lock out hyperscaler-resold serving.

### Segment D — Enterprise on-prem and in-VPC inference

> Anchor: ~$5–15B in 2026 (capex + operational cost). The largest customers running their own GPUs to escape per-token markup.

**Who they are.** Largest banks (JPMC has 18,000+ H100 equivalents in self-disclosed AI infra), telcos (AT&T, Verizon), insurance (Allianz, AXA), pharma (Pfizer, Novartis), defense, large healthcare systems. Plus "private inference platforms" — Databricks Mosaic AI, Snowflake Cortex, IBM watsonx, NVIDIA DGX Cloud, Dell APEX AI, HPE Private Cloud AI — that sell on-prem-equivalent inference under managed contracts.

**Buyers (and their reasons).**
- **Sovereign data**: HIPAA, PCI, GDPR, regulatory disclosure prevent cloud routing.
- **Cost at scale**: F500 with >1B tokens/month on a workload finds Bedrock/Vertex/Databricks markup uneconomic vs. running their own H100/H200/Trainium2/MI300X cluster.
- **Latency / co-location**: trading firms, telco voice, real-time recommendation systems where round-trip-time to a hyperscaler region kills the use case.
- **IP protection**: not wanting prompts/outputs to land in any third-party log.

**What they buy.** Hardware + software + power. Inference $/Mtok is internal accounting only.

### Segment E — Edge / on-device inference

> Anchor: <$1B revenue today but growing fast. Mostly device-OEM-bundled.

**Who they are.** Apple Intelligence (on-device for Pro models, private-cloud-compute for harder), Google Gemini Nano on Pixel, Qualcomm AI on Snapdragon X, AMD Ryzen AI, NVIDIA DGX Spark and equivalents, Tinybox.

**Buyers.** Smartphone OEMs, PC OEMs, prosumer AI hobbyists, vertical workstation buyers (covered separately in `../workstation/gpu-workstation-business-analysis.md`).

**Why the proprietary-chip DC business probably skips this segment.** Edge inference is bundled into hardware sales, not sold per-token; competing here means a different business model entirely. **In scope for this analysis: Segments A–D.**

### Segment summary

| Segment | 2026 size | Dominant buyers | Typical $/Mtok | Why this segment exists |
|---|---|---|---|---|
| A — Frontier API direct | $25–40B | App-layer AI cos, F500 R&D | $1.25–$25 | Frontier reasoning quality |
| B — Hyperscaler resale | $15–25B | Regulated F500, lakehouse customers | $0.50–$30 (with batch discount 50%) | VPC, audit, procurement |
| C — Inference-as-a-Service | $3–6B | OS-power users, voice, companions | $0.05–$1 | OS-model serving + low TTFT |
| D — Enterprise on-prem / in-VPC | $5–15B | Banks, telco, insurance, pharma, defense | $0 marginal (capex amortized) | Sovereignty, scale economics |
| E — Edge / on-device | <$1B | Device OEMs | bundled into HW | Privacy, offline, hardware sales |

> Compressed: **the proprietary-chip DC business goes to market against Segments B and C in year 1, expands into Segments A (workloads where buyer trust permits OS substitution) and D (managed-private offering) in years 2–3.**

---

## 2. How Each Segment Is Served Today — The Existing Stack

**Q: Which vendors do the buyers in each segment actually buy from, and how does the money flow?**

### Segment A — Frontier API direct

Money flows: **buyer → OpenAI/Anthropic/Google → silicon (Azure NVIDIA / AWS Trainium / Google TPU)**.

- **OpenAI** runs on Azure (Microsoft committed $100B+ in compute), with workloads hosted on H100/H200/B200 GPUs and the recent first-tranche of Stargate-class capacity. Some routing to Oracle (Stargate) and CoreWeave starting 2026.
- **Anthropic** is split between **AWS Trainium 2 / Trainium 3** ($100B Amazon commit through 2030) and **Google TPU v5p / Ironwood** (Oct 2025 deal "up to 1M TPUs"). Recently growing TPU share for Claude consumer; Trainium for enterprise Bedrock workloads.
- **Google Gemini** runs entirely on **TPU v5p / Ironwood / TPU 8i** (announced Cloud Next April 2026).

Gross margins of the frontier labs themselves are roughly **50–65% on inference revenue** (estimated; not publicly disclosed at workload granularity). Subsidized by training-cost amortization. Price-setting for the entire industry.

### Segment B — Hyperscaler resale

Money flows: **buyer → AWS / Azure / GCP → frontier-lab royalty + hyperscaler infrastructure margin → silicon**.

- **Bedrock** lists Claude / Llama / Mistral / Cohere / Nova at standard list prices (Sonnet 4.6 = $3/$15/Mtok, same as Anthropic direct). Anthropic earns a royalty (estimated 60–80% of list); AWS earns the remainder plus DC margin. Bedrock Batch is 50% off list and is the dominant SKU for regulated F500 high-volume workloads.
- **Vertex AI** lists Gemini at parity with Google AI direct, plus third-party (Anthropic, Llama, Mistral) at similar economics to Bedrock. Vertex Batch at 50% off.
- **Azure OpenAI** runs on dedicated PTU (provisioned throughput units) for enterprise customers. PTUs are sold for fixed monthly fees, providing dedicated throughput at typically 20–40% effective discount over pay-as-you-go list. Microsoft's enterprise AI customers are funneled here.

Hyperscaler gross margin on this resale segment is roughly **30–50%** after frontier-lab royalty — call it $0.5–$1.50/Mtok captured by AWS/Google/MSFT on a $3 input price.

### Segment C — Inference-as-a-Service

Money flows: **buyer → IaaS → GPU rental + custom silicon + DC + power**.

- **Together AI, Fireworks, DeepInfra, Anyscale, Modal, Baseten, RunPod**: rent H100 / H200 / B200 from CoreWeave / Lambda / Crusoe / Together's own cluster, run vLLM / SGLang / TRT-LLM, sell per-Mtok at typically 3–10× markup over BOM.
- **Groq, Cerebras, SambaNova**: own custom silicon (LPU, CS-3, RDU). Sell per-Mtok at price points competitive with H100 IaaS but with dramatically better TTFT (Groq 315 t/s on 70B; Cerebras 1,800+ t/s).
- **OpenRouter**: pure aggregator / router on a 5% take rate. Routes to whichever provider has the cheapest viable serving for a given model. Reports Chinese OS models hit 61% of token volume in February 2026.

Gross margins for IaaS on rented GPUs: **20–40%** (need to cover the H100 rental rate plus their own ops). Groq/Cerebras: harder to estimate publicly; capex-heavy with potentially higher long-run gross margin once the silicon is amortized.

### Segment D — Enterprise on-prem / in-VPC

Money flows: **buyer → NVIDIA / AMD / hyperscaler private offering → DC + power + ops**.

- **Direct H100/H200/B200/MI300X procurement** — JPMC, banks, telcos buy directly from NVIDIA (or via OEM resellers like Dell, HPE, SuperMicro), deploy in own DC with own ops team running vLLM/SGLang/TRT-LLM.
- **Managed-private** — Databricks Mosaic AI, Snowflake Cortex, NVIDIA DGX Cloud, IBM watsonx run on customer-pay-for-it-but-vendor-manages-it model. Customer pays vendor a managed-service premium.
- **Hyperscaler private** — AWS Outposts, Google Distributed Cloud, Azure Stack — hyperscaler DC delivered to customer's premises.

Effective $/Mtok COGS to the customer is whatever they amortize their capex + opex to — typically **$0.04–$0.10/Mtok for 8B serving on H100 at realistic 30–50% utilization**, much higher for 70B / 405B class.

---

## 3. The Proprietary Chip — Cost Basis

**Q: What does the proprietary chip cost per token, how does that compare to the existing market BOM, and which model classes does it serve?**

The chip serves **8B dense, 30B dense, 100B-MoE** decoder models plus **BGE-L** (encoder embedding) and **SAM-H** (vision segmentation) specialty SKUs. Cost basis at four process nodes:

### Decoder LLM SKUs ($/Mtok input/output)

| Tier | Node | Input $/Mtok | Output $/Mtok | Blended (Pf 3:1) $/Mtok |
|---|---|---:|---:|---:|
| 8B | N16 | $0.003658 | $0.003853 | $0.003707 |
| 8B | N7 | $0.002408 | $0.002536 | $0.002440 |
| 8B | N5 | $0.001694 | $0.001848 | $0.001732 |
| 8B | N3 | $0.001297 | $0.001595 | **$0.001371** |
| 30B | N16 | $0.024163 | $0.025524 | $0.024503 |
| 30B | N7 | $0.010942 | $0.011558 | $0.011096 |
| 30B | N5 | $0.007436 | $0.007856 | $0.007541 |
| 30B | N3 | $0.005121 | $0.005409 | **$0.005193** |
| 100B-MoE | N16 | $0.001387 | $0.003449 | $0.001902 |
| 100B-MoE | N7 | $0.000750 | $0.001436 | $0.000921 |
| 100B-MoE | N5 | $0.000440 | $0.001254 | $0.000643 |
| 100B-MoE | N3 | $0.000285 | $0.001014 | **$0.000468** |

### Specialty SKUs

| Model class | Unit | N16 | N7 | N5 | N3 |
|---|---|---:|---:|---:|---:|
| BGE-L (encoder embedding, 335M) | $/Mqueries | $0.0345 | $0.0185 | $0.0135 | **$0.0101** |
| SAM-H (vision segmentation, 636M) | $/Mimages | $3.3415 | $1.7059 | $1.2233 | **$0.8953** |

### Reference: H100-served BOM (rough; not the proprietary chip)

For comparison, an H100 at $30K capex over a 5-year amortization at $2.50/hr pricing serves ~5K tok/s for 8B inference at ~30% utilization. Effective BOM is **~$0.04–$0.10/Mtok for 8B**, **~$0.30–$0.80/Mtok for 70B**, and **higher for MoE depending on expert routing efficiency**.

### What's included in the chip $/Mtok numbers above

The $/Mtok values in the tables above are the **all-in TCO output** of the proprietary chip's costing model, not silicon-BOM-only. They include silicon amortization, per-chip non-silicon overhead, DC buildout amortization, electricity, and per-chip OpEx. NRE is excluded by default. The full TCO model parameters are described in §5.

### Headline cost-basis comparisons

- **8B at N3 BOM**: $0.001371/Mtok (Pf 3:1 blended). vs. H100-rented BOM ~$0.05/Mtok → **~36× cheaper**.
- **100B-MoE at N3 BOM**: $0.000468/Mtok blended. vs. Llama-70B Together list price $0.88 → **~1,880× cheaper**.
- **BGE-L at N3 BOM**: $0.0101/Mqueries. vs. DeepInfra BGE-large at $2.56/Mqueries (256-tok chunk) → **~253× cheaper**.
- **SAM-H at N3 BOM**: $0.8953/Mimages. vs. Replicate-hosted SAM-H ~$1,500/Mimages → **~1,675× cheaper**.

### Key structural property: 100B-MoE is the dominant SKU at every node

The 100B-MoE is **cheaper than 30B dense and cheaper than 8B dense at every node and every P:D ratio**, because MoE active-params-per-token is bounded (~22B) while expert routing amortizes total parameter weight-streaming. Strategic implication: **the product line collapses to two decoder SKUs (8B for latency-bound, 100B-MoE for everything else) plus the two specialty SKUs**.

For full per-workload multipliers across N16/N7/N5/N3, see `../llm-inference-workloads/accelerator-workload-fit-analysis.md` §7.

---

## 4. The Top-5 Most Promising Workload Segments

**Q: Of the addressable workloads, which five are the highest-priority go-to-market wedges, and what is the per-token cost improvement we deliver?**

Selection criteria — combining accelerator fit, market size, GTM credibility, and customer concentration:

1. **Buyer pool is large and named** — we can list 10+ specific customers to cold-call.
2. **Quality-trust hurdle is low** — we don't have to convince the buyer that OS quality matches frontier.
3. **Multiplier is large enough to fund the GTM** — at least 100× cost advantage at N16 (conservative node).
4. **Workload has structural moat** — once captured, it's sticky (latency-locked, sovereignty-locked, integration-locked).

The five winning workload segments:

### Wedge 1 — Hierarchical worker layer for frontier-planner agentic systems

**The pitch.** Cursor Agent, Claude Code, OpenAI Deep Research, OpenAI Operator, Devin/Cognition, Replit Agent are all **frontier-planner + OS-L-worker** by architecture. The user-visible model is the frontier planner; the bulk of the token volume is in the OS-L worker layer (Llama-70B class) running search/read/summarize subtasks. Currently served on Together/Fireworks at $0.88/Mtok with 200–500ms TTFT.

**Cost improvement.** At N3, 100B-MoE Pf 5:1 blended = $0.000407/Mtok. **2,162× cheaper than Llama-70B Together**, with sub-100ms TTFT (2–5× faster). Tail-latency-bound trajectory: planner blocks on slowest worker, so latency cut compounds.

**Why it's the cleanest pitch.** No quality-trust hurdle — the customer keeps their frontier planner exactly as-is, just swaps the worker layer. Customer set is concentrated and named: Anthropic, OpenAI, Cursor (Anysphere), Devin/Cognition, Replit, every code-agent and deep-research product team.

**TAM.** Estimated $500M–$2B in current worker-layer token spend (no public number; inferred from Cursor + Claude Code + Deep Research agent volume). Growing 200%+ YoY.

### Wedge 2 — Voice agent LLM passthrough

**The pitch.** Voice agent platforms (Vapi, Retell, Bland, Synthflow, ElevenLabs Agents, Hippocratic AI) are latency-bound: 200–300ms total round-trip budget; LLM TTFT must be <400ms; small models forced. Currently Llama-70B on Groq at $0.59/$0.79 per Mtok blended ~$0.69/Mtok. About $0.04/min on Sonnet, $0.01/min on Groq Llama-70B at typical 30 turns/min.

**Cost improvement.** At N3, 8B Bal 1:1 blended = $0.001446/Mtok. Per-minute LLM cost drops to **~$0.00005/min** — an **800× drop** on the LLM line item. The LLM stops being a meaningful component of the call's $0.12–$0.40/min total; the floor reverts to STT + TTS + telephony at $0.05–$0.15/min.

**Why it's a binding-constraint sale.** Voice latency budgets *literally break the conversation* if missed — your low-TTFT property is license to operate, not an optimization. Customer set: Vapi, Retell, Bland (already on in-house Llama-70B), ElevenLabs Agents, Hippocratic AI (HIPAA → self-hosted), telephony BPOs (TaskUs, Concentrix, Teleperformance) deploying voice for enterprise outsourcing.

**TAM.** Voice agent platform revenue ~$300–600M in 2026, with token spend a portion (~$50–100M). Very high-growth (Bland 100k+ daily calls; Vapi/Retell at scale).

### Wedge 3 — Customer-service text agents (Sierra/Decagon-class)

**The pitch.** Sierra ($1B+ ARR cluster), Decagon, Cresta, Ada, Intercom Fin, Zendesk AI Resolve all run multi-turn agentic conversations on Claude Sonnet ($3/$15) or GPT-5 ($1.25/$10) with heavy prompt caching. Pf 5:1 typical (long policy + history context, modest output). Currently $5/Mtok blended at Sonnet 5:1.

**Cost improvement.** At N3, 100B-MoE Pf 5:1 blended = $0.000407/Mtok. **12,285× cheaper** than Sonnet at the same P:D ratio. Even at a 5× retail markup, the accelerator's effective retail $/Mtok is **$0.00204 — 2,455× cheaper than Sonnet retail at Pf 5:1**.

**Why it works.** F500 deploying customer-service agents are price-sensitive and increasingly testing OS alternatives. Headline buyer pool: every named CS-AI vendor plus their F500 customers (banks running CS bots, telcos deploying tier-1 support, retail running 24/7 chat). Quality-trust hurdle is moderate but conquerable — Sierra-class platforms already do their own evals and can deploy mixed Fr+OS stacks.

**TAM.** CS-agent token spend estimated $300M–$1B in 2026, growing 100%+ YoY (Menlo: $657M of $7.3B departmental enterprise AI is "Customer Success").

### Wedge 4 — Batch summarization / ETL for F500 lakehouse

**The pitch.** F500 enterprises running Bedrock Batch / Vertex Batch / Databricks FM Serving / Snowflake Cortex for nightly ETL — support-ticket digesting, call-center transcription summary, e-discovery, lakehouse annotation. Currently Llama-70B on Databricks at ~$1/$3/Mtok (50% Bedrock Batch discount), or Sonnet 4.6 batch at $1.50/$7.50.

**Cost improvement.** At N3, 100B-MoE Pf 3:1 blended = $0.000468/Mtok. **3,205× cheaper** than Databricks Llama-70B at the same P:D. Pure cost play — no latency or quality trade-off.

**Why it's strategically high-leverage.** Bedrock + Vertex + Databricks process >50% of all enterprise LLM tokens by volume in 2026 (industry estimate). The buyer is already paying a per-Mtok markup over raw GPU cost; the proprietary chip lets them recover that markup. Easy "drop-in cost cut" pitch for F500 buyers who already have an ML team running on OS models.

**TAM.** Enterprise batch / ETL inference spend estimated $5–10B in 2026.

### Wedge 5 — BGE-L embedding for RAG / semantic search

**The pitch.** Every enterprise RAG pipeline, vector database, semantic search system, recommendation engine runs an embedding model. Market today bifurcates: frontier embedders at $0.10–$0.18/Mtok (OpenAI 3-large, Voyage-3-large, Cohere v3) or OS-served BGE / GTE / E5 at $0.005–$0.012/Mtok (DeepInfra, Cloudflare).

**Cost improvement.** At N3, BGE-L = $0.0101/Mqueries (256-tok). vs. DeepInfra BGE-large at $2.56/Mqueries → **253× cheaper apples-to-apples**. vs. OpenAI text-embedding-3-large at $33.28/Mqueries → **3,295× cheaper tier-down**. vs. Voyage-3-large at $46.08/Mqueries → **4,562×**.

**Why it's the volume wedge.** Embeddings are the load-bearing primitive of every grounded LLM application. Buyer set is ubiquitous: Pinecone, Weaviate, Qdrant, Chroma, Turbopuffer customers; every F500 with an internal RAG project; every enterprise search vendor (Glean, Microsoft Copilot M365 retrieval layer, every legal/healthcare/financial-services AI vendor with corpus indexing).

**TAM.** Embedding inference spend estimated $300M–$800M in 2026. Smaller per-customer than LLM workloads, but cumulative.

### Top-5 summary table

| Wedge | Workload | Cost basis (N3) | Current market | Multiplier | TAM (2026) | Time-to-first-revenue |
|---|---|---|---|---|---|---|
| 1 | Hierarchical worker layer | $0.000407/Mtok | $0.88 (Llama-70B Together) | **2,162×** | $500M–$2B | 6–9 months |
| 2 | Voice agent LLM | $0.001446/Mtok blended | $0.69 (Groq Llama-70B) | **477×** (LLM line); **800× /min** drop | $50–100M | 9–12 months (latency cert) |
| 3 | CS text agents | $0.000407/Mtok | $5 (Sonnet 5:1) | **12,285×** | $300M–$1B | 12 months (eval cycles) |
| 4 | Batch ETL F500 | $0.000468/Mtok | $1.50 (Databricks 3:1) | **3,205×** | $5–10B | 9–12 months (procurement) |
| 5 | BGE-L embedding | $0.0101/Mqueries | $2.56 (DeepInfra BGE) | **253×** | $300–800M | 6–9 months |

> **Combined Year-1 addressable revenue if all five wedges land**: $50–150M with even modest market share. Year-3 if the same wedges scale: $500M–$1.5B.

---

## 5. Datacenter Business Margins — Unit Economics vs. Incumbents

**Q: What does the P&L of the datacenter inference business look like, and how do margins compare to current incumbents (frontier, hyperscaler resale, IaaS, on-prem)?**

### The TCO model — operational and financial parameters

The $/Mtok values in §3 are the all-in TCO output of a **power-bounded fleet model** with the following defaults:

**Operational parameters:**

| Parameter | Default | Notes |
|---|---|---|
| Datacenter IT power budget | **100 MW** | Power-bounded fleet sizing |
| PUE | **1.1** | Affects electricity bill, not chip count |
| Chip utilization | **90%** | Steady-state with batching/scheduling/cold-start overhead |
| Fleet sizing | IT power ÷ chip power | Chip-only by default; non-chip host/network/rack power optional |
| Other constraints | none beyond power | No explicit rack-space, cooling-capacity, or networking-topology limits in the model |

**Financial parameters:**

| Parameter | Default | Annual cost at 100 MW |
|---|---|---|
| Electricity | **$0.07/kWh** | $67.5M/yr (110 MW × 8,760 h × $0.07) |
| Hardware lifetime | **3 years** | All hardware capex amortized over 3 yr |
| Per-chip overhead (board / chassis / NIC / PSU) | **$200/chip** | Amortized over 3 yr |
| Datacenter buildout | **$10/W of IT load** | $1B at 100 MW; amortized over 3 yr → ~$333M/yr |
| Annual per-chip OpEx | **$50/chip/yr** | Operations, monitoring, replacement |
| NRE | excluded by default | When enabled: amortized over `nre_amortization_volume × lifetime` |

### Fleet-level economics at 100 MW

At a 500W-per-chip representative chip power, the 100 MW fleet sizes to **200,000 chips**. Annual cost stack:

| Cost component | Annual amount | Per chip per year |
|---|---|---|
| Electricity | $67.5M | $338 |
| DC buildout (3-yr amortization) | $333M | $1,667 |
| Chip silicon (amortized over 3 yr) | varies by chip cost | varies |
| Per-chip overhead amortized | $13M | $67 ($200 / 3 yr) |
| Annual OpEx | $10M | $50 |
| **Subtotal (excluding silicon)** | **~$424M/yr** | **~$2,122 / chip / yr** |

Plus silicon: chip BOM amortized over 3 years × 200K chips. Total silicon contribution depends on chip cost per unit; the per-Mtok output in §3 reflects the full stack at the proprietary chip's silicon cost.

### What changes if assumptions move

The model is sensitive to four levers. Sensitivities at 100 MW:

| Lever | Default | If varied… | Annual fleet cost impact |
|---|---|---|---|
| DC IT MW budget | 100 MW | 50 MW → ~$210M/yr; 200 MW → ~$850M/yr | linear in MW |
| Electricity rate | $0.07/kWh | $0.04 (Texas/Iceland) → −$29M/yr; $0.12 (CA/NY) → +$48M/yr | varies by site |
| Hardware lifetime | 3 yr | 5 yr → DC + chip overhead amortization drop ~40% | meaningful; depends on chip degradation reality |
| PUE | 1.1 | 1.05 (sub-tropical liquid-cooled) → −$3M; 1.2 (older facility) → +$6M | smaller |

**The largest cost lever is DC buildout amortization** ($333M/yr at 100 MW with 3-yr life, $10/W). If we extend hardware lifetime to 5 yr (more typical for DC infrastructure even if chips refresh at 3 yr), DC capex amortization drops to $200M/yr, and total annual fleet cost (ex-silicon) drops from $424M to ~$290M.

### Pricing power vs. all-in COGS

Using the $/Mtok values from §3 as already-all-in COGS:

| Workload class | All-in COGS (N3) | Retail price scenarios | Gross margin |
|---|---|---|---|
| 100B-MoE Pf 5:1 (worker layer) | $0.000407 | $0.10 → **99.6%**; $0.05 → **99.2%**; $0.01 → **95.9%**; $0.001 (DeepSeek-floor) → **59%** |  |
| 8B Bal 1:1 (voice) | $0.001446 | $0.10 → 98.6%; $0.05 → 97.1%; $0.01 → 85.5% |  |
| 100B-MoE Dc 1:5 (synthetic data) | $0.000893 | $0.10 → 99.1%; $0.05 → 98.2%; $0.01 → 91.1% |  |
| BGE-L (per Mqueries) | $0.0101/Mqueries | $0.50/Mq → 98.0%; $0.10/Mq → 89.9%; $0.05/Mq → 79.8% |  |

### Margin comparison table

| Vendor / segment | Typical $/Mtok price | Estimated COGS | Gross margin |
|---|---|---|---|
| **Frontier API direct** (OpenAI, Anthropic) | $1.25–$25 | $0.30–$3 | **65–85%** (+ training amortization) |
| **Hyperscaler resale** (Bedrock, Vertex, Azure) | $1.25–$25 | $0.50–$5 (royalty + DC) | **30–50%** (after frontier-lab royalty) |
| **IaaS on rented H100** (Together, Fireworks, DeepInfra) | $0.05–$1 | $0.04–$0.20 | **20–40%** |
| **Custom-silicon IaaS** (Groq, Cerebras) | $0.10–$1 | unclear; likely lower at scale | **30–60%** (estimated) |
| **Enterprise on-prem self-hosted** | internal cost only | $0.04–$0.10 (8B); higher for 70B+ | n/a (not a business) |
| **Proprietary-chip DC (us)** at retail $0.10/Mtok | $0.10 | **$0.000468 (N3, 100B-MoE Pf 3:1)** | **99.5%** |
| Proprietary-chip DC (us) at retail $0.05/Mtok | $0.05 | $0.000468 | **99.1%** |
| Proprietary-chip DC (us) at retail $0.01/Mtok | $0.01 | $0.000468 | **95.3%** |

### Two readings

**Reading 1 — Pricing power.** The proprietary chip can profitably charge **anywhere from $0.005/Mtok (deep DeepSeek-floor) up to $1+/Mtok (frontier-undercut)**, and still deliver 90%+ gross margin. This is structurally different from every incumbent in the table — none of them can undercut to $0.005/Mtok and still profit. **The chip's all-in COGS is so far below incumbents that *price-setting becomes a strategic choice rather than a cost-driven floor*.**

**Reading 2 — Where the all-in $/Mtok comes from.** Per the TCO model in §5, the $0.000468/Mtok at N3 (100B-MoE Pf 3:1) decomposes into silicon amortization, DC buildout amortization, electricity, per-chip overhead, and OpEx — all rolled in. The largest single line item at the model's 3-year hardware-lifetime assumption is **DC buildout** ($333M/yr at 100 MW). At a more typical 5–10 year DC-infrastructure life, electricity becomes the dominant ex-silicon cost (35%+ of fleet OpEx), which is where the **energy-efficiency property earns its keep**: a 2× perf-per-watt advantage over H100 translates to roughly 15–20% lower long-run DC TCO, meaningful at hyperscaler-class deployments where TCO is the procurement criterion.

### How this compares to the incumbents on a "competitive price-cut" scenario

If the proprietary-chip DC business **prices at 50% of Bedrock Sonnet 4.6 Batch ($0.75 input / $3.75 output → blended $1.125 at 1:1, $0.875 at Pf 5:1)**:
- Customer saves: 50% on Bedrock bill.
- **We earn**: ~$0.875/Mtok at Pf 5:1, COGS $0.000407. Gross margin **99.95%**.

If we **price at 10% of current Llama-70B Together ($0.88 → $0.088)**:
- Customer saves: 90% — dramatic, lands every CFO meeting.
- **We earn**: $0.088/Mtok with COGS $0.000468. Gross margin **99.5%**.

If we **price at 1% of current Llama-70B Together ($0.88 → $0.0088)**:
- Customer saves: 99% — disrupts the entire OS-IaaS market.
- **We earn**: $0.0088/Mtok with COGS $0.000468. Gross margin **94.7%**.

**Strategic conclusion: pricing should be set by competitive dynamics (what we need to charge to displace incumbents), not by COGS-floor + markup math.** The COGS advantage is so large that any price competitive with current OS-IaaS leaves >95% gross margin headroom even after the full TCO stack (silicon + DC buildout + electricity + OpEx) is accounted for.

### Operating margin (after S&M + R&D + G&A)

Going from ~99% gross to operating margin requires layering in:
- **R&D amortization**: ~$200M annual chip team + $50M software stack = $250M/year. At $200M revenue, that's 125% of revenue (loss-making). At $1B revenue, it's 25%. At $5B, it's 5%. **R&D dilutes margins until revenue scales past ~$1B.**
- **S&M**: enterprise sales motion targeting F500 + frontier labs. Estimated 15–25% of revenue at scale. Lower than typical SaaS because the value prop is concrete and quantifiable per buyer.
- **G&A**: ~5–10%.
- **Capex**: chip development (NRE + mask sets per node), DC build-out, networking. Treated as capex; doesn't hit gross margin but does hit cash flow.

**Steady-state operating margin at $5B revenue and N3 silicon: estimated 50–65%.** Comparable to peak-NVIDIA gross margin plus DC-style operating leverage. This is what makes the business attractive — the unit economics are structurally better than any incumbent in the inference market.

---

## 6. GTM and Sales Motion

**Q: How do we actually get customers to buy tokens from us?**

The proprietary-chip DC business is an enterprise infrastructure sale. Five viable channels, in rough order of revenue concentration:

### Channel 1 — Direct API (api.us.com)

Self-serve API endpoint with OpenAI-compatible schema. Buyer signs up, pastes API key, immediately runs Llama-70B-equivalent or 100B-MoE-equivalent at our prices. **Customer acquisition cost is low** (developer-led, no sales motion); **revenue per customer is variable** but skews toward power-users and OS-experimenting startups.

- **Land**: integrate with OpenRouter, list on Together / Fireworks comparison pages, sponsor LangChain / LlamaIndex examples.
- **Expand**: auto-detect customer's high-volume workloads, pitch dedicated-throughput contracts.

Channel 1 captures Segment C buyers (OS-power users, voice platforms, companions). Year-1 target: $5–15M ARR.

### Channel 2 — White-label / OEM resale to enterprise platforms

Sell tokens at wholesale rate to platforms that resell under their own brand: Salesforce Einstein, ServiceNow Now Assist, Workday AI, SAP Joule. They want a sub-frontier-cost option to bundle into their own pricing without compromising margin.

- **Land**: 6–12 month enterprise sales cycle; first deal is hardest.
- **Expand**: every platform has a multi-product roadmap and naturally extends LLM use across modules.

Channel 2 captures Segment B economics (platform serves Segment B customers, we serve the platform). Higher per-customer revenue, longer cycle. Year-2 target: $20–50M ARR.

### Channel 3 — Frontier-lab worker-layer partnership

Anthropic, OpenAI, Cursor, OpenAI Operator, Devin, Replit Agent: pitch as "we host your worker layer at 2,000× lower cost." Frontier lab keeps its planner; our chip serves the worker fleet underneath their product.

- **Land**: requires API and tooling parity (vLLM/SGLang compatibility, OpenAI-schema, model-versioning, observability hooks).
- **Expand**: as agentic products grow, worker-layer volume compounds.

Channel 3 is the highest-leverage single deal. One Anthropic worker-layer partnership = $50–200M ARR estimated. Year-2 target: 1–2 named partnerships.

### Channel 4 — Hyperscaler Marketplace listing

AWS Marketplace, Azure Marketplace, Google Cloud Marketplace. List the proprietary-chip endpoint as a third-party model option inside Bedrock-compatible APIs. Customer pays through their existing cloud bill; we earn revenue minus hyperscaler take rate (typically 20%).

- **Pro**: zero-friction procurement for regulated F500 already on AWS / Azure / GCP.
- **Con**: hyperscaler take rate; we don't own the customer relationship.

Channel 4 captures Segment B buyers indirectly. Year-2–3 target: $10–30M ARR.

### Channel 5 — Sovereign / managed-private deployments

For F500 customers who refuse cloud (banks, telcos, defense, healthcare), offer the chip in an on-prem rack delivered to their DC. Capex sale + recurring software/support contract. Adjacent to NVIDIA DGX / Dell APEX positioning.

- **Land**: 12–18 month enterprise sales cycle, first deal requires reference architecture and at least one operational reference site.
- **Expand**: every regulated F500 has multiple workloads; first rack expands to fleet.

Channel 5 is the highest-revenue-per-customer channel. Year-3 target: $50–200M ARR from 5–15 reference accounts.

### Sales motion sequencing

| Year | Primary channels | Revenue target | Headcount |
|---|---|---|---|
| 1 (post first-silicon) | Channel 1 (direct API) + Channel 3 (1 frontier-lab pilot) | $5–20M | 5–15 GTM |
| 2 | Channel 1 expand + Channel 2 (2–3 white-label) + Channel 3 (1–2 partnerships) | $50–150M | 25–50 GTM |
| 3 | All 5 channels active | $200–500M | 75–150 GTM |

> Compressed: **Channel 1 (direct API) lands the dev community and creates pricing reference; Channel 3 (worker-layer partnerships) is the highest-leverage single deal; Channel 5 (sovereign managed-private) is the long-tail high-revenue annuity.** Channels 2 and 4 are filler.

---

## 7. Defensibility — What Compounds Beyond Silicon BOM

**Q: Once we're successful, what stops AWS Trainium 3, NVIDIA Blackwell, Groq, Cerebras, or any well-funded entrant from copying us?**

The silicon BOM advantage is the wedge, not the moat. If the moat is only "we have a better chip," every credible answer to "what about a year from now" requires having a chip lead at every node, forever — which is implausible. Four moats compound after the chip:

### Moat 1 — Customer-side state and integration

Every customer that lands on the proprietary-chip API accumulates **prompt cache state** (a 90%-cached input prompt at $0.000028/Mtok is wildly more valuable than a fresh prompt elsewhere), **fine-tune weights** (custom LoRAs / DPO-tuned variants stored in our vault), **observability history** (eval logs, A/B traces, anomaly patterns), and **routing rules** (custom prompt templates, system prompts, tool definitions). Migration cost grows linearly with usage.

Comparable: Snowflake's integration moat over Redshift even when Redshift is technically competitive. The customer doesn't churn unless a competitor is *dramatically* better, not marginally.

### Moat 2 — Energy-efficiency-driven sovereign / edge DC footprint

The third accelerator property (energy efficiency) translates to **lower-power racks → cheaper-to-build-and-operate sovereign DCs** in jurisdictions with constrained power: Singapore, Tokyo, London, Sydney, Frankfurt, GovCloud regions. Once we land sovereign customers (e.g., a UK bank, a Singaporean ministry, a German pharma), those become locked-in references that hyperscalers can't easily replicate without their own custom silicon at our perf-per-watt point.

NVIDIA Blackwell is power-hungry; AWS Trainium 3 is improving but is still rack-power-bound. Groq/Cerebras are optimized for compute-density at high power. **Energy efficiency at deployable density is the structural advantage that compounds geographically.**

### Moat 3 — Software stack and developer experience

Customers don't buy chips, they buy **APIs and SDKs**. If the proprietary-chip API has the cleanest:
- OpenAI-schema compatibility
- vLLM / SGLang / TRT-LLM model-portability
- Prompt cache implementation (key-prefix-aware, multi-turn-aware, tool-call-aware)
- Observability hooks (LangSmith, Datadog AI, custom eval frameworks)
- Multi-tenancy SLAs

then the developer / ML-engineer choice is sticky beyond pure cost. The user-facing API is the moat, not the silicon. Comparable: Stripe's "developer experience moat" over Adyen even when Adyen is technically more capable.

### Moat 4 — Workload-specific optimizations the chip exploits but rivals can't

The proprietary chip is purpose-built (we assume) for the four properties: low TTFT, fast tok/s, energy efficiency, 8B–100B-MoE class. Software optimizations that the chip exploits — e.g., **disaggregated prefill/decode for Pf-heavy workloads** (LMCache-style), **expert routing prefetch for MoE**, **speculative decoding tuned to our chip's pipeline**, **custom KV cache quantization** — accumulate as a software-stack advantage that's not portable to Blackwell or Trainium.

This is the "narrow but deep" moat: we beat NVIDIA on a specific operating-point, not on general-purpose AI compute. As long as the inference market continues fragmenting by workload (which the taxonomy doc argues it is), workload-specific moats matter more than general silicon advantages.

### What's NOT a moat

- **Just having a chip lead.** Every competitor's roadmap will catch up at some node.
- **Customer logos.** Easy to lose to a cheaper competitor without other moats.
- **Patents.** Useful for licensing leverage but not for product differentiation.
- **Founder network / hiring quality.** Necessary but not sufficient.

---

## 8. Risks and Kill-Shots

**Q: What can kill this thesis?**

Six structural risks, in rough order of severity:

### Risk 1 — Frontier API price collapse (DeepSeek-style)

DeepSeek shipping V3 / R1 at sub-$1/Mtok output dropped the entire OS-tier price floor by 5–10× within a quarter. If OpenAI / Anthropic / Google decide to crush the open-weight tier — e.g., Anthropic ships Claude Haiku 5 at $0.10/Mtok input — our 2,000× cost advantage vs. their $5 list price compresses to 200× vs. $0.10. Still a win, but dramatically less headline.

**Mitigation.** The energy-efficiency moat survives any pricing war. Frontier labs amortizing training cost into per-token prices have a structural floor below which they can't profitably go. We don't.

### Risk 2 — Hyperscaler vertical integration

AWS Trainium 3 (rumored 2026 launch), Google TPU v6 / Ironwood / 8i, Azure Maia (rumored late-2026): all targeting the same OS-class workloads we serve. Hyperscalers have unlimited capex and customer-facing distribution. If they hit our perf-per-watt point at any node, they take our customers via Bedrock / Vertex / Azure direct.

**Mitigation.** Sovereign DC strategy (out-of-cloud) and direct-customer relationships. Hyperscaler chips serve hyperscaler clouds; we can be in jurisdictions / contracts they aren't.

### Risk 3 — Foundry slot loss

TSMC N3 slots are constrained; an N3 tape-out requires Apple-class commitment ($100M+ NRE per design plus wafer commit) or a foundry slot via Samsung / Intel that may have lower yield. If the chip has to retreat to N5 or N7 for first silicon, our headline multipliers drop 2–4×.

**Mitigation.** N7 first silicon at multipliers still 5–6,000× vs. frontier is defensible; N16 still 2,300–2,900×. The pitch lands at every node with named foundry slot (covered in `accelerator-workload-fit-analysis.md` §9).

### Risk 4 — Software stack incompatibility

If the chip can't run vLLM / SGLang / TRT-LLM out of the box, every customer integration becomes a custom port. Groq and Cerebras hit this hard early; both spent 2–3 years building software stacks before customers could deploy at scale. SambaNova still hasn't fully solved it.

**Mitigation.** Day-1 OpenAI-schema API, day-1 vLLM compatibility, day-1 prompt-cache parity. Software is the moat (Moat 3); shortcut here costs years.

### Risk 5 — MoE serving complexity

100B-MoE is the dominant SKU in our cost model, but MoE is also harder to serve efficiently than dense models. Expert routing has tail-latency variance; load balancing across experts is non-trivial; a single hot expert can crater throughput. If our chip's MoE serving doesn't deliver the projected $0.000285 input cost in real production, the whole thesis weakens.

**Mitigation.** MoE-specific kernel optimizations + customer pilots that validate real-world MoE economics before scale-out commit.

### Risk 6 — Customer concentration risk

A worker-layer partnership with Anthropic or Cursor (Channel 3) could be 50%+ of Year-2 revenue. If they renegotiate, build their own silicon, or get acquired into a hyperscaler, the revenue base implodes.

**Mitigation.** Channel diversification — Channels 1, 2, 4, 5 prevent any single customer from being existential. Revenue concentration above 25% from any single customer flagged as risk.

### Lower-priority risks (worth tracking)

- Macro / capital-markets shutdown forcing premature monetization
- US-China export controls expanding to memory / packaging / rare-earth supply chains
- Litigation: model-weight licensing (Llama community license terms could shift; Qwen is Alibaba-owned)
- Talent — chip design + ML systems + enterprise sales is a rare combination

---

## 9. Roadmap and Milestones — 12 / 24 / 36 Months

**Q: What does the 12/24/36-month financing-and-proof-point ladder look like?**

### Months 0–12 (now → 2027-05) — Tape-out and first silicon

- **Q1–Q2 2026**: Tape-out at N7 or N5 (foundry slot dependent). Software stack development in parallel: OpenAI-schema API, vLLM compatibility, prompt cache implementation, observability hooks.
- **Q3 2026**: First silicon back from foundry. Bring-up, validation, MLPerf-style internal benchmarks.
- **Q4 2026**: Closed beta with 5–10 design partners (mix of Channel 1 dev community + 1–2 Channel 3 frontier-lab pilots).
- **Q1 2027**: GA launch on direct API (Channel 1). First $1–5M ARR.

**Financing milestone**: Series B or equivalent ($150–300M) to fund first silicon + bring-up.

### Months 12–24 (2027-05 → 2028-05) — Commercial scale-out

- **Q2 2027**: First worker-layer partnership signed (Channel 3 — Anthropic, OpenAI, Cursor, or equivalent).
- **Q3 2027**: Direct API ARR $20–50M; first white-label deal (Channel 2).
- **Q4 2027**: First sovereign managed-private deployment (Channel 5 — F500 bank, telco, or insurance).
- **Q1 2028**: ARR $100–200M. Begin N3 tape-out.

**Financing milestone**: Series C ($500M–$1B) to fund DC capex (multi-MW colo deals) + N3 tape-out.

### Months 24–36 (2028-05 → 2029-05) — Multi-channel scale

- **Q2 2028**: All five channels active. Hyperscaler Marketplace listings live.
- **Q3 2028**: First N3 silicon back from foundry. 4–7× cost advantage over N7-era pricing.
- **Q4 2028**: ARR $300–500M. 2–3 sovereign-DC sites operational (US, EU, Asia).
- **Q1 2029**: ARR $500M–$1B. Operating-margin positive. Public-company-ready or strategic-acquisition-ready.

**Financing milestone**: IPO or strategic exit. Comparable references: Groq ($2.8B post-money 2024), Cerebras (filed S-1), Tenstorrent, Etched.

### Proof-points the deck needs to land each financing round

| Round | Proof-point bar |
|---|---|
| Series B (Year 1) | Tape-out commit + 3 named LOIs + foundry slot |
| Series C (Year 2) | First $20M ARR + 1 worker-layer partnership + 3 reference customers |
| Pre-IPO / late-stage | $200M+ ARR + path to operating-margin-positive within 12 months + multi-region DC footprint |

---

## 10. Open Questions / Next Steps

- **Foundry slot strategy.** TSMC N3 slot or Samsung 3GAA? When does the slot need to be locked? Mask-set NRE budget and amortization assumption.
- **Chip-level specs at validated benchmarks.** What's the *measured* TTFT / tok-per-sec / perf-per-watt on first silicon at customer-representative workloads (not synthetic). MLPerf-style validation pre-customer-pilot is essential.
- **Pricing strategy commit.** Where on the spectrum (1% / 10% / 50% of incumbent list) do we set retail prices for each wedge? Different wedges may justify different markups.
- **Channel-3 first partner.** Which frontier-lab worker-layer partnership do we pursue first? (Anthropic vs. OpenAI vs. Cursor vs. Replit.) Each has different procurement dynamics and reference-account value.
- **Software stack scope.** Do we build proprietary serving (more control, slower) or fork vLLM / SGLang (faster, less differentiation)? Resourcing and timeline implications.
- **DC strategy.** Build, lease, or co-locate? Tier 3 vs Tier 4 site selection. Energy sourcing (renewable PPAs, sovereign jurisdiction). Networking partner.
- **30B SKU decision.** Per `accelerator-workload-fit-analysis.md` §11, 30B is dominated by 100B-MoE at every node. Confirm before committing 30B-specific masks.
- **Vision-language SKU.** Multimodal Q&A (Row 32) is currently out of scope. Is there a fifth model SKU for VL (LLaVA / Qwen-VL class) worth adding?
- **Roadmap on re-ranker SKU.** Cross-encoder rerank is currently uncovered. Worth a sixth SKU?

---

## Sources

**Workload taxonomy and operating-point characterization:**
- `../llm-inference-workloads/llm-inference-workloads-taxonomy.md` — the canonical workload reference
- `../llm-inference-workloads/accelerator-workload-fit-analysis.md` — per-workload accelerator multipliers across N16/N7/N5/N3

**Adjacent business archetype docs:**
- `batch-inference-business-analysis.md` (archetype 1)
- `agentic-inference-business-analysis.md` (archetype 3)
- `tokens-too-cheap-to-meter-thesis.md` (cost-curve framing)

**Market sizing and customer references** (drawn from the taxonomy doc; see that document's Sources section for full URL list):
- Anthropic Economic Index (Sept 2025 / Jan 2026 / Mar 2026)
- OpenAI / NBER WP 34255 ("How People Use ChatGPT", Sept 2025)
- a16z Top 100 Gen AI Consumer Apps 6th ed. (Mar 2026)
- Menlo Ventures 2025 State of Generative AI in the Enterprise (Dec 2025) — $37B aggregate enterprise AI spend
- LangChain State of Agent Engineering 2025 — 57.3% of orgs with agents in production
- SemiAnalysis Coding Assistant Breakdown — Claude Code 100:1 input:output ratio
- Hamming AI / Twilio voice latency analyses
- Mistral OCR 3 launch (Dec 2025) — $2/1K pages reset of IDP pricing

**Inference-provider pricing references** (May 2026 list prices):
- OpenAI API pricing
- Anthropic API pricing
- Google Vertex / Gemini API pricing
- Together AI, Fireworks, Groq, Cerebras, DeepInfra, Cloudflare Workers AI pricing
- AWS Bedrock + Vertex AI batch pricing tables
- Databricks Foundation Model Serving pricing
- Snowflake Cortex AI Functions pricing

---

*Document compiled 2026-05-02.*
