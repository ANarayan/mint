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
| Modalities | Text-decoder primary (8B / 30B / 100B-MoE); **specialty SKUs** for encoder embedding (BGE-L) and image segmentation (SAM-H). Vision-language models (LLaVA, Qwen-VL class) and re-ranking cross-encoders not yet addressed. |

**Cost basis (BOM, $/Mtok by process node and model size):**

| Tier | Node | Input $/Mtok | Output $/Mtok | Blended (Pf 3:1) $/Mtok |
|---|---|---:|---:|---:|
| 8B Llama-3.1-8B | N16 | $0.003658 | $0.003853 | $0.003707 |
| 8B Llama-3.1-8B | N7 | $0.002408 | $0.002536 | $0.002440 |
| 8B Llama-3.1-8B | N5 | $0.001694 | $0.001848 | $0.001732 |
| 8B Llama-3.1-8B | N3 | $0.001297 | $0.001595 | **$0.001371** |
| 30B Gemma4-31B | N16 | $0.024163 | $0.025524 | $0.024503 |
| 30B Gemma4-31B | N7 | $0.010942 | $0.011558 | $0.011096 |
| 30B Gemma4-31B | N5 | $0.007436 | $0.007856 | $0.007541 |
| 30B Gemma4-31B | N3 | $0.005121 | $0.005409 | **$0.005193** |
| 100B Qwen-MoE | N16 | $0.001387 | $0.003449 | $0.001902 |
| 100B Qwen-MoE | N7 | $0.000750 | $0.001436 | $0.000921 |
| 100B Qwen-MoE | N5 | $0.000440 | $0.001254 | $0.000643 |
| 100B Qwen-MoE | N3 | $0.000285 | $0.001014 | **$0.000468** |

> **Three structural observations from the input/output split.**
>
> **(1) 100B-MoE input is dramatically cheaper than 30B and 8B input — at every node.** At N3, 100B-MoE input is $0.000285/Mtok, **18× cheaper than 30B input** ($0.005121) and **4.6× cheaper than 8B input** ($0.001297). The MoE active-params-per-token advantage shows up most where prefill dominates, which is most workloads (rows 4, 5, 9, 10, 11, 14, 15, 17, 18, 22, 23, 24, 26, 27, 28, 29, 30 in the master table all carry Pf labels).
>
> **(2) 100B-MoE output cost is 3.6× the input cost (at N3) — a structural ratio.** $0.000285 in / $0.001014 out reflects the MoE serving asymmetry: prefill batches expert-routes efficiently; decode is autoregressive and bounded by weight-streaming. The ratio holds across nodes: 2.5× at N16 ($0.001387 → $0.003449), 1.9× at N7, 2.85× at N5, 3.6× at N3 — increasingly output-dominant as the node shrinks. For decode-heavy workloads (rows 2, 8, 21 — long-form writing, companions, synthetic data) the cost advantage is smaller but still 4–10× over current OS-L list prices.
>
> **(3) 100B-MoE is unconditionally cheaper than 8B at N3 across all P:D ratios.** Both 100B-MoE input ($0.000285 < $0.001297) AND output ($0.001014 < $0.001595) are below 8B. This means there is **no P:D ratio at which 8B beats 100B-MoE on cost at N3**. The 8B SKU only wins when latency, on-device deployment, or quality-overkill considerations override pure cost. Even more striking: 30B is uncompetitive vs 100B-MoE at every node — same logic applies, with an 18× gap on input and 5.3× on output. Strategic implication: **the 30B SKU is hard to justify in the product roadmap unless it serves a specific architectural niche** (e.g., dense-only customers who refuse MoE, or sub-segment latency profile where 30B dense beats 100B-MoE on TTFT).

**Specialty SKU cost basis (BOM, by process node):**

| Model class | Unit | N16 | N7 | N5 | N3 |
|---|---|---:|---:|---:|---:|
| BGE-L (encoder-only embedding, ~335M params, 1024-dim) | $/Mqueries | $0.0345 | $0.0185 | $0.0135 | **$0.0101** |
| SAM-H (vision segmentation, 636M params, image-conditioned) | $/Mimages | $3.3415 | $1.7059 | $1.2233 | **$0.8953** |

