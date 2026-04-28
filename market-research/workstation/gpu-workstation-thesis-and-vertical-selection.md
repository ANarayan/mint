# GPU Desktop Workstation — Market Thesis & Vertical Selection

*A working conversation exploring the opportunity to build a GPU workstation business: historical analogies, the core thesis, single-vertical discipline, and the search for a beachhead.*

---

## 1. The Opening Question: Historical Corollaries to Desktop GPU Workstations

**Q: I want to understand the opportunities of doing something like a desktop "GPU" workstation. Something like selling individual GPU per employee. Are there any corollaries to previous computing revolutions.**

There are two strong historical corollaries here, and they teach almost opposite lessons. Worth knowing both because the desktop-GPU thesis lives or dies on which one is more relevant.

### Corollary 1: The Engineering Workstation Era (1982–2000)

Sun, SGI, Apollo, NeXT. This is the closest direct analogy. A new class of compute workload (CAD, 3D graphics, scientific computing, EDA) was too demanding for the PC and too latency-sensitive for time-sharing on a mainframe. So a generation of companies built premium per-seat workstations: an Apollo or SGI box on every engineer's desk at $20k–$80k a head, sold into Boeing, Pixar, Ford, Lockheed, Wall Street.

The economics were extraordinary while they lasted. SGI hit $3.7B in revenue at peak. Sun crossed $18B. The pattern was vertically integrated: proprietary CPU (MIPS, SPARC), proprietary OS (IRIX, Solaris), proprietary graphics, sold direct or through specialty VARs, with high-margin service contracts.

Then they all died — or got bought for scrap. Commodity x86 + Linux + consumer GPUs caught up on the price/performance curve in the late 90s, and the workloads bled out to PCs running Maya or AutoCAD. The vertical stack became a liability the moment the commodity stack was "good enough."

**The lesson: a desktop workstation business can be huge for 10–15 years, but the exit door slams shut when commodity hardware closes the gap.** You need to either harvest hard during the window or build a software/services moat that survives the hardware commoditization.

### Corollary 2: Mainframe → Minicomputer → PC (1965–1985)

The more famous arc. DEC took compute from the glass house to the department (PDP-11, VAX). Then Apple, Compaq, and IBM took it to the individual desk. The driving economic force was clear: when the cost of idle capacity at the edge falls below the latency and control cost of sharing, decentralization wins.

This is the optimistic case for the desktop GPU thesis. If a developer waiting 40 seconds for a cloud H100 to allocate is the new "waiting in line for the mainframe," and if a $4k box on the desk eliminates that friction, the same dynamic should play out.

### Corollary 3 (Counter-example): The Cloud Era (2006–today)

This one cuts the other way. From 2006 onward, compute *re-centralized* — into AWS, GCP, Azure. Why? Variable workloads, scale economies, undifferentiated heavy lifting, and capex avoidance. The pendulum can swing back.

A relevant data point: Lambda Labs, which built its early business selling on-premise deep learning workstations, ended its on-premise hardware business (Vector, Vector One, Vector Pro workstations and Scalar and Hyperplane servers) on August 29, 2025 to focus entirely on cloud. They concluded the per-seat hardware business wasn't where the money was.

### Current Market State

NVIDIA started shipping DGX Spark in October 2025 — a desktop AI supercomputer with 128GB unified memory delivering up to 1 petaflop, sold through Acer, ASUS, Dell, GIGABYTE, HP, Lenovo, MSI, and Micro Center, at $4,699 retail. Tinybox sits in the prosumer slot at $15k+. AMD's Strix Halo plays the commodity x86 angle. The OEMs (Dell, HP, Lenovo) are building variants on the NVIDIA reference.

### Where the Real Opportunity Lives

The pure "GPU per employee" pure-hardware play is most likely a 10-year window before commoditization, like SGI was. The compounding business is the software, services, and fleet layer that survives the hardware becoming a commodity:

- **Per-seat IT layer**: Fleet management, model/weights distribution, license tracking, security posture
- **Vertical workstation**: Pre-configured, certified, supported for a specific workflow
- **Financing/leasing model**: Recurring revenue rather than one-shot hardware margin
- **Hybrid orchestration**: Software that makes desktop GPU and cloud cluster feel like one system

---

## 2. The Core Thesis

**Q: Let's summarize what the core thing I need to do to be a successful GPU desktop workstation business.**

### The core thing: don't sell a box, sell a *seat*

