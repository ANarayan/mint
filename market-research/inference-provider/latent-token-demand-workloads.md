# Latent Token Demand — Workloads That Don't Exist at Current Prices

*A working catalogue of the specific workload classes that are uneconomical or impossible at today's inference cost / latency / energy curves but become real when those curves move. The companion to `tokens-too-cheap-to-meter-thesis.md` (which argues the cost curve will move) and `datacenter-inference-business-analysis.md` (which describes the business that captures today's demand). This document operationalizes the "expand-the-pie" thesis with named workloads, TAM intuitions, gating factors, and 2026/2028/2030 sequencing.*

> Companion to `tokens-too-cheap-to-meter-thesis.md` (cost-curve framing, historical precedents, unmetering inflection) and `datacenter-inference-business-analysis.md` (current-demand business case + top-5 wedges). Read both first; this one assumes the cost curve moves and asks: *what specifically gets unlocked, by how much, and when?*

> Builds on the workload taxonomy at `../llm-inference-workloads/llm-inference-workloads-taxonomy.md` and the accelerator-fit analysis at `../llm-inference-workloads/accelerator-workload-fit-analysis.md`. Where the taxonomy describes 32 workloads that exist today, this document names the ~10 that don't yet exist at scale because the cost / latency / energy ceiling hasn't lifted.

---

## 1. The Framing — What Constrains Demand Today

**Q: Why isn't every conceivable AI use case already a deployed workload?**

Five structural constraints, in rough order of severity:

1. **Cost** — per-inference economics fail above a threshold. A use case that fires 1B times/day at $0.001 each is $1M/day; the same use case firing at $0.00001 is $10K/day, a fraction of the value created. *The cost ceiling is the most-cited but not always the binding one.*
2. **Latency** — TTFT or end-to-end latency too slow for the user experience. Voice agents at 800ms break the conversation; live captioning at 1s lags speech; per-frame video AI at 100ms misses real-time. *Latency caps frequency-of-inference.*
3. **Energy** — the device or deployment site can't sustain the wattage. Always-on wearables, in-cabin automotive, on-device assistants at the edge. *Energy caps where inference can physically live.*
4. **Sovereignty / data residency** — data can't legally leave certain jurisdictions or networks. HIPAA, GDPR, defense, court records. *Sovereignty caps what cloud serving can address.*
5. **Distribution / integration** — the API isn't yet exposed to the consuming surface (keyboard, OS, car, appliance, game engine). *Distribution lag is a 24–36 month catalysis, separate from the others.*

**The proprietary chip's three properties + cheap COGS lift constraints (1), (2), (3) simultaneously.** Constraint (4) is partially addressed by sovereign DC deployments. Constraint (5) is independent — even with cheap, fast, energy-efficient inference, integration into Apple/Android/Windows/auto-OEM/game-engine stacks is a separate timeline.

> **The thesis of this document:** *some token demand is gated by a single constraint (cost OR latency OR energy). That demand expands incrementally as constraints lift one at a time. But the most valuable latent demand is gated by **multiple** constraints simultaneously — and that demand only emerges when a single supplier addresses all three at once. The proprietary chip is exactly such a supplier; the workloads that emerge first are those at the intersection.*

---

## 2. Eight Categories of Latent Demand

Each subsection: what's blocked today, what unlocks at our pricing/latency/energy point, named first-customer companies, rough TAM intuition, what the chip needs to prove.

### 2.1 Per-frame / per-event inference (frequency unlock)

**The constraint today.** Inference is too expensive to fire at native event rate. Video moderation samples 1-in-N frames; security analysis batches transactions; database enrichment runs nightly batches.

**What unlocks at $0.001/Mtok (decoder) or $0.001/Mimage (segmentation):**