These are specialty model classes outside the 8–31B decoder LLM envelope but architecturally within reach of the same accelerator silicon (encoder forward-pass for BGE-L; vision-encoder + mask-decoder for SAM-H). They unlock two adjacent workload classes — embeddings/RAG (BGE-L) and image segmentation (SAM-H) — that the master taxonomy table previously listed as "specialty / out of scope." Both are now addressable, with their own per-unit pricing primitives that don't fit the $/Mtok blended math used for decoder LLMs.

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
| 32 | Multimodal Q&A | Vision-language model quality gap (LLaVA / Qwen-VL class still trail Gemini / Claude vision); SAM-H addresses *segmentation* but not VL Q&A |
| 20 | Embedding generation | **Now addressable** via the **BGE-L SKU** (see §1 specialty cost basis and §7 specialty workloads table) |
| 22 | Re-ranking | Specialty cross-encoder model class — separate SKU question (Cohere Rerank / BGE-Reranker style); not yet covered by the BGE-L embedder SKU |

---

## 6. Where the Accelerator Expands the Pie

Workloads not currently in the master table that become economical with the accelerator's properties:

### A. Hierarchical worker sub-agents (§9.3 of the taxonomy doc)

The single biggest non-obvious target. Cursor Agent, Claude Code, OpenAI Deep Research, OpenAI Operator are all **frontier-planner + OS-L-worker** by architecture. The worker layer is:

- Bulk-of-token-volume in any agent trajectory.
- Sh–Med reasoning per call.
- Currently served on Together/Fireworks Llama-70B at $0.88/Mtok with 200–500ms TTFT.
- Tail-latency-bound: planner blocks on the slowest worker.

Accelerator on 100B-MoE at Pf 5:1 (typical worker P:D) blended cost: **$0.001731 at N16, $0.000864 at N7, $0.000576 at N5, $0.000407 at N3**. Versus the $0.88 worker-layer market price, that's **508× / 1,019× / 1,528× / 2,162× cheaper** depending on node. TTFT drops from 200–500ms to sub-100ms across nodes — the latency cut compounds across the trajectory because tail latency dominates.

The pitch positions to Anthropic / OpenAI / Cursor / OpenAI Operator as a *worker-layer accelerator* without ever competing on the planner. Their cost goes down 500–2,200× depending on node; throughput goes up; quality is unchanged.

### B. Real-time multimodal at the edge

Live captioning, mid-conversation translation in headphones, real-time dubbing. None of this works at frontier-API latency. Existing voice+translation infra is bolted-on; first-party low-TTFT 8–31B inference makes the round-trip clean.

### C. Universal autocomplete (not just IDE)

Keyboard predictions, email/doc autocomplete, terminal autocomplete, every input field. Currently nonexistent at scale because per-keystroke economics fail above ~$0.0001/inference. The accelerator's BOM at N3 ($0.001297 input / $0.001595 output for 8B per Mtok) puts this in range.

### D. Tool-call routing layers in agentic systems

Fast classifier to pick the right tool / sub-agent before any frontier reasoning fires. Today's routing layer is either keyword regex (cheap, brittle) or a frontier API call (slow, expensive). A sub-100ms 8B classifier eliminates the trade-off.

### E. Embedding-at-scale and high-volume RAG (BGE-L SKU)

Every enterprise RAG pipeline, semantic search system, vector database, and recommendation engine runs an embedding model. The market today routes through OpenAI text-embedding-3-large ($0.13/Mtok), Voyage 3-large ($0.18), Cohere Embed v3 ($0.10), AWS Titan v2 ($0.02), or self-hosted BGE / GTE / E5 ($0.005–$0.012 via DeepInfra/Cloudflare). Total embedding token volume is dominated by *indexing passes* (one-time at corpus ingestion) plus serving-time query embedding (continuous).

The BGE-L SKU at $0.0345/Mqueries (N16) → $0.0101/Mqueries (N3) targets this directly. At a 256-token typical RAG chunk that translates to ~$0.04/Mtok at N3 (vs DeepInfra BGE at $0.01 — within 4× of the cheapest OS-served competitor at the silicon-only layer), with the energy-efficiency wedge unlocking deployments that don't exist today (per-keystroke embedding for autocomplete; on-device semantic search; sovereign embedding for healthcare/legal corpora). See §7 specialty table for multipliers.