Every workstation business that died (SGI, Sun, Apollo, Lambda's hardware arm) sold boxes. Every one that compounded sold something the box was attached to.

**Pick one vertical workflow, own it end-to-end, and price per seat per month — not per box.**

That means:

- **One vertical, deeply.** Not "AI developers." Pick something with three properties: data that legally or practically can't go to the cloud, a workflow with painful iteration loops today, and buyers who already pay $10k+/seat/year for tools.
- **Own the stack above the silicon.** Hardware is the wedge, not the product. The product is the certified workflow on top.
- **Recurring revenue or you're dead.** Lease the hardware, charge monthly per seat. A $15k box sold once is an SGI business. A $1,200/month seat is a compounding business.
- **Hybrid by default.** Local for sensitive/iterative work, cloud for burst. The orchestration layer is the actual moat.

> Compressed to one sentence: **be the Bloomberg Terminal of [vertical] AI workstations, not the Dell of them.**

---

## 3. Single Vertical vs. Multi-Vertical

**Q: Is it unrealistic to think we can do multiple verticals?**

Short answer: yes, at the start. Almost certainly fatal if you try.

### Why one vertical, ruthlessly, at the start

What makes a vertical workstation business defensible isn't the hardware — NVIDIA has that. It's the *depth* in the workflow: certifications, integrations to the system-of-record, pre-validated models, regulatory posture, relationships with the 50–200 buyers who actually matter, and domain-specific support.

Each is a multi-year build. A radiology sales cycle teaches you nothing about selling to chip designers. ITAR compliance doesn't transfer to HIPAA. Splitting effort means losing to focused competitors in every segment.

Historical pattern is brutal:
- **SGI** tried film + defense + scientific + web servers → died
- **Sun** spread across financial services + telco + web + scientific → died
- **Bloomberg** refused to leave finance for 20 years → unassailable position
- **Epic** refused to leave healthcare → dominant
- **Veeva** refused to leave life sciences → dominant
- **Palantir** spent first decade in intel/defense before expanding

### When multi-vertical becomes realistic

After you have a defensible position in one. Veeva is the canonical playbook: pharma CRM first, total domination, *then* clinical, *then* quality — each a separate product on the shared Vault platform. They didn't expand to vertical #2 until #1 was unambiguously won (~7 years).

### The dangerous middle path

"We'll start with two adjacent verticals to de-risk." Sounds like portfolio thinking, actually focus dilution dressed up as prudence. Two verticals isn't half the risk — it's double the work for the same revenue.

---

## 4. Vertical Selection: The Five Criteria Screen

**Q: What is the vertical that could be the beach-head? I want a vertical that:**
1. **is willing to buy (don't build in-house)**
2. **has shared commonality across companies in that sector**
3. **sales cycles are short**
4. **don't pinch on price**
5. **use AI abundantly today**

### Screening Table

| Vertical | Buys | Common | Cycle | Price | AI Use | Verdict |
|---|---|---|---|---|---|---|
| Clinical radiology/pathology | ✓ | ~ | 9–18mo | Pinches | Heavy | Fails on cycle/price |
| Biotech/drug discovery | ✓ | ✓ | 6–12mo | OK | Heavy | Strong but slow |
| EDA/chip design | ✓ | ✓ | 6–12mo | OK | Growing | Fails on #5 today |
| VFX/animation | ✓ | ✓ | 2–4mo | Pinches hard | Heavy | Fails on price |
| Quant trading | Reluctant | Varies | 1–3mo | OK | Heavy | Fails on #1, #2 |
| **Big Law** | **✓** | **✓** | **30–90d** | **No pinch** | **Heavy** | **Strong on all five** |
| Architecture/AEC | ✓ | ✓ | 3–6mo | Moderate | Growing | Fails on #5 |
| Management consulting | ✓ | ✓ | 60–90d | OK | Heavy | Strong |
| Big 4 audit | ✓ | ✓ | 6–9mo | Moderate | Growing | Borderline |
| Genomics/academic | ✓ | ✓ | Long | Pinches | Heavy | Fails on #3, #4 |

### The Pick: Big Law

Cleanest fit on all five criteria, underrated as a beachhead because most AI-infrastructure founders have no relationships there.

- **Buying culture**: AmLaw 200 firms have spent 30 years buying expensive software (Relativity, Kira, Westlaw, iManage) at $5k–$25k/seat/year. Partners aren't engineers and don't want to be.
- **Commonality**: Litigation review at Kirkland looks identical to Latham. Discovery is discovery.
- **Cycle**: 30–90 days with a partner champion. "The partner says yes, IT installs it, the firm bills it back to clients."
- **Price**: Associates billed at $800–$1,500/hr, partners at $1,500–$3,000/hr. A $5,000/seat/month tool that saves 2 hours/week pays back in less than a day.
- **AI usage**: Harvey, CoCounsel, Hebbia, Robin AI, Spellbook, EvenUp — proven the firms will buy. The gap: all run in cloud, but firms have intense data sovereignty concerns (privilege, ethics walls, jurisdictional restrictions).

**The wedge product**: per-attorney AI workstation running sensitive matter work locally — discovery review, privileged documents, deal-room contracts — with data never leaving the firm. "Harvey, but documents never leave the building." Bundle hardware + pre-configured legal models + iManage/Relativity integration + ethics-wall enforcement + 24/7 support at $30k–$60k/attorney/year.

TAM: ~50,000 partners and 150,000 associates at AmLaw 200. Multi-billion dollar market in one vertical.

---

## Open Questions / Next Steps

- Validate Big Law buying behavior with 5–10 AmLaw 200 IT/innovation leaders before committing
- Map second-best verticals (management consulting, biotech) as fallback if Big Law discovery fails
- Identify which incumbent legal-AI vendor (Harvey, CoCounsel, Hebbia) is most exposed to a sovereignty-led local play
- See companion document on agentic-developer workstations for an alternate vertical thesis with different product, GTM, and economics

---

*Document compiled April 27, 2026.*