- **Per-frame video segmentation, tracking, content moderation at native 30/60fps.** A 1-hour stream is 108K frames; today's $1,500/Mimages → $0.16/hr per stream, manageable for a few high-value streams. At our SAM-H pricing ($0.90/Mimages at N3) → $0.0001/hr — every Twitch stream, every TikTok upload, every Zoom call, every YouTube live moderated frame-by-frame.
- **Per-transaction LLM reasoning for fraud / claims / risk.** Visa processes 700M+ transactions/day; at sub-cent-per-inference, full LLM reasoning per transaction becomes viable (today's rule-based + ML classifier stack misses subtle fraud patterns LLM reasoning catches).
- **Per-keystroke security telemetry.** Every command typed in production environments analyzed for anomaly. SIEM tools (Splunk, Datadog Security, Snyk) sample today.
- **Per-row analytics enrichment.** Every database row continuously enriched with LLM-extracted features. dbt + Snowflake / Databricks customers running batch nightly today move to real-time. Census, Hightouch, Materialize integration.

**Named first customers.** Twitch / TikTok / YouTube (video moderation), Hive / Cinder / Sentropy successors (T&S vendors), Visa / Mastercard / Adyen / Stripe (txn fraud), Datadog Bits AI / Splunk / Snyk (sec ops), dbt Labs / Census / Hightouch (data eng).

**TAM intuition.** Video moderation alone is ~$2–3B annual spend (heavily human-reviewed today). Transaction-level fraud reasoning at the top-10 card networks: ~$500M–$1B addressable. SIEM AI: ~$1B emerging. **Combined latent TAM: $5–10B by 2030.**

**Gating factors.** Cost is the dominant lifting; latency matters for live moderation; integration into existing pipelines (FFmpeg, Kafka, dbt) is the catalysis.

---

### 2.2 AI-native gaming and procedural content

**The constraint today.** Generative content costs $0.05–$0.50 per interaction at frontier prices. A $0.99 mobile game can't afford $0.50 per NPC line; an MMO with 100K concurrent players can't afford $0.05 per interaction × 100K × dozens-of-interactions-per-session.

**What unlocks:**

- **LLM-native NPCs at platform scale.** Every NPC has dynamic dialogue, persistent memory, and reactive behavior. Inworld AI, Convai (NVIDIA ACE partner) — today demo-grade. At our pricing, Riot, Activision-Blizzard, EA, Tencent, NetEase, Roblox can deploy across tentpole titles.
- **Procedural quest / level generation per player.** Every player gets a unique narrative thread. Today: bespoke (Replica AI, AI Dungeon) or seeded (No Man's Sky-style procedural).
- **Real-time speech-driven game interaction.** Voice in / dynamic NPC voice out at sub-300ms TTFT. The voice-agent property meets gaming. Roblox already has voice; LLM-driven NPC voice generation is the next layer.
- **Spectator AI commentary.** Automatic real-time commentary on any esports stream, any live event. Riot, ESL, Activision (CDL/OWL), every league.
- **AI-native game design tools.** UEFN-style AI assistance for game devs (procedural asset generation, scripting, balance tuning).

**Named first customers.** Inworld AI (already shipping), Convai, Roblox Studio (UEFN-style), Riot (Champions / League NPCs), Epic Games (Fortnite), Microsoft Gaming / Xbox, EA (FIFA/EA Sports characters), Tencent (mobile gaming dominance), NetEase, Activision-Blizzard (post-Microsoft acquisition).

**TAM intuition.** Roblox 70M DAU; if 1% of session interactions used AI tokens at our pricing, that's ~10B tokens/day at trivial unit cost — tens of millions of dollars in tokens spent annually by Roblox alone. Activision-Blizzard, EA, Tencent each at least 5–10× larger. Gaming has historically been ~$200B/yr revenue; AI-token-as-input-cost layer plausibly $5–20B by 2030.

**Gating factors.** Cost is binding; latency for voice/real-time NPCs; *but the bigger gate is game-engine integration*. Unreal Engine, Unity, proprietary engines need plug-in support. ~24-month catalysis from cost-availability to platform deployment.

---

### 2.3 Always-on personal AI / wearables

**The constraint today.** Continuous inference on a wearable today costs $5–$20/day per user (rough estimate based on 1-2K turns × $0.04/turn); nobody pays that for a $9.99/month service. The category exists (Friend, Limitless, bee.computer, Plaud) but is rate-limited by COGS.

**What unlocks:**

- **Continuous companion AI** — always-listening, always-thinking AI on the wrist, lapel, neckband, or earbuds. Cost per user-day at our pricing drops to **cents**, making subscription viable at $9.99/mo with healthy margin.
- **Always-on transcription + memory** — every meeting, every call, every conversation captured + embedded. Otter / Granola / Limitless at universal scale, not just knowledge-worker premium tier.
- **Continuous health monitoring with LLM reasoning** — Apple Watch / Whoop / Oura combined with LLM interpretation of every signal stream. Today health AI is reactive (you ask a question); always-on makes it proactive (it tells you when something's anomalous).
- **Real-time interpretive AI** — earbuds with continuous translation (Pixel Buds Pro extends), accessibility (live captioning for deaf users), context augmentation (live fact-check, identity recall).
- **Vehicle conversational AI** — every car interaction is voice-driven and LLM-mediated. Currently bottlenecked by patchy connectivity (cloud-routed AI fails in tunnels, rural). Energy-efficient on-device + cellular fallback unlocks.

**Named first customers.** Friend (Avi Schiffmann), Limitless (Dan Siroker, ex-Optimizely), bee.computer (Maria Zhang ex-Apple), Plaud, Humane successors, Apple (Apple Intelligence on Watch — rumored), Meta (Ray-Ban / Quest), Pixel Buds, Sennheiser/Bose hearables, Whoop, Oura, Garmin, BMW / Mercedes / Tesla in-cabin AI, Rivian.

**TAM intuition.** Apple sells 80M+ watches/year, Meta Ray-Ban has 2M+ pairs, Whoop/Oura combined ~5M paying users, automotive AI on 100M+ new vehicles/yr globally. **Wearables + automotive addressable for always-on AI: ~150M units/year by 2028, $20–50B/year inference TAM at our prices.**

**Gating factors.** Energy efficiency is the binding constraint (battery on wearables; cabin power on automotive). Cost is necessary but not sufficient. Latency matters (sub-1s for conversation). *Distribution into the OS / device-OEM layer is the slowest gate* — Apple Intelligence / Android Gemini Nano are 2–3 year cycles.

---

### 2.4 Per-customer / per-creative personalization at scale

**The constraint today.** Marketing-tech platforms generate 5–10 cohort variants for each campaign because LLM-generated personalization at 100M-user scale costs more than the lift it produces.

**What unlocks:**

- **Per-user email subject lines, push-notification copy, ad creative.** Klaviyo, Braze, Iterable, Twilio Segment customer base shifts from cohort-level A/B to user-level personalization. 100M-recipient campaign × 1 LLM call per recipient at $0.01/Mtok = ~$1K total — completely viable.
- **Per-user product description / page copy.** Amazon-class commerce with truly personalized PDP text per user-session. Shopify Plus, BigCommerce, Salesforce Commerce Cloud customers extend.
- **Per-customer support persona tuning.** Sierra / Decagon CS agents that not only answer your question but match your communication style, sensitivity, language, prior history. *Already happening at the high end; becomes universal at our prices.*
- **Per-search-query result rendering + summarization.** Every search query gets a custom-rendered result page synthesizing the relevant results. Google AI Overviews does this for some queries; expands to all with cheap inference.
- **Per-investor analyst report rendering.** Every Bloomberg / Refinitiv / FactSet user gets a custom-rendered analyst summary on every ticker they follow.

**Named first customers.** Klaviyo (~$1B ARR), Braze, Iterable, Twilio Segment, MailChimp / Intuit, Amazon (PDP), Shopify / Shop, Walmart, Target, Salesforce (Marketing Cloud + Service Cloud), HubSpot, Mailgun, SendGrid, Bloomberg AI, FactSet, Refinitiv.

**TAM intuition.** Marketing-tech inference today ~$1B; could plausibly 5–10× to $5–10B with per-customer generation universal.

**Gating factors.** Cost binding; quality acceptable on Sh-Med reasoning (no frontier needed for marketing copy); integration with existing marketing platforms ~12-month catalysis.

---

### 2.5 Continuous evaluation, fine-tuning, and indexing

**The constraint today.** Evals are sampled (Patronus / Braintrust / Arize / LangSmith / W&B Weave all batch-test); fine-tunes are static (DPO / RLAIF research-grade); embedding indexes are rebuilt nightly. Continuous-everything would multiply infra cost 100×+.

**What unlocks:**

- **Eval-as-a-service at population scale.** Every LLM deployment continuously tested against millions of synthetic test cases. Today: 10K-test-case sampled batches. Becomes universal — every prompt change, every model swap, every system-prompt update auto-evaluated against full eval set.
- **Continuous DPO / RLAIF.** Every customer-facing LLM deployment auto-tunes on its own usage data daily. Currently a research project (Anthropic's "Constitutional AI" auto-loop); becomes the default deployment pattern.
- **Live embedding refresh on document change.** Every contract, every policy doc, every wiki edit triggers immediate re-embedding. Today: nightly batch (Pinecone, Weaviate, Qdrant, Chroma all assume static-mostly indexes).
- **Continuous model distillation.** Every fine-tune of a frontier model auto-distills to OS-100B for serving cost. The hierarchical-worker pattern (described in `accelerator-workload-fit-analysis.md` §6.A) becomes self-maintaining: when Anthropic ships Opus 5, the worker layer auto-redistills against new outputs.
- **Continuous data-quality monitoring.** Every row of every database column continuously LLM-checked for anomaly, schema drift, quality degradation.

**Named first customers.** Patronus AI, Braintrust, Arize, LangSmith (LangChain), Weights & Biases Weave, Helicone, Galileo, Databricks Mosaic AI Eval, Snowflake Cortex Eval, Anthropic / OpenAI internal eval pipelines (huge consumers), every F500 ML team, Pinecone, Weaviate, Qdrant, Chroma, Turbopuffer.

**TAM intuition.** AI infrastructure tooling is currently ~$300M ARR collectively (LangChain ~$40M, W&B ~$200M, Arize ~$50M, Patronus / Braintrust smaller). 5–10× expansion plausible at unlock → $2–4B by 2030. *And the underlying token consumption is much larger than the tooling revenue — these tools consume 10–100M tokens per customer per day at unlock, multiplying inference spend.*

**Gating factors.** Cost binding. Software integration is mostly already there (the eval/fine-tune frameworks exist and are framework-agnostic). Quality: the eval/distill loops need careful design but are research-mature.

---

### 2.6 Sovereign AI for tier-2/3 nations

**The constraint today.** Building national AI capacity requires capex/expertise at NVIDIA-scale; tier-2 nations can't compete with US/China hyperscalers. Sovereign data residency is the explicit reason; energy density is the implicit one (smaller national power grids).

**What unlocks:**

- **Energy-efficient deployable AI for emerging markets.** Singapore, UAE, Saudi (already moving), Egypt, Turkey, Vietnam, Indonesia, Philippines, Mexico, Brazil, Chile, Colombia, South Africa, Kenya, Nigeria, Pakistan. Each wants its own AI-as-public-good infrastructure inside its own jurisdiction.
- **Public-sector AI inside national jurisdiction.** Every government agency, court, regulator, ministry, public-health-system. Today: a fraction of what's possible because cloud-based AI loses to data-residency rules in regulated sectors.
- **Defense / intelligence AI for non-US/non-China allies.** Australia, UK, Germany, France, Italy, Spain, Netherlands, Japan, Korea, India, Israel. Sovereign defense AI is an explicit national-security priority for each — UK AI Safety Institute, French Mistral state-aligned investment, German Aleph Alpha public funding, Japan's METI AI strategy, India's IndiaAI Mission.
- **Sovereign developer-cloud.** Each tier-2 nation hosting their own Cohere/Mistral/Together-equivalent inside-jurisdiction.

**Named first customers.** UAE G42 / TII (Abu Dhabi sovereign), Saudi HUMAIN / SDAIA, India IndiaAI / TRAI / RBI, Singapore IMDA / GovTech / DSTA, France BPI / Mistral state contracts, UK AISI / NHS / MoD, Germany Aleph Alpha (state-aligned), Japan METI / NICT, Korea NIA, Israel MoD / INSA, Australia DoD / DSTG.

**TAM intuition.** NVIDIA's Sovereign AI revenue is forecast at $5–10B for 2026 alone, growing rapidly. By 2030, sovereign AI infra spend plausibly $50–100B aggregated — and energy-efficient deployable density is the binding selection criterion (NVIDIA Blackwell at high power consumption is hard to deploy in power-constrained sovereign DCs).

**Gating factors.** Energy efficiency is binding (sovereign DC power constraints in non-hyperscaler jurisdictions). Cost matters but is secondary to "sovereign" status. Sales motion is government / state-owned-enterprise procurement, 18–36 month cycles.

---

### 2.7 Industrial / scientific archive-scale inference

**The constraint today.** Running modern AI over an enterprise's full historical archive costs more than the value of the insights. So AI gets applied to *new* data going forward, never to the historical corpus.

**What unlocks:**

- **Every medical image in a hospital archive analyzed.** Not just new scans. A hospital's 10-year imaging archive is 100M+ studies; segmenting + reasoning over all of it at our SAM-H pricing is *hundreds* of dollars instead of *millions*. Aidoc, Rad AI, Annalise, Caption Health customers.
- **Every scientific paper continuously synthesized + linked.** Elicit / Consensus / Scite currently sample (or batch-process at premium tiers). At our pricing, every PubMed / arXiv / SSRN paper auto-summarized + linked + cross-referenced as it's published. Plus retroactive-against-archive: 50M+ papers.
- **Drug discovery generative loops at scale.** Every candidate molecule analyzed; the screening pipeline today gates by inference cost. Insilico Medicine, Recursion Pharma, Schrödinger, Insitro, Isomorphic Labs (Alphabet) could 100× their generative throughput.
- **Materials science design loops.** Every candidate material analyzed before synthesis. Materials Project (Berkeley), Microsoft Quantum / Materials, Citrine Informatics. Battery-materials, semiconductor-materials, catalyst design pipelines.
- **Legal e-discovery at full corpus scale.** Every document in a litigation production analyzed by LLM, not just keyword-flagged ones. Relativity, Reveal, Everlaw customers.
- **Insurance underwriting at full claims-history scale.** Every prior claim, every medical record, every police report analyzed for new policy quotation. Allianz, AXA, Liberty, Travelers, Zurich.

**Named first customers.** Aidoc, Rad AI, Annalise, Tempus / Foundation Medicine (oncology imaging + genomics), Insilico Medicine ($1B+ valuation), Recursion Pharma (Roche partner), Schrödinger ($3B market cap), Insitro, Isomorphic Labs, Microsoft Materials, Citrine, Relativity (~$1B revenue), Reveal, Everlaw, Allianz / AXA / Travelers / Liberty Mutual, Pfizer / Merck / Novartis (pharma R&D).

**TAM intuition.** Healthcare AI inference forecast at $5–10B by 2028; legal AI ~$3–5B; scientific R&D AI ~$2–4B. *All currently bottlenecked at the indexing/inference cost layer*, not at model-quality or distribution. **Combined latent TAM: $15–25B by 2030 if cost ceiling lifts.**

**Gating factors.** Cost is binding (the canonical "we'd run AI over our archive but the inference bill exceeds our budget" complaint). Sovereignty for healthcare/legal/insurance (HIPAA, attorney-client privilege, customer data — favors on-prem or sovereign DC). Quality is generally acceptable on OS-100B class for these workloads.

---

### 2.8 Education at universal-tutoring scale

**The constraint today.** Khanmigo / Duolingo Max / MagicSchool / Khan Academy AI / Quizlet rate-limit free tiers and price institutional licensing low ($5–$20/student/year) because per-student inference cost over a school year is meaningful at frontier prices.

**What unlocks:**

- **Continuous AI tutoring for every K-12 student in every public school system.** Today districts pay $5–$20/student/year for AI-augmented tools; with our cost basis, $50/student/year buys *unlimited* AI tutoring for a full school year (10K+ student-tutor turns).
- **AI-native universities.** Arizona State (560K students, already heavy AI investment), SNHU, Western Governors University (already ~150K students using AI tutoring extensively), Coursera ($600M revenue), Pluralsight, Guild Education extend AI into every interaction.
- **Workforce reskilling at population scale.** Coursera, Pluralsight, Guild, LinkedIn Learning, Udemy running personalized AI tutoring for tens of millions of working learners.
- **Language learning at conversational scale.** Duolingo Max currently rate-limits AI conversation features; at our pricing, every Duolingo session is a full conversation with an AI tutor, every learner.
- **Special-needs AI accompaniment.** Speech therapy, autism social-skills training, ADHD coaching, dyslexia reading-assistance — all economically viable at population scale.

**Named first customers.** Khan Academy / Khanmigo (Google partnership), Duolingo (50M+ DAU), MagicSchool (8M+ teachers), Quizlet, Coursera, Pluralsight, Guild Education, Chegg, Course Hero, Cengage / McGraw-Hill / Pearson (textbook publishers' AI extensions), Class Technologies, Replit (CS education), Carnegie Learning, Newsela, Outschool. Plus: state-level school district contracts (NYC DOE, LAUSD, Chicago Public Schools, Texas TEA).

**TAM intuition.** Global K-12 ed-tech is ~$100B; AI-tutoring slice was <$1B in 2025. At unlocked prices, plausibly **$10–20B by 2030**, with the largest single buyer being state/national school systems doing population-scale rollouts.

**Gating factors.** Cost is binding (the frontier-API-priced version has been priced out of mass-market education). Quality on OS-100B is sufficient for K-12 tutoring. Sovereignty matters in some jurisdictions (student data, COPPA, FERPA). Integration with LMS (Canvas, Schoology, Google Classroom) is the catalysis lag.

---

## 3. Two Second-Order Patterns

### 3.1 Demand that emerges only when *multiple* constraints lift simultaneously

The most valuable workloads sit at the intersection of cost + latency + energy:

- **Always-on personal AI** (§2.3) needs cheap **and** low-latency **and** energy-efficient. A chip that's cheap but power-hungry doesn't fit a wearable; a chip that's energy-efficient but slow can't drive conversation.
- **Per-frame video AI** (§2.1) needs cheap **and** low-TTFT **and** high-throughput. SAM-H at $0.001/Mimage but with 1-second TTFT misses the live-stream use case.
- **Sovereign tier-2 AI** (§2.6) needs cheap **and** energy-efficient (small power grids) **and** small-form-factor (DC-buildout cost).

Single suppliers that address one constraint (Groq does TTFT, NVIDIA does FLOPS, AMD does cost-per-FLOP) leave the multi-constrained workloads uneconomical. **The strategic position of the proprietary chip is exactly at the intersection — and it's the multi-constrained workloads that are the most valuable expansion targets.**

### 3.2 Distribution-side latency

Even with cheap, fast, energy-efficient inference available, some workloads expand only when *integration* catches up. Categories:

- **OS-level integration** — Apple Intelligence, Google AI on Android, Microsoft Copilot at OS level. Took 2 years from chip availability to OS shipment. Always-on personal AI (§2.3) and universal autocomplete depend on OS-level integration.
- **Vehicle integration** — automotive infotainment cycles are 4–6 years. In-cabin AI deployment (§2.3) requires OEM commits 2–3 years before consumer availability.
- **Game-engine integration** — Unreal Engine, Unity, proprietary engines. NVIDIA's ACE (Audio2Face + Riva ASR + Audio2Gesture) launched 2023; meaningful game shipping with it now in 2025–2026. AI gaming (§2.2) is ~24 month catalysis from cost-availability.
- **Medical-device integration** — FDA cycles for clinical-grade AI integration: 12–24 months per indication. Healthcare archive analysis (§2.7) has the longest catalysis.
- **Education LMS integration** — Canvas / Schoology / Blackboard / Google Classroom integration timelines: 6–18 months per platform.

> **Compressed:** *cost-curve unlocks the workload on paper. Distribution unlocks the workload in practice. The two timelines are independent, and the second is usually the binding one.*

---

## 4. TAM Aggregation and Sequencing

### 4.1 Aggregated latent demand

Sum of the eight categories, by **2030 TAM** if all unlocks land:

| Category | 2030 TAM intuition |
|---|---|
| §2.1 Per-frame / per-event inference | $5–10B |
| §2.2 AI-native gaming | $5–20B |
| §2.3 Always-on personal AI / wearables | $20–50B |
| §2.4 Per-customer personalization | $5–10B |
| §2.5 Continuous eval / fine-tune / indexing | $2–4B |
| §2.6 Sovereign AI for tier-2/3 nations | $50–100B |
| §2.7 Industrial / scientific archive-scale | $15–25B |
| §2.8 Education at universal-tutoring scale | $10–20B |
| **Total latent TAM by 2030** | **$110–240B** |

For comparison, the *existing* enterprise + consumer LLM inference market in 2026 is ~$50–70B (per `datacenter-inference-business-analysis.md` §1). The latent demand is **2–4× the size of the entire current market** — and is what the cost-curve thesis (`tokens-too-cheap-to-meter-thesis.md`) is asserting will get unlocked.

### 4.2 Sequencing — when does each unlock land?

| Category | First commercial deployment | Mass-market | Why the timing |
|---|---|---|---|
| §2.5 Continuous eval / fine-tune | 2026 | 2027–28 | Software-only; fastest catalysis. Drops in as soon as cheap inference exists. |
| §2.4 Per-customer personalization | 2026 | 2027–28 | API-level integration, marketing-tech ready. |
| §2.7 Industrial archive-scale | 2026 (legal e-discovery) | 2028–30 | Healthcare requires FDA / HIPAA cycles; legal first. |
| §2.1 Per-frame inference | 2027 | 2028–29 | Pipeline integration into video/security stacks. |
| §2.6 Sovereign AI | 2026 (signed contracts) | 2027–30 | Government procurement timelines + DC build-out. |
| §2.8 Education universal | 2027 | 2029–30 | LMS integration + state-level procurement. |
| §2.2 AI-native gaming | 2027 (NPCs in tentpole title) | 2028–30 | Game-engine integration + dev cycle (~3 years per AAA title). |
| §2.3 Always-on personal AI | 2027 (Apple Intelligence Watch?) | 2029–30 | OS-level integration + device ship cycle. |

**For the proprietary-chip business plan:**
- Year 1 (2026–27): wedge §2.5, §2.4, §2.7-legal — software-integration-fast, no hardware-distribution lag.
- Year 2 (2027–28): wedge §2.1, §2.6 (early sovereign deals), §2.7-healthcare. Begin §2.2 game pilots.
- Year 3+ (2028–30): mass-market §2.3, §2.8, §2.6 (scaled sovereign), §2.2.

The Year-1 unlocks are roughly **$8–18B aggregate TAM**; Year-3+ unlocks add the larger $100B+. Crucially, the sequencing is governed by *integration timelines, not chip availability*.

---

## 5. What the Chip Needs to Prove for Each Unlock

| Category | Required chip property | Validation milestone |
|---|---|---|
| §2.1 Per-frame | SAM-H + decoder both at low cost; latency for live streams | Per-frame video moderation pilot at Twitch / TikTok scale |
| §2.2 Gaming | Sub-300ms TTFT for NPC voice; energy efficiency for portable gaming | NVIDIA-ACE-equivalent stack on the chip; tentpole-game pilot |
| §2.3 Wearables | **Energy efficiency** is binding; <1W per inference (much smaller than DC chip) | A "wearable variant" of the chip — different SKU, same architecture |
| §2.4 Personalization | Cost only; throughput at marketing-tech scale | Klaviyo / Braze partnership for per-user generation |
| §2.5 Continuous eval | Cost only | Patronus / Braintrust integration |
| §2.6 Sovereign | Energy efficiency + deployable density; software stack mature | First named sovereign DC deployment (UAE / Saudi / Singapore) |
| §2.7 Archive-scale | Cost only; batch throughput | Healthcare imaging archive pilot (10M-study scale) at one hospital system |
| §2.8 Education | Cost + sovereignty (student data) | State-level pilot (one US state DOE) |

**The wearables variant (§2.3) is the most product-roadmap-significant gating factor.** A DC chip optimized for 100W+ TDP doesn't fit a 1W wearable budget. If wearable AI is part of the long-term thesis, a "lite" SKU at <5W TDP needs to be on the roadmap. This is a separate tape-out and 18–24 month additional cycle.

---

## 6. Risks and Counterfactuals

### 6.1 What kills the demand-expansion thesis?

- **Frontier-tier price collapse independent of our chip.** If OpenAI / Anthropic / Google compete each other to $0.001/Mtok via amortizing training cost into perpetual marginal discounts, our cost advantage compresses. Several latent demands (§2.4, §2.5, §2.8) get unlocked anyway — but by frontier labs, not us. Our share of the unlocked TAM shrinks.
- **Quality ceiling on OS-100B.** If continuous-frontier-only workloads expand faster than OS-tier-able ones (i.e., demand goes upmarket faster than it goes downmarket), our addressable share of the unlocked TAM shrinks. Hard to predict; depends on whether the *shape* of demand is bimodal or evenly distributed.
- **Distribution lag stretches longer than expected.** If OS / vehicle / game-engine integration takes 5 years instead of 2–3, we enter a "valley of death" between cheap inference availability and consumed demand. Capital structure needs to survive 5+ years.
- **Hyperscaler vertical integration captures the unlocks.** If AWS / Google / Azure ship custom silicon at our perf-per-watt point and bundle it into their existing cloud relationships before we land standalone DC customers, the latent unlocks land in their cloud, not ours.

### 6.2 What does the "no-unlock" scenario look like?

Even if *none* of these unlocks happen at the scale described, the **current** market (per `datacenter-inference-business-analysis.md`) is $50–70B and growing 50%+ YoY. The chip's COGS advantage delivers 95%+ gross margins capturing existing demand, before any expand-the-pie thesis lands. **The latent-demand thesis is upside, not the base case.**

The base case ROI works on existing demand. The expand-the-pie unlocks make it 5–10× larger.

---

## 7. Strategic Implications

### 7.1 Pricing strategy at the unlock thresholds

For each latent-demand category, there is a **price threshold** below which the workload becomes economically viable. Identifying these thresholds and pricing slightly below each is the most lucrative strategic posture:

| Category | Estimated unlock price threshold | Today's market price | Our N3 COGS |
|---|---|---|---|
| Per-frame video AI | <$0.01/Mimage | $1,500/Mimages | $0.90/Mimages |
| Per-event fraud reasoning | <$0.001/inference (~$0.001/Mtok) | $0.05+/inference | $0.000468/Mtok |
| Always-on wearable AI | <$0.10/user-day | $5–20/user-day | <$0.01/user-day at typical usage |
| Per-customer marketing copy | <$0.001/recipient | $0.05/recipient (frontier) | $0.0001/recipient |
| Continuous eval | <$0.10/Mtok | $0.88/Mtok (Llama-70B) | $0.000468/Mtok |
| Sovereign AI | TCO + data residency parity | hyperscaler-bundled | sovereign DC at 95%+ margin |
| Archive-scale healthcare | <$0.01/study | $1.50/study (current) | $0.0009/study |
| K-12 universal tutoring | <$50/student-year | $200+/student-year (frontier) | <$5/student-year |

**Strategic posture:** announce pricing at each threshold *just before* the workload would otherwise be uneconomical. The chip's COGS is so low that the announced price is still 90%+ gross margin. Each price-threshold announcement triggers the unlock; the announcement is the marketing.

### 7.2 The category-creator move

Most chip companies sell into existing markets. The latent-demand opportunity invites a different posture: **don't just sell tokens; visibly fund and orchestrate the unlocks that make the new categories economical.**

Concretely:
- Co-fund pilots at named first customers (Inworld for gaming, Limitless for wearables, Klaviyo for personalization, Aidoc for healthcare, Khanmigo for education) — provide free / heavily subsidized inference for 6–12 months in exchange for case-study + reference-customer status.
- Sponsor open-source frameworks that catalyze each category (Inworld plugins, eval-as-a-service tooling, embedding-pipeline templates, gaming SDKs).
- Publish prominent benchmarks at price thresholds — make the unlock visible to the broader ecosystem.

This is a different go-to-market motion than "sell tokens to existing buyers." It's slower per-deal but compounds: each unlocked category expands the addressable inference market by tens of billions and creates a customer base that didn't exist when sales started.

---

## 8. Open Questions / Next Steps

- **TAM derivations need stress-testing.** Each category's $X–$YB intuition is roughly anchored to a comparable existing market × an assumed expansion multiple. A bottom-up customer-by-customer derivation (e.g., "Twitch has X moderators-of-streams, Y videos/year, Z frames per video, AI-token-cost per frame") for each category would tighten the numbers materially.
- **Wearable-variant chip SKU.** §2.3 (always-on personal AI) requires a dedicated low-TDP variant. Is this on the product roadmap? Tape-out timing and incremental NRE budget?
- **Sovereign DC strategy specifics.** Which 3–5 sovereign customers do we target first (UAE / Saudi / Singapore / India / Brazil)? Each has different procurement dynamics; sequencing matters.
- **Distribution-side partnership map.** For categories with integration lag (gaming, automotive, OS, LMS, medical devices), which 1–2 platform partners do we sign deep partnerships with to accelerate? The platform partners are the gating factor more than the chip.
- **Pricing-threshold validation.** The price-threshold table in §7.1 is intuitive; each threshold should be validated by actual customer-economic-model interviews. (e.g., Klaviyo would actually deploy per-recipient AI at $0.001 but not $0.01? Need to test.)
- **Competitive response timing.** When do hyperscalers / frontier labs respond to each unlock? At what price and timeline?
- **Risk mitigation: distribution lag.** Capital structure to survive 4–5 years of integration catalysis. Bridge financing or anchor customer commitments to derisk.

---

## Sources

**Internal companion documents:**
- `tokens-too-cheap-to-meter-thesis.md` — cost-curve framing, 10× rule, historical precedents
- `datacenter-inference-business-analysis.md` — current-market business case, top-5 wedges, GTM channels, defensibility, risks
- `batch-inference-business-analysis.md`, `agentic-inference-business-analysis.md` — adjacent business archetypes
- `../llm-inference-workloads/llm-inference-workloads-taxonomy.md` — 32-workload current-demand taxonomy
- `../llm-inference-workloads/accelerator-workload-fit-analysis.md` — per-workload accelerator multipliers across N16/N7/N5/N3

**External / market data references:**
- NVIDIA Sovereign AI revenue forecasts (~$5–10B FY26, growing)
- Anthropic Economic Index (Sept 2025 / Jan 2026 / Mar 2026) — current LLM workload distribution
- Menlo Ventures 2025 State of Generative AI in the Enterprise — $37B aggregate enterprise AI spend
- a16z Top 100 Gen AI Consumer Apps (6th ed., Mar 2026)
- LangChain State of Agent Engineering 2025 — agent deployment rates
- Healthcare AI / legal AI / education-tech 2030 forecasts (Grand View Research, Markets and Markets, public-equity-analyst comps)
- Hyperscaler sovereign-AI announcements (UAE G42, Saudi HUMAIN, India IndiaAI Mission, Singapore IMDA, France Mistral state-aligned investment, German Aleph Alpha)

---

*Document compiled 2026-05-02.*