### F. Image segmentation as a primitive (SAM-H SKU)

SAM-H (Segment Anything Model — Huge, 636M params) is the canonical open-source image-segmentation primitive. Used in:
- **Image editing** (Adobe Firefly, Canva AI, Photoshop AI generative-fill — every "select object" workflow)
- **Medical imaging** (radiology AI segmenting tumors, organs, lesions on CT/MRI)
- **IDP regional extraction** (segment regions of a page before OCR — improves accuracy on complex layouts)
- **Robotics scene understanding** (object segmentation from RGB cameras)
- **AR / spatial computing** (Apple Vision Pro and equivalents do real-time scene segmentation)

Currently SAM-H is served via Replicate-hosted endpoints (~$0.001–$0.002/image, equivalent to $1,000–$2,000/Mimages) or self-hosted on H100s in vendor-specific stacks. The accelerator's SAM-H SKU at $3.34/Mimages (N16) → $0.90/Mimages (N3) is **500–2,200× cheaper** than current hosted serving — sufficient to make per-frame video segmentation economical (currently impossible at $1,500/Mimages × 30fps).

---

## 7. Cheaper-by Multipliers Across All Process Nodes

All multipliers are **BOM-to-list-price** ratios computed using each workload's actual prefill:decode ratio (e.g., a Pf 8:1 summarization workload uses 8 input tokens and 1 output token in the blend). They tell you the gross-margin headroom available, not the implied retail price (see §10 caveats).

**Computation:** User blend = (P × in_cost + 1 × out_cost) / (P + 1), where P is the workload's P:D ratio from §4 of the taxonomy doc. Market blend uses the same weighting on the listed input/output prices. Latency-bound workloads (voice, IDE inline) use **8B** because TTFT favors small-dense over MoE routing overhead; all other workloads default to **100B-MoE** because at every node it is unconditionally cheaper than 8B (see §1, observation 3).

The table below covers all 22 addressable workloads sorted by N3 multiplier descending, with the full N16 → N3 progression. The "Currently on" column shows the dominant production deployment today; the "Current $/Mtok" column is the blended market price at the workload's P:D ratio.

| # | Workload | Model · P:D | Currently on | Current $/Mtok | N16 × | N7 × | N5 × | N3 × |
|---|---|---|---|---|---|---|---|---|
| 2 | Long-form writing | 100B-MoE · Dc 1:5 | Fr (Sonnet 4.6) | $13 | 4,187× | 9,834× | 11,628× | **14,558×** |
| 21 🟢 | Synthetic data generation | 100B-MoE · Dc 1:5 | Fr batch (GPT-5.5) | $12.92 | 4,161× | 9,773× | 11,557× | **14,468×** |
| 10 | In-IDE chat | 100B-MoE · Pf 5:1 | Fr (Sonnet) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 15 | Enterprise search & grounded chat | 100B-MoE · Pf 5:1 | Fr (Sonnet) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 18 🟢 | IDP text extraction | 100B-MoE · Pf 5:1 | Fr (Sonnet) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 24 | Customer-service agent (text) | 100B-MoE · Pf 5:1 | Fr (Sonnet) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 27 | Workflow / sales / RPA agent | 100B-MoE · Pf 5:1 | Fr (Sonnet) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 28 | Healthcare scribe (text) | 100B-MoE · Pf 5:1 | Fr (Sonnet, BAA) | $5 | 2,889× | 5,787× | 8,681× | **12,285×** |
| 14 | Code search / repo Q&A | 100B-MoE · Pf 10:1 | Fr (Sonnet) | $4.27 | 2,713× | 5,259× | 8,307× | **12,165×** |
| 4 🟢 | Summarization, single document | 100B-MoE · Pf 8:1 | OS-L / Fr (Sonnet) | $4.33 | 2,679× | 5,242× | 8,170× | **11,831×** |
| 1 🟢 | Short conversational Q&A | 100B-MoE · Bal 1:1 | Fr (GPT-5) | $5.63 | 2,328× | 5,151× | 6,647× | **8,662×** |
| 7 | Practical guidance / tutoring | 100B-MoE · Bal 1:1 | Fr (GPT-5) | $5.63 | 2,328× | 5,151× | 6,647× | **8,662×** |
| 23 | Batch summarization / ETL | 100B-MoE · Pf 3:1 | OS-L (Databricks 70B) | $1.50 | 789× | 1,629× | 2,333× | **3,205×** |
| 31 | Education tutor | 100B-MoE · Bal 1:1 | Fr (Gemini 3 Flash) | $1.75 | 724× | 1,601× | 2,066× | **2,692×** |
| — | Hierarchical worker layer (§9.3) | 100B-MoE · Pf 5:1 | OS-L (Llama-70B Together) | $0.88 | 508× | 1,019× | 1,528× | **2,162×** |
| 17 | Open-domain citation chatbot | 100B-MoE · Pf 5:1 | OS-L (Sonar / Llama-70B) | $0.88 | 508× | 1,019× | 1,528× | **2,162×** |
| 3 | Editing, paraphrasing, critique | 8B · Bal 1:1 | Fr / OS-S (Haiku) | $1 | 266× | 405× | 565× | **692×** |
| 25 | Voice agent | 8B · Bal 1:1 | OS-S (Groq Llama-70B) | $0.69 | 184× | 279× | 390× | **477×** |
| 6 | Translation | 100B-MoE · Bal 1:1 | OS-L (Llama-70B DeepInfra) | $0.315 | 130× | 288× | 372× | **485×** |
| 8 | Companions / roleplay | 100B-MoE · Dc 1:5 | OS-L (in-house est. mid) | $0.35 | 113× | 265× | 313× | **392×** |
| 9 | IDE inline completion | 8B · Pf 100:1 | OS-S (custom small) | $0.05 | 13.7× | 20.8× | 29.5× | **38.5×** |
| 19 | Classification & moderation | 8B · Pf 50:1 | OS-S (Llama-8B) | $0.05 | 13.7× | 20.8× | 29.5× | **38.4×** |

