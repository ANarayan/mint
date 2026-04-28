# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project goal

Build a business pitch deck for a company focused on **local AI, desktop GPUs, and commoditizing AI** — making tokens "too cheap to meter." The vision: put **Aristotle-level intelligence in everyone's pocket, on everyone's desk, and in every device.**

The output of work in this repo is:

1. A **business pitch deck**.
2. **Market opportunity identification** that supports the pitch.

## What to focus on

Work here should advance one or both of:

- **Market dynamics**
  - Sizing (TAM/SAM/SOM, segment growth)
  - Competition (incumbents, near-peers, substitutes)
  - Margins (BOM, gross margin, unit economics, pricing power)
  - Why now (tech/cost/regulatory inflection that makes this possible *this year*)
  - Adoption curve / diffusion shape (10-year harvest window vs. compounding platform)
  - Regulatory & policy environment (data residency, export controls, sector compliance — central to a *local* AI thesis)

- **Business plans**
  - Buyer (who, what they pay for today, willingness-to-pay)
  - GTM strategy (channel, motion, sequencing)
  - Initial product offering (wedge SKU, what to ship first, what to skip)
  - Moat / defensibility (what compounds after hardware commoditizes — distribution, fleet/orchestration software, certifications, data)
  - Risks & kill-shots (API price collapse, NVIDIA forward-integration, cloud closing on latency/sovereignty — name them and pre-empt)
  - Roadmap & milestones (12/24/36-month financing/proof-point ladder)
  - Team / founder–market fit (hardware ops + dev-tools software + enterprise sales — who needs to be in the room)

If a piece of analysis doesn't feed the deck or sharpen one of the bullets above, it's out of scope.

## Repo layout

```
market-research/
  workstation/         — desktop GPU workstation thesis
  inference-provider/  — inference provider market
  edge-inference/      — edge inference market
```

Each subdirectory is a distinct vertical/angle on the broader local-AI thesis. Most `README.md` files are empty placeholders today; the substantive document is `market-research/workstation/gpu-workstation-business-analysis.md`.

This is a **prose/research repository** — Markdown only, no build, no tests, no tooling. Don't propose CI, scaffolding, or code infrastructure unless asked.

## Document style

Match the existing format in `gpu-workstation-business-analysis.md` when extending or adding analyses:

- **Conversational Q&A structure** — sections open with `**Q: ...**`, then an answer.
- **Numbered top-level sections** (`## 1. ...`) separated by `---`.
- **Strong, opinionated theses.** Bold one-line takeaways; blockquotes for the compressed sentence.
- **Concrete numbers and named companies** — cite specific revenue, dates, products, historical analogies. Avoid hand-wavy framing.
- **Close with `## Open Questions / Next Steps`** as a bulleted list, then `*Document compiled <date>.*`.

Use absolute dates, never relative ones.
