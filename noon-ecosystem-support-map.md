# noon-ecosystem-support-map.md

**Written:** 2026-04-23 Thursday end-of-session (Leger strategy, today).
**Status:** This is the strategic map — which organizations can amplify Leger, in what order, via what mechanism. Peer to `noon-scoping-policy-handoff.md`.
**Source:** Tom's own research, verbatim below. Orchestrator additions at §End: Cloudflare-maximalist overture, HuggingFace direct contact, solo-dev-dogfood blueprint variant, Discord setup, accelerator-sequencing doctrine.

---

# Leger — ecosystem support map

Who can help you, ranked by fit and near-term actionability. Research current to April 2026.

## TL;DR

You sit at a rare intersection: **AMD Strix Halo** (a platform AMD is actively promoting), **Framework Desktop** (a hardware company whose community is literally the Strix Halo home-lab scene), and **on-prem local AI for small business** (the loudest unsolved problem in the AI-for-SMB conversation right now). Four organizations have clear, actionable programs that map to what you're doing: AMD, Framework, Cloudflare, and Anthropic. Two tiers below that, there's a dense grassroots community (Strix Halo Home Lab, Lemonade) that is the right first audience. One tier below that is federal (SBIR/ARPA-H) — real money but slow, worth considering only when Sodimo is a year old and you have outcomes data.

The move is to stack three or four of these in parallel, not pick one. They're non-exclusive and most of them are explicitly designed to be stackable.

## Tier 1 — programs you should apply to now

### AMD AI Developer Program

- **What it is.** Free, no-hardware-required membership. amd.com/en/developer/ai-dev-program.html. Step-by-step application at developer.amd.com.
- **What you get immediately.** $100 AMD Developer Cloud credits (MI300X access, DigitalOcean-backed), private Discord with AMD engineers, AI Academy courses, one month complimentary DeepLearning.AI Pro, early invites to AMD events. Monthly hardware sweepstakes for Radeon GPUs / Ryzen AI PCs.
- **What you get if you engage.** The real value is the "Showcase Opportunities" path: *"Submit your project to ai_dev_program@amd.com. If approved, an AMD AI Developer Program manager will be in contact with you to get you highlighted on AMD Developer | LinkedIn."* That's the Tetramatrix playbook.
- **Why Leger fits.** Strix Halo on Framework Desktop running local AI for small business is close to a showcase-case-study-by-construction. Nobody on the public AMD blog is telling *this* story — the managed on-prem deployment story for non-technical SMBs. AMD has Lemonade, GAIA, Sorana, and Dayhoff Health (genomic analysis on Radeon AI PRO 9700) as their current narrative surfaces. You'd be the "small-business ops" piece.
- **Specific event to target.** **AMD AI DevDay 2026, San Francisco, April 30, 2026** (developer.amd.com event listing — note this is 7 days from now as of writing). *"AMD engineers, technical leaders, ecosystem partners, open-source contributors, and AI developers."* If you're in SF or can get there on short notice, this is the room.
- **Action.** Apply today. Join the Discord. Start posting in the Lemonade-relevant channels. Submit Leger to ai_dev_program@amd.com once you have a public landing page.

### Framework Computer — no formal program, but an unusually active community channel

- **What exists formally.** Framework has **no partner program** and has said so explicitly (community.frame.work/t/msp-and-partner-program/8203, 2021, no subsequent change visible). Their investor list includes Spark Capital, Buckley Ventures, Anzu Partners, Cooler Master, Pathbreaker Ventures (frame.work/blog/frameworks-series-a-1-and-community-participation). Cooler Master is a strategic-partner-turned-investor — shipping products in the Framework ecosystem is how you become a partner, not the other way round.
- **What exists informally — the real story.** Framework is **directly shipping hardware to community members doing Strix Halo AI work.** Evidence:
  - *"I built a 2-node Strix Halo/Framework Desktop setup using RDMA (RoCE/Intel E810) and vLLM tensor parallelism... Thank you Framework for providing the two boards and one of the cards!"* — Feb 2026, community.frame.work/t/80482.
  - *"Thanks to support from the Strix Halo Home Lab community, Framework, and AMD, I've continued to maintain these 'Toolboxes'..."* — Marco (kyuz0) of the Strix Halo Toolboxes project, strix-halo-toolboxes.com.
  - *"I was given the go ahead a while back to mention that the testing has been supported by AMD and Framework"* — AMD Strix Halo GPU LLM Performance Tests thread, community.frame.work/t/72521.