**Excluded workloads (Frontier-only, not addressable by 8B / 30B / 100B-MoE):** rows 5 (multi-doc synthesis), 11 (code review), 12 (long-horizon code agent), 13 (app builder), 16 (deep research), 26 (browser/computer-use), 29 (legal contract), 30 (financial research), 32 (multimodal Q&A — VL-model gap), 22 (re-ranking — separate cross-encoder SKU not yet in product). Note: row 12 and row 16 worker-layer fragments are addressable via the hierarchical worker pattern (already in the table); the 7B-class decoder embedder subset of row 20 (NV-Embed-v2, E5-mistral-7B, gte-Qwen2-7B) is also addressable on the 8B / 100B-MoE SKUs but warrants its own row in a v2 of this table.

### Specialty Workloads — Addressable via BGE-L and SAM-H SKUs

Per-query / per-image pricing (different unit from $/Mtok), so kept in a separate sub-table. Multipliers below assume a 256-token typical RAG chunk for BGE-L (linear scaling: at 512-token chunks, multipliers double); SAM-H is an inherently per-image primitive.

| Workload | Model · per-unit | Currently on | Current price | N16 × | N7 × | N5 × | N3 × |
|---|---|---|---|---|---|---|---|
| 20 | Embedding (encoder, BGE-class) — apples-to-apples vs OS market | BGE-L · per query | OS (DeepInfra BGE-large) | $2.56/Mqueries | 74× | 138× | 190× | **253×** |
| 20a | Embedding tier-down vs frontier (OpenAI text-embedding-3-large) | BGE-L · per query | Fr (OpenAI 3-large) | $33.28/Mqueries | 965× | 1,799× | 2,465× | **3,295×** |
| 20b | Embedding tier-down vs Voyage-3-large (Anthropic-recommended) | BGE-L · per query | Fr (Voyage 3-large) | $46.08/Mqueries | 1,335× | 2,491× | 3,413× | **4,562×** |
| — | Image segmentation (SAM-class) | SAM-H · per image | OS hosted (Replicate, mid-est.) | $1,500/Mimages | 449× | 879× | 1,226× | **1,675×** |
| — | Image segmentation tier-down vs hosted-frontier (cloud Vision API class) | SAM-H · per image | Fr hosted ($0.0015/image) | $1,500/Mimages | 449× | 879× | 1,226× | **1,675×** |

