# Accelerator–Workload Fit Analysis — 8–31B Class, Low TTFT, Energy-Efficient

*A companion analysis to `llm-inference-workloads-taxonomy.md`. Maps the 32 catalogued workloads against an inference accelerator with three properties — (1) super-low TTFT, (2) very fast tokens-per-second, (3) energy-efficient — and a hard constraint of 8–31B parameter models (with 100B-class MoE permissible because active-params-per-token fits the budget). Reports per-workload fit strength, cost-basis advantage at N16/N7/N5/N3, and the strategic takeaways.*

> Companion to `llm-inference-workloads-taxonomy.md`. Read the taxonomy first — this document references rows by number and assumes the §4 master table is loaded.

---

## 1. The accelerator's properties and constraints

| Dimension | Property |
|---|---|
| TTFT | Super low (sub-200ms target; comparable to Groq / Cerebras class) |
| Throughput | Very fast tokens/second |
| Power | Energy-efficient (perf-per-watt advantage) |
| Model size | **Hard constraint: 8–31B parameter models** (active-params-per-token; 100B-class MoE permissible if active params fit the budget) |
| Modalities | Text-decoder; vision and embedding workloads not addressed |

**Cost basis (BOM, $/Mtok input / output, by process node and model size):**

| Tier | N16 in / out | N7 in / out | N5 in / out | N3 in / out |
|---|---|---|---|---|
| 8B Llama-3.1-8B | $0.0037 / $0.0039 | $0.0024 / $0.0025 | $0.0017 / $0.0018 | **$0.0013 / $0.0016** |
| 30B Gemma4-31B | $0.0242 / $0.0255 | $0.0109 / $0.0116 | $0.0074 / $0.0079 | **$0.0051 / $0.0054** |
| 100B Qwen-MoE | $0.0014 / $0.0034 | $0.0007 / $0.0014 | $0.0004 / $0.0013 | **$0.0003 / $0.0010** |

> **Three structural observations from the input/output split.**
>
> **(1) 100B-MoE input is dramatically cheaper than 30B and 8B input — at every node.** At N3, 100B-MoE input is $0.0003/Mtok, **17× cheaper than 30B input** ($0.0051) and **4.3× cheaper than 8B input** ($0.0013). The MoE active-params-per-token advantage shows up most where prefill dominates, which is most workloads (rows 4, 5, 9, 10, 11, 14, 15, 17, 18, 22, 23, 24, 26, 27, 28, 29, 30 in the master table all carry Pf labels).
>
> **(2) 100B-MoE output cost is 3.3× the input cost (at N3) — a structural ratio.** $0.0003 in / $0.0010 out reflects the MoE serving asymmetry: prefill batches expert-routes efficiently; decode is autoregressive and bounded by weight-streaming. For decode-heavy workloads (rows 2, 8, 21 — long-form writing, companions, synthetic data) the cost advantage is smaller but still 8–18× over current OS-L list prices.
>
> **(3) 100B-MoE is unconditionally cheaper than 8B at N3 across all P:D ratios.** Both 100B-MoE input ($0.0003 < $0.0013) AND output ($0.0010 < $0.0016) are below 8B. This means there is **no P:D ratio at which 8B beats 100B-MoE on cost at N3**. The 8B SKU only wins when latency, on-device deployment, or quality-overkill considerations override pure cost. Even more striking: 30B is uncompetitive vs 100B-MoE at every node — same logic applies, with a 17× gap on input and 5× on output. Strategic implication: **the 30B SKU is hard to justify in the product roadmap unless it serves a specific architectural niche** (e.g., dense-only customers who refuse MoE, or sub-segment latency profile where 30B dense beats 100B-MoE on TTFT).

---

## 2. Tier 1 — Accelerator is a Forcing Function (binding constraint, not a nice-to-have)

These are workloads where one or more of the accelerator's three properties is the literal license-to-operate, not an upgrade.

### Workload 25 — Voice agent

The single cleanest fit.