- **What this means.** Framework's support pathway is: build something visible on a Framework Desktop, share it in the community forum, and the team will send you boards if what you're doing amplifies the platform. There's no application form because the signal *is* the contribution.
- **Why Leger fits.** You're already using a Framework Desktop as your reference hardware. The Sodimo engagement is exactly the kind of case study that Framework's forum would boost. "Lone engineer deploys full local-AI + mail + CRM stack on Framework Desktop for a real small business, 3-month handoff" is better narrative material than most of what's in the community already (which tilts toward benchmarks and hobbyist clusters).
- **Action.**
  1. Start posting in community.frame.work under "Framework Desktop" when you have a concrete milestone — first working bootc image, first Sodimo production cutover, your hardware-profile detection service.
  2. Target the same community thread shape as kyuz0 and Marco: documented, reproducible, useful to other people.
  3. Once visible, email framework-desktop@frame.work or DM Nirav Patel on LinkedIn with a specific ask — e.g., "I'd like a second board to prove a failover configuration for SMB deployments" — grounded in the work already visible.

### Cloudflare for Startups + Workers Launchpad

Two stacked programs — apply to both.

#### Cloudflare for Startups (the credits program)

- **What it is.** Up to **$250,000 in Cloudflare credits over one year**, across Workers, R2, Durable Objects, Zero Trust, Cloudflare Access, Turnstile, Workers AI, and more. blog.cloudflare.com/startup-program-250k-credits, blog.cloudflare.com/startup-program-v2.
- **Four tiers.** $5,000 / $25,000 / $100,000 / $250,000. Base tier doesn't require VC backing. Higher tiers require you to be in a "Tier 1 VC and accelerator network" or in Workers Launchpad.
- **Bootstrapped path.** If you haven't raised $50k, apply with promo code **BOOTSTRAPPED** for a lower-credit tier. Explicitly called out by Cloudflare.
- **Eligibility.** Building a software product/service with an active website, not already a Cloudflare contract customer, and (for higher tiers) <$5M raised and currently in or recently graduated from an accelerator.
- **Rolling deadline** — no cohort cycle; applications typically reviewed within ~2 weeks.

#### Workers Launchpad (the quarterly cohort program)