**Reading the BGE-L multipliers.** The 253× apples-to-apples figure is the realistic cost-arbitrage number against OS-served BGE on commodity GPU clouds. The 3,295× and 4,562× figures vs. OpenAI/Voyage frontier embeddings are tier-down opportunities — relevant for the slice of buyers who default to frontier embeddings out of convenience, not quality necessity (BGE-L on MTEB is competitive with text-embedding-3-large for general English retrieval).

**Reading the SAM-H multipliers.** The 1,675× number assumes a $1,500/Mimages mid-estimate for current Replicate-hosted SAM-H serving. Cloud Vision APIs (AWS Rekognition Custom Labels, GCP Vision AI for similar capabilities) cluster at $1,000–$1,500/Mimages, so the multiplier holds across hosted alternatives. The pitch lands hardest where per-frame/per-image volume is large and currently uneconomical — video segmentation, real-time AR scene parsing, high-volume IDP regional extraction.

**Three readings of the column patterns:**

1. **The N3 → N16 multiplier compression is steepest on prefill-heavy 100B-MoE workloads** (rows 4, 14, and the Pf 5:1 cluster). N3 to N7 keeps ~50% of the N3 multiplier; N3 to N16 keeps ~25%. This is because 100B-MoE input cost is the steepest-compressing cell in §1's table — the strategic value of N3 is concentrated where input dominates the blend.
2. **Decode-heavy workloads compress less aggressively** (rows 2, 21 — long-form writing, synthetic data). N3 → N16 keeps ~30%. This is because output cost (which dominates Dc blends) compresses 3.4× across nodes vs. input's 4.7×.
3. **Same-tier OS-S workloads (rows 9, 19) compress nearly linearly** with node — 13.5× (N16) → 38× (N3). At N16 the multiplier is no longer headline-grade for these workloads; selling them needs N5 or better.

---

## 8. How the Multiplier Bands Compress Across Nodes

The table's 22 rows fall into roughly five multiplier bands. The band shifts across nodes are predictable and worth memorizing for the pitch:

| Multiplier band | Pattern | At N3 | At N7 | At N5 | At N16 |
|---|---|---|---|---|---|
| **Tier A** — Fr-deployed Dc-heavy | Long-form writing, synthetic data | ~14,500× | ~9,800× | ~11,600× | ~4,200× |
| **Tier B** — Fr-deployed Pf 5–10:1 | In-IDE chat, search, IDP, CS, RPA, healthcare, code search, single-doc summary | 11,800–12,300× | 5,200–5,800× | 8,200–8,700× | 2,700–2,900× |
| **Tier C** — Fr-deployed Bal | Short Q&A, practical guidance | ~8,660× | ~5,150× | ~6,650× | ~2,330× |
| **Tier D** — OS-L deployed Pf-mid | Batch ETL, education tutor, hierarchical worker, citation chatbot | 2,160–3,200× | 1,020–1,630× | 1,530–2,400× | 510–790× |
| **Tier E** — OS-L deployed Bal/Dc, latency-bound 8B | Translation, voice, companions, editing | 390–690× | 265–405× | 310–570× | 110–270× |
| **Tier F** — Same-tier OS-S | IDE inline, classification | ~38× | ~21× | ~30× | ~13.7× |

**Three-way structural read:**

- **Tier A and B survive node degradation gracefully.** Even at N16 the multipliers are 2,700–4,250×, more than enough to anchor any sales conversation. **The Fr-displacement pitch works at every node.**
- **Tier D begins losing headline strength at N16** (500–800×). Still strong but less viral. Pitch needs to lean on TTFT or energy rather than $/Mtok at older nodes.
- **Tier F (OS-S same-tier) only works at advanced nodes.** At N16 the multiplier is 13.5×, which is real but unimpressive vs. existing Llama-8B serving. **Don't lead with these workloads if your tape-out is N16.**

**Strategic node-by-portfolio recommendation:**

- **Initial customer portfolio = Fr-displacement (Tiers A/B/C):** Tape out at N7 or even N16 and the multipliers still carry the pitch. The MoE input compression dominates economics and is meaningful even at older nodes.
- **Initial customer portfolio = OS-L cost-arbitrage (Tier D/E):** Tape out at N5 minimum. At N16 these workloads' multipliers (110–800×) are real but compete against Together / Fireworks / DeepInfra economics that are themselves improving over time.
- **Initial customer portfolio = OS-S head-on (Tier F):** Don't pitch at N16. Need N5 or N3 to clear the credibility bar against existing $0.05/Mtok serving.