- **TTFT** budget of 200–300ms ([Hamming AI voice latency analysis](https://hamming.ai/resources/voice-ai-latency-whats-fast-whats-slow-how-to-fix-it)) literally breaks the conversation if missed. Low TTFT is not a feature; it's the license to operate.
- **Size constraint** is already forced by latency budget — production voice deployments converge on 7B–11B models (Vapi/Retell defaults; Bland's in-house Llama-70B-class is the upper bound). 8–31B is the **native** size band.
- **Fast tok/s** matters: ~100 generated tokens within a 200ms window for natural turn-taking.
- **Energy efficiency** lands the kill blow on per-minute economics ($0.12–$0.40/min current all-in) AND on telephony-DC co-location (cell-site / PoP racks are power-constrained).
- **Direct competition**: Groq, Cerebras. Differentiation: perf-per-watt + physical footprint.
- **Customers to land**: Bland AI (already runs in-house Llama-70B-class on its own GPU cluster — knife-fighting cost), Vapi/Retell BYO-LLM users, Hippocratic AI (HIPAA forces self-hosted), telephony BPOs (TaskUs, Concentrix, Teleperformance).

### Workload 9 — IDE inline completion (FIM)

- **TTFT SLO 0.125s** ([DistServe](https://arxiv.org/abs/2401.09670)) — *tighter than voice*. Sub-200ms TTFT is the literal product.
- Already an OS-S workload by construction (Cursor Tab, Copilot completion are sub-7B specialty models). 8–31B is over-spec for the inline-FIM case but a perfect fit for the next-tier "agentic completion" segment.
- **Tok/s** matters less here (decode is 1–13 tokens) — properties (1) and (3) are what carry.
- **Energy efficiency** matters because every keystroke triggers inference across millions of developers.
- **Customers to land**: Cursor (would love to migrate Tab off bespoke silicon), JetBrains AI, Sourcegraph Cody, every IDE plugin vendor without their own training stack.

---

## 3. Tier 2 — Strong Fit, Compete on Cost/Efficiency

These workloads benefit from the accelerator's properties but compete with viable alternatives. The accelerator's wedge here is cost basis + perf-per-watt.

### Workload 8 — Companions / roleplay

- Decode-heavy + massively parallel + sub-$0.0001/message economics. Fast tok/s lands directly on per-message COGS.
- Already runs on 13B–30B-class in-house finetunes (Chai's "Chaiverse," Spicychat, Joyland, Polybuzz, Janitor.AI's BYOK long tail). 8–31B is the native model size.
- Frontier API safety filters reject this content, so the competitive set is DeepInfra, Together, OpenRouter Chinese-model passthrough — not OpenAI/Anthropic.
- **Customers**: Character.AI is captive on Google TPU post-acquihire; the addressable buyers are the 5–50M-user long tail (Chai, Spicychat, Polybuzz, Talkie, Janitor users via OpenRouter).

### Workload 19 — Classification & moderation

- Online moderation needs high TTFT (every post/comment/message); offline is throughput-bound.
- Reasoning is Sh — 8B is sufficient. Already deployed on Llama-3.1-8B at $0.05/$0.05 per Mtok.
- Energy efficiency at massively-parallel scale is the wedge.
- Tokens/sec doesn't matter (output is 1–10 tokens). Two of three properties leveraged.
- **Customers**: every social platform, every CSAM/abuse vendor (Hive, Cinder, Sentropy successors), every payments-ops fraud team.

### Workload 1 — Short conversational Q&A 🟢

- The largest token-volume workload globally. 700M–900M ChatGPT WAU + Claude.ai + Gemini consumer.
- Sh→Med reasoning. The 🟢 dot says OS-100B is sufficient; in practice Llama-3-8B handles a meaningful share of consumer queries.
- High TTFT sensitivity (consumer chat UX).
- Energy efficiency matters because frontier free-tier serving is structurally subsidized (Altman: "hundreds of millions/quarter") — anyone offering consumer chat at sub-frontier cost is in the conversation.
- **Direct competition**: DeepSeek (its own clusters), Mistral Le Chat (Cerebras), every consumer-chat surface backstopping free tiers with cheap OS.
- **Customers**: aggregators (OpenRouter), consumer-chat startups without frontier deals, Mistral Le Chat–style vendors, white-label chat for enterprise.

---

## 4. Tier 3 — Plausible but Not Differentiating

The accelerator can address these but doesn't win on its three properties. Cost win is real but a price war.

- **Row 3 (Editing)** — already at $0.10/$0.10 OS-S retail in the market for the cost-conscious slice; quality slice is on Haiku.
- **Row 6 (Translation)** — 8B competent for major language pairs but quality-conscious enterprise (DeepL, Smartling, Lokalise) defaults to OS-L 70B per WMT25 results. Size-constrained out of the premium tier.
- **Row 17 (Open-domain citation chatbot)** — Perplexity Sonar already dominates on Llama-70B finetunes. 8–31B is a step down on quality.
- **Row 23 (Batch ETL)** — TTFT doesn't matter, so property (1) is wasted. Tok/s and efficiency still matter, but Bedrock Batch Llama-70B at $1.50/$7.50 is a tough comp.

---

## 5. Tier 4 — Out of Reach

Workloads requiring Deep reasoning that 8–31B can't credibly deliver in May 2026, plus specialty model classes:

| # | Workload | Why excluded |
|---|---|---|
| 5 | Multi-doc synthesis | Deep reasoning required |
| 11 | Code review / PR comments | Med→Dp; nuanced reasoning matters |
| 12 | Long-horizon code agent | Deep — but the OS-L worker layer (§6 below) **is** addressable |
| 13 | App builder / vibe coding | Deep |
| 16 | Web-grounded research / deep research | Deep — but again, worker layer is addressable |
| 26 | Browser / computer-use agent | Deep + vision |
| 28 | Healthcare ambient scribe | Frontier today (BAA constraint); OS-100B-feasible but trust-gated |
| 29 | Legal contract analysis | Deep |
| 30 | Financial research | Deep |
| 32 | Multimodal Q&A | Vision quality gap |
| 20 | Embedding generation | Specialty model class — separate SKU question |
| 22 | Re-ranking | Specialty model class — separate SKU question |

---

## 6. Where the Accelerator Expands the Pie

Workloads not currently in the master table that become economical with the accelerator's properties:

### A. Hierarchical worker sub-agents (§9.3 of the taxonomy doc)

The single biggest non-obvious target. Cursor Agent, Claude Code, OpenAI Deep Research, OpenAI Operator are all **frontier-planner + OS-L-worker** by architecture. The worker layer is:

- Bulk-of-token-volume in any agent trajectory.
- Sh–Med reasoning per call.
- Currently served on Together/Fireworks Llama-70B at $0.88/Mtok with 200–500ms TTFT.
- Tail-latency-bound: planner blocks on the slowest worker.

Accelerator at N3 100B-MoE serves at $0.0005/Mtok with sub-100ms TTFT — **1,760× cheaper per token, 2–5× faster TTFT**. The latency cut compounds across the trajectory because tail latency dominates.

The pitch positions to Anthropic / OpenAI / Cursor / OpenAI Operator as a *worker-layer accelerator* without ever competing on the planner. Their cost goes down 1,000×; throughput goes up; quality is unchanged.

### B. Real-time multimodal at the edge

Live captioning, mid-conversation translation in headphones, real-time dubbing. None of this works at frontier-API latency. Existing voice+translation infra is bolted-on; first-party low-TTFT 8–31B inference makes the round-trip clean.

### C. Universal autocomplete (not just IDE)

Keyboard predictions, email/doc autocomplete, terminal autocomplete, every input field. Currently nonexistent at scale because per-keystroke economics fail above ~$0.0001/inference. The accelerator's BOM at N3 ($0.0014 for 8B per Mtok) puts this in range.

### D. Tool-call routing layers in agentic systems

Fast classifier to pick the right tool / sub-agent before any frontier reasoning fires. Today's routing layer is either keyword regex (cheap, brittle) or a frontier API call (slow, expensive). A sub-100ms 8B classifier eliminates the trade-off.

---

## 7. Cost Basis at N3 — Multiplier Tables

All multipliers below are **BOM-to-list-price** ratios computed using each workload's actual prefill:decode ratio (so a Pf 8:1 summarization workload uses 8 input tokens and 1 output token in the blend, not a 1:1 average). They tell you the gross-margin headroom available, not the implied retail price (see caveats in §10).

**Computation:** User blend = (P × in_cost + 1 × out_cost) / (P + 1), where P is the workload's P:D ratio from §4 of the taxonomy doc. Market blend uses the same weighting on the listed input/output prices. Latency-bound workloads (voice, IDE inline) use 8B because TTFT favors small-dense over MoE routing overhead; all other workloads default to 100B-MoE because at N3 it is unconditionally cheaper than 8B (see §1 observation 3).

### Latency-leveraged workloads (properties 1+2+3 *all* land)

| # | Workload | Model · P:D | User N3 blend $/Mtok | A2A market $/Mtok | A2A ratio | Fr-deployed $/Mtok | Tier-down ratio |
|---|---|---|---|---|---|---|---|
| 25 | **Voice agent** | 8B · Bal 1:1 | $0.00145 | $0.69 (Groq Llama-70B) | **475×** | $9 (Sonnet 1:1) | **6,200×** |
| 9 | **IDE inline completion** | 8B · Pf 100:1 | $0.00130 | $0.05 (custom-small) | **38×** | $0.05 (already OS-S) | match |
| 1 | 🟢 **Short conversational Q&A** | 100B-MoE · Bal 1:1 | $0.00065 | $0.88 (Llama-70B Together) | **1,354×** | $5.63 (GPT-5 1:1) | **8,654×** |
| 8 | **Companions / roleplay** | 100B-MoE · Dc 1:5 | $0.00088 | $0.20–$0.50 (in-house OS-L est.) | **225–565×** | (already OS-L) | match (BOM win) |
| 19 | **Classification & moderation** | 8B · Pf 50:1 | $0.00131 | $0.05 (Llama-8B DI) | **38×** | $0.05 (already OS-S) | match |
| 7 | **Practical guidance / tutoring** | 100B-MoE · Bal 1:1 | $0.00065 | $0.88 (Llama-70B) | **1,354×** | $5.63 (GPT-5 1:1) | **8,654×** |
| 10 | **In-IDE chat** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (Llama-70B) | **2,110×** | $5 (Sonnet 5:1) | **11,990×** |
| 24 | **Customer-service agent (text)** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (Llama-70B) | **2,110×** | $5 (Sonnet 5:1) | **11,990×** |
| 31 | **Education tutor** | 100B-MoE · Bal 1:1 | $0.00065 | $1.75 (Gemini 3 Flash 1:1) | **2,692×** | (already on Flash) | match |

### Throughput-leveraged workloads (fast tok/s + efficiency win; low TTFT wasted)

| # | Workload | Model · P:D | User N3 blend $/Mtok | A2A market $/Mtok | A2A ratio | Fr-deployed $/Mtok | Tier-down ratio |
|---|---|---|---|---|---|---|---|
| 4 | 🟢 **Summarization, single doc** | 100B-MoE · Pf 8:1 | $0.000378 | $0.88 (Llama-70B 8:1) | **2,328×** | $4.33 (Sonnet 8:1) | **11,455×** |
| 6 | **Translation** | 100B-MoE · Bal 1:1 | $0.00065 | $0.315 (Llama-70B DeepInfra 1:1) | **485×** | (already OS-L) | match |
| 14 | **Code search / repo Q&A** | 100B-MoE · Pf 10:1 | $0.000364 | $0.88 (Llama-70B) | **2,418×** | $4.27 (Sonnet 10:1) | **11,730×** |
| 15 | **Enterprise search & grounded chat** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (Llama-70B) | **2,110×** | $5 (Sonnet 5:1) | **11,990×** |
| 17 | **Open-domain citation chatbot** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (Sonar / Llama-70B 5:1) | **2,110×** | (already OS-L) | match (BOM win) |
| 27 | **Workflow / sales / RPA agent** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (Llama-70B 5:1) | **2,110×** | $5 (Sonnet 5:1) | **11,990×** |
| 23 | **Batch summarization / ETL** | 100B-MoE · Pf 3:1 | $0.000475 | $1.50 (Databricks Llama-70B 3:1) | **3,158×** | (already OS-L) | match (BOM win) |
| 21 | 🟢 **Synthetic data generation** | 100B-MoE · Dc 1:5 | $0.000883 | $12.92 (GPT-5.5 batch 1:5) | — | $12.92 | **14,634×** |
| 18 | 🟢 **IDP text extraction** | 100B-MoE · Pf 5:1 | $0.000417 | $0.88 (extraction LLM step) | **2,110×** | $5 (Sonnet 5:1) | **11,990×** |

**Two findings the input/output split changes vs. the previous version of this section:**

1. **Prefill-heavy workloads on 100B-MoE win even harder than the blended numbers suggested.** Pf 10:1 (code search) at N3 is **$0.000364/Mtok** because input dominates and 100B-MoE input ($0.0003) is the cheapest cell in the entire grid. The previous flat-$0.0005 number underestimated the prefill advantage by ~25%.
2. **Decode-heavy workloads on 100B-MoE see a real cost penalty (but still win at scale).** Synthetic data and companions hit $0.000883/Mtok blended at N3 — 2.4× higher than the prefill-heavy cells, because output cost ($0.0010) is 3.3× input cost ($0.0003). The win vs. market is still 14,000× on the GPT-5.5 batch comparison, but the *gross-margin headroom* for these workloads is meaningfully tighter than for Pf-heavy ones. Pricing strategy should reflect this: charge more per output token to mirror your own COGS structure.

---

## 8. Cost Basis at N16 — Conservative Tape-Out Scenario

Same workloads, conservative node. 8B = $0.0037 in / $0.0039 out, 30B = $0.0242 in / $0.0255 out, 100B-MoE = $0.0014 in / $0.0034 out per Mtok. Computation method matches §7.

### Latency-leveraged at N16

| # | Workload | Model · P:D | User N16 blend $/Mtok | A2A market $/Mtok | A2A ratio | Fr-deployed $/Mtok | Tier-down ratio |
|---|---|---|---|---|---|---|---|
| 25 | **Voice agent** | 8B · Bal 1:1 | $0.0038 | $0.69 (Groq Llama-70B) | **182×** | $9 (Sonnet 1:1) | **2,368×** |
| 9 | **IDE inline completion** | 8B · Pf 100:1 | $0.00370 | $0.05 (custom-small) | **14×** | $0.05 (already OS-S) | match |
| 1 | 🟢 **Short conversational Q&A** | 100B-MoE · Bal 1:1 | $0.0024 | $0.88 (Llama-70B) | **367×** | $5.63 (GPT-5 1:1) | **2,344×** |
| 8 | **Companions / roleplay** | 100B-MoE · Dc 1:5 | $0.00307 | $0.20–$0.50 (in-house OS-L) | **65–163×** | (already OS-L) | match (BOM win) |
| 19 | **Classification & moderation** | 8B · Pf 50:1 | $0.00370 | $0.05 (Llama-8B DI) | **14×** | $0.05 (already OS-S) | match |
| 7 | **Practical guidance / tutoring** | 100B-MoE · Bal 1:1 | $0.0024 | $0.88 (Llama-70B) | **367×** | $5.63 (GPT-5 1:1) | **2,344×** |
| 10 | **In-IDE chat** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (Llama-70B) | **508×** | $5 (Sonnet 5:1) | **2,885×** |
| 24 | **Customer-service agent (text)** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (Llama-70B) | **508×** | $5 (Sonnet 5:1) | **2,885×** |
| 31 | **Education tutor** | 100B-MoE · Bal 1:1 | $0.0024 | $1.75 (Gemini 3 Flash 1:1) | **729×** | (already on Flash) | match |

### Throughput-leveraged at N16

| # | Workload | Model · P:D | User N16 blend $/Mtok | A2A market $/Mtok | A2A ratio | Fr-deployed $/Mtok | Tier-down ratio |
|---|---|---|---|---|---|---|---|
| 4 | 🟢 **Summarization, single doc** | 100B-MoE · Pf 8:1 | $0.00162 | $0.88 (Llama-70B 8:1) | **543×** | $4.33 (Sonnet 8:1) | **2,673×** |
| 6 | **Translation** | 100B-MoE · Bal 1:1 | $0.0024 | $0.315 (Llama-70B DeepInfra 1:1) | **131×** | (already OS-L) | match |
| 14 | **Code search / repo Q&A** | 100B-MoE · Pf 10:1 | $0.00158 | $0.88 (Llama-70B) | **557×** | $4.27 (Sonnet 10:1) | **2,703×** |
| 15 | **Enterprise search** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (Llama-70B) | **508×** | $5 (Sonnet 5:1) | **2,885×** |
| 17 | **Open-domain citation chatbot** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (Sonar / Llama-70B 5:1) | **508×** | (already OS-L) | match (BOM win) |
| 27 | **Workflow / sales / RPA agent** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (Llama-70B 5:1) | **508×** | $5 (Sonnet 5:1) | **2,885×** |
| 23 | **Batch summarization / ETL** | 100B-MoE · Pf 3:1 | $0.00190 | $1.50 (Databricks 3:1) | **789×** | (already OS-L) | match (BOM win) |
| 21 | 🟢 **Synthetic data generation** | 100B-MoE · Dc 1:5 | $0.00307 | $12.92 (GPT-5.5 batch 1:5) | — | $12.92 | **4,213×** |
| 18 | 🟢 **IDP text extraction** | 100B-MoE · Pf 5:1 | $0.00173 | $0.88 (extraction LLM) | **508×** | $5 (Sonnet 5:1) | **2,885×** |

**The N16 → N3 spread compresses for prefill-heavy 100B-MoE workloads.** A code-search Pf 10:1 workload runs at $0.00158 on N16 vs. $0.000364 on N3 — a 4.3× speedup. But for *balanced* and *decode-heavy* workloads on 100B-MoE, the N16 → N3 ratio is closer to 3–4× (governed by output-cost reduction, which is smaller in absolute terms). This means **the strategic case for taping out at N3 is strongest if your initial customer mix is prefill-heavy** (summarization, code search, enterprise search, IDP, batch ETL); a launch portfolio dominated by chat or companions captures less of the N3 advantage and might justify N7 instead.

---

## 9. Node Sensitivity — How N16/N7/N5/N3 Changes the Pitch

The ranking of which workloads win **does not change** with node — voice, IDE inline, hierarchical worker, companions, and consumer-chat-tier-down stay the top fits at every node. What changes is how aggressively the accelerator can price.

**Input / output cost spread by node and model:**

| Tier | 8B in / out | 30B in / out | 100B-MoE in / out | Best-fit workload pitch |
|---|---|---|---|---|
| **N16** | $0.0037 / $0.0039 | $0.0242 / $0.0255 | $0.0014 / $0.0034 | "100–600× cheaper than current OS-L for prefill-heavy workloads. Conservative pitch, defensible numbers." |
| **N7** | $0.0024 / $0.0025 | $0.0109 / $0.0116 | $0.0007 / $0.0014 | "300–1,200× cheaper. Credible Series-B middle ground." |
| **N5** | $0.0017 / $0.0018 | $0.0074 / $0.0079 | $0.0004 / $0.0013 | "600–2,000× cheaper. Where you want to be at scale." |
| **N3** | $0.0013 / $0.0016 | $0.0051 / $0.0054 | $0.0003 / $0.0010 | "1,300–2,400× cheaper for prefill-heavy 100B-MoE. Headline-friendly but only credible with named foundry slot." |

### What the input/output split exposes about node strategy

- **The 100B-MoE input cost compresses dramatically across nodes** — $0.0014 (N16) → $0.0003 (N3), a 4.7× ratio that is the single biggest economic mover in the table. Output cost only compresses 3.4× (N16 $0.0034 → N3 $0.0010). This means **the strategic value of moving down nodes is concentrated in prefill-heavy workloads** because input cost dominates the blend.
- **The 8B input/output ratio stays roughly 1:1 across nodes** ($0.0037/$0.0039 at N16; $0.0013/$0.0016 at N3). 8B economics are flat in the input/output dimension; the only thing that changes with node is total cost. This makes 8B a node-insensitive product — pick whichever node hits the right price/quality balance.
- **The 30B input/output ratio also stays ~1:1** but at 5–17× higher cost than 100B-MoE on input. **The 30B SKU is dominated by 100B-MoE at every node and every P:D ratio.** Strategic recommendation: **drop the 30B SKU from the product roadmap unless it serves a specific architectural niche** (latency where MoE routing overhead is unacceptable, or dense-only customers). Replacing 30B with 100B-MoE in the customer pitch is essentially free upgrade in both quality and cost.

### Anchor against H100 BOM

- At **N16**, the 8B blended ~$0.0038 is comparable to DeepInfra's likely H100 BOM (~$0.005–$0.010, inferred from 5× retail markup on the $0.029 8B list price). Cost win at N16 is modest at the silicon-only layer; the bigger wedge is **energy efficiency**, which doesn't show up in $/Mtok at all.
- At **N3**, the 100B-MoE at $0.000378 (Pf 8:1) is **15–25× below H100 BOM** on a like-for-like model class. Pitch becomes "we beat everyone on $/Mtok before energy enters the conversation." This is the slide that makes a buyer's eyebrow move.

### Node-by-workload strategy

- **Pitch on N3, tape out at N7** for first silicon (4-month NRE saving, ~25% higher cost than N3 but 5× cheaper than N16). N7 still delivers 100B-MoE at $0.0007/$0.0014 — Pf 8:1 blended $0.000478, ~1,840× cheaper than Llama-70B Together. Defensible.
- **Pitch on N16, upgrade-path to N3** in the deck (under-promise, over-deliver). At N16 the 100B-MoE Pf 8:1 blended is $0.00162 — ~543× cheaper than market. Still a strong number.
- **Differentiate by node per workload**: N3 dies for prefill-heavy and latency-leveraged workloads (where extra perf and lowest input cost are monetized); N16 dies for decode-heavy and balanced workloads where additional N3 perf isn't fully captured.

---

## 10. Caveats on the Multipliers

The headline 2,110× and 14,634× numbers are silicon-BOM-vs-current-list-price. They are useful for thinking about gross margin headroom (BOM = 0.007% of Sonnet retail at Pf 5:1 → 99.99% gross margin if charging frontier prices), **not** for advertising as retail savings.

Real retail price layers in:
- **Data-center cost** — typically 3–10× over silicon BOM.
- **Power** — meaningful at scale; one of the accelerator's pitched advantages.
- **Networking** — depends on disaggregated-prefill architecture.
- **Gross margin** — typical inference-as-a-service vendors run 3–10× markup over BOM.
- **R&D and training amortization** — frontier labs amortize multi-billion training runs into per-token prices.

**At a 5× retail markup, the accelerator's effective retail blended $/Mtok would be:**

| Workload class | User N3 blend | 5× retail | vs. apples-to-apples market | vs. current Fr deployment |
|---|---|---|---|---|
| Voice (8B · Bal 1:1) | $0.00145 | $0.0073 | $0.69 (Groq 70B) → **95×** | $9 (Sonnet) → **1,233×** |
| Short Q&A (100B-MoE · Bal 1:1) | $0.00065 | $0.00325 | $0.88 (Llama-70B) → **271×** | $5.63 (GPT-5) → **1,731×** |
| In-IDE chat (100B-MoE · Pf 5:1) | $0.000417 | $0.00209 | $0.88 (Llama-70B) → **421×** | $5 (Sonnet 5:1) → **2,395×** |
| Single-doc summary (100B-MoE · Pf 8:1) | $0.000378 | $0.00189 | $0.88 (Llama-70B) → **466×** | $4.33 (Sonnet 8:1) → **2,291×** |
| Synthetic data (100B-MoE · Dc 1:5) | $0.000883 | $0.00442 | — | $12.92 (GPT-5.5 batch) → **2,924×** |

The advantage survives any reasonable markup assumption. Even at a **10× markup**, the worst-performing 100B-MoE workload (synthetic data) is still ~1,500× cheaper than current GPT-5.5 batch retail; the best (in-IDE chat) is still ~1,200× cheaper than Sonnet 5:1 retail.

---

## 11. Strategic Takeaways — Where to Lead the Pitch

Two slides essentially write themselves:

### Slide 1 — "We make the OS-L worker layer 2,100× cheaper and 2–5× faster"

Aimed at frontier labs and code-agent / deep-research products. The §9.3 hierarchical-worker thesis. The math:

- Worker calls have Pf 5:1 P:D (planner-dispatched read/search/summarize sub-tasks).
- Current worker layer: Llama-70B Together at $0.88/$0.88 = $0.88/Mtok blended. TTFT ~200–500ms.
- User N3 100B-MoE at Pf 5:1: $0.000417/Mtok. TTFT sub-100ms.
- Multiplier: **2,110×** on cost, **2–5×** on TTFT.

No quality-trust hurdle because the accelerator does not displace the user-visible planner. Customer set: Anthropic, OpenAI, Cursor, OpenAI Operator, Devin/Cognition, Replit Agent.

### Slide 2 — "We drop voice-agent LLM cost to a rounding error"

Current voice all-in is $0.12–$0.40/min; the LLM is currently $0.01–$0.04/min depending on tier. The math at N3:

- 8B blended (Bal 1:1) = $0.00145/Mtok.
- A typical voice turn: ~1K input + ~200 output. ~30 turns/min in steady conversation.
- Per-minute LLM cost on the accelerator: **~$0.00005/min** (vs. $0.04/min on Sonnet → **800× drop** on the LLM line item).

The LLM stops being a meaningful line item. The call's all-in floor reverts to **STT + TTS + telephony** ($0.05–$0.15/min combined). Everyone with telephony footprint becomes addressable: Vapi, Retell, Bland, ElevenLabs Agents, Hippocratic AI, telephony BPOs (TaskUs, Concentrix, Teleperformance).

### Where *not* to lead

The headline-grabbing target is workload **1 (consumer chat)** because the multiplier is ~8,650× vs. GPT-5, but it's the *least* differentiated by the accelerator's three properties — TTFT/tok/s/efficiency moat doesn't matter much there since DeepSeek and OpenAI free tier have already commoditized cost via subsidy. Lead with where the physics + cost story together carry the pitch:

1. **Voice agent (25)** — only workload where all three properties are simultaneously binding. 800× drop on LLM-per-minute math.
2. **Hierarchical worker layer (§9.3 of taxonomy)** — the largest hidden token pool, no quality-trust hurdle. 2,110× cost drop, 2–5× TTFT improvement.
3. **IDE inline completion (9)** — same physics story as voice, different buyer set, sitting ducks for energy/$ improvements.

Then expand into companions (8), online moderation (19), and short conversational Q&A (1) as second-wave addressable.

### Product-roadmap implication: drop the 30B SKU

The input/output split makes this concrete. **30B is dominated by 100B-MoE at every node and every P:D ratio.** At N3, 30B blended Pf 5:1 is $0.00515 vs. 100B-MoE Pf 5:1 at $0.000417 — a 12.4× gap. There is no workload class where 30B beats 100B-MoE on cost. The only reasons to keep 30B in the roadmap are:

- **Latency-niche:** 30B dense might hit lower TTFT than 100B-MoE on a single die (no expert-routing overhead). If the telco / voice / IDE-inline customer demands sub-100ms TTFT and routing latency is non-trivial on the accelerator, 30B dense fills that slot.
- **Trust-niche:** customers who refuse MoE for opaque reasons (regulatory, audit, deterministic reproducibility). Unclear how large this segment is.

If neither niche materializes, dropping 30B simplifies the product line, the SKU pricing, and the customer narrative. The accelerator's 8B and 100B-MoE SKUs cover the entire useful operating envelope.

---

## Open Questions / Next Steps

- **N16 vs N3 silicon-cost validation.** The BOM numbers in §1 anchor every multiplier in this document. A foundry-quote-validated cost basis is the prerequisite for any of these numbers landing in a deck.
- **Energy efficiency $/kWh translation.** The third property doesn't show up in $/Mtok comparisons. A perf-per-watt benchmark vs. H100 / B200 / Groq Language Processing Unit would let the energy story carry quantitative weight.
- **Worker-layer token-share measurement.** §9.3 of the taxonomy doc claims the OS-L worker layer is the largest hidden token pool. No public number found. A measurement (even a leaked log from one production agent product) would pin down the slide-1 TAM.
- **MoE serving viability on the accelerator.** The 100B Qwen-MoE entry assumes active-params-per-token fits the 8–31B budget. Memory bandwidth for the inactive expert weights (streaming or sharded) is the implementation question; the pitch depends on the answer.
- **Voice-agent buyer interviews.** Bland AI, Vapi, Retell, Hippocratic AI — first-conversation calls to validate the latency + perf-per-watt willingness-to-pay before silicon commits.

---

*Document compiled 2026-05-01.*