- **What it is.** Quarterly-cohort program layered on top of the Startup Plan. ~$2B potential funding pool across 40+ VC partners. Cohort #7 applications closed March 13, 2026; Cohort #8 should open in the next review cycle — **apply now regardless, late apps roll forward**.
- **Eligibility.** Currently using Workers or other Developer Platform components, funded up to Series B, founded in the last 5 years.
- **Real-world filter (per Cohort #2 founder interview).** No hard headcount/ARR bar — ELYD got in at 2 people, zero revenue, deep tech. The actual bar is whether you *meaningfully use Workers* and will enrich the ecosystem.
- **What you get beyond the Startup Plan credits.**
  - **Bootcamp sessions** — pricing strategy, PMF, scaling sales, company-building topics with Cloudflare leadership and PMs.
  - **Solutions Architect office hours** — weekly technical guidance. Specifically called out by the founder I spoke to: *"the best part was the incredible support from Cloudflare's internal teams — hands-on migration help, rapid feedback loops, genuine partnership from engineers."*
  - **VC introductions** — warm intros from the partner network, carrying Cloudflare's endorsement. Founder described the VC list as "top tier."
  - **Demo Day** — pitch event at cohort close, streamed on Cloudflare TV.
  - **Peer network** — other cohort founders, repeatedly flagged as one of the most valuable parts.
  - **Early access** to beta Cloudflare products — Launchpad companies get to preview roadmap items with PMs.
  - **Office space access** (starting Jan 2026) — SF, Austin, London, Lisbon; Launchpad participants/alumni eligible to apply.
- **Quiet benefit: M&A pipeline.** Cloudflare uses the program as an acquisition-discovery channel. Public acquisitions from prior cohorts: **Nefeli Networks** (Cohort #2), **Outerbase** (Cohort #4). Founder caveat: most of these have been companies founded by ex-Cloudflare staff, so calibrate.

#### Why Leger fits exceptionally well

Leger's architecture *already* depends on Cloudflare: Cloudflare Tunnel is how the on-prem box reaches the internet, Cloudflare Access is the external auth perimeter, and app.leger.run (the config UI) could plausibly run on Workers + R2 + D1. The LP-XX policy pack mentions cosign-signed images, which pairs cleanly with Cloudflare's supply-chain messaging. Jade Wang runs the startup program (cloudflare.tv/birthday-week).

#### Showcase surfaces (no program, just distribution)

- **Built on Cloudflare** — official company showcase at cloudflare.com/built-with-workers. The Startup Program blog explicitly invites you: *"If you're actively using Cloudflare's Developer Platform, we'd love to hear more about what you're building and share it on our Built with Cloudflare site."*
- **Small App Garden** — newer curated showcase at developers.cloudflare.com/garden. Indie-flavored. Submit once the Sodimo + Lemonade + app.leger.run story is public.

#### Application strategy (from founder interview)

1. **Lean into Workers + R2 usage.** Baseline expectation. Demonstrate you actually build on the platform.
2. **Adopt beta/newer products.** "The newer and fresher/beta-er the better" — pub/sub, Workers AI, Workflows, Secrets. Signal ecosystem enrichment.
3. **Name-drop external validation.** NVIDIA Inception, Microsoft for Startups, paying customers, investor backing all help.
4. **Watch competitive positioning.** If your product overlaps with Cloudflare's own roadmap (Workflows, agent orchestration, AI Gateway), frame around the complementary angle, not the overlap.
5. **Be vocal in the Cloudflare Developer Discord.** Community engagement counts more than public-social presence.
6. **Accelerator / referral affiliation** gives a bonus on the Startup Plan form (dropdown: "Referred by" or your accelerator name).

#### Action

- Apply to **Cloudflare for Startups** now via cloudflare.com/forstartups/. In "what you're building," explicitly mention Cloudflare Tunnel + Access as the core of Leger's external-reachability design.
- Simultaneously apply to **Workers Launchpad** at cloudflare.com/lp/workers-launchpad/#form_workerslaunchpad.
- If Anthropic ends up introducing you, mention that — Tier 1 credit tiers open up for VC-backed/accelerator-backed companies.
- Contact: startups@cloudflare.com.

### Anthropic Startup Program

- **What it is.** Anthropic's official program for startups. claude.com/programs/startups. Free Claude API credits, priority rate limits, invitations to founder events, Anthropic team office hours.
- **Eligibility.** Must be **VC-backed by an Anthropic partner investor**. This is the sharp edge. Per Anthropic's own terms (anthropic.com/startup-program-official-terms): *"evaluation criteria... business traction, investment, and funding, as well as Claude integration and usage, but ultimate eligibility is determined in Anthropic's sole discretion."*
- **Why Leger might fit.** Leger's core runbook model (Claude Code as narration layer over the `leger` CLI) is a non-trivial Claude integration. Your LP-XX skills + the docs-are-the-prompt-corpus pattern is exactly the kind of agent pattern Anthropic wants to showcase. But without VC backing, you likely land in the "individual developer, not VC-backed" bucket — which means the Campus Club / Builder Club programs apply, which aren't your audience.
- **Alternative path if you're not VC-backed.** Anthropic's Cookbook / Skills repo (github.com/anthropics/skills) accepts community contributions. A **`leger-bootstrap-cf` SKILL.md** submitted as an example skill, or a **runbook-as-skills pattern writeup**, gets you visibility with the team without needing program membership. The Block Engineering blog post on agent skills (engineering.block.xyz/blog/3-principles-for-designing-agent-skills) is the kind of content that gets amplified by Anthropic's developer-relations team on social.
- **Action.** Don't apply to the Startup Program yet if you're not VC-backed. Instead: write a technical post about the LP-XX pattern once you've shipped Sodimo's first version. Tag Alex Albert (@alexalbert_) and Barry Zhang (@barry_zhang_) — they're the Anthropic DevRel people actively amplifying agent/skill patterns on Twitter. If that post lands, the Startup Program becomes a natural follow-up conversation.

## Tier 2 — the grassroots community (start here regardless)

### Strix Halo Home Lab

- **What it is.** strixhalo.wiki and strixhalo-homelab.d7.wtf, with a Discord server. Maintained by "deseven" and a small team. Comprehensive docs across every Strix Halo hardware platform (Framework Desktop, GMKtec EVO-X2, Beelink GTR9 Pro, Bosgame M5, Corsair AI Workstation 300, GEEKOM A9 Mega, HP Z2 Mini G1a, Minisforum MS-S1 MAX, NIMO AI MiniPC, Peladn YO1, etc.). AI guides covering clustering, RDMA, llama.cpp performance, vLLM, Kubernetes on Strix Halo. Both AMD and Framework are supporting this community (per kyuz0's toolboxes acknowledgments).
- **Why Leger fits.** This is your *first real audience* — the people who already have Strix Halo boxes and would actually try a managed deployment layer on top. They're not your customer (they're technical), but they're your **beta testers, amplifiers, and eventual FDE-network recruits**.
- **Specific contribution ideas that would land.**
  - A wiki page on "Fedora bootc for Strix Halo" — nobody has one yet; their AI guides are all Ubuntu/Fedora-standard. The closest is community.frame.work/t/75856 "AMD Strix Halo Llama.cpp Installation Guide for Fedora 42" which is useful but doesn't address bootc at all.
  - A guide on "rootless Podman Quadlets on Strix Halo" covering the GPU passthrough issues you've presumably already solved.
  - Your hardware-detection-service pattern (from RQ7 of the research) — a standalone `strix-hardware-setup` generalized from Bazzite's pattern would be widely useful.
- **Action.** Join the Discord today. Lurk for a week. Post once you have something concrete and reproducible.

### Lemonade community (AMD-backed)

- **What it is.** lemonade-server.ai, github.com/lemonade-sdk/lemonade, Discord. AMD-maintained OpenAI-compatible local inference server. "Extra optimized for Ryzen AI, Radeon, and Strix Halo PCs." Lemonade 10.1 shipped Feb 2026 with Linux NPU support finally usable (phoronix.com/news/Lemonade-10.1-Released).
- **Why Leger fits.** Direct technical opportunity from the last response: add a Lemonade Quadlet to your catalog. Positions Leger as a Lemonade integration, gets you into AMD's "apps built on Lemonade" showcase (visible on lemonade-server.ai marketplace).
- **Action.** Build the Quadlet, submit a PR to the Lemonade repo's marketplace (github.com/lemonade-sdk/lemonade — *"Want your app featured here? Just submit a marketplace PR!"*).

### uBlue / BlueBuild community

- **What it is.** universal-blue.org, docs.projectbluefin.io, blue-build.org. The bootc-for-desktop community. Not directly aligned with Leger (they do desktop, you do on-prem SMB) but they're the only group regularly shipping bootc at production-scale-for-end-users, and they'd be receptive to your findings about the Anaconda `bootc` kickstart command, hardware detection services, and yafti-on-Niri patterns.
- **Why Leger fits (indirectly).** Cross-pollination. If you open-source a piece of the harness (per the earlier note: "bootc-image-builder GitHub Action with hardware-profile detection"), the uBlue community is where it gets adopted and refined. That's how you get external contributors to Leger's infrastructure pieces without having to hire them.
- **Action.** Lower priority than Strix Halo Home Lab. Worth lurking in their Discord once you have something to share.

## Tier 3 — longer horizon, higher ceiling

### SBIR / ARPA-H

- **SBIR Phase I: up to $600k, Phase II: up to $3.5M, non-dilutive.** arpa-h.gov/explore-funding/sbir. Current status as of April 2026: "There are currently no open SBIR or STTR solicitations" on ARPA-H — check back after May 2026 cycle. NIH/AHRQ has an R21/R33 program targeting AI-based clinical decision support and workflow optimization for hospitals/clinics (socialroots.ai/grant-success-stories/nih-ahrq-digital-healthcare-solutions-grants-2026).
- **Why Leger fits on a 1-2 year horizon.** The expected 2026 HIPAA Security Rule update *"removes the addressable designation for encryption and tightens requirements around asset inventories, network maps, and breach timelines. Those requirements are easier to satisfy when the GPU doing the inference is in a rack the hospital owns"* (histalk2.com, Apr 22 2026). If Sodimo or a future customer is a clinic, ambulatory surgery center, or specialty practice, Leger's "borrow HIPAA vocabulary and shape" positioning aligns with exactly what SBIR/ARPA-H wants to fund — "on-prem AI that meets the 2026 Security Rule."
- **Why not now.** You need outcomes data first. SBIR reviewers will ask "how many deployments, what's the clinical or operational outcome, who's the PI." Sodimo is week 1 of that answer.
- **Action.** File it as a 12-month-from-now item. Start collecting Sodimo outcome data with SBIR-shaped metrics in mind: time-to-deploy, FTE hours saved per month, cost-vs-cloud, incident count, rollback count. When ARPA-H publishes its next solicitation (subscribe to Vitals Newsletter), you'll be the person who already has the data.

### Fal, Modal, Groq startup programs

- Lower fit. These are cloud-inference platforms; Leger is on-prem-first. Don't apply unless you end up with a hybrid-cloud story where Leger falls back to Groq/Modal for specific workloads.

## Tier 4 — think twice before applying

### Google for Startups, Microsoft for Startups, AWS Activate

- Real credits ($5k–$200k+), but every generic SaaS startup is in these programs. They won't amplify you the way AMD or Framework would, and they don't have an on-prem narrative to attach to.
- Worth $5k-$25k of AWS credits to run Leger-the-SaaS (app.leger.run, leger-docs builds, CI) if you want to save cash. Not a strategic bet.

## The one-page action plan

**This week:**
1. Apply to AMD AI Developer Program (amd.com/en/developer/ai-dev-program.html). Activate $100 cloud credit.
2. Apply to Cloudflare for Startups (cloudflare.com/forstartups/) **and** Workers Launchpad (cloudflare.com/lp/workers-launchpad/). Mention Cloudflare Tunnel + Access as core architecture.
3. Join Strix Halo Home Lab Discord (link from strixhalo.wiki).
4. Join Lemonade Discord (github.com/lemonade-sdk/lemonade).
5. Join the Cloudflare Developer Discord (discord.com/invite/cloudflaredev) — visibility there matters for future Launchpad selection.
6. If in SF on April 30: attend AMD AI DevDay. If not, follow the post-event recap.

**This month:**
7. Ship one public artifact under a Leger Labs GitHub org that's genuinely useful to the Strix Halo community independently of Leger. Candidate: `strix-hardware-setup` — a generalized hardware-profile detection service for bootc on Strix Halo. Post it in community.frame.work and strixhalo-homelab.
8. Add a Lemonade Quadlet to Leger's catalog. Submit a marketplace PR to lemonade-sdk/lemonade so Leger appears on lemonade-server.ai.
9. Write a technical post: "On-prem AI for small business on Framework Desktop: what we learned deploying Sodimo" — published once Sodimo is in production. Tag @amdradeon, @FrameworkPuter, @alexalbert_, @barry_zhang_.

**When Sodimo is 3+ months in and you have data:**
10. Submit Leger to ai_dev_program@amd.com for LinkedIn/blog feature.
11. Email framework-desktop@frame.work with a specific hardware ask grounded in forum visibility you've built.
12. Submit Leger to the "Built on Cloudflare" showcase and Small App Garden (developers.cloudflare.com/garden).
13. If the Claude Code narration layer is working and demoable, pitch Anthropic's DevRel via Discord / Twitter as a skill/agent pattern showcase.

**When you have 3+ customers and a year of data:**
14. Evaluate SBIR/ARPA-H. Check for solicitations targeting on-prem healthcare AI infrastructure.
15. Evaluate VC — at which point the Anthropic Startup Program and higher Cloudflare credit tiers become accessible.

## One framing point to close on

Tetramatrix's Sorana is being amplified by AMD because it's a consumer-ish app running on Ryzen. You occupy a different slot: **the managed deployment layer between small-business customers and local-AI infrastructure**. That slot has nobody in it right now. Framework's community is full of "here's my cluster" posts and zero "here's how I deployed this for a real business with a non-technical ops lead" posts. AMD's Lemonade marketplace has dozens of end-user apps and zero deployment-layer products. Cloudflare's startup program has thousands of SaaS companies and almost no on-prem-first ones.

This is the rare case where the right play is not to fight for attention in a crowded field — it's to show up in *multiple* ecosystems at once with a genuinely novel story. Three to four of these organizations will amplify you in parallel if you give them each one concrete thing that advances their narrative. That's the goal.

---

# Orchestrator additions (Apr 23 close-out)

The sections below are consolidation + extensions by the orchestrator, building on Tom's research above. Flagged separately so it's clear where his authoritative research ends and where the additions begin.

## §Add-A — HuggingFace as a Tier-1 partner (direct contact)

Promote HuggingFace from "Blueprint-3 partner (year-2, partnership-gated)" to a **Tier-1 named relationship** — Tom has a direct, warm contact there who can help meaningfully. Actions gated by that contact, not by a general-DevRel cold-email cycle.

- **What HF can plausibly unlock, given a warm contact.**
  - Featured HF Spaces deployment or HF blog post co-authored with Leger once there's a runnable reference blueprint (the insurance-finetune Blueprint 3 is the clearest fit, but NOT the only one — a HuggingFace-pipeline-to-Framework-Desktop pattern works independently).
  - `leger-hf-pull` skill-and-Quadlet co-promoted as the canonical pattern for "finetune on HF, serve on your own box" — makes Leger the reference integration.
  - Early access to HF AutoTrain / TRL / Spaces features that matter for on-prem serving (signed GGUF publication, cosign on HF Hub if that lands, etc.).
  - Co-marketed post on the AMD + HF + Strix Halo triangle — AMD's existing posture ("apps built on Lemonade") + HF's existing pipeline authority + Leger's deployment layer is a three-way showcase.
- **Action now:** warm-contact a pre-announcement ping describing what Leger is and what "would be mutually interesting." Don't commit to a deliverable yet — the HF conversation IS the deliverable until a concrete joint artifact is agreed.
- **Action after Sodimo first production:** propose a joint writeup template. HF contacts generally respond well to "here's a draft of what we'd publish together; tell me what's true and what isn't."
- **Nothing to apply to externally** — direct contact supersedes formal pathways. Leave the HF Startup Program etc. off the list unless Tom's contact explicitly points there.

## §Add-B — Cloudflare maximalist overture (the separate investigation Tom flagged)

Tom's note: worth doing a separate investigation once we have the full picture of "what being a Cloudflare maximalist looks like." Excluding items that are private-beta or frozen internally (e.g., fully-managed email inboxes), but including deeply-integrated ecosystem pieces like **Enterprise MCP** (blog.cloudflare.com/enterprise-mcp).

This is a deferred investigation, not a current answer. Its purpose: for every infrastructure decision Leger is about to make, ask "is there a Cloudflare product that does this, and does using it deepen the Workers-Launchpad narrative?" The answer is sometimes no — but often yes, in non-obvious ways.

Known CF surfaces Leger already uses or plans to use:
- **Workers** — `app.leger.run` backend; the MCP Worker for Sodimo (`sodimo/mcp`).
- **Pages** — `app.leger.run` frontend, `leger.run/docs`, `<customer>.leger.run/changelog`.
- **D1** — per-engagement configurator storage; eventual `run_ledger` Worker-side aggregate.
- **R2** — age-encrypted backup mirror per LP-13; potentially `leger-release-manifest` artifact hosting.
- **Tunnel** — the outbound-dial posture from every customer Framework Desktop per LP-04.
- **Access** — external auth perimeter per LP-01; gates `app.leger.run`, `<customer>.leger.run/*`, Cockpit.
- **Queues** — `email_outbox` pull-queue per LP-04 / LP-15; the symmetric-application mechanism.
- **Turnstile** — not yet deployed; candidate for `leger.run` signup gating (when signup ships).
- **Workers AI** — not yet adopted; potential escalation-path for cloud-heavy LLM inference per LP-07 (as an alternative to Anthropic when the Path-c constraint says "never make Anthropic load-bearing").
- **DNS** — per-customer zone management; partly automated via `app.leger.run` config.

Surfaces the investigation should evaluate:
- **Enterprise MCP** (blog.cloudflare.com/enterprise-mcp) — Tom's specific flag. Understand how it relates to the single-MCP-Worker-per-engagement pattern Leger already has. Is it a distribution mechanism, a governance layer, a hosted alternative? Does it subsume `sodimo/mcp` or complement it?
- **Workers AI** — can Leger's "cloud-heavy" escalation path route through Workers AI instead of Anthropic directly, preserving no-long-lived-creds + adding Cloudflare as the inference gateway?
- **AI Gateway** — is this a drop-in replacement for LiteLLM in the cloud-escalation path? Does it have token-accounting primitives that dovetail with `run_ledger` counterfactual tracking?
- **Durable Objects** — relevant for any state the configurator needs that doesn't belong in D1 (ephemeral OAuth device-grant state, GH App install-ID rotation, etc.).
- **Workflows** — long-running orchestration (per LP-03 rotation sequences, per LP-13 backup drills). Does it compete with or complement `leger bootstrap`/`leger rotate` being CLI-side?
- **Zero Trust beyond Access** — WARP client for FDEs, Browser Isolation for compliance workflows, DLP for outbound data policies.
- **Pages Functions vs Workers** — Leger's changelog renderer. Which is cheaper / better-cached / more-maintainable?
- **Hyperdrive** — irrelevant unless Leger ever needs to aggregate across customer `run_ledger.db` files (opt-in telemetry).
- **Secrets Store** — for the leger-release-manifest signing key (if signed keys ever land Cloudflare-side rather than cosign-local).
- **Images** — possibly relevant for the per-customer `<customer>.leger.run` asset-hosting story.
- **Email Workers / Email Routing** — frozen internally per Tom's note; SKIP.
- **Fully-managed inboxes** — private beta per Tom's note; SKIP.

**Investigation output** (when done, not now): a decision matrix — for each Leger architectural choice, "use CF product X" vs "use alternative Y" vs "use both / hybrid." This matrix becomes load-bearing for the Workers Launchpad application and for the Cloudflare-for-Startups eligibility tier.

## §Add-C — The solo-dev-startup dogfood blueprint (variant on Blueprint-2)

Tom's note: he could be dogfooding Leger for a solo-dev startup that also runs on Framework Desktop. This is not Blueprint-2 (Leger-Labs as its own back-office), it's a **sibling blueprint** — a separate solo-dev-startup venture Tom runs, using Leger as its infrastructure.

Why this matters:
- It's a second "no-NDA, can-publish-everything-about-it" case study, running on a second Framework Desktop — independent of Leger-Labs itself but on the same stack.
- It strengthens the "one box per customer" and "static at handoff" product doctrine with a second dogfood example.
- It amplifies the Framework Desktop + Strix Halo narrative — a solo-dev startup running its product development on a Framework Desktop is exactly the forum post that community.frame.work rewards.
- It unblocks a year-1 outbound demo that's not FMCG-shaped, countering the FMCG-typecasting risk flagged in earlier deliberation.

Structurally, this becomes:
- **Blueprint-2a: Leger Labs itself** (existing, already planned) — back-office dogfood; repos `leger-labs/leger-infra` + `leger-labs/leger-dotfiles`.
- **Blueprint-2b: Solo-dev startup dogfood** (new) — product-dev dogfood; repos `<solo-startup>/infra` + `<solo-startup>/dotfiles` under whatever the solo-startup's GitHub org is. Technical architecture is identical to any other customer; Tom is the customer.
- **Blueprint-1: Sodimo** (existing, live) — the FMCG-for-real-money engagement.
- **Blueprint-3: Insurance finetune with HuggingFace** — year-2; unlocked by the HF contact per §Add-A.

The narrative arc:
- Q2 2026: Sodimo production cutover (publish case study).
- Q2-Q3 2026: solo-startup spin-up on second Framework Desktop (publishes dogfood post in parallel).
- Q4 2026: Leger-Labs-itself back-office migration (third dogfood post).
- Q1-Q2 2027: HF + insurance blueprint.

Three independent production-grade engagements in 2026 — two of which are Tom-as-customer so there's zero NDA friction for publishing everything about them. This also means the Strix Halo Home Lab community sees THREE reproducible Framework Desktop deployments on Leger, not one.

**Sprint implication:** Block 6's 60-min Blueprint-2 seed (per `99-consolidated.md` §E) expands to seed THREE repo pairs instead of one. Still ~60 min total — git init + README x 3. The additional repos: `<solo-startup>/infra` and `<solo-startup>/dotfiles`. Tom needs to name the solo-startup for the README to be meaningful.

## §Add-D — Leger needs a Discord (decision + setup plan)

Tom's explicit: "Leger absolutely needs a Discord." Every comparable product in the landing-references file (Msty, Flowise, a dozen others) runs its community via Discord. Every Tier-1 and Tier-2 ecosystem partner in this map has a Discord Tom should be in (AMD Developer Program Discord, Strix Halo Home Lab, Lemonade, Cloudflare Developer Discord, HuggingFace Discord, uBlue). The asymmetry — joining five Discords but having none of his own — undersells Leger.

Setup plan (runbook shape, ~2 hours):

1. **Server creation.** `Leger` server. Categories: `#welcome`, `#announcements`, `#general`, `#help-deployment`, `#help-runbooks`, `#help-lp-xx`, `#framework-desktop`, `#strix-halo`, `#bug-reports`, `#feature-requests`, `#showcase` (customer-consent-gated), `#dev` (leger-labs contributors).
2. **Roles.** `FDE` (customer-facing role for Tom + future FDEs), `Customer-<slug>` (per-engagement, invite-only), `Contributor` (leger-labs GitHub contributors, auto-assigned by bot), `Community` (default).
3. **Bots.**
   - GitHub bot — posts `leger-labs/*` releases + merged PRs into `#dev` / `#announcements`.
   - A small bot that posts Leger-release-manifest version tags when they publish.
   - Anti-spam defaults (Wick or similar).
4. **Invite policy.** Public invite link on `leger.run/discord`. No gated signup — honest-by-default. Customer-specific channels are manually-assigned.
5. **Cross-linking.** Pin the `leger.run` + GitHub + docs URLs in `#welcome`. Link to the Discord from the `leger-docs` footer + `app.leger.run` footer + every `leger.run/<surface>` page.
6. **Moderation posture.** Light. Tom as sole admin at v1; add a second admin only when cohort grows beyond manual moderation.
7. **Relationship to customer-specific communication.** Discord is for public + community + FDE-prospect conversation. Customer engagement-specific comms stay in the customer's own channels (email, Signal, Slack Connect). The `#showcase` channel is opt-in; no customer talk without explicit consent.

**Action:** create the server Friday pre-launch (Block 5 or Block 6). Put the invite link on `leger.run` before any ecosystem-map action-list item ships that references Leger publicly. A Lemonade marketplace PR or a Framework forum post that doesn't link a Discord is a missed amplification.

## §Add-E — The accelerator-sequencing doctrine (implicit in Tom's ecosystem stack)

Tom's earlier framing (from `~/mecattaf/notes/agency/leger-labs/`) suggested thinking about accelerators in sequence rather than picking one. The ecosystem map above reads cleanly as that sequence — not a picked-one, but a stacked-many. The doctrine made explicit:

- **Always-on (no application, contribution-gated):** Framework community, Strix Halo Home Lab, Lemonade community, uBlue, Cloudflare Developer Discord. These reward shipping + posting.
- **Rolling-application (credits + ecosystem entry):** AMD AI Developer Program, Cloudflare for Startups. Apply now.
- **Cohort-cadence (selective, time-boxed):** Workers Launchpad. Apply now regardless of cohort timing; late apps roll forward.
- **VC-gated (needs investor in their network):** Anthropic Startup Program. Route via Cohort-cadence outcomes + investor intros.
- **Partnership-gated (needs named sponsor-inside):** HuggingFace (via Tom's direct contact — §Add-A), eventually AMD for Developer Program showcases, eventually Cloudflare for Built-on-CF / Small App Garden listings.
- **Outcomes-gated (needs a year of production data):** SBIR/ARPA-H, HIPAA-adjacent healthcare programs.

The sequence across time:
- **Weeks 1-4 (this month):** Always-on contributions + rolling applications + Workers Launchpad application + Discord server online.
- **Months 2-6:** Sodimo production + one public artifact + solo-startup dogfood cutover + HF warm-contact intro.
- **Months 6-12:** Showcases (AMD LinkedIn feature, Cloudflare "Built-on-CF", Framework forum spotlight), Anthropic community writeup, HF joint post.
- **Year 2:** VC-gated programs, SBIR/ARPA-H if a healthcare-adjacent customer lands.

The key insight: stacking 5-7 of these in parallel is realistic because they each need one concrete thing from Leger and the concrete things overlap. A single "Sodimo production cutover" post satisfies the Strix Halo Home Lab ask, the Framework forum ask, the AMD showcase ask, and the Cloudflare Built-on-CF ask, if written for all four audiences simultaneously (different angles highlighted). Reuse is the mechanism that makes the parallelism cheap.

---

*End of file. Authoritative strategic map for Leger ecosystem support as of 2026-04-23. Revisit monthly; the landscape shifts fast.*