---

## 9. Node Sensitivity — How N16/N7/N5/N3 Changes the Pitch

The ranking of which workloads win **does not change** with node — voice, IDE inline, hierarchical worker, companions, and consumer-chat-tier-down stay the top fits at every node. What changes is how aggressively the accelerator can price.

**Input / output cost spread by node and model (transposed view of §1):**

| Tier | 8B in / out | 30B in / out | 100B-MoE in / out | Best-fit workload pitch |
|---|---|---|---|---|
| **N16** | $0.003658 / $0.003853 | $0.024163 / $0.025524 | $0.001387 / $0.003449 | "110–2,900× cheaper than current OS-L for prefill-heavy workloads. Conservative pitch, defensible numbers." |
| **N7** | $0.002408 / $0.002536 | $0.010942 / $0.011558 | $0.000750 / $0.001436 | "265–5,800× cheaper. Credible Series-B middle ground." |
| **N5** | $0.001694 / $0.001848 | $0.007436 / $0.007856 | $0.000440 / $0.001254 | "310–8,700× cheaper. Where you want to be at scale." |
| **N3** | $0.001297 / $0.001595 | $0.005121 / $0.005409 | $0.000285 / $0.001014 | "390–12,300× cheaper for prefill-heavy 100B-MoE. Headline-friendly but only credible with named foundry slot." |

### What the input/output split exposes about node strategy

- **The 100B-MoE input cost compresses 4.87× across nodes** — $0.001387 (N16) → $0.000285 (N3) — the single biggest economic mover in the table. Output cost compresses 3.40× (N16 $0.003449 → N3 $0.001014). This means **the strategic value of moving down nodes is concentrated in prefill-heavy workloads** because input cost dominates the blend.
- **The 8B input/output ratio stays roughly 1:1 across nodes** ($0.003658/$0.003853 at N16; $0.001297/$0.001595 at N3). 8B economics are flat in the input/output dimension; the only thing that changes with node is total cost. This makes 8B a node-insensitive product — pick whichever node hits the right price/quality balance.
- **The 30B input/output ratio also stays ~1:1** but at 4.6–18× higher cost than 100B-MoE on input. **The 30B SKU is dominated by 100B-MoE at every node and every P:D ratio.** Strategic recommendation: **drop the 30B SKU from the product roadmap unless it serves a specific architectural niche** (latency where MoE routing overhead is unacceptable, or dense-only customers). Replacing 30B with 100B-MoE in the customer pitch is essentially free upgrade in both quality and cost.

### Anchor against H100 BOM

- At **N16**, the 8B blended ~$0.003707 is comparable to DeepInfra's likely H100 BOM (~$0.005–$0.010, inferred from 5× retail markup on the $0.029 8B list price). Cost win at N16 is modest at the silicon-only layer; the bigger wedge is **energy efficiency**, which doesn't show up in $/Mtok at all.
- At **N3**, the 100B-MoE at $0.000366 (Pf 8:1 blended) is **15–25× below H100 BOM** on a like-for-like model class. Pitch becomes "we beat everyone on $/Mtok before energy enters the conversation." This is the slide that makes a buyer's eyebrow move.

### Node-by-workload strategy

- **Pitch on N3, tape out at N7** for first silicon (4-month NRE saving, ~2× higher per-token cost than N3 but ~2× cheaper than N16). N7 still delivers 100B-MoE at $0.000750/$0.001436 — Pf 8:1 blended $0.000826, ~5,242× cheaper than Sonnet 8:1. Defensible.
- **Pitch on N16, upgrade-path to N3** in the deck (under-promise, over-deliver). At N16 the 100B-MoE Pf 8:1 blended is $0.001616 — ~2,679× cheaper than Sonnet at the same P:D. Still a strong number.
- **Differentiate by node per workload**: N3 wins for prefill-heavy and latency-leveraged workloads (where extra perf and lowest input cost are monetized); N16 is sufficient for decode-heavy and balanced workloads where additional N3 perf isn't fully captured.

