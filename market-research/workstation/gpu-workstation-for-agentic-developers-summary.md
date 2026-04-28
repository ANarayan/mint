# GPU Workstations for Agentic Developers — Summary & Key Takeaways

*Distilled takeaways from `gpu-workstation-for-agentic-developers.md`. The full document carries the reasoning, scenarios, and unit economics; this file is the deck-ready compression.*

---

## Key Takeaways

1. **The product is "stop paying frontier prices for `read_file`."** Cost is the thesis: 60–75% of agent calls are mechanical (file reads, edits, tests, lints) and run fine on 30B-class local models. You're arbitraging that bottom tier against API costs and routing the rest.

2. **Prosumer first: the developer is the buyer.** Day one this is sold direct to the developer — a workstation they can put on a corporate card (or buy outright near the $1k BOM) without involving procurement, IT, or security review. The buyer/user split only emerges later, in Phase 3, when 30+ devs at the same company show up on the dashboard and the CTO becomes the enterprise buyer. The deck should treat this as a *sequencing* point, not a positioning compromise: PLG into the developer, then escalate to the org.

3. **What you're actually selling is observability + cost attribution + routing.** Not "local inference." Local hardware is the wedge that earns the right to own that data — and the dashboard becomes the artifact a CTO eventually claims, which is what flips the motion from prosumer to enterprise.

4. **OpenAI-compatible endpoint is the entire integration story.** Drop-in for Cursor, Cline, Aider, Continue, Claude Code, Codex. Don't ask developers to change tools — that's how you lose to incumbents.

5. **Pain ranking matters for sales scripts.** Cost opens the deal. Reliability closes it. Rate limits are #3. Lead every conversation with cost.

6. **The corporate-card threshold is the GTM design constraint.** $250/mo lease (or ~$1k one-time) is sized to bypass procurement entirely. Break that threshold and you break the entire motion — the developer can no longer self-serve, and you're back to a 6-month enterprise cycle on day one. Cursor's $100M ARR in 12 months is the proof point.

7. **The CTO dashboard is the enterprise wedge inside a prosumer product.** Free forever, claimed with a corporate email, surfaces shadow IT *that saves money*, and quantifies value before any sales call. It's how you transition from "developer expensed it" to "company signed a contract."

8. **Two hardware paths, two business outcomes:**
   - DGX Spark resell: 40% gross margin, $101/mo contribution → SGI-trap reseller economics, 1–2x revenue multiple.
   - $1k custom BOM: 69% gross margin, $172/mo contribution → SaaS-grade margins, 5–8x revenue multiple, real venture outcome.
   - A100 DIY at $250/mo loses money on every seat. Non-starter.

9. **Don't ship custom hardware in Year 1.** Resell DGX Spark, take the 40% margin, validate the prosumer motion, gather workload telemetry. Year 2 you design the $1k box around what you actually learned. Companies that skip Phase 1 ship the wrong hardware or run out of money.

10. **Caching/prefix sharing is a 5–10x cost lever the cloud can't match.** When 50 engineers run agents over the same codebase, prompts overlap massively. Team-level KV cache is a structural advantage of on-prem that should show up prominently in the deck.

11. **Three existential risks to name and pre-empt in the deck:**
    - Anthropic/OpenAI cut API prices 50% → cost-arbitrage thesis squeezes
    - NVIDIA's stack absorbs routing → you become irrelevant
    - Hyperscalers ship native equivalent → you get out-distributed

    Window to dominant share, acquisition, or pivot: **3–4 years (2026–2029).**

---

*Document compiled April 27, 2026.*
