# LLM Inference Workloads — A Taxonomy Across Nine Operating-Point Axes

*A descriptive catalogue of publicly-documented LLM inference workloads, labeled across nine axes (reasoning depth, domain specialization, concurrency profile, prefill:decode ratio, TTFT sensitivity, agent end-to-end latency sensitivity, total tokens per session, tool-use intensity, price sensitivity) that drive hardware, latency, pricing, and locality decisions for any inference business.*

> Companion to the inference-provider archetype docs (`batch-inference-business-analysis.md`, `agentic-inference-business-analysis.md`, `tokens-too-cheap-to-meter-thesis.md`). Where those documents pick a *business* — batch, agentic, or interactive — this one labels the *workloads* those businesses serve. It is a reference layer, deliberately descriptive: no recommendations, no thesis-mapping, no positioning calls. Use it to ground claims in the deck and downstream documents.

---

## 1. The Taxonomy Problem

**Q: Why doesn't a unified public taxonomy of LLM workloads exist?**

Because four large, well-funded measurement programs each slice usage by a *different* primitive — and none of those primitives is "what is happening on the inference server."

| Source | Primitive | What it captures |
|---|---|---|
| Anthropic Economic Index ([Sept 2025](https://www.anthropic.com/research/anthropic-economic-index-september-2025-report), [Jan 2026](https://www.anthropic.com/research/anthropic-economic-index-january-2026-report), [Mar 2026](https://www.anthropic.com/research/economic-index-march-2026-report)) | **O*NET occupational task** | Top-down task code → conversation. "Modify software to correct errors" = 6% of Claude.ai, 10% of API. |
| OpenAI / NBER WP 34255 ([Chatterji et al., Sept 2025](https://www.nber.org/papers/w34255)) | **Conversational intent (Ask/Do/Express)** | Why the user opened the conversation. Asking 49% / Doing 40% / Expressing 11%. |
| a16z Top 100 Consumer AI Apps ([6th ed., Mar 2026](https://a16z.com/100-gen-ai-apps-6/)) | **Consumer product/app category** | What product the user installs. ChatGPT, Sora, Character.AI, OpenClaw, Suno. |
| Menlo Ventures *State of Generative AI in the Enterprise* ([Dec 2025](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/)) | **Enterprise department / line item** | Where the budget is booked. Coding 55% of departmental AI spend, IT 10%, Marketing 9%, CS 9%, Design 7%, HR 5%. |

These lenses don't union cleanly. "Coding" looks like 55% of the world (Menlo, enterprise spend), 35% of the world (Anthropic, Claude.ai conversations), and 4.2% of the world (OpenAI consumer NBER paper) — all simultaneously true. Each is measuring a different pie.

The [Stanford HAI AI Index 2026](https://hai.stanford.edu/ai-index/2026-ai-index-report) Economy chapter and the [REALM dataset](https://aclanthology.org/2025.findings-acl.437/) (94k real-world LLM use cases mapped to occupations, ACL Findings 2025) add two more lenses (national-level adoption and a 15-category use-case taxonomy: Content Creation, Content Synthesis, Decision Making, Information Retrieval, Process Automation, etc.). [LMSYS Chatbot Arena](https://www.lmsys.org/blog/2024-05-17-category-hard/) adds an eighth lens organized around evaluation skill (Writing, Roleplay, Extraction, Reasoning, Math, Coding, Knowledge I/II, Hard Prompts, Long Context).

None of these are wrong. They are answering different questions.

**Q: What lens is this document using?**

A workload here is a **(task × deployment-mode) pair distinguishable by its inference operating point.** "Coding" is not a workload because IDE inline completion (sub-200ms TTFT, prompt 1,500 tokens / output 13 tokens — [Splitwise](https://arxiv.org/abs/2311.18677), ISCA '24) and a long-running code agent (100K–285K tokens per session, 17 tool calls per task — [MCPMark](https://arxiv.org/abs/2509.24002), [Anthropic engineering on advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use)) share neither prompt shape, latency budget, hardware preference, nor pricing tier. They are different workloads even though Menlo books them on the same line.

The result: a workload list of ~30 entries — finer than any single source above, coarser than the long tail of the LMSYS-Chat-1M cluster trees. The cuts follow inference operating point, not occupation, intent, or product category. Where two workloads share the same operating-point fingerprint we collapse them; where one occupation cell contains two operating points we split them.

> **The thesis of this document:** *the right axis to taxonomize LLM workloads — for anyone building hardware, an inference business, or pricing — is operating point, not occupation. Public reports give us occupation; this document translates.*

---

## 2. The Nine Axes

The nine axes are reproduced verbatim from the framework supplied by the user. Each axis has a small set of allowed values and a primary downstream decision it informs.

| # | Axis | Values | Primary decision it drives |
|---|---|---|---|
| 1 | **Reasoning depth** | Shallow / Medium / Deep | Model tier: open-source 8–30B / open-source 70–120B / frontier API |
| 2 | **Domain specialization** | General / Specialized | Whether fine-tunes or vertical models matter |
| 3 | **Concurrency profile** | Single-stream / Small-batch / Massively parallel | Desktop GPU vs. on-prem cluster vs. cloud |
| 4 | **Prefill : decode ratio** | Prefill-heavy / Balanced / Decode-heavy | Hardware spec (compute-bound vs. bandwidth-bound) |
| 5 | **TTFT sensitivity** | Low / High | Streaming architecture, edge placement |
| 6 | **Agent end-to-end latency sensitivity** | Low / High | Co-location of model + tools, routing topology |
| 7 | **Total tokens per session** | Short / Medium / Long | KV cache sizing, memory tier |
| 8 | **Tool-use intensity** | Pure generation / Light / Heavy | Routing complexity, observability needs |
| 9 | **Price sensitivity** | Low / Medium / High | Pricing tier and packaging |

A few definitions and conventions used throughout:

- **Reasoning depth.** Shallow ≈ pattern-matching, classification, format conversion (8–30B sufficient). Medium ≈ multi-step composition, summarization, basic agents (70–120B sufficient). Deep ≈ frontier-required reasoning (current frontier API: GPT-5, Claude Opus 4.7, Gemini 2.5 Pro / 3 Pro).
- **Concurrency profile.** Single-stream = one task at a time on one accelerator (desktop GPU, dedicated tenancy). Small-batch = tens to hundreds of concurrent streams (departmental cluster). Massively parallel = thousands+ (hyperscaler / inference-provider scale).
- **Prefill : decode.** Anchored on real production traces. Prefill-heavy = input compute > output compute (e.g., [Splitwise](https://arxiv.org/abs/2311.18677) coding trace, 1,500:13 input:output). Balanced = within ~3× of 1:1 (e.g., [BurstGPT](https://arxiv.org/abs/2401.17644) ChatGPT trace, 575:340). Decode-heavy = output dominates (e.g., [InferenceMAX](https://newsletter.semianalysis.com/p/inferencemax-open-source-inference) reasoning config, 1,024:8,192).
- **TTFT sensitivity.** Low = batch / off-line (tens of seconds acceptable). High = interactive / streaming (sub-second). [DistServe](https://arxiv.org/abs/2401.09670) (OSDI '24) anchors specific SLOs: code completion 0.125s, chatbot 2.5s, summarization 15s on OPT-66B.
- **Agent E2E latency.** Distinct from TTFT. A code agent with high TTFT (fast first token) can still have low E2E sensitivity if the user is willing to wait minutes for the trajectory to finish. Voice agents have *both* high TTFT and high E2E sensitivity (sub-300ms TTFT, sub-800ms turn — [Hamming AI voice latency analysis](https://hamming.ai/resources/voice-ai-latency-whats-fast-whats-slow-how-to-fix-it)).
- **Total tokens per session.** Short = <2K, Medium = 2K–32K, Long = 32K+. Anchors: BurstGPT chat ~915 total (Short); LMSYS multi-turn 20–40K (Medium); Claude Code 100K–285K (Long).
- **Tool-use intensity.** Pure generation = no tools. Light = 1–5 tool calls (RAG, single search). Heavy = >5 (MCPMark canonical "long-horizon tool use" workload averages **17.4 tool calls per task** across 16.2 turns).
- **Price sensitivity.** "How much does the buyer care about per-token price?" High = consumer chat (margins crushed by free tiers). Medium = enterprise SaaS workflows. Low = vertical AI where the comparison is to a $40/hr human or a $400/hr lawyer (healthcare scribe at $600M, +2.4× YoY per Menlo; legal at ~$650M).

**Caveats on reading the table.**

- **The axes are not independent.** Deep reasoning + heavy tool use + long sessions cluster ("agent profile"). Massively parallel + balanced P:D + high TTFT + high price sensitivity cluster ("interactive chat profile"). The master table in §4 should not be read as 9 independent dimensions but as ~3 latent profiles plus boundary cases. Section 5 makes this explicit.
- **Concurrency profile is a deployment choice, not a workload property in isolation.** The same workload (e.g., document understanding) can be served single-stream (one PDF on a desktop) or massively parallel (a thousand PDFs through batch). Where ambiguous, the label reflects the dominant production mode.
- **Many cells are estimates.** Where a public number anchors the label we cite it. Where no public number exists we mark it "estimated" — most often on per-session token counts for niche enterprise workloads.

---

## 3. The Workload Catalogue

Thirty-two workloads, grouped into seven families for readability only. The master table in §4 is flat. Each entry has: a one-paragraph description with citations to the public sources where the workload appears, then a justification for non-obvious axis labels with the technical citation.

### 3.1 Text-Production Workloads (General Chat, Writing, Translation)

#### Workload 1 — Short conversational Q&A

The dominant ChatGPT/Claude.ai/Gemini operating point. Single-turn or short multi-turn questions: factual lookups, advice, brainstorming, "what does this mean," "how do I do X." [OpenAI/NBER WP 34255](https://www.nber.org/papers/w34255) attributes 49% of ChatGPT consumer messages to "Asking" intent, with "Practical Guidance" and "Seeking Information" together accounting for ~53% of all conversations by June 2025. [Anthropic Economic Index Mar 2026](https://www.anthropic.com/research/economic-index-march-2026-report) reports the top-10 O*NET tasks now cover 19% of Claude.ai conversations (down from 24%) — a *diversification*, but the bulk is still short Q&A.

*Operating point:* [BurstGPT](https://arxiv.org/abs/2401.17644) (10.31M Azure OpenAI traces) — average 575 input / 340 output tokens. [InferenceMAX](https://newsletter.semianalysis.com/p/inferencemax-open-source-inference) standardized "Chat" config — 1,024 input / 1,024 output. Massively parallel (ChatGPT: 900M weekly actives — [a16z](https://a16z.com/100-gen-ai-apps-6/)). High price sensitivity (free-tier baseline).

#### Workload 2 — Long-form writing / drafting

Composition of multi-paragraph or document-length output: blog posts, emails, marketing copy, fiction, reports. NBER attributes 24% of June 2025 ChatGPT messages to "Writing" — 40% of *work-related* messages. [Anthropic Economic Index Jan 2026](https://www.anthropic.com/research/anthropic-economic-index-january-2026-report) shows Arts/Design/Entertainment at 11% of Claude.ai. [a16z State of Consumer AI 2025](https://a16z.com/state-of-consumer-ai-2025-product-hits-misses-and-whats-next/) gives prosumer writing tools 26% of generative content traffic.

*Operating point:* output tokens dominate (decode-heavy). Streaming UX is non-negotiable — TTFT high. Sessions medium length (one-shot drafts) but creative-writing chat sessions can run long.

#### Workload 3 — Editing, paraphrasing, critique

User pastes existing text and asks for improvements. NBER explicitly notes "two-thirds of Writing is *editing/summarizing/translating* user-provided text rather than de novo generation." This is the volume share of "Writing" in ChatGPT.

*Operating point:* balanced or slightly prefill-heavy (input ≈ output). Reasoning is shallow-to-medium (most edits are local). General-purpose 8–30B models suffice; 70B used for stylistic polish.

#### Workload 4 — Summarization, single-document

User pastes a document, asks for a summary or key points. [Vidur](https://arxiv.org/abs/2405.05465) (Microsoft) Arxiv-Summarization trace is the canonical reference. [InferenceMAX](https://newsletter.semianalysis.com/p/inferencemax-open-source-inference) "Summarization" config: 8,192 input / 1,024 output (8:1, prefill-heavy).

*Operating point:* TTFT can be relaxed (15s acceptable per [DistServe](https://arxiv.org/abs/2401.09670) summarization SLO). Hardware compute-bound on input ingestion. [LMCache](https://arxiv.org/abs/2510.09665) reports 1.53–1.84× mean TTFT reduction with PD disaggregation on 8K-input/200-output document analysis.

#### Workload 5 — Summarization / synthesis, multi-document

Synthesis across 5–100+ documents: research briefings, "summarize this Slack channel," "what changed in this corpus." Vidur BWB-4K (Bilingual Web Book) trace shows 10× longer decode and 2× longer prefill than Chat-1M, P:D ratio 0.65 (decode-dominated), 4K-token contexts. [REALM](https://aclanthology.org/2025.findings-acl.437/) classifies this as "Content Synthesis" — one of its two most prevalent use cases.

*Operating point:* long context, hybrid prefill/decode shape. Tokens-per-session long. Often run as a batch job with low TTFT sensitivity.

#### Workload 6 — Translation

Cross-lingual rewriting. NBER includes translation under Writing (sub-share of 24% June 2025). Specialized for legal/medical/literary translation; otherwise general.

*Operating point:* balanced P:D (input ≈ output). Streaming UX matters in conversational settings; batch for document translation. High volume, massively parallel deployment, high price sensitivity (commodity).

#### Workload 7 — Practical guidance / how-to / tutoring

Step-by-step instructions, life advice, problem-solving help. NBER's largest steady category — "Practical Guidance" ≈ 29% of all ChatGPT messages, constant across the year. [Anthropic Mar 2026](https://www.anthropic.com/research/economic-index-march-2026-report) shows Educational Instruction at 12% of Claude.ai (down from 19% as coursework migrates).

*Operating point:* short-to-medium sessions, balanced P:D, massively parallel, high TTFT sensitivity (interactive UX).

#### Workload 8 — Companions, roleplay, emotional support

Character.AI, Replika, ChatGPT "Express" intent (11% of NBER messages). LMSYS Chatbot Arena category includes Roleplay. [a16z 6th edition](https://a16z.com/100-gen-ai-apps-6/) lists multiple companion apps in the consumer Top 100.

*Operating point:* sessions are often very long (Character.AI per-user retention is built on multi-hour cumulative sessions). Decode-heavy (long persona-driven generation). Persona context cached aggressively. General models, sometimes with character LoRAs (specialization-light). Massively parallel; price-sensitive (free-to-cheap consumer tier).

### 3.2 Code Workloads

The single largest department-of-spend per [Menlo](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/) ($4.0B of $7.3B departmental AI in 2025), the largest occupational cluster on Claude.ai (35–36% — [Anthropic Mar 2026](https://www.anthropic.com/research/economic-index-march-2026-report)) and the largest 1P-API category (44–46%). Splits into six distinct operating points.

#### Workload 9 — IDE inline completion (FIM / autocomplete)

Cursor Tab, GitHub Copilot, JetBrains AI Assistant. The user types; the model fills in 1–10 tokens. [Splitwise](https://arxiv.org/abs/2311.18677) "coding" trace: median prompt 1,500 tokens, median output 13 tokens — extreme prefill skew (>100:1). DistServe assigns code completion the strictest SLO of any workload tested: TTFT 0.125s on OPT-66B, TPOT 0.2s.

*Operating point:* the canonical *prefill-bound, TTFT-bound* workload. Hardware preference is high prefill compute and tight latency tail. Small models often fine — Cursor and GitHub Copilot have used dedicated specialty models below 7B parameters. Massively parallel (every keystroke). High price sensitivity (per-seat ARR economics).

#### Workload 10 — In-IDE chat / inline question

User highlights code and asks "what does this do" or "refactor for clarity." Cursor Chat, Continue, Copilot Chat. Operates over a window of repo context (typical files 3K–5K tokens — [Cursor dynamic context](https://www.infoq.com/news/2026/01/cursor-dynamic-context-discovery/)).

*Operating point:* prefill-heavy (repo context) but with streaming-output decode. Tokens per session medium. Tool-use light unless agent-mode invoked. Massively parallel. Price sensitivity medium-high.

#### Workload 11 — Code review / PR comment generation

Greptile, Diamond, GitHub Copilot review, CodeRabbit, Anthropic Claude in CI. Asynchronous, multi-file context, structured output (review comments).

*Operating point:* prefill-heavy (full diff + linked context), low TTFT sensitivity (background), small-batch concurrency, light-to-heavy tool use (linters, search). Price sensitivity medium.

#### Workload 12 — Long-horizon code agent

Claude Code, Cursor Agent, OpenAI Codex (CLI agent), Devin/Cognition, Replit Agent. Multi-turn autonomous code editing across a repo. [SemiAnalysis Coding Assistant Breakdown](https://newsletter.semianalysis.com/p/the-coding-assistant-breakdown-more) reports Claude Code average input:output ratio ≈ **100:1**; Codex ≈ 80:1. [Anthropic on advanced tool use](https://www.anthropic.com/engineering/advanced-tool-use): tool-definitions alone can occupy 134K tokens; 5-server MCP setup with 58 tools = 55K tokens of overhead per call. [Cursor dynamic context](https://www.infoq.com/news/2026/01/cursor-dynamic-context-discovery/): typical agent run consumes 50K+ tokens just exploring the codebase.

*Operating point:* the canonical "agent profile." Single-stream concurrency (one task at a time per user). Strongly prefill-heavy (every turn re-ingests context). Tokens-per-session long (100K–285K — [Verdent on Claude Code pricing](https://www.verdent.ai/guides/claude-code-pricing-2026)). Heavy tool use (16.2 turns / 17.4 tool calls per task — [MCPMark](https://arxiv.org/abs/2509.24002)). Agent E2E latency is high-sensitivity within-task (faster tokens → faster turns) but tolerant per-task (users wait 1–10 min). Reasoning *currently* deep-frontier (Claude Opus, GPT-5) but rapidly migrating to open-weight Qwen3-Coder / DeepSeek-Coder — see Anthropic Mar 2026 on the +14pp Computer/Math API share shift driven by Claude Code.

#### Workload 13 — App builder / vibe coding

Lovable, v0, Bolt, Replit Agent in app-builder mode, Manus, OpenClaw. Generates full apps from natural-language briefs. [a16z Mar 2026 Top 100](https://a16z.com/100-gen-ai-apps-6/): OpenClaw 68k GitHub stars in weeks; Manus acquired by Meta Dec 2025 (~$2B); Bolt token quotas 150K–300K/day per user.

*Operating point:* like a code agent but with longer per-task token budgets (whole-app generation), fewer interactive turns, and more tolerance of long E2E latency (user waits for an "app to be built"). Tool use heavy (file system, package management, test running, deploy).

#### Workload 14 — Code search / repo Q&A

"Where is `parseUserConfig` called from?" / "Summarize the auth flow." Sourcegraph Cody, Cursor Ask, Glean for engineering. Distinct from in-IDE chat in that the deployment scales horizontally over corporate codebases and is often background-indexed.

*Operating point:* prefill-heavy (RAG over indexed code), light tool use (vector search), small-batch concurrency, medium TTFT sensitivity. Tokens per session medium. Price sensitivity medium-low (enterprise budget).

### 3.3 Search and Retrieval-Grounded Workloads

#### Workload 15 — Enterprise search & grounded chat

Glean, Microsoft Copilot M365, Notion AI, ChatGPT Enterprise with corporate connectors. [a16z Enterprise CIO 2025](https://a16z.com/ai-enterprise-2025/): "internal enterprise search" is the second-most named use case after software development; >90% of CIOs surveyed are testing third-party customer support apps (a related grounded-chat workload). [OpenAI State of Enterprise AI 2025](https://openai.com/index/the-state-of-enterprise-ai-2025-report/) puts research alongside writing/programming/analysis as a top-four work task.

*Operating point:* prefill-heavy (RAG context), small-batch (per-tenant) concurrency, high TTFT (interactive). Domain-specialized to each corpus. Price sensitivity medium (enterprise pays for the connector value, not the tokens).

#### Workload 16 — Web-grounded research / "deep research"

OpenAI Deep Research, Gemini Deep Research, Perplexity Pro Search, Anthropic Claude with web tools. [GAIA](https://arxiv.org/pdf/2311.12983) is the canonical academic proxy: Level 3 tasks take "up to 50 steps for humans, any number of tools." Real deployed deep-research products run 5–30 minutes per task and consume hundreds of thousands of tokens.

*Operating point:* mirrors the code-agent profile but with web tools instead of code tools. Long sessions, deep reasoning (frontier required for synthesis), heavy tools, low TTFT but high agent E2E sensitivity (faster = better but minutes are tolerated).

#### Workload 17 — Open-domain citation chatbot / Q&A

Perplexity standard search, Phind, You.com, the "Search GPT" mode in ChatGPT. Distinct from §16 in that the answer is one-shot (no agent loop) and the tool call is a single search.

*Operating point:* prefill-heavy (web snippets), high TTFT (interactive UX), massively parallel, tokens-per-session short-to-medium. Light tool use.

### 3.4 Data-Plane / Extraction Workloads (Batch-Adjacent)

The set of workloads characterized in `batch-inference-business-analysis.md`. They share an operating point: prefill-heavy, latency-tolerant, massively parallel, high-throughput.

#### Workload 18 — Document understanding (IDP, PDF→JSON)

Reducto, Unstructured.io, AWS Textract+LLM, Google Document AI, LandingAI Agentic Document Extraction. Pull invoices, contracts, claims, statements into structured fields. [Menlo Ventures Vertical AI](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/): healthcare ambient scribes alone $600M (+2.4× YoY) — much of that is document-understanding-adjacent.

*Operating point:* prefill-heavy (5K–50K input → 1K–5K JSON output per `batch-inference-business-analysis.md` reference table). Specialized schemas per vertical. Massively parallel batch jobs. Low TTFT sensitivity. Low price sensitivity at the enterprise tier (replaces an offshore OCR/keying contract).

#### Workload 19 — Classification & moderation

Sentiment, intent, content moderation, customer-support routing, trust-and-safety scoring. [OpenAI Moderation API](https://openai.com/index/the-state-of-enterprise-ai-2025-report/) and similar; LMSYS-Chat-1M's "Unsafe content" cluster motivates production deployment of moderation classifiers. REALM "Detection and Monitoring" category.

*Operating point:* prefill-heavy in the extreme (input → 1–10 token label). Reasoning shallow (8B sufficient). Massively parallel. Online moderation is high-TTFT; offline scoring is low.

#### Workload 20 — Embedding generation

OpenAI text-embedding-3, Cohere Embed, Voyage, Jina, BGE, NV-Embed. The bulk volume driver of "AI infrastructure" line items in [Menlo's $1.5B AI infra spend](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/).

*Operating point:* prefill-only (no autoregressive decode). Reasoning-depth axis is N/A; the closest analog is "Shallow." Massively parallel, low TTFT sensitivity, high price sensitivity (commodity).

#### Workload 21 — Synthetic data generation

Output of training data, evaluation data, augmentation. Mentioned in `batch-inference-business-analysis.md`; growing line item with the rise of post-training pipelines.

*Operating point:* decode-heavy (output dominates). Shallow-to-medium reasoning. Massively parallel batch. Low TTFT.

#### Workload 22 — Re-ranking

LLM-as-reranker over candidate documents in RAG pipelines. Cohere Rerank, OpenAI re-rank, and bespoke deployments.

*Operating point:* prefill-heavy (k candidate documents in, score out). Online deployment in serving paths makes some re-ranking workloads TTFT-sensitive; offline re-indexing is not.

#### Workload 23 — Batch summarization / ETL

Bulk summarization or transformation of corpora: support-ticket digests, research-paper indexing, news clustering. The largest line item in the batch-inference market by token volume.

*Operating point:* prefill-heavy, low TTFT, massively parallel, high price sensitivity.

### 3.5 Agent Workloads (Non-Code)

Per [LangChain State of Agent Engineering 2025](https://www.langchain.com/state-of-agent-engineering): 57.3% of orgs have agents in production; top use cases customer service (26.5%), research/data analysis (24.4%), internal workflow automation (18%). [Menlo](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/) qualifier: "Only 16% of enterprise and 27% of startup deployments qualify as true agents" — most are fixed-sequence pipelines. The numbers below describe the *true agent* subset.

#### Workload 24 — Customer-service agent (text)

Sierra, Decagon, Cresta, Ada, Intercom Fin. Multi-turn conversations grounded in policy + ticket history + product knowledge, with tool calls into Zendesk/Salesforce/internal APIs. The canonical eval is [τ-bench](https://github.com/sierra-research/tau-bench) (Sierra + Anthropic): retail and airline simulations with multi-turn dialogues, tool calls, and policy adherence. >90% of [a16z CIOs](https://a16z.com/ai-enterprise-2025/) testing this workload. [Menlo](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/) reports CS at $657M of $7.3B departmental AI spend.

*Operating point:* specialized per company. Massively parallel (concurrent conversations). Prefill-heavy (policy + history dominate). High agent E2E latency sensitivity (sub-2s response targets in chat). Tool-use heavy (3–10 calls per typical conversation per industry chatter; no public number found for definitive averages). Tokens per session medium.

#### Workload 25 — Voice agent

Bland, Vapi, Retell, ElevenLabs Voice Agents, Synthflow. The most latency-bound LLM workload in production. [Hamming AI voice latency analysis](https://hamming.ai/resources/voice-ai-latency-whats-fast-whats-slow-how-to-fix-it) and [Twilio's guide](https://www.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents): total round-trip budget 200–300ms ("neurologically hardwired"); LLM TTFT must be ≤400ms; sub-200ms TTFT requires 7B–11B class models at 500–1,000 token context. [Retell benchmarks 2025](https://www.retellai.com/resources/ai-voice-agent-latency-face-off-2025): production end-to-end 600–800ms is best-in-class for telephony.

*Operating point:* the *only* LLM workload where TTFT and agent-E2E latency are both very-high sensitivity simultaneously. Hardware co-location (model + STT + TTS) is the binding constraint. Tokens-per-session short (1K–10K per call), but token-per-second budget is brutal: ~100 tokens generated within a 200ms window for natural turn-taking. Specialized per use case (sales, support, scheduling). Small-batch concurrency (per-call dedicated capacity). Price sensitivity medium.

#### Workload 26 — Browser / computer-use agent

Anthropic Computer Use, OpenAI Operator (Computer-Using Agent), Manus, OpenClaw. Multi-step UI manipulation: screenshots, DOM, click/type/scroll. Evals: [WebArena](https://webarena.dev/), [VisualWebArena](https://jykoh.com/vwa), [OSWorld](https://os-world.github.io/). OSWorld OpenAI CUA: 38.1% success; agents capped at 15–50 steps per task. [OSWorld-Human](https://arxiv.org/html/2506.16042v1) finds 75–94% of agent latency is *planning/reflection*, not action execution.

*Operating point:* deep reasoning (frontier), single-stream, prefill-heavy (screenshots are token-expensive — ~1,500 tokens per Anthropic vision encoder, more for higher-resolution). Long sessions. Heavy tool use. Agent E2E latency high-sensitivity within-task. Price sensitivity low-to-medium (enterprise willingness to pay $200–$500/seat-month for an agent that replaces RPA).

#### Workload 27 — Workflow / sales / RPA-replacement agent

11x.ai, Artisan, Regie.ai, Tessl, Lindy. Background agents that drive multi-step workflows: outbound prospecting, multi-system data ops, scheduled tasks. [LangChain](https://www.langchain.com/state-of-agent-engineering): 18% of agent deployments are internal workflow automation.

*Operating point:* small-batch concurrency, prefill-heavy, low TTFT (background), low agent E2E latency sensitivity (the agent runs while the user does something else), heavy tools, medium-to-long sessions. Specialized per workflow.

### 3.6 Vertical / Specialized Workloads

Per [Menlo Vertical AI 2025](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/) breakdown: Healthcare 43% (~$1.5B; ambient scribes alone $600M), Legal ~19% (~$650M), Government ~10% (~$350M), Creator tools ~10% (~$360M), other (finance, manufacturing, education) ~8% combined.

#### Workload 28 — Healthcare ambient scribe

Abridge, Nuance DAX (Microsoft), Suki, DeepScribe, Augmedix, Heidi Health. Real-time transcription + clinical-note generation from doctor-patient conversations. Largest single vertical AI line item per Menlo.

*Operating point:* specialized (ICD-10, SNOMED, payer-specific note formats). Small-batch (per-clinician). Prefill-heavy (long transcripts). Low TTFT sensitivity (note generated post-encounter or with seconds of buffer). Tokens per session medium-to-long. Light tool use (EHR write-back). Low price sensitivity (replaces $40/hr scribe labor or recovers physician time worth $200+/hr).

#### Workload 29 — Legal contract analysis / e-discovery

Harvey, Thomson Reuters CoCounsel, Spellbook, Hebbia, Eve Legal, Casetext. Long-document review, contract abstraction, due diligence, e-discovery search.

*Operating point:* deep reasoning (frontier required for nuanced legal language). Specialized models common (Harvey markets a custom-finetuned model; CoCounsel uses GPT-4-class with legal-specific tooling). Strongly prefill-heavy (contracts are long). Long sessions (multi-hundred-page diligence). Light-to-heavy tool use. Low price sensitivity ($400/hr lawyer comparison).

#### Workload 30 — Financial research / earnings analysis

AlphaSense, Daloopa, Hebbia, Bloomberg AI, Rogo. Earnings-call transcript synthesis, equity research, regulatory filings analysis.

*Operating point:* deep reasoning, specialized (financial language and structured data). Prefill-heavy. Sessions long. Heavy tool use (data lookups, charting). Latency-medium (some real-time use cases — alpha-generating signal extraction — bump TTFT sensitivity high). Low price sensitivity (sell-side seat pays $20K+/year for terminals; buy-side higher).

#### Workload 31 — Education tutor

Khan Academy Khanmigo, Duolingo Max, Chegg, Quizlet, Magic School. [Anthropic Mar 2026](https://www.anthropic.com/research/economic-index-march-2026-report) coursework category: 12% of Claude.ai (down from 19%). The largest share of Claude.ai use among teenagers and college students.

*Operating point:* general-to-specialized (subject-specific). Massively parallel. Balanced P:D. High TTFT (interactive). Light tool use. High price sensitivity (consumer subsidized by free tiers; institutional licensing low ARPU).

### 3.7 Multimodal / Mixed Workloads

#### Workload 32 — Multimodal Q&A / image-conditioned chat

Image-grounded LLM queries: "what's in this screenshot," "extract data from this chart," visual reasoning. The substrate of computer-use agents (§26), but distinct as a stand-alone workload — e.g., GPT-4V/o-style ad-hoc screenshot Q&A in ChatGPT, Claude.ai, and Gemini. [a16z](https://a16z.com/100-gen-ai-apps-6/) on Nano Banana (Gemini image-aware): 200M images served in first week. NBER reports multimedia ≈ 7% of ChatGPT messages (rose post-April 2025 image-update launch).

*Operating point:* prefill-heavy (image tokens cost ≥ 1K per encoded image; high-res can hit 10K+). Massively parallel. High TTFT (chat UX). General-purpose. Tokens per session medium. Light tool use (vision reasoning is inline, not external-tool-driven). High price sensitivity (commoditized at the chat tier).

*Note on scope:* pure-generation diffusion workloads (text-to-image, text-to-video, voice synthesis) are covered by [a16z State of Generative Media 2026](https://a16z.com/the-state-of-generative-media-2026/) and are out of scope here — the prefill/decode framing does not apply identically. They are large in token-equivalent volume but architecturally distinct.

---

## 4. Master Table — Workloads × Nine Axes

Compact labels: **Sh**allow / **Med**ium / **Dp** (Deep). **Gen**eral / **Spec**ialized. **SS** (single-stream) / **SB** (small-batch) / **MP** (massively parallel). **Pf** (prefill-heavy) / **Bal** (balanced) / **Dc** (decode-heavy). TTFT and Agent-E2E: **L**ow / **H**igh / **VH** (very high). Tokens/session: **S**hort / **M**edium / **L**ong. Tool use: **PG** (pure generation) / **Lt** (light) / **Hv** (heavy). Price sens.: **L** / **M** / **H**.

**Most-deployed model tier (May 2026)**: **OS-S** = open-source 8–30B (Llama-3-8B, Qwen3-8B, Phi-4, Gemma-3, specialty code-completion models). **OS-L** = open-source 70–120B (Llama-3.1-70B, Qwen3-72B, DeepSeek-V3-distill, Mistral-Large). **Fr** = frontier API (Claude Opus 4.7 / Sonnet 4.6, GPT-5 / o3, Gemini 2.5 / 3 Pro). **Spec.** = specialty model outside the three tiers (embedders, rerankers). The "Reasoning" column states the depth the workload *requires*; "Deployed" states what the *current production stack* mostly uses. Where these diverge (e.g., short Q&A: Sh→Med reasoning required, Fr deployed) the gap signals room for downward model-tier migration as open-source quality closes.

**New columns added in this revision:**
- **Best-fit tier** — opinionated recommendation for what model class the workload *should* run on given quality, latency, cost, and specialization constraints (May 2026 quality landscape). Values: **OS-30B** (8–30B sufficient), **OS-100B** (70–120B sufficient), **Frontier** (frontier-only justified), **Spec.** (dedicated embedding/rerank). Where the workload genuinely splits (e.g., simple text vs. complex vision), the cell shows both with the dominant case first.
- **Cost (5/2026)** — dollars per million tokens (input / output) on the *currently dominant* production deployment. For frontier rows this is the public list price of the typical model used (e.g., Claude Sonnet 4.6 at $3/$15); for OS-L/OS-S rows it's a representative serving provider's published rate (Together, Groq, DeepInfra, Bedrock batch). Where the workload uses a non-token unit (per-page, per-resolution, per-minute), the cell reports a token-equivalent estimate; non-token primary unit is footnoted.

🟢 **Green dot** = rows where reasoning is **Sh→Med** AND deployed includes **Fr** — the cleanest "tier-down opportunity" cells, where frontier is currently absorbing token volume that an OS-L (or even OS-S) model could plausibly serve as open-weight quality closes.

| # | Workload | Reasoning | **Best-fit** | **Deployed (5/2026)** | **Cost $/Mtok in/out** | Specializ. | Concurrency | P:D | TTFT | Agent E2E | Tokens/sess. | Tool use | Price sens. |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | 🟢 Short conversational Q&A | Sh→Med | **OS-100B** | Fr ᵃ | $1.25 / $10 (GPT-5) | Gen | MP | Bal | H | L | S | PG→Lt | H |
| 2 | Long-form writing / drafting | Med | OS-100B | Fr | $3 / $15 (Sonnet) | Gen | MP | Dc | H | L | M→L | PG | H |
| 3 | Editing, paraphrasing, critique | Sh | **OS-30B** | Fr / OS-S ᵇ | $1 / $5 (Haiku) | Gen | MP | Bal | H | L | M | PG | H |
| 4 | 🟢 Summarization — single document | Sh→Med | **OS-100B** | OS-L / Fr | $0.88 / $0.88 (Llama-70B) | Gen | SB→MP | Pf | L | L | M | PG | H |
| 5 | Summarization — multi-doc / synthesis | Med→Dp | **Frontier** | Fr | $3 / $15 (Sonnet) | Gen | SB | Pf+Dc¹ | L | L | L | Lt | M |
| 6 | Translation | Sh→Med | **OS-100B** | OS-L ᶜ | $0.23 / $0.40 (Llama-70B DI) | Gen→Spec² | MP | Bal | H | L | S→M | PG | H |
| 7 | Practical guidance / tutoring | Med | **OS-100B** | Fr | $1.25 / $10 (GPT-5) | Gen | MP | Bal | H | L | S→M | PG→Lt | H |
| 8 | Companions / roleplay | Sh→Med | **OS-100B** | OS-L ᵈ | ~$0.20 / $0.50 (in-house) | Gen³ | MP | Dc | H | L | L | PG | H |
| 9 | IDE inline completion (FIM) | Sh | **OS-30B** | OS-S ᵉ | ~$0.05 / $0.05 (custom) | Gen | MP | Pf | **VH** | L | M | PG | H |
| 10 | In-IDE chat | Med | **OS-100B** | Fr | $3 / $15 (Sonnet) | Gen | MP | Pf | H | L | M | Lt | M→H |
| 11 | Code review / PR comments | Med→Dp | **Frontier** | Fr | $3 / $15 (Sonnet) | Gen | SB | Pf | L | L | M | Lt→Hv | M |
| 12 | Long-horizon code agent | Dp⁴ | **Frontier** + OS-L worker ⁶ | Fr ᶠ | $5 / $25 (Opus) | Gen | SS | Pf | H | **H** | L | Hv | L |
| 13 | App builder / vibe coding | Med→Dp | **Frontier** | Fr | $3 / $15 (Sonnet) | Gen | SS | Pf | H | H | L | Hv | M |
| 14 | Code search / repo Q&A | Med | **OS-100B** + Spec. | Fr + Spec. ᵍ | $3/$15 + $0.13 (Sonnet+embed) | Gen | SB | Pf | M | L | M | Lt | M |
| 15 | Enterprise search & grounded chat | Med | **OS-100B** | Fr | $3 / $15 (Sonnet) | Spec | SB | Pf | H | L | M | Lt | M |
| 16 | Web-grounded research / deep research | Dp | **Frontier** + OS-L worker ⁶ | Fr | $5 / $25 (Opus) | Gen | SS | Pf | L | H | L | Hv | M→L |
| 17 | Open-domain citation chatbot | Med | **OS-100B** | Fr / OS-L ʰ | $0.88 / $0.88 (Sonar/Llama-70B) | Gen | MP | Pf | H | L | S→M | Lt | H |
| 18 | 🟢 Document understanding (IDP) | Sh→Med | **OS-100B** (text) / **Frontier** (vision) | Fr / OS-L ⁱ | **$2 / 1K pages** (Mistral OCR) ⁷ | Spec | MP | Pf | L | L | M | PG | L |
| 19 | Classification & moderation | Sh | **OS-30B** | OS-S | $0.05 / $0.05 (Llama-8B) | Gen→Spec | MP | Pf | L→H⁵ | L | S | PG | H |
| 20 | Embedding generation | n/a (Sh) | **Spec.** | Spec. ʲ | $0.13 input only (text-emb-3-large) | Gen | MP | Pf-only | L | L | n/a | PG | H |
| 21 | 🟢 Synthetic data generation | Sh→Med | **OS-100B** | Fr | $2.50 / $15 batch (GPT-5.5) ⁸ | Gen | MP | Dc | L | L | M | PG | H |
| 22 | Re-ranking | Sh | **Spec.** | Spec. ᵏ | ~$0.001 / query (Cohere Rerank) ⁹ | Gen | MP | Pf | L→H⁵ | L | S | PG | H |
| 23 | Batch summarization / ETL | Sh→Med | **OS-100B** | OS-L | $1 / $3 (Databricks Llama-70B) | Gen | MP | Pf | L | L | M | PG | H |
| 24 | Customer-service agent (text) | Med | **OS-100B** + Fr escalation | Fr ˡ | $3 / $15 (Sonnet) | Spec | MP | Pf | H | **H** | M | Hv | M |
| 25 | Voice agent | Sh→Med | **OS-30B** | OS-S ᵐ | $0.59 / $0.79 (Groq Llama-70B) | Spec | SB | Bal | **VH** | **VH** | S→M | Lt→Hv | M |
| 26 | Browser / computer-use agent | Dp | **Frontier** | Fr | $5 / $25 (Opus) | Gen | SS | Pf | H | H | L | Hv | L→M |
| 27 | Workflow / sales / RPA agent | Med | **OS-100B** | Fr / OS-L | $3 / $15 (Sonnet) | Spec | SB | Pf | L | L | M→L | Hv | M |
| 28 | Healthcare ambient scribe | Med | **Frontier** (today, BAA) → **OS-100B** | Fr ⁿ | $3 / $15 (Sonnet under BAA) | Spec | SB | Pf | L | L | M→L | Lt | L |
| 29 | Legal contract analysis / e-discovery | Dp | **Frontier** | Fr | $5 / $25 (Opus) | Spec | SB | Pf | L | L | L | Lt→Hv | L |
| 30 | Financial research / earnings | Dp | **Frontier** | Fr | $5 / $25 (Opus) | Spec | SB | Pf | M→H | M | L | Hv | L |
| 31 | Education tutor | Med | **OS-100B** | Fr | $0.50 / $3 (Gemini Flash) | Gen→Spec | MP | Bal | H | L | M | Lt | H |
| 32 | Multimodal Q&A / image chat | Med | **Frontier** (vision gap) | Fr ᵒ | $0.50 / $3 (Gemini Flash) | Gen | MP | Pf | H | L | M | PG→Lt | H |

**Footnotes to the table (operating-point):**
1. Multi-document synthesis combines a heavy prefill (corpus ingestion) with a non-trivial decode (long synthesis output). Vidur's BWB-4K shows P:D = 0.65 (decode-leaning).
2. Translation is general-purpose for casual/business prose; legal, medical, and literary translation use specialized tunes.
3. Companion apps frequently apply persona LoRAs but base model is general.
4. Reasoning depth for code agents is "currently deep-frontier; rapidly migrating to open-weight 70–120B" — see Anthropic Mar 2026 commentary on the API Computer/Math share shift.
5. Online moderation and online re-ranking are TTFT-sensitive in the serving path; offline batch versions are not.
6. **Hierarchical pattern.** Best-fit reflects the dominant token-volume layer once the workload is decomposed into planner + worker calls. The frontier planner does the Deep reasoning; OS-L workers execute search/read/summarize subtasks and account for the bulk of token consumption. See §9 for the full architectural argument.
7. **Per-page primary unit.** IDP is priced per page in production (Mistral OCR 3 list $2 / 1K pages, batch $1 / 1K pages — Dec 2025; AWS Textract Analyze $50–65 / 1K pages; Reducto starts at $0.015/page). Token-equivalent at ~1,500 input tokens/page is ~$1.30/Mtok input for the OCR step plus a downstream LLM extraction call.
8. **Batch primary unit.** Synthetic data runs almost exclusively on batch endpoints (50% off list); GPT-5.5 batch input ≈ $2.50/Mtok, output $15/Mtok. Equivalent batch pricing on Claude Opus 4.7 ≈ $2.50/$12.50; on Llama 3.3 70B (Together batch) ≈ $0.88/$0.88, no batch discount needed.
9. **Per-query primary unit.** Cohere Rerank v3 lists at ~$2 per 1K queries with up to 100 documents per query; per-token equivalents are not the natural unit for re-ranking and roughly map to $0.10–$0.30 per Mtok input.

**Best-fit vs. Deployed — descriptive tally.** Across the 32 rows: **18 match** (deployed tier equals or is the cheapest member of the best-fit set), **5 partial match** (deployed tier includes the best-fit but also runs a more expensive tier in parallel), **9 mismatch** (deployed tier is materially above best-fit). The mismatch cluster is rows **1 (Short Q&A), 2 (Long-form writing), 7 (Practical guidance), 10 (In-IDE chat), 14 (Code search), 15 (Enterprise search), 21 (Synthetic data), 24 (Customer-service agent text), 31 (Education tutor)** — disproportionately the high-volume consumer chat surfaces and the enterprise interactive-but-not-deep tier. The matches concentrate at the two ends: latency-bound (rows 9, 25 → OS-S) and frontier-required (rows 5, 11, 12, 13, 16, 26, 29, 30 → Frontier). Specialty rows (20, 22) match by construction. This is a snapshot of May 2026, not a prediction; the partial-match rows are the most likely to flip to fully matched as buyers test OS-L variants.

**Footnotes to the table (deployed-tier):**
- a. Consumer chat volume is dominated by GPT-5/-4o, Claude Sonnet 4.6, Gemini 2.5/3 Pro running ChatGPT (900M WAU), Claude.ai, and Gemini surfaces. Llama-3-8B / Qwen3-8B exist for cost-sensitive deployments and on-device but are a small share of conversation volume today.
- b. Free-tier consumer editing is frontier; programmatic editing in writing-tools APIs (Grammarly, content platforms) is increasingly OS-S finetunes.
- c. Llama-3.1-70B and Qwen3-72B are competitive with frontier on most language pairs; Google Translate / DeepL run proprietary stacks adjacent to OS-L. Frontier used where document-level pragmatics matter.
- d. Character.AI (in-house tunes), Replika, and Janitor.AI all operate near the OS-L tier; smaller "uncensored" deployments live at OS-S.
- e. Cursor Tab is a custom small model; GitHub Copilot's completion model is sub-7B specialty; Codeium uses small specialty models. Sub-200ms TTFT requirement forces the tier down.
- f. Claude Code → Claude Opus 4.7; Cursor Agent → Claude Sonnet 4.6 / Opus 4.7; OpenAI Codex CLI → GPT-5 / o3; Devin → GPT-5. Open-weight Qwen3-Coder and DeepSeek-V3 are closing on these evals but production deployments remain frontier-dominated as of May 2026.
- g. Code search uses frontier for the LLM-Q&A stage and specialty embedding/re-ranker models for retrieval. Sourcegraph Cody, Greptile, Glean for engineering all follow this pattern.
- h. Perplexity defaults to its in-house **Sonar** family (OS-L finetunes of Llama 3.1 70B and DeepSeek), with frontier (GPT-5, Claude Opus, Gemini) as a paid upgrade. ChatGPT Search uses GPT-5.
- i. Vision-heavy IDP (form fields, handwritten, multi-language) is frontier (GPT-4o-class, Claude vision); text-heavy or schema-bound IDP uses OS-L (Reducto, Unstructured pipelines often use Llama 3.1 70B or DeepSeek-V3 internally).
- j. Embeddings are served by dedicated dual-encoder models outside the three tiers: text-embedding-3-large/small, Cohere Embed v3, BGE-M3, NV-Embed-v2, Voyage. Typical sizes 100M–7B.
- k. Re-rankers are also dedicated cross-encoder models: Cohere Rerank v3, BGE-Reranker, mxbai-rerank, Jina-Reranker. Typical sizes 300M–4B.
- l. Sierra, Decagon, Cresta deploy Claude Opus / GPT-5 with extensive prompt caching and tool harness work. Smaller-tier customer-support bots (Intercom Fin, Ada) mix GPT-4o-mini / Claude Haiku 4.5 (frontier-cheap-tier) with retrieval.
- m. Voice latency budget (≤400ms LLM TTFT) forces small models. Vapi/Retell defaults are typically Llama-3-8B / Qwen3-8B served on Groq / Cerebras; some deployments use Claude Haiku 4.5 with aggressive caching as a frontier-cheap option.
- n. Abridge, Nuance DAX (Microsoft), Suki run on GPT-4-class / Claude Sonnet under BAA. Some scribe vendors fine-tune OS-L for specific specialties but production-grade clinical reasoning is still frontier-dominated.
- o. Vision quality on frontier is materially ahead of OS-L vision models (Llama-3.2-Vision, Qwen-VL); production multimodal Q&A in consumer surfaces is GPT-5/-4o, Gemini 2.5/3 Pro, Claude Sonnet vision.

---

## 5. Cross-cuts and Correlations

**Q: Which axes cluster?**

The 32-row table looks like nine independent dimensions. It is not. Three latent profiles account for most of the rows; a small number of workloads sit on the boundary.

### Profile A — "Interactive chat"

Massively parallel, balanced or slightly prefill P:D, high TTFT, low agent-E2E, short-to-medium sessions, light or no tools, high price sensitivity, general-purpose.

Members: short Q&A (1), long-form writing (2), editing (3), translation (6), practical guidance (7), companions (8), in-IDE chat (10), open-domain citation chatbot (17), education tutor (31), multimodal Q&A (32).

What unites them: the human is in the loop, waiting on a streamed reply. Hardware is bandwidth-bound at decode; price competition is brutal because consumer free tiers set the ceiling.

### Profile B — "Batch / data-plane"

Massively parallel, strongly prefill-heavy, low TTFT, low agent-E2E, short-to-medium sessions, pure generation or light tools, high price sensitivity.

Members: summarization single-doc (4), document understanding (18), classification (19), embedding (20), synthetic data (21), re-ranking (22), batch summarization (23). Code workloads 11 (PR review) and 14 (repo Q&A) sit adjacent.

What unites them: the human is not in the loop. Throughput-per-dollar is the only metric that matters. This is the operating-point fingerprint of the `batch-inference-business-analysis.md` archetype.

### Profile C — "Long-horizon agent"

Single-stream, strongly prefill-heavy, high agent-E2E latency sensitivity (within-task), long sessions, heavy tools, low-to-medium price sensitivity (replaces hours of human labor).

Members: long-horizon code agent (12), app builder (13), web-grounded deep research (16), browser/computer-use agent (26), legal contract analysis (29), financial research (30). Customer-service text agent (24) is adjacent but at higher concurrency.

What unites them: one task at a time per user; the user trades wall-clock for autonomy. KV-cache is the load-bearing data structure (Mooncake's disaggregated cache shows up to 2.36× reuse improvement; ~48% prefill compute saved). This is the operating-point fingerprint of the `agentic-inference-business-analysis.md` archetype.

### Boundary / hybrid workloads

A handful of workloads do not fit any one profile cleanly:

- **IDE inline completion (9).** Massively parallel like Profile A, but TTFT-bound to 0.125s (DistServe) — tighter than any chat workload — and prefill skew >100:1 (Splitwise). It looks like "interactive chat with batch-style P:D and code-completion-only TTFT." A profile of one.
- **Voice agent (25).** Both TTFT and agent-E2E latency are very-high sensitivity simultaneously. No other workload pairs these. Drives a unique hardware co-location requirement (model + STT + TTFT < 800ms round-trip).
- **Customer-service text agent (24).** Massively parallel like Profile A, but sessions are agentic (heavy tools, multi-turn, tool-call latency dominated). Sits between A and C.
- **Healthcare scribe (28), legal contract analysis (29), financial research (30).** Specialized verticals at small-batch concurrency with low price sensitivity. They look like Profile C but with lower agent-E2E sensitivity (offline or near-real-time, not interactive) and lighter tool use. A "vertical specialist" profile.

**Q: Which workloads sit on the boundary, and why does that matter (descriptively)?**

The boundary workloads are the ones whose dominant *deployment mode* could plausibly migrate. IDE inline completion could migrate from massively-parallel (cloud) to single-stream (on-device) without changing what it is. Voice agent could migrate from cloud (Vapi/Retell hosted) to edge (telephony-DC co-located) or to-device (pixel-watch / earbuds). Customer-service agent could migrate from cloud to per-tenant on-prem.

This is descriptive, not prescriptive. The point is: **most boundary workloads share the property that their constraint is co-location of model + tools + state, not raw compute.** Whether that observation maps to a thesis is the work of the next document, not this one.

---

## 6. Coverage and Gaps

What this taxonomy covers well, and where the public record is thin.

**Well-covered.** Coding (best-instrumented workload in the literature: Splitwise, Anthropic Economic Index, SemiAnalysis, Cursor public posts, Claude Code public usage data, MCPMark, BFCL). Summarization (DistServe, Vidur, LMCache, InferenceMAX). Chat (BurstGPT, Mooncake, Splitwise conversation trace, KVCache-in-the-Wild). Voice (multiple infra writeups; latency budgets are well-known). Long-horizon agents (LangChain survey; MCPMark; SWE-bench).

**Thinly covered.** Per-workload TTFT *distributions* (Azure publishes raw CSV but no per-percentile summaries by category). Mooncake / Kimi median and p99 input-output (only means published). Anthropic-specific average tokens per Claude.ai session (only Claude Code session ranges, third-party). Embedding and classification token distributions (the inference-research papers are decoding-system papers and do not characterize embedding workloads). τ-bench per-domain turn averages for customer-service evals (paper appendix has them; PDFs are scan-rendered).

**Effectively absent.** Public per-workload prefill:decode for healthcare scribe, legal contract analysis, financial research (these are vertical-AI deployments and the operating-point data is internal to each vendor). Public agent-step distributions for browser-use products in real production (versus benchmarks). Voice-agent token distributions (industry benchmarks publish latency, not per-call token shape).

The cells in §4 marked Sh→Med, Med→Dp, etc. tend to correspond to the thinly-covered ranges. They are honest splits, not hedges.

---

## 7. Open Questions / Next Steps

- **Reasoning-depth migration.** The single biggest dynamic in the table is workloads whose reasoning depth is moving down — e.g., code agent (12) currently labelled Deep but rapidly migratable to Medium as Qwen3-Coder / DeepSeek-V3 / open-weight 70–120B close on frontier coding evals. A version of this doc cut a year from now will have several Dp cells flipping to Med. Tracking *which* workloads flip first is the live research question.
- **Tool-use intensity is rising across the board.** [MCP downloads](https://mcpmanager.ai/blog/mcp-adoption-statistics/): 2M (Nov 2024) → 97M (Mar 2026); 10,000+ active public MCP servers. Many workloads currently labelled "Light" may be "Heavy" within a year as MCP connectors propagate into chat surfaces (Claude.ai, ChatGPT Plus, Gemini).
- **The "agent-E2E latency: High" workloads need their own characterization doc.** Profile C is the most internally heterogeneous — code agents, deep-research agents, computer-use agents differ on every other axis. A follow-up doc that subdivides Profile C by *tool topology* (filesystem-based, browser-based, API-based, voice-mediated) would sharpen the table.
- **Vertical workloads are under-instrumented.** Healthcare, legal, finance, education each get a single row here; in practice each contains sub-workloads with materially different operating points (e.g., legal contract review vs. e-discovery vs. case prediction). Worth a vertical-only deep-dive.
- **The price-sensitivity axis needs a quantitative anchor.** "High / Medium / Low" is the user-supplied scale; mapping it to actual $/Mtok WTP per workload would let the table predict pricing tiers, not just categorize them. Public data is partial: Menlo's $/department spend gives a denominator; Cursor / Anthropic / OpenAI public ARRs give a per-product numerator; bridging the two is downstream work.
- **Mapping this taxonomy to the local-AI / desktop-GPU / commoditization thesis is explicitly deferred to a separate document.** This file is a reusable descriptive reference, not a positioning argument.

---

## 8. Inference Economics for the Sh→Med Workloads (Deep Dive)

This section drills into the eight workloads with **Sh→Med reasoning depth required** — rows 1, 4, 6, 8, 18, 21, 23, 25 in the master table — and reports, per workload: (A) **price-per-token today**, (B) **named customers**, (C) **where the inference is sourced.** Four of these are 🟢 "tier-down candidates" (Fr deployed despite Sh→Med required); four already operate at OS-L or OS-S. The contrast between the two halves is the most concrete evidence available for how fast workloads migrate down-tier when the economics force it.

### 8.1 Short Conversational Q&A 🟢 *(currently Fr-deployed)*

**(A) Pricing.**

| Tier | Model | Input $/Mtok | Output $/Mtok |
|---|---|---|---|
| Frontier | GPT-5 | $1.25 | $10.00 |
| Frontier | GPT-5.5 (Apr 2026) | $5.00 | $30.00 |
| Frontier | Claude Sonnet 4.6 | $3.00 | $15.00 |
| Frontier | Claude Haiku 4.5 | $1.00 | $5.00 |
| Frontier | Gemini 3 Flash | $0.50 | $3.00 |
| OS-L | Llama 3.3 70B / Groq | $0.59 | $0.79 |
| OS-L | Llama 3.3 70B / DeepInfra | $0.23 | $0.40 |
| OS-L | DeepSeek V3.2 (api.deepseek) | $0.28 | $0.42 |
| OS-S | Llama 3.1 8B / DeepInfra | $0.029 | $0.05 |

Effective rates: ~50% of list for enterprise commits; OpenAI/Anthropic prompt caching at 90% off cached input; Anthropic batch and OpenAI Batch at 50% off. Free-tier consumer chat is heavily subsidized — Sam Altman has stated free-tier users cost OpenAI hundreds of millions per quarter.

**(B) Customers.**
- **ChatGPT** — 700–900M WAU; Free on GPT-5.3 Instant capped 10 msg/5h; Plus $20/mo (projected ~9M subs in 2026, down from 44M in 2025); Go ~$5/mo ad-supported (launched Feb 9, 2026, ~112M subs projected); Pro $200/mo on GPT-5.5 Pro.
- **Claude.ai** — Pro $20/mo on Sonnet 4.6 / Opus 4.7; consumer tier direct on Anthropic.
- **Gemini consumer** — Google AI Pro $19.99/mo on Gemini 3 Pro; free tier on Gemini 3 Flash.
- **Perplexity** — 22M+ MAU; free tier on Claude Haiku via Bedrock + in-house Sonar (OS-L finetune of Llama 3.1 70B); Pro $20/mo with user-selectable frontier.
- **DeepSeek** (chat.deepseek.com) — 100% free, runs DeepSeek V3.2 / R1 on its own China-domiciled Huawei Ascend + H800 clusters.
- **Mistral Le Chat** — free 25 msg/day; Pro €14.99/mo; "Flash Answers" claims 1,000 wps via Cerebras partnership.
- **xAI Grok** — free with X account; SuperGrok $30/mo.
- **Klarna AI Assistant** — 1.3M chats/mo (≈2/3 of CS), GPT-class direct API.

**(C) Inference source.** Frontier labs serve their own free tiers from in-house clusters: OpenAI on Azure ($100B+ Microsoft commitment); Anthropic split between AWS Trainium (the $100B AWS commit) and Google TPUs (the Oct 2025 "up to 1M TPUs" deal); Google Gemini fully on TPU v5p / Ironwood. **OpenRouter** (≈$50M ARR on a 5% take rate, implying ~$1B routed token spend) reports **Chinese open-weight models hit 61% of token volume in February 2026** — the single most important data point in this workload. DeepSeek's free web app handles enormous traffic on its own GPU clusters. Le Chat sits on a mix of self-hosted H100s and Cerebras CS-3s (Cerebras Llama 3.3 70B at 1,800+ t/s).

### 8.2 Summarization, Single Document 🟢 *(currently OS-L / Fr-mixed)*

**(A) Pricing — per 1-hour meeting (≈10K input, ≈1K output transcript→summary).**

| Backend | $/meeting |
|---|---|
| Claude Sonnet 4.6 | ≈ $0.045 |
| GPT-5 | ≈ $0.0225 |
| Gemini 3 Flash | ≈ $0.008 |
| Llama 3.3 70B / Groq | ≈ $0.007 |

A $20/seat-month Otter/Fireflies subscription with ~30 meetings/user produces ≈$0.67/meeting in revenue — frontier inference COGS run 3–7%, OS-on-fast-silicon under 1%.

**(B) Customers.**
- **Granola AI** — Brex named customer; $43M raise at $250M valuation; $14–$35/user-month tiers.
- **Fireflies.ai** — $1B valuation; 500K+ orgs; **Uber, Netflix, Nike, Salesforce** named.
- **Otter.ai** — SMB/individual; $8.33–$20/user-month.
- **Gong** — surpassed **$300M ARR Jan 2025**, ~$4.5B 2026 secondary valuation; "Ask Anything" up 400% YoY; F500 sales-org buyer.
- **Fathom, Chorus, Read.ai, tl;dv** — same architectural pattern.
- **NotebookLM (Google)** — Gemini 3 Pro on paid; Gemini 2.5 Flash on free.

**(C) Inference source.** **Frontier-LLM-as-a-service** dominates: Granola's stack is Deepgram/AssemblyAI ASR + GPT-4o + Claude direct API. Fireflies routes through OpenAI/Anthropic. Gong is multi-LLM Anthropic + OpenAI. NotebookLM is 100% in-house Gemini. **Almost no one in this category runs on OS-L yet** — summary quality is user-visible and per-meeting cost is already low enough that the engineering cost of a self-hosted vLLM cluster does not pay back. The exception is Gong-class enterprise tier where contracts route through Bedrock or Vertex with negotiated rates. The 🟢 dot says the *opportunity* exists; today's vendors haven't taken it.

### 8.3 Translation *(currently OS-L-deployed)*

**(A) Pricing.** The translation market shows the largest unit-cost spread of any workload in the table.

| Backend | $/Mchar | Notes |
|---|---|---|
| Google Translate Cloud Basic (NMT) | $20 | ~100ms latency |
| Google Translate "Translation LLM" tier | ~$60 | "largely a rebrand of NMT with premium price" |
| DeepL API Pro | $25 + $5.49/mo | 500K free / month on free tier |
| Gemini 3 Flash | ≈ $0.75–$1.50 | $0.50/$3 per Mtok |
| Llama 3.3 70B / DeepInfra | ≈ $0.40 | $0.23/$0.40 per Mtok |
| NLLB-200 (self-hosted, OSS) | infra-only | 200-language coverage |
| RWS Language Weaver Pro | enterprise quote | 100B+ dedicated MT, built with Cohere |

WMT25 (April 2026): **Gemini 2.5 Pro #1 across 14 of 16 language pairs** in human evaluation; LLMs occupy the top 4 positions; Google Translate, DeepL, Microsoft Translator rank mid-table. Dedicated-MT lost to LLMs at the top of the quality curve sometime in 2024–2025.

**(B) Customers.**
- **DeepL** — ~€500M ARR; 100K+ business customers including **Tencent, Mercedes, Coursera**; runs proprietary encoder-decoder NMT on its own H100 clusters in Germany + a Llama-3-derived "DeepL Next-Gen" tier (2025).
- **Smartling** — multi-engine routing across DeepL/Google/Microsoft/OpenAI plus its own LanguageAI; **Vimeo, Lyft, Pinterest, IHG**.
- **Lokalise** — own "Lokalise AI"; **Revolut, Yelp, Notion**.
- **Phrase (formerly Memsource)** — **Uber, Bosch, Shopify, Puma**.
- **Crowdin AI** — game/SaaS localization; **Wargaming, Discord, Khan Academy**.
- **RWS Language Weaver Pro** (March 2026 launch) — F500 legal/regulated; Cohere partnership.
- **TransPerfect** — claims 90% of F500.
- **Unbabel** — fine-tuned Llama-3.1-70B + NLLB blend; **Booking.com, Microsoft, Logitech**.
- **Riot, Activision-Blizzard** — game localization on OpenAI API + in-house glossary RAG.
- **Lara (Translated)** — adaptive MT for legal/medical; Llama-3.1-70B finetunes.

**(C) Inference source.** Three layers. **High-volume real-time** (Google Translate consumer, Apple Translate, Bing, captioning) still on dedicated NMT on TPU/in-house silicon — latency keeps it there. **Enterprise localization workflow** (Lokalise/Smartling/Phrase/Crowdin) runs an aggregator pattern: pull from DeepL + Google + OpenAI + Anthropic, route by quality+cost, with TM/glossary RAG in front. **Premium quality content** (legal, gaming, marketing) is shifting hard to LLM — Gemini 2.5/3 Pro direct, GPT-5, Llama 3.1 70B served on Together/Fireworks/DeepInfra. The Llama-3.3-70B-on-DeepInfra path is ≈50× cheaper per character than DeepL Pro list. DeepL's pricing power survives on brand + zero-config quality, not infrastructure economics.

### 8.4 Companions / Roleplay *(currently OS-L-deployed)*

**(A) Pricing.** Subscription tiers as proxy; daily-message limits indicate per-message COGS.

| Provider | Subscription | Inferred backend |
|---|---|---|
| Character.AI | Free; c.ai+ $9.99/mo | In-house OS-L finetunes on Google TPU |
| Replika | Pro $7.99 / Ultra ~$13 / Platinum higher | In-house finetuned transformer on AWS |
| Janitor.AI | Free; user BYOK | OpenRouter / Claude API direct (user-paid) |
| Chai | Premium $13.99/mo | "Chaiverse" OS-L finetunes ~13B class |
| Talkie (MiniMax) | $9.99/mo | MiniMax MoE on its own infra |
| Kindroid | Lite $9.99 / Pro $19.99 | In-house finetunes |
| Spicychat | Free; Premium $4.95 / Pro $14.95 | OpenRouter + in-house OS-L |

Companion apps must clear sub-$0.001/message inference cost or unit economics fail — at 50 msgs/day × 30 days, $9.99/mo only works on Llama-class self-hosted serving.

**(B) Customers / scale.**
- **Character.AI** — 20M MAU (down from 28M peak), $50M revenue 2025, projected $60M 2026; **Google paid $2.7B Aug 2024** to license tech + acquihire Noam Shazeer / Daniel De Freitas; inference moved to Google Cloud TPU with publicly cited **3.8× cost improvement**.
- **Replika** — ~30M registered, 2M+ paying; Pro $79.99/yr (~$6.67/mo); proprietary in-house LLM, no API access.
- **Janitor.AI** — 130M+ monthly visits; BYOK — material share of OpenRouter volume.
- **Chai Research** — claimed 10M+ MAU at peak; community-finetuned 13B–34B models.
- **Talkie (MiniMax-owned)** — 10M+ downloads.
- **Polybuzz** — 5M+ DAU claimed.

**(C) Inference source.** The **most OS-L-dominated consumer workload**. Drivers: (a) frontier API safety filters refuse most romantic content, forcing operators off frontier; (b) per-message economics demand sub-$0.0001 cost; (c) finetune control matters more than reasoning quality. **Character.AI** is the canonical case: pre-Google, ~20K H100-equivalents in-house; post-acquihire, migrated to Google TPU v5e/v5p at 3.8× cost improvement. **Replika** has always run in-house finetuned models on AWS — refused to switch to GPT-4 even when cheap. **Janitor.AI is a routing layer** — token volume flows through OpenRouter (the 61% Chinese-model share is partially explained by Janitor users gravitating to DeepSeek and Qwen3 for uncensored content). The long tail (Spicychat, Polybuzz, Joyland) hits **DeepInfra + Together for Llama 3.1 70B / Mistral Nemo / Qwen 2.5 32B at $0.20–$0.50/Mtok** — the cheapest production-grade serving in the market.

### 8.5 Document Understanding (IDP) 🟢 *(currently Fr / OS-L mixed)*

**(A) Pricing.** The largest unit-cost dispersion in the table — and the most disruptive 2025-2026 pricing event.

| Vendor / Service | Unit | List price |
|---|---|---|
| **AWS Textract Analyze Document** | per page | $0.065 (first 1M/mo), $0.050 thereafter |
| AWS Textract Custom Queries | per 1K pages | $25 |
| **Mistral OCR 3** (Dec 18, 2025) | per 1K pages | **$2.00 list / $1.00 batch** |
| **Reducto** | per page | starts at $0.015/page; volume-tiered |
| **LlamaParse** | per page | $0.00125 basic → $0.1125 agentic Sonnet 4.0 mode |
| Google Document AI | per page | ~$0.03–$0.10 |
| Frontier vision DIY (Claude Sonnet 4.6, ~2K in / 1K out per page) | per 1M pages | ~$21,000 |

At 1M pages: AWS Textract Analyze $50–65K, Google Document AI $30–100K, Mistral OCR 3 batch **$1,000**, Reducto enterprise ~$5–15K. Mistral's $2/1K-page launch undercuts Textract by ~97% and Document AI by ~93% — the load-bearing 2025-2026 pricing event in IDP.

**(B) Customers.**
- **Hyperscience** — **Mars, US Department of Veterans Affairs, Guardian Life, HM Revenue & Customs, International Rescue Committee**; F500 insurer case study cites ~85% reduction in claims processing time.
- **Rossum** — 450+ orgs; **Bosch, Siemens, Panasonic, Flexport, Morton Salt, The Master Trust Bank of Japan**; proprietary "Rossum Aurora" transactional LLM.
- **Allianz** — 71% claims-automation rate; lifecycle reduced 19 days → 4; uses Project Nemo + Incognito agentic stack.
- **Zurich** — Agentic AI Hyperchallenge (2025): 1,000 participants, 218 prototypes, 5 in production.
- **Reducto** — $24.5M Series A; published customers include **Meta, Vanta, Scale AI, Anthropic** (ingestion pipelines).
- **Mistral OCR 3** — early adopters **BNP Paribas, French government agencies**.
- **Instabase** — late-stage IDP; financial services + public sector.
- AWS Textract / Google Document AI / Azure Form Recognizer — every F500 with mortgage/insurance-claims/KYC pipelines runs at least one as the OCR backbone.

**(C) Inference source.** Two-stage architecture. **Stage 1** (OCR/parse): Textract, Mistral OCR, Reducto, Document AI, LlamaParse. **Stage 2** (LLM extraction/normalization): historically GPT-4o and Claude Sonnet, now increasingly **Gemini Flash, Llama 3.3 70B on Bedrock/Vertex/Together, Mistral OCR's bundled extraction**. Mistral OCR 3 at $2/1K pages forces every IDP vendor to rebuild gross-margin assumptions. **Hyperscience and Rossum are stack vendors** — they own ingestion + human-in-loop + audit; the underlying LLM is a substituted commodity. Enterprise IDP buyers in regulated verticals (insurance, banking, healthcare) increasingly demand on-prem or VPC inference, which is why Hyperscience runs Llama-class models on customer infra (often AWS GovCloud or on-prem). The Mistral OCR 3 launch is the clearest single 2026 datapoint that the 🟢 tier-down dynamic is already happening in this workload.

### 8.6 Synthetic Data Generation 🟢 *(currently Fr-deployed)*

**(A) Pricing.** Output-token-dominated; runs on batch endpoints.

| Tier | Model | Output $/Mtok (batch) | $/1B tokens batch |
|---|---|---|---|
| Frontier | Claude Opus 4.7 | $12.50 | $12,500 |
| Frontier | GPT-5.5 | $15.00 | $15,000 |
| Frontier | GPT-5 | $5.00 | $5,000 |
| Frontier | Claude Sonnet 4.6 | $7.50 | $7,500 |
| Frontier | Gemini 3 Flash batch | $1.50 | $1,500 |
| OS-L | Llama 3.3 70B / Together batch | ~$0.88 | $880 |
| OS-L | Qwen 3 72B / Fireworks | ~$0.90 | $900 |
| OS-S | Llama 3.1 8B / Together | ~$0.18 | $180 |

A 1-billion-token fine-tune dataset costs **~$15K on GPT-5.5 batch, ~$12.5K on Claude Opus batch, <$1K on Llama 70B**.

**(B) Customers.**
- **Scale AI** — Meta acquired 49% at $14.3B (mid-2024), hollowing out third-party serving; shifted upmarket to defense/government.
- **Snorkel AI** — Foundation Model Data Platform / Foundry; **5 of the top 10 US banks, BNY Mellon, Memorial Sloan Kettering, Wayfair**.
- **Gretel.ai** — **acquired by NVIDIA March 2025**; folded into NeMo as **NeMo Data Designer** (open-sourced Apache 2.0).
- **Mostly AI** — synthetic structured data for big banks and insurance.
- **Tonic AI** — financial services, healthcare, e-commerce.
- **Datology AI** — pre-training data quality; $46M Series A 2024; customers include **major frontier labs**.
- **Argilla / Distilabel (HuggingFace)** — open-source tooling layer.
- **AI labs themselves** — **OpenAI, Anthropic, Mistral, Meta, Google** all run synthetic data at trillion-token scale on their own training clusters. Anthropic's Constitutional AI is the canonical public method.
- **Bloomberg** and other financial firms — internal synthetic augmentation for in-house models.

**(C) Inference source.** Frontier-lab synthetic generation runs **on the lab's own training cluster**, not a paid endpoint. The interesting paying buyer is the **Tier B/C enterprise post-training shop** (Predibase, Together Custom Models, Fireworks Build, Snorkel Foundry, in-house bank ML teams). Their pattern: **OpenAI Batch + Anthropic Batch for "teacher" generation** (frontier quality on a small, high-leverage subset — instruction-tune seeds, reasoning chains), then **Llama 3.3 70B / Qwen 3 72B on Together or DeepInfra batch / their own H100s** for bulk volume expansion. The 50% batch discount is critical because the workload is genuinely offline. Increasingly the teacher→student pattern routes through **Bedrock Batch Claude** because regulated buyers need the model in-VPC. A 100-person bank ML team can burn $1M of batch tokens in a quarter — which is why this workload is the most price-sensitive, highest-volume token consumer in the enterprise, and exactly why the 🟢 frontier-deployed today / OS-L-feasible tomorrow gap is so live.

### 8.7 Batch Summarization / ETL *(currently OS-L-deployed)*

**(A) Pricing.** Per-Mtok rates after batch discount; enterprises increasingly buy through the data-platform contract, not the model API.

| Platform | Model | Input $/Mtok | Output $/Mtok | Notes |
|---|---|---|---|---|
| **AWS Bedrock Batch** | Claude Sonnet 4.6 | $1.50 | $7.50 | 50% off list |
| AWS Bedrock | Llama 3.3 70B | ~$2.65 | ~$3.50 | 50% batch |
| AWS Bedrock | Nova Pro | $0.80 | $3.20 | 50% batch |
| **Vertex AI Batch** | Gemini 3 Flash | $0.25 | $1.50 | 50% off |
| Vertex AI Batch | Gemini 3 Pro | ~$0.625 | ~$5.00 | 50% off |
| **Snowflake Cortex** | Llama 3.1 70B | 1.21 cred/Mtok ≈ $2.66 | -- | -- |
| Snowflake Cortex | Claude Opus 4 | up to 12 cred/Mtok | -- | -- |
| **Databricks FM Serving** | Llama 3.3 70B | ~$1.00 | ~$3.00 | "up to 80% TCO" claim |
| Databricks | gpt-oss-20B | $0.13 blended | -- | -- |
| **Zendesk Advanced AI** | per-resolution | $1.00–$2.00 | -- | -- |
| **Intercom Fin AI Agent** | per-resolution | $0.99 + $49.50/mo min | -- | -- |

**(B) Customers.**
- **Intercom Fin** — 7,000+ customers; 67% average resolution rate; Zendesk/Salesforce/HubSpot installs.
- **Zendesk AI** — F500 contact-center buyers.
- **Databricks Mosaic / FM Serving** — **Block, Comcast, Walgreens, Rivian** publicly disclosed.
- **Snowflake Cortex** — F500 batch-summarization (anecdotal "$5K single-query" from industry coverage); Cortex Search and AI_SUMMARIZE the dominant function calls.
- **AWS Bedrock Batch** — every regulated F500 with a Bedrock contract; dominant "VPC-native" batch endpoint.
- **Vertex Batch** — Workspace customers + GCP F500.
- **Relativity / Reveal** (legal e-discovery) — Bedrock/Vertex case summarization at multi-TB scale; **Kirkland, Latham, Skadden** as end users.
- **Government / public sector** — DoD JWCC contracts route through Bedrock GovCloud and Azure Government.

**(C) Inference source.** This is the **single biggest US enterprise token-volume workload**. Platforms have **bundled the LLM into the data-platform contract**: Databricks customer pays in DBUs; Snowflake customer pays in credits; Bedrock customer pays AWS. Behind the scenes, **Databricks runs Llama 3.3 70B and gpt-oss-20B on its own GPU fleet** (50% per-token reduction, "up to 80% TCO" savings); Snowflake routes Llama 3.1 70B at 1.21 credits/Mtok and Claude Opus 4 at up to 12 credits/Mtok via API integration; Bedrock Batch is the dominant endpoint for Claude Sonnet 4.6 + Llama + Nova in regulated F500 at 50% off list. **Bedrock + Vertex + Databricks collectively process >50% of all enterprise LLM tokens by volume in May 2026** (industry estimate, no audited figure). The on-prem angle is real: very large F500 buyers (large banks, insurers, telcos) are starting to deploy vLLM/SGLang Llama 3.3 70B and Qwen3-72B on their own H100/H200/MI300X clusters specifically to escape per-token markup from Databricks/Snowflake/Bedrock for high-volume nightly ETL.

### 8.8 Voice Agent *(currently OS-S-deployed)*

**(A) Pricing.** All-in cost = orchestration + STT + LLM + TTS + telephony.

| Platform | Headline $/min | True all-in $/min |
|---|---|---|
| Vapi | $0.05 (orchestration only) | $0.12–$0.35 |
| Retell AI | $0.07 (PAYG) | $0.15–$0.30 |
| Bland AI | $0.09 (plan-based since Dec 2025) | $0.15–$0.40, $359/500 min Build plan |
| Synthflow | $0.08 ($0.07 enterprise) | $0.15–$0.37 |
| ElevenLabs Agents | $0.08–$0.12 (bundled premium voice) | $0.12–$0.25 |

LLM passthrough at typical conversational density:

| Model | $/min |
|---|---|
| GPT-4.1 mini | ~$0.02 |
| GPT-5 / -5.1 / -5.2 | ~$0.04 |
| Claude Sonnet 4.6 | ~$0.04 |
| Llama 3.3 70B / Groq (315 t/s) | ~$0.01 |
| Llama 3.3 70B / Cerebras (1,800+ t/s) | ~$0.015 |

**(B) Customers.**
- **Domino's Pizza** — Bland AI for outbound order confirmation and missed-delivery callbacks (regional pilot 2025).
- **Sears, Better.com** — Bland AI public case studies.
- **Klarna** — OpenAI-direct GPT-class on 1.3M chats/mo with growing voice rollout.
- **TaskUs / Concentrix / Teleperformance** — BPO Vapi/Retell deployments for client outsourcing.
- **Carvana / Tonal / Chime** — typical mid-market Retell customers.
- **JetBlue / Mercury Insurance / Allstate** — enterprise Vapi pilots.
- **DoorDash** — agent-routing pilot, mix of in-house + Anthropic.
- **Notion / Ramp / Mercury** — outbound sales / SDR replacement on Vapi or Retell.
- **Hippocratic AI** (healthcare voice) — separate stack; runs in-house finetuned **Polaris 70B-class** on its own GPUs for HIPAA compliance.

**(C) Inference source.** Latency owns the architecture decision. End-to-end target 500–800ms; STT 100–200ms (Deepgram Nova / AssemblyAI), TTS 100–200ms (ElevenLabs Flash 2.5 / Cartesia Sonic), leaves **200–400ms for the LLM**. That budget drives **Groq and Cerebras as default LLM backends**: Groq Llama 3.3 70B at 315 tok/s with ~290ms TTFT; Cerebras at 1,800+ tok/s — both clear the budget where Bedrock-Claude-Sonnet at 600–800ms TTFT does not. Vapi explicitly supports BYO LLM endpoint and Groq is the most-used non-OpenAI LLM on Vapi by a wide margin. ElevenLabs Agents defaults to GPT-4o with optional swap. **Bland AI runs in-house finetuned Llama-3-70B-class on its own GPU cluster** — pitched as their differentiator (no LLM markup, sub-400ms). Hippocratic runs entirely on its own GPU cluster. Industry estimate: **Groq serves ~25–30% of voice-agent LLM tokens in May 2026**; Cerebras smaller but growing fast (won the ElevenLabs internal speed bake-off late 2025). Almost zero voice-agent volume goes through "Pro" frontier tiers (Opus 4.7, GPT-5.5) — too slow and too expensive for sub-second turn-taking.

### 8.9 Cross-cuts across the eight workloads

A few observations the per-workload data makes visible:

1. **Mistral OCR 3 at $2/1,000 pages (Dec 2025) is the clearest 2025–2026 pricing inflection in any of the eight workloads** — a 97% undercut of AWS Textract list price, with named European enterprise adopters (BNP Paribas, French government). It is concrete evidence that the 🟢 tier-down dynamic, when it triggers, moves fast.
2. **Batch is now table-stakes pricing.** 50% off list on OpenAI Batch, Anthropic Batch, Bedrock Batch, Vertex Batch. For the offline-feasible workloads (4, 18, 21, 23) this collapses economics by 2× at zero engineering cost.
3. **The OS-L gap to frontier has compressed to ~3–10× per token, but per-product COGS is what matters.** Llama 3.3 70B on Groq at $0.59/$0.79 vs. Claude Sonnet 4.6 at $3/$15 is a ~5–10× cost advantage per token; the equivalent at the meeting-summarizer per-meeting level is a 6× advantage; at synthetic-data 1B-token scale it's a 15× advantage. Vendors choose tier on whether the per-product economics force a switch — not on whether the per-token price exists.
4. **The largest enterprise inference pool by volume sits in workload 23 (batch ETL).** Bedrock + Vertex + Databricks collectively process >50% of US enterprise LLM tokens. Databricks's "up to 80% TCO savings on Llama 3.3" is a direct invitation to F500 buyers to escape platform markup.
5. **The companions workload (8) is the most-migrated of the eight** — almost no volume on frontier APIs. Drivers are non-economic (safety filters) as much as economic (per-message COGS). This is the lower bound for how far OS-L can absorb a workload when the conditions allow.
6. **Voice (25) is OS-S not OS-L because of latency, not cost.** Groq-served Llama 70B is the dominant choice precisely because the latency budget excludes Bedrock-Claude or even Together-served Llama in default config. Hardware co-location (model + STT + TTS) is the architectural variable.
7. **Frontier subsidization at the consumer chat tier is structural.** OpenAI's hundreds-of-millions/quarter free-tier cost and Jensen Huang's stated 35× hardware $/token drop in two years are the reason short Q&A has not migrated despite Sh→Med reasoning being sufficient. The migration here is gated by whether subsidized frontier free tiers continue to exist.

---

## 9. Long-Horizon Agentic Workloads at Sh–Med Reasoning — An Emerging Class

**Q: Are there any long-horizon / agentic workloads that run at Shallow or Medium reasoning, not Deep?**

Yes, but the master table under-represents them. The dominant cultural picture of "agent" in 2026 is Claude Code / Cursor Agent / Devin / OpenAI Operator — all Deep / frontier. The Sh–Med agentic class is real and rising in token volume, and most of it is invisible because the workloads either (a) sit inside the Heavy-tool-use cells of existing rows but at a sub-flavor the row doesn't name, or (b) operate as *worker sub-agents* dispatched by a frontier planner, where the worker layer is what consumes the bulk of the tokens.

> The picture: **Deep agents are visible because they ship as branded products. Sh–Med agents are invisible because they ship as glue or as worker layers.** Token volume in 2026 is increasingly the invisible kind.

### 9.1 What the existing table already captures

Three rows partially fit "long-horizon Sh–Med agentic," each with a caveat:

| Row | Workload | Reasoning | Tools | Tokens/sess. | Long-horizon? |
|---|---|---|---|---|---|
| 24 | Customer-service agent (text) | Med | Hv | M | No — bounded by single conversation (5–20 turns). |
| 25 | Voice agent | Sh→Med | Lt→Hv | S→M | No — single call, <10 min. |
| **27** | **Workflow / sales / RPA agent** | **Med** | **Hv** | **M→L** | **Yes** — multi-day sequences, background execution. The cleanest existing fit. |

Row 27 is the only stand-alone Sh–Med long-horizon agentic row in the catalogue. Rows 24 and 25 are agentic in the sense of heavy tool-use but are *not* long-horizon — their sessions are bounded by a single human-in-the-loop conversation.

### 9.2 Workloads not in the table that fit the Sh–Med long-horizon agentic profile

Five categories surface from public deployments and academic patterns that the master table does not yet name as stand-alone rows:

#### A. Outbound sales / SDR replacement agent

Examples: **11x.ai** (Alice, Jordan), **Artisan** (Ava), **Regie.ai**, **Lindy**, **Tessl**. Multi-day prospecting sequences: research lead → draft email → send → wait → handle reply → follow up → log to CRM. Each per-step reasoning is Sh–Med (template selection, light personalization, classification of replies); the *horizon* is days-to-weeks of asynchronous execution.

Operating point: Med reasoning, Specialized (per ICP), Single-stream per task, Prefill-heavy (CRM context + thread history), low TTFT (background), low agent-E2E (asynchronous), tokens-per-session **Long** when measured across a multi-week sequence, Heavy tools, low-medium price sensitivity. Currently deployed: mostly on Claude Sonnet / GPT-4-class with heavy templating; a meaningful share of the per-step reasoning would run on Llama-3.3-70B if buyer trust permitted.

#### B. Narrow code agent (test generation, doc generation, mechanical refactor)

Examples: **Qodo** (formerly Codium AI), **CodiumAI test gen**, **Tabnine refactor mode**, **Continue.dev narrow agents**, **GitHub Copilot Workspace tasks**, **Sweep.dev**, **Mentat**. Tasks: "generate unit tests for this module," "bump dependency X across the repo and fix breakage," "regenerate docs from these signatures," "convert this class from JS to TS."

These are agentic (multi-turn, repo-scoped tool use, file-edit loops) but the per-step reasoning is mechanical relative to a Cursor Agent or Claude Code session. They are sub-flavors of Row 12 that the catalogue collapses into the Deep label because the dominant volume in Row 12 is the long-horizon Claude Code use case. **Operating point**: Med reasoning, General (code), Single-stream, Prefill-heavy, low TTFT, low agent-E2E, **Medium** tokens-per-session (10K–50K, an order of magnitude shorter than Claude Code), Heavy tools, medium price sensitivity. Most-deployed today: **Fr (Sonnet 4.6 / GPT-5)** but the workload is the canonical "Med-required-Fr-deployed" code subcategory — a 🟢-equivalent for the agentic half of the table.

#### C. Hierarchical worker / sub-agent in multi-agent systems

Pattern: a frontier-tier **planner** decomposes a task and dispatches sub-tasks to **worker** sub-agents. The planner is Deep / Fr; the workers are Sh–Med / OS-L (or even OS-S). Examples of frameworks where this pattern is now dominant: **Anthropic multi-agent research patterns** (the agent-team SDK), **OpenAI Swarm** and its successor-frameworks, **LangGraph hierarchical**, **CrewAI**, **AutoGen worker layer**, **MetaGPT engineer/architect/QA roles**. Production examples: deep-research products that fan out to dozens of parallel worker calls (each does one search + one summary, planner aggregates); code agents that spawn worker sub-agents per file; customer-service systems that dispatch a Med worker per ticket-classification step.

Operating point: depends on the layer. **The worker layer**: Sh–Med reasoning, General, **Massively parallel** (many workers fan out concurrently), Prefill-heavy (worker re-ingests its slice of context), low TTFT, **High agent-E2E** (planner is blocked on workers), Medium tokens per worker call but **Long when summed across the trajectory**, Heavy tools, medium-low price sensitivity (worker token volume is what drives the COGS bill).

This is the architectural pattern that creates the largest hidden Sh–Med token pool. The user-visible product is "Claude Opus did my research"; the actual token economics are dominated by ~20 OS-L worker calls. Not yet a stand-alone row in the master table, but arguably the single most important addition for v2.

#### D. Data ops / ETL agent

Examples: **Hex Magic**, **Mode AI**, **Sigma AI**, **Airbyte AI**, **Fivetran AI**, **dbt Copilot**, **Cube AI**, **Snowflake Cortex Agent**. Tasks: "build me a pipeline that joins these three tables and computes weekly active users," "find anomalies in last week's load," "summarize what changed in this dbt model." Multi-step (introspect schema → write SQL → run → validate → present), heavy tool use (database, dbt, dashboarding APIs), background or near-interactive.

Operating point: Med reasoning, Specialized (per data platform), Small-batch concurrency, Prefill-heavy (schema + sample data + user goal), low-medium TTFT, low agent-E2E, **Medium-Long** tokens-per-session (multi-step plans + tool I/O), Heavy tools, medium price sensitivity. Most-deployed today: Fr (Claude Sonnet, GPT-5) on Bedrock or direct API; OS-L deployments emerging on Databricks FM Serving and Snowflake Cortex.

#### E. DevOps / SRE / ops agent

Examples: **Cleric**, **Komodor AI**, **Resolve AI**, **Lightlytics AI**, **Datadog Bits AI**, **PagerDuty AIOps**, plus internal "auto-remediation" agents at hyperscalers. Tasks: "investigate this alert," "open a PR to bump the failing dependency," "tune autoscaling on this service," "summarize the last 24h of incident channel into a postmortem draft." Multi-step (logs → metrics → traces → infer cause → propose action → execute or open PR), heavy tools (Kubernetes API, cloud SDKs, observability stack), continuous background execution.

Operating point: Med reasoning, Specialized (per infrastructure stack), Small-batch, Prefill-heavy, low-medium TTFT, medium agent-E2E (incidents have time pressure but minutes are tolerated), **Medium-Long** tokens-per-session, Heavy tools, low-medium price sensitivity (replaces an SRE on-call shift). Most-deployed today: Fr direct API; this is one of the workloads where OS-L self-hosted is most appealing because the *infra context itself is sensitive* (auth tokens, infra topology) and many operators do not want to ship it to a frontier API.

### 9.3 The hierarchical pattern is reshaping the agent token mix

Rows 12, 16, 26 in the master table currently carry a Deep-only label because the *user-visible product* runs on a frontier planner. But once you decompose those products into planner + worker calls, the worker layer is materially Sh–Med and represents the bulk of the token volume. Public anchors:

- **Anthropic's research-agent walkthrough** (2025) explicitly describes a planner-orchestrator pattern with worker sub-agents handling each search/read/summarize subtask.
- **OpenAI Deep Research** is publicly described as a multi-step plan with web browsing and reading sub-calls; per [OpenAI's own communication](https://openai.com/index/introducing-deep-research/), a single Deep Research run can issue dozens of tool calls and many minutes of model time — the breakdown by reasoning tier is not published, but the architecture clearly has a Med worker layer.
- **Cursor Agent and Claude Code** have begun mixing "small model" calls inside the loop for line-search, file-read planning, and patch-formatting subtasks (see [Cursor on dynamic context](https://www.infoq.com/news/2026/01/cursor-dynamic-context-discovery/)) — the worker calls are Sh–Med even when the planner is Opus 4.7.

This pattern means a meaningful share of the *Deep-labeled* rows in the master table is already an OS-L worker pool wearing a frontier brand. The token-volume share that is genuinely frontier-required is smaller than the table's Deep cells imply.

### 9.4 Suggested table additions for v2

If the master table is regenerated, three new rows would be defensible:

| Proposed row | Reasoning | Specializ. | Concurrency | P:D | TTFT | Agent E2E | Tokens/sess. | Tool use | Price sens. |
|---|---|---|---|---|---|---|---|---|---|
| Outbound SDR / sales agent | Med | Spec | SS | Pf | L | L | L | Hv | M |
| Narrow code agent (test/doc/refactor) | Sh→Med | Gen | SS→SB | Pf | L | L | M | Hv | M |
| Hierarchical worker sub-agent | Sh→Med | Gen | **MP** | Pf | L | **H** | M (per worker) | Hv | M→L |
| Data-ops / ETL agent | Med | Spec | SB | Pf | L | L | M→L | Hv | M |
| DevOps / SRE agent | Med | Spec | SB | Pf | L | M | M→L | Hv | L→M |

The hierarchical-worker row is the most distinctive. It is the only proposed row that pairs **massively-parallel concurrency** with **high agent-E2E latency sensitivity** — workers fan out concurrently and the planner blocks on the last-finishing worker, which makes tail latency the binding constraint. No row currently in the table has that fingerprint.

### 9.5 Why the current table buries this class

Two reasons. First, public usage data (Anthropic Economic Index, OpenAI NBER, Menlo) categorizes agents by *user-visible product* (Claude Code, deep research, customer service), not by the internal planner/worker decomposition. So the public reports we drew from in §3 don't surface this class. Second, "agentic" in the cultural picture of 2026 is synonymous with Deep frontier — the public benchmark suite (SWE-bench, GAIA, OSWorld) measures the planner's success, not the workers' efficiency. The tooling catalogue (LangGraph, AutoGen, MetaGPT, CrewAI) names the pattern but doesn't quantify the token mix.

Both gaps are tractable: the worker-layer share is in principle measurable from any large agent product's API logs. No public number found as of May 2026.

---

## Sources

**Usage and adoption (market lens):**
- [Anthropic Economic Index — Sept 2025](https://www.anthropic.com/research/anthropic-economic-index-september-2025-report)
- [Anthropic Economic Index — Jan 2026, "Economic Primitives"](https://www.anthropic.com/research/anthropic-economic-index-january-2026-report)
- [Anthropic Economic Index — Mar 2026, "Learning Curves"](https://www.anthropic.com/research/economic-index-march-2026-report)
- [Anthropic Economic Index — arXiv 2511.15080](https://arxiv.org/abs/2511.15080)
- [OpenAI / NBER WP 34255 — "How People Use ChatGPT" (Sept 2025)](https://www.nber.org/papers/w34255)
- [OpenAI — State of Enterprise AI 2025](https://openai.com/index/the-state-of-enterprise-ai-2025-report/)
- [a16z — Top 100 Gen AI Consumer Apps, 6th Ed. (Mar 2026)](https://a16z.com/100-gen-ai-apps-6/)
- [a16z — State of Consumer AI 2025](https://a16z.com/state-of-consumer-ai-2025-product-hits-misses-and-whats-next/)
- [a16z — How 100 Enterprise CIOs Are Buying AI 2025](https://a16z.com/ai-enterprise-2025/)
- [a16z — State of Generative Media 2026](https://a16z.com/the-state-of-generative-media-2026/)
- [Menlo Ventures — 2025 State of Generative AI in the Enterprise (Dec 2025)](https://menlovc.com/perspective/2025-the-state-of-generative-ai-in-the-enterprise/)
- [Stanford HAI — AI Index 2026, Economy chapter](https://hai.stanford.edu/ai-index/2026-ai-index-report)
- [REALM — Real-World LLM Use Cases (ACL Findings 2025)](https://aclanthology.org/2025.findings-acl.437/)
- [LMSYS — Chatbot Arena Hard Prompts category](https://www.lmsys.org/blog/2024-05-17-category-hard/)

**Inference operating-point traces and serving systems:**
- [DistServe (arXiv 2401.09670, OSDI '24)](https://arxiv.org/abs/2401.09670)
- [Mooncake (arXiv 2407.00079, FAST '25 Best Paper)](https://arxiv.org/abs/2407.00079)
- [Splitwise (arXiv 2311.18677, ISCA '24)](https://arxiv.org/abs/2311.18677)
- [Vidur (arXiv 2405.05465)](https://arxiv.org/abs/2405.05465)
- [BurstGPT (arXiv 2401.17644)](https://arxiv.org/abs/2401.17644)
- [LMCache (arXiv 2510.09665)](https://arxiv.org/abs/2510.09665)
- [DuetServe (arXiv 2511.04791)](https://arxiv.org/abs/2511.04791)
- [KVCache in the Wild (arXiv 2506.02634)](https://arxiv.org/abs/2506.02634)
- [Azure Public LLM Inference Datasets](https://github.com/Azure/AzurePublicDataset)
- [SemiAnalysis — Coding Assistant Breakdown](https://newsletter.semianalysis.com/p/the-coding-assistant-breakdown-more)
- [SemiAnalysis — InferenceMAX](https://newsletter.semianalysis.com/p/inferencemax-open-source-inference)

**Agent and tool-use prevalence:**
- [LangChain — State of Agent Engineering 2025](https://www.langchain.com/state-of-agent-engineering)
- [Berkeley Function Calling Leaderboard V4](https://gorilla.cs.berkeley.edu/leaderboard.html)
- [MCPMark (arXiv 2509.24002)](https://arxiv.org/abs/2509.24002)
- [τ-bench](https://github.com/sierra-research/tau-bench)
- [GAIA](https://arxiv.org/pdf/2311.12983)
- [WebArena](https://webarena.dev/)
- [OSWorld](https://os-world.github.io/)
- [OSWorld-Human (arXiv 2506.16042)](https://arxiv.org/html/2506.16042v1)
- [Anthropic — Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic — Advanced Tool Use](https://www.anthropic.com/engineering/advanced-tool-use)
- [Anthropic — SWE-bench Sonnet](https://www.anthropic.com/research/swe-bench-sonnet)
- [Cursor Dynamic Context (InfoQ Jan 2026)](https://www.infoq.com/news/2026/01/cursor-dynamic-context-discovery/)
- [MCP Adoption Statistics 2026](https://mcpmanager.ai/blog/mcp-adoption-statistics/)

**Voice agent latency:**
- [Hamming AI — Voice AI Latency](https://hamming.ai/resources/voice-ai-latency-whats-fast-whats-slow-how-to-fix-it)
- [Twilio — Core Latency in AI Voice Agents](https://www.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents)
- [Retell AI — Voice Agent Latency Face-Off 2025](https://www.retellai.com/resources/ai-voice-agent-latency-face-off-2025)

**Frontier API and serving pricing (Section 8):**
- [OpenAI API Pricing](https://openai.com/api/pricing/)
- [GPT-5.5 pricing (Apr 2026)](https://apidog.com/blog/gpt-5-5-pricing/)
- [Anthropic Claude pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- [Claude Opus 4.7 pricing analysis](https://www.finout.io/blog/claude-opus-4.7-pricing-the-real-cost-story-behind-the-unchanged-price-tag)
- [Gemini API pricing 2026](https://ai.google.dev/gemini-api/docs/pricing)
- [Together AI pricing](https://www.together.ai/pricing)
- [Fireworks AI pricing](https://fireworks.ai/pricing)
- [Groq pricing](https://groq.com/pricing)
- [Cerebras pricing & speed](https://tokenmix.ai/blog/cerebras-api-key-access-speed-tests-2026)
- [DeepInfra pricing](https://deepinfra.com/pricing)
- [DeepSeek pricing](https://api-docs.deepseek.com/quick_start/pricing)
- [Llama 3.3 70B providers comparison — Artificial Analysis](https://artificialanalysis.ai/models/llama-3-3-instruct-70b/providers)
- [OpenRouter revenue / Sacra](https://sacra.com/c/openrouter/)
- [OpenRouter Chinese models 61% market share](https://dataconomy.com/2026/02/25/chinese-ai-models-hit-61-market-share-on-openrouter/)
- [AWS Bedrock pricing 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)
- [Vertex AI batch 2026 pricing](https://tokenmix.ai/blog/vertex-ai-pricing)
- [Snowflake Cortex pricing](https://yukidata.com/snowflake-cortex-pricing/)
- [Databricks Foundation Model Serving](https://www.databricks.com/product/pricing/foundation-model-serving)
- [Databricks Llama 3.3 80% savings](https://www.databricks.com/blog/making-ai-more-accessible-80-cost-savings-meta-llama-33-databricks)

**Consumer products / customer references (Section 8):**
- [Character.AI revenue and usage 2026 — Business of Apps](https://www.businessofapps.com/data/character-ai-statistics/)
- [Replika subscription pricing 2026](https://help.replika.com/hc/en-us/articles/39551043419149-Choosing-a-Subscription)
- [ChatGPT subscription decline projections](https://www.wheresyoured.at/openai-projects-chatgpt-plus-subscriptions-to-drop-by-80-from-44-million-in-2025-to-9-million-in-2026-made-up-using-cheaper-subscriptions-somehow/)
- [Mistral Le Chat](https://mistral.ai/news/all-new-le-chat)
- [Klarna AI assistant case study (OpenAI)](https://openai.com/index/klarna/)
- [Klarna walking back AI customer service](https://www.customerexperiencedive.com/news/klarna-reinvests-human-talent-customer-service-AI-chatbot/747586/)
- [Perplexity Bedrock case study](https://aws.amazon.com/solutions/case-studies/perplexity-bedrock-case-study/)

**Translation (Section 8):**
- [DeepL API plans](https://support.deepl.com/hc/en-us/articles/360021200939-DeepL-API-plans)
- [RWS Language Weaver Pro launch](https://www.businesswire.com/news/home/20260325603562/en/RWS-Launches-Language-Weaver-Pro-the-Worlds-Most-Advanced-AI-Translation-Solution)
- [Smartling AI translation tools 2026](https://www.smartling.com/blog/ai-translation-tools)

**Voice agents (Section 8):**
- [Vapi pricing](https://vapi.ai/pricing)
- [Retell AI voice agent pricing 2026](https://www.retellai.com/blog/ai-voice-agent-pricing-full-cost-breakdown-platform-comparison-roi-analysis)
- [Bland AI pricing 2026](https://callbotics.ai/blog/bland-ai-pricing)
- [Synthflow pricing](https://synthflow.ai/pricing)
- [ElevenLabs Agents pricing](https://elevenlabs.io/pricing/agents)

**IDP and document understanding (Section 8):**
- [Reducto pricing](https://reducto.ai/pricing)
- [Reducto Series A](https://reducto.ai/blog/reducto-series-a-funding)
- [AWS Textract pricing](https://aws.amazon.com/textract/pricing/)
- [Mistral OCR 3 launch — VentureBeat](https://venturebeat.com/technology/mistral-launches-ocr-3-to-digitize-enterprise-documents-touts-74-win-rate)
- [Mistral OCR 3 technical review](https://pyimagesearch.com/2025/12/23/mistral-ocr-3-technical-review-sota-document-parsing-at-commodity-pricing/)
- [Hyperscience insurance solutions](https://www.hyperscience.ai/solutions/insurance/)
- [Rossum platform](https://rossum.ai/)
- [LlamaParse pricing](https://www.llamaindex.ai/pricing)
- [Allianz AI claims case — Emerj](https://emerj.com/artificial-intelligence-at-allianz-two-use-cases/)

**Note-takers / summarizers (Section 8):**
- [Granola pricing teams](https://www.granola.ai/blog/granola-pricing-teams-per-user-enterprise)
- [Fireflies vs Otter (Sybill comparison)](https://www.sybill.ai/blogs/fireflies-vs-otter-ai)
- [Gong $300M ARR](https://techcrunch.com/2025/03/05/revenue-prediction-startup-gong-surpasses-300m-arr-indicating-potential-ipo-path/)
- [NotebookLM Gemini model tiers 2026](https://gemilab.net/en/articles/gemini-basics/notebooklm-pro-advanced-ultra-gemini-model-comparison-2026)

**Synthetic data and ETL (Section 8):**
- [Snorkel AI Foundry — VentureBeat](https://venturebeat.com/business/snorkel-ai-introduces-new-foundation-model-data-platform-to-bring-the-power-of-programmatic-data-development-to-generative-ai)
- [NVIDIA acquires Gretel](https://www.gretel.ai/)
- [Synthetic data in LLMs — Interconnects](https://www.interconnects.ai/p/llm-synthetic-data)
- [Intercom Fin pricing](https://minami.ai/blog/intercom-fin-ai-agent-pricing)
- [Zendesk vs Intercom AI pricing 2026](https://myaskai.com/blog/zendesk-ai-intercom-ai-comparison-2026)

---

*Document compiled 2026-05-01.*