---

## 10. Caveats on the Multipliers

The headline 2,162× and 14,468× numbers are silicon-BOM-vs-current-list-price. They are useful for thinking about gross margin headroom (BOM = 0.008% of Sonnet retail at Pf 5:1 → 99.99% gross margin if charging frontier prices), **not** for advertising as retail savings.

Real retail price layers in:
- **Data-center cost** — typically 3–10× over silicon BOM.
- **Power** — meaningful at scale; one of the accelerator's pitched advantages.
- **Networking** — depends on disaggregated-prefill architecture.
- **Gross margin** — typical inference-as-a-service vendors run 3–10× markup over BOM.
- **R&D and training amortization** — frontier labs amortize multi-billion training runs into per-token prices.

**At a 5× retail markup, the accelerator's effective retail blended $/Mtok at N3 would be:**

| Workload class | User N3 blend | 5× retail | vs. apples-to-apples market | vs. current Fr deployment |
|---|---|---|---|---|
| Voice (8B · Bal 1:1) | $0.001446 | $0.00723 | $0.69 (Groq 70B) → **95×** | $9 (Sonnet) → **1,245×** |
| Short Q&A (100B-MoE · Bal 1:1) | $0.000650 | $0.00325 | $0.88 (Llama-70B) → **271×** | $5.63 (GPT-5) → **1,732×** |
| In-IDE chat (100B-MoE · Pf 5:1) | $0.000407 | $0.00204 | $0.88 (Llama-70B) → **432×** | $5 (Sonnet 5:1) → **2,455×** |
| Single-doc summary (100B-MoE · Pf 8:1) | $0.000366 | $0.00183 | $0.88 (Llama-70B) → **481×** | $4.33 (Sonnet 8:1) → **2,366×** |
| Synthetic data (100B-MoE · Dc 1:5) | $0.000893 | $0.00447 | — | $12.92 (GPT-5.5 batch) → **2,894×** |

The advantage survives any reasonable markup assumption. Even at a **10× markup**, the worst-performing 100B-MoE workload (synthetic data) is still ~1,450× cheaper than current GPT-5.5 batch retail; the best (in-IDE chat) is still ~1,225× cheaper than Sonnet 5:1 retail.

---

## 11. Strategic Takeaways — Where to Lead the Pitch

Four slides essentially write themselves:

### Slide 1 — "We make the OS-L worker layer 2,160× cheaper and 2–5× faster"

Aimed at frontier labs and code-agent / deep-research products. The §9.3 hierarchical-worker thesis. The math:

- Worker calls have Pf 5:1 P:D (planner-dispatched read/search/summarize sub-tasks).
- Current worker layer: Llama-70B Together at $0.88/$0.88 = $0.88/Mtok blended. TTFT ~200–500ms.
- User N3 100B-MoE at Pf 5:1: $0.000407/Mtok. TTFT sub-100ms.
- Multiplier: **2,162×** on cost, **2–5×** on TTFT.

No quality-trust hurdle because the accelerator does not displace the user-visible planner. Customer set: Anthropic, OpenAI, Cursor, OpenAI Operator, Devin/Cognition, Replit Agent.

### Slide 2 — "We drop voice-agent LLM cost to a rounding error"

Current voice all-in is $0.12–$0.40/min; the LLM is currently $0.01–$0.04/min depending on tier. The math at N3:

- 8B blended (Bal 1:1) = $0.001446/Mtok.
- A typical voice turn: ~1K input + ~200 output. ~30 turns/min in steady conversation.
- Per-minute LLM cost on the accelerator: **~$0.00005/min** (vs. $0.04/min on Sonnet → **800× drop** on the LLM line item).

The LLM stops being a meaningful line item. The call's all-in floor reverts to **STT + TTS + telephony** ($0.05–$0.15/min combined). Everyone with telephony footprint becomes addressable: Vapi, Retell, Bland, ElevenLabs Agents, Hippocratic AI, telephony BPOs (TaskUs, Concentrix, Teleperformance).

### Slide 3 — "We make embedding 253× cheaper than the cheapest OS competitor (3,295× vs OpenAI)"

The BGE-L SKU pitch. Embeddings are the load-bearing primitive of every RAG pipeline, semantic search system, vector database, and recommendation engine — and the market today bifurcates between expensive frontier embedders ($0.13–$0.18/Mtok at OpenAI / Voyage / Cohere) and cheap OS-served BGE / GTE / E5 ($0.005–$0.012/Mtok at DeepInfra / Cloudflare). The accelerator collapses both:

- Vs. **OS-served BGE** (apples-to-apples): **253× cheaper** at N3 ($0.0101/Mqueries vs $2.56/Mqueries on DeepInfra at 256-tok queries). Buyers already on OS embeddings get a drop-in cost cut.
- Vs. **frontier OpenAI text-embedding-3-large**: **3,295× cheaper** at N3. Buyers using OpenAI for "convenience embeddings" can migrate to BGE-L on the accelerator without quality loss for general English retrieval (BGE-L is competitive with text-embedding-3-large on MTEB).
- Vs. **Voyage 3-large** (Anthropic-recommended): **4,562× cheaper** at N3.

Customer set: every enterprise running RAG (the entire enterprise-search and grounded-chatbot category — rows 14, 15, 17, 27, 28 of the taxonomy), every vector-DB customer (Pinecone, Weaviate, Qdrant, Chroma, Turbopuffer), every recommendation-engine team. The wedge unlocks deployments that don't exist today — per-keystroke embedding for autocomplete, sovereign embedding for healthcare/legal corpora, on-device semantic search.

### Slide 4 — "We make image segmentation 1,675× cheaper, enabling per-frame video segmentation"

The SAM-H SKU pitch. Current SAM-H serving on Replicate clusters at $1,000–$2,000/Mimages — sufficient for one-shot segmentation in image editors but uneconomical at video frame rate (30fps × 1 hour = 108K images = $108–$216 per hour of video). The accelerator's $0.8953/Mimages at N3 makes a one-hour video segmentable for **$0.10**, which unlocks:

- **Per-frame video segmentation** for video editing (Adobe Premiere AI, DaVinci Resolve, Runway, Pika)
- **Real-time AR scene parsing** for Vision Pro / Quest / smart-glasses (currently latency-bound, not just cost-bound, but the cost cut funds dedicated edge silicon)
- **High-volume IDP regional extraction** (segment regions of every page before OCR — currently a "premium" feature priced at the high end of IDP)
- **Medical imaging at scale** (segment every slice of every CT/MRI volume in a hospital's archive — currently impossible at $1,500/Mimages)
- **Robotics scene understanding** at frame rate

Customer set: image-editing and creative-media platforms, AR/spatial-computing OEMs, medical-imaging AI vendors (Aidoc, Annalise, Rad AI), robotics platforms (Figure, 1X, Boston Dynamics).

### Where *not* to lead

The headline-grabbing target is workload **1 (consumer chat)** because the multiplier is ~8,650× vs. GPT-5, but it's the *least* differentiated by the accelerator's three properties — TTFT/tok/s/efficiency moat doesn't matter much there since DeepSeek and OpenAI free tier have already commoditized cost via subsidy. Lead with where the physics + cost story together carry the pitch:

1. **Voice agent (25)** — only workload where all three properties are simultaneously binding. 800× drop on LLM-per-minute math.
2. **Hierarchical worker layer (§9.3 of taxonomy)** — the largest hidden token pool, no quality-trust hurdle. 2,162× cost drop, 2–5× TTFT improvement.
3. **IDE inline completion (9)** — same physics story as voice, different buyer set, sitting ducks for energy/$ improvements.
4. **BGE-L embedding SKU** — 253× vs OS market, 3,295× vs OpenAI. Drop-in for every RAG pipeline.
5. **SAM-H image segmentation SKU** — 1,675× vs hosted SAM-H. Unlocks per-frame video segmentation.

Then expand into companions (8), online moderation (19), and short conversational Q&A (1) as second-wave addressable.

### Product-roadmap implication: drop the 30B SKU

The input/output split makes this concrete. **30B is dominated by 100B-MoE at every node and every P:D ratio.** At N3, 30B blended Pf 5:1 is $0.005169 vs. 100B-MoE Pf 5:1 at $0.000407 — a 12.7× gap. There is no workload class where 30B beats 100B-MoE on cost. The only reasons to keep 30B in the roadmap are:

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
