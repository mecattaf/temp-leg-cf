# 99 — Consolidated master synthesis (product + release shape)

**Author:** orchestrator (team-lead, `leger-scaffolding`)
**Date:** 2026-04-23 Thursday afternoon — rewritten after Tom's clean-up directive
**Scope:** product architecture + release sequence only. No pitching, pricing, revenue, or buyer discussion.
**Status of this document:** this is the **shape** of the work. The actual *decisions* and the *sequences* for externalizing Sodimo-specific material into Leger-generic repos — ending in "`leger deploy` on the immutable Linux harness which lives in the Leger repo, no longer user-specific" — have not yet been produced. This document defines the container those decisions will land in.
**Inputs:** `01-product-doctrine-final.md`, `02-policy-format-final.md`, `03-delivery-sprint-final.md`, `98-devils-advocate.md`, plus the noon-wave priming (noon-leger-progress.md, morning FINAL-SYNTHESIS.md, midday-conceptualizing-leger wave).

---

## A. Headline

Three structural decisions, converged across three Opus deliberators and a Sonnet devil's-advocate pass:

1. **Leger is three surfaces** (`app.leger.run` stateless configurator / signed `leger` CLI / public 14-LP policy pack) whose product property is a *shape* — running config IS the compliance artefact, the customer holds every long-lived credential, the vendor holds nothing runtime-critical.
2. **The LP-XX pack gets a new document shape** that CITES compiled/schema/skill rules and CARRIES only genuinely-prose rules, with `status: draft|stub|shipped` honesty on every citation, against a pinned release manifest. `0.1-controls-evidence-map.md` is the machine-runnable spine consumed by `leger doctor --map-row <n>`.
3. **The 24-hour sprint ships the release-level scaffold** — 7 new repos `git init`'d, a working demo spine end-to-end, a 30-row evidence-map prefix, LP-02 rewritten in the new shape as the reference, Blueprint-2 placeholder seeds — explicitly NOT the full externalization of Sodimo-specific → Leger-generic material. That externalization is the named gap (§D).

The devil's-advocate found three architectural under-specifications that this consolidation absorbs: the "Leger goes dark" claim needs a survivability matrix, Block 4 was 8h of work in 4h and `leger doctor --map-row` exec gets deferred to post-sprint, and Anthropic Path (c) (Claude Code optional at install time, never a load-bearing dependency) is the honest default.

---

## B. Product definition (shape, not contents)

### Mission

Leger runs a small operation's on-prem AI + back-office stack as a signed, audited, rollbackable unit — one Framework Desktop per customer, no vendor holding long-lived credentials, every LLM call ledgered against a cloud-cost counterfactual.

### Three surfaces

1. **`app.leger.run`** — the ONLY vendor-hosted surface. **Stateless for secrets.** React + shadcn SPA over the `leger.config.yaml` JSON-Schema envelope, CF Worker backend. On "apply" it emits a PR to the customer's `<customer>-infra` repo via a GitHub App; the customer merges. After merge, the customer's running stack has zero runtime dependency on `app.leger.run` being reachable. This is the architectural enforcement of the "no long-lived vendor creds" property.
2. **`leger` CLI** — signed Go binary, delivered via Cloudflare-hosted DNF repo, baked into the Leger harness image. Sprint-phase surface: `login / logout / whoami / pull / plan / apply / install-claude-code / doctor` (8 verbs). Post-sprint: `bootstrap {cf, vault, workspace, github, mail, accounts, tailscale, leger-run}`, `rotate {ssh, admin, service-token, vault-master}`, `models {list, pin, bump, verify, pull}`, `quadlets {list, status, restart, logs, upgrade}`, `report {engagement, rotation-evidence, handoff-test, mail, backup-evidence, vendor-contract}`, `changelog render`, `update`.
3. **`leger-docs`** — public Astro pack at `leger.run/docs`. **14 LPs** (collapsed from 17) in the new shape + `0.1-controls-evidence-map.md` + risk-assessment-checklist + approved-tools-snapshot + 3 appendices (upstream-bugs, silicon-guidance, skills-conventions).

### 8 product doctrine rules (architectural, not marketing)

Each rule is a constraint on the running system. They live across the four enforcement surfaces per §C. Numbered for reference from the policy pack and from this document; ordering is not priority.

1. **One box per customer.** [prose] No multi-tenant runtime. Framework Desktop is the unit. v1 is single-site only; multi-site is an unrealized blueprint, DR is a scoped re-engagement, scale-up beyond ~2× is a scoped re-engagement. This scope boundary is a required scope-section entry in `0.0-overview.md`.
2. **Static at handoff.** [prose + schema] Config frozen at handoff. Roll-forward is deliberate, not a cron job. No auto-upgrades.
3. **Symmetric application.** [compiled + prose] Local co-location is not architectural permission. Every side-effect — local or remote — routes through the same gateway and the same ledger. Load-bearing across LP-04, LP-05, LP-12.
4. **Token accounting with a counterfactual.** [compiled + schema] The `run_ledger` 14-column schema is D-179-locked. `cost_eur_if_cloud` is computed per-call. Savings is `SUM(cost_eur_if_cloud) - SUM(cost_eur)` — a SQL query against the customer's own database.
5. **Paper break-glass.** [prose] Physical envelope in a safe. No email-support-path for critical creds. The recovery path is physical and does not require any SaaS reachable, any operator alive, or any cloud online.
6. **Vendor holds no long-lived creds.** [compiled + prose] `app.leger.run` is stateless for secrets; GitHub App only emits PRs the customer merges; OAuth is a 60-second mint window; the DNF repo serves signed packages, not secrets. Pair with the survivability matrix.
7. **Conceptual fork over upstream.** [compiled] Modified upstream components are forked and pinned, never chased. Upstream changes trigger a decision, not an auto-update.
8. **No inbound ports.** [compiled] Pull-queue posture; outbound dial only. Pair with the symmetric-application rule — any "temporary" inbound opens violate both.

### Survivability matrix (required `0.0-overview.md` artefact)

The "if Leger goes dark the box keeps running" property is *architecturally real* but only if its boundaries are stated. Four known failure modes:

| Scenario | What breaks | Customer mitigation |
|---|---|---|
| OAuth provider changes device-grant behavior | `leger login` fails; can't rotate local auth | `leger login --no-browser` PAT fallback; paper-envelope recovery per LP-02 |
| GitHub App deregistered (Leger disappears, payment lapses, GH policy change) | `app.leger.run` PR-emit is dead | Existing config still applies via CLI; new config authored as hand-written YAML in customer's infra repo |
| DNF repo on CF goes away | `bootc switch` against Leger image fails; no new quadlet upgrades | Local mirror of current image; stack freezes in current state until FDE re-engages with alternate package source |
| Anthropic deprecates or changes Claude Code | Skills layer breaks; LPs (self-contained prose) are unaffected | CLI core verbs have zero Claude Code dependency (Path c, §E.3); skills migrate to plain-prompt or another CLI runtime |

This matrix is a documented architectural posture. It does not depend on vendor partnerships or favorable terms.

---

## C. The new LP-XX document shape

### Four enforcement surfaces

1. **Compiled** — quadlet field, systemd unit, CLI verb, schema validation. The rule IS the code.
2. **Prose** — out-of-band human workflow or named physical artefact. Paper envelope, DMARC ramp cadence, four-token CF split.
3. **Schema** — JSON-Schema / Zod validator enforced by `app.leger.run` at authoring time.
4. **Skill** — Claude Code narration delivered at invocation time. A delivery mechanism, not rule content. Survives Claude Code going away (the rule still exists in prose + citations; only the narration path changes).

### Per-LP file shape (one file per policy)

```yaml
---
lp: LP-02
title: Credential Vault & Recovery
status: shipped            # draft | stub | shipped — whole-LP state
surfaces: [compiled, prose, schema, skill]
surface_status:            # optional; per-surface when they diverge
  compiled: shipped
  prose: shipped
  schema: shipped
  skill: stub              # leger-labs/leger-skills deferred this sprint
verbs: [leger bootstrap vault, leger rotate vault-master]
profile_scope: [leger-smb-baseline, leger-smb-hipaa-adjacent]
external_refs: [HIPAA:164.308(a)(5)(ii)(D), HITRUST:01.d, SOC2:CC6.1, GDPR:Art.32]
---
```

Required sections:
1. **Intent** — 2–4 paragraph prose opener.
2. **Applicable Artefacts** — 6-col table: `What / Surface / Citation / Verify / Cadence / Status`.
3. **Compiled-in Rules** (conditional) — citation-only. **Paraphrase is forbidden.**
4. **Prose Rules** (conditional) — numbered, skill-readable.
5. **Schema & Profile Toggle** (conditional).
6. **Operational Commands** — `leger` verb table.
7. **Verification** — `leger doctor --<subject>` invocation.
8. **Out of Scope.**

### Citation grammar

- `@quadlet/<name>#<field>`
- `@manifest/<version>#<path>`
- `@schema/<file>#<json-pointer>`
- `@lp/<n>#<section>`
- `@skill/<name>` (will be `status: stub` for months)
- `@cli/<file>#<symbol>`
- `@decision/<D-num>`
- `@worker/<c>-core/<file>#<symbol>`
- `@cf-access/<customer>#<app>`
- `@timer/<name>`

Resolver runs at render time against the pinned `leger-release-manifest`. `status: shipped` rows must resolve against the pinned manifest (build fails on mismatch). `status: stub` rows must be on an allowlist file. Until citation-resolution CI is built (NOT in the sprint), LP-02-v2 ships with top-level `status: stub`; promotion to `shipped` is a post-sprint release gate.

### The master map — `0.1-controls-evidence-map.md`

Single flat table, one row per RULE (not per policy). 10 columns: `Rule / LP / Surface / Artefact / Verify / Cadence / Attestation / Profile / External / Status`.

- **Sprint:** 30-row prefix covering LP-02 + LP-05 + LP-10 (the three most quadlet-heavy LPs).
- **Post-sprint v0.2:** full ≈100 rows across all 14 LPs.
- Machine-runnable `Verify` column doubles as the `leger doctor --map-row <n>` test manifest. Status marker lets stubs return a graceful `skip: dependency not installed` rather than `fail`.

### 17 → 14 LP structural collapse

| LP | Verdict |
|---|---|
| LP-01 Identity & Access | rewrite |
| LP-02 Credential Vault | rewrite (sprint reference; `status: stub`) |
| LP-03 Key Rotation | keep, add frontmatter |
| LP-04 No Inbound Ports | collapse & cite |
| LP-05 Single-Surface AI Audit | rewrite (first post-sprint migration) |
| LP-06 Token Accounting | fold into LP-05 |
| LP-07 On-Prem LLM | keep, repaginate |
| LP-08 Image Provenance | merge with LP-17 |
| LP-09 Atomic OS | keep, narrow |
| LP-10 Rootless | collapse |
| LP-11 Secrets at Rest | rewrite |
| LP-12 Mail Sovereignty | keep |
| LP-13 Backup & Restore | keep |
| LP-14 Baseline Accounts | keep, deepen prose |
| LP-15 Symmetric-Application | **undecided**: demote to invariant marker (cleanest) OR keep as standalone LP (pitch-visible was the earlier argument; in a product-only framing the question becomes "does a named LP make the rule easier to enforce in CI?"). Recommendation: keep standalone because it crosses LP-04/LP-05/LP-12 and having a named file lets CI point at it. |
| LP-16 Black-Box Contract | keep |
| LP-17 Static-at-Handoff | merge into LP-08 |

Net: 14 files, same content redistributed.

### Format rollout cadence — 12 weeks

Sprint: ONE reference LP-02-v2 (`status: stub`) + 30-row evidence-map prefix. Post-sprint: LP-05 first (holds `run_ledger` schema, highest citation ROI). Then LP-08+LP-17 merge. Then LP-15 decision. Then the remaining 10 LPs in priority order driven by which LP the next deployment exercises first. Migration cadence does NOT block new content — a new LP (e.g. LP-18 finetune provenance) can ship in old shape and migrate on its own schedule.

---

## D. The externalization arc — what Tom has yet to produce

This is the named gap in this document. Everything above defines the **container**. The actual *decisions* and the *sequences of items to complete* — all the way to "`leger deploy` on the immutable Linux harness living in the Leger repo, no longer user-specific" — have not been produced.

### Today's state

- `sodimo/harness` is a Sodimo-specific bootc image (customer name in paths, customer-specific kargs pending, customer-specific repo URLs).
- `sodimo/dotfiles` holds both Sodimo-specific chezmoi (customer domains, mailbox counts, 33 mailboxes, CNIL notice) AND Leger-generic patterns (quadlet layout, user-scope systemd discipline, rootless posture).
- `sodimo/mcp`, `sodimo/etl`, `sodimo/mail` — similar mix of customer-specific and Leger-generic.
- `sodimo-dev/changelog` — already has `scope: leger | sodimo | both` field on 40 chapters; 9 leger + 14 both are the source material for the Leger-generic lift.
- `~/leger-new/` — the midday wave produced scoping docs but not extractions.
- `~/leger-labs/` — legacy reference; holds a partial `app.leger.run/` + `leger-legacy/` that was the pre-reset direction.

### Target state

- `leger-labs/harness` — generic bootc image. No customer name in paths. Customer-agnostic kargs applied via runtime hardware detection (Bazzite pattern, `/usr/libexec/leger-hardware-setup`). OS-layer only.
- `leger-labs/template-dotfiles` — Leger-generic chezmoi patterns. Customer template; customer-specific values rendered from `leger.config.yaml` at `chezmoi apply` time.
- `leger-labs/template-{mcp, etl, mail}` — parallel.
- `leger-labs/leger-docs` — everything `scope: leger` from the Sodimo manual, rewritten in the new LP-XX shape, citations resolved to Leger-generic artefacts.
- `leger-labs/leger-cli`, `leger-labs/app.leger.run`, `leger-labs/leger-schema`, `leger-labs/leger-release-manifest` — already named in §E repo scaffolds.
- `leger-labs/leger-infra`, `leger-labs/leger-dotfiles` — Blueprint-2 (Leger-Labs-as-its-own-customer).
- `sodimo/*` repos retain only Sodimo-specific content + `leger.config.yaml` pinning a Leger release.
- End-state behavior: `leger deploy` takes a `leger.config.yaml` + a pinned release manifest, produces a customer-agnostic bootc image + chezmoi tree + quadlet set, and applies them to ANY Framework Desktop without Sodimo-specific assumptions anywhere in the Leger tree.

### The sequences that have not been produced

Four sequences are required before a second customer engagement can start from `leger-labs/*` without Sodimo-specific leakage. They are not produced in this document and not produced by the midday wave.

1. **The harness extraction sequence.** What exactly moves from `sodimo/harness` to `leger-labs/harness`. Which kargs are generic (Strix Halo kargs) vs customer-specific (none that we know of today but need to confirm). Which system-scope quadlets (`tailscaled`, `cloudflared`, `cockpit`) are generic vs Sodimo-specific. What the new harness image's build pipeline looks like (mkosi + cosign + signed DNF repo). What "`leger deploy`" actually does at the harness layer.
2. **The dotfiles extraction sequence.** Decomposition of `sodimo/dotfiles` into `leger-labs/template-dotfiles` (generic) + `sodimo/dotfiles` (residual Sodimo-specific values). Templating boundary — which values become `leger.config.yaml` fields, which are fixed, which are per-profile. Interaction with the `app.leger.run` Nunjucks rendering.
3. **The LP-XX citation rewrite sequence.** Every `@lp/<n>` and `@quadlet/<name>` citation in the LP files currently points at Sodimo-manual line numbers (per policy-format v1 critique). The rewrite to Leger-generic citations (`@quadlet/vaultwarden.container` as a generic quadlet, not `sodimo-dev/changelog/src/content/manual/en/38-vault.md:21`) is real content-production work. The citation grammar is defined; the citations themselves are not yet written.
4. **The `leger.config.yaml` envelope lock.** The schema envelope `leger_config_version`, `compat`, `engagement`, `services`, `x-accounts`, `x-docs`, `x-finetunes` is named in `app-leger-run/02-schema-envelope.md` but not locked. The lock is what lets `leger deploy` be deterministic across customers. Until locked, every customer's config risks drift.

These four sequences are the work between "sprint shipped" and "second customer engagement from Leger-generic repos." They are out of sprint scope. They are in the 12-week post-sprint window and need to be produced before the Sodimo engagement closes so handoff lands on a Leger-generic harness, not a Sodimo-specific one.

---

## E. The 24-hour sprint — release milestones

The sprint purpose: ship the release-level scaffold that makes §D's externalization sequences possible. Not "produce a demo." The end-state is a set of releasable repos + one end-to-end working vertical proof.

### Six blocks (T+0 → T+24)

**Block 1 — T+0 to T+4 (Thu 14:00–18:00): Kickoff + Framework Desktop unbox + CLI scaffold**
- Cofounder onboarding with `noon-leger-progress.md` §5, `05-cli-scoper-findings.md`, `02-policy-format-final.md` §3 (evidence-map format), this document.
- `leger` CLI scaffold (cobra + `login` + `doctor` skeleton).
- USB-install stock F42 on FD; Strix Halo recognition check.
- Clone `sodimo/harness` + `sodimo/dotfiles` as a reference (not the build target).
- Claude team A: begin `app.leger.run` lift (fork `leger-labs/leger-legacy/frontend/`, strip Beam.cloud + multi-tenant D1, keep FormRenderer patterns).
- **Checkpoint T+4:** FD boots; `pnpm dev` renders; `go run . version` + `go run . doctor` print.

**Block 2 — T+4 to T+8 (Thu 18:00–22:00): CLI verbs + bootc switch + Path (c) wiring**
- `leger login` OAuth device-grant against CF Worker (`wrangler dev`).
- `leger doctor --map-row` entry point (reads YAML, prints row, exec stub — devil's fix: print-only in sprint, exec deferred).
- `bootc switch` on FD. Apply harness image. niri up.
- **Path (c) wiring** (~3h — moved here from Block 4 per close-out fix): `leger install-claude-code` scaffolded as optional-only. Verify none of `login/pull/plan/apply/doctor` imports Claude Code or breaks without it. Write a short architecture note to `leger-cli/README.md` ("Claude Code accelerates runbook narration; it is not required for stack operation"). Baseline the no-Claude path with a smoke test.
- Claude team B: draft evidence-map rows 1–15 from LP-02 / LP-05 / LP-10 content in policy-format's 10-column grammar.
- **Checkpoint T+8:** `leger login` prints JWT; FD boots niri; `install-claude-code` is optional-only and the CLI's core verbs pass smoke without Claude Code installed; evidence map ≥15 rows with real citations.

**Block 3 — T+8 to T+12 (Thu 22:00–Fri 02:00): Sleep + async agents**
- Humans sleep 4h.
- Claude team A: React component lift + basic tests.
- Claude team B: evidence-map rows 16–30 + Verify-column commands.
- Claude team C: `~/leger-new/leger-docs/` → `leger-labs/leger-docs` with Starlight; preview deploy to CF Pages.
- Claude team D: seed 5–10 synthetic `run_ledger` rows + Blueprint-2 README placeholders at `leger-labs/leger-infra` + `leger-labs/leger-dotfiles`.
- **Checkpoint T+12:** team PRs surface; review on wake.

**Block 4 — T+12 to T+16 (Fri 06:00–10:00): Wire end-to-end**
- Apply-button → CF Worker → real GH PR against `leger-labs/test-sandbox` via pre-registered App. PR body matches `04-pr-to-infra-output.md` template + includes one citation-grammar link (+15 min).
- `leger doctor --map-row` ships print-only (reads YAML, prints row, TODO comment for exec in `leger-cli#12`). Finalize the 30-row evidence-map prefix shipped by overnight teams; verify it renders in `leger.run/docs`.
- On FD: `leger pull` + `plan` + `apply` against post-merge sandbox. 3 demo rows written to `run_ledger.db`. `RENDER_TARGET=sodimo pnpm build` → localhost serve.
- Note: Path (c) wiring is NOT in Block 4 (moved to Block 2 per close-out fix). Block 4 budget is ~4h of work in 4h.
- **Checkpoint T+16:** end-to-end smoke: click Apply → PR → merge → `leger apply` → changelog shows new entry. `leger doctor --map-row 3` prints a real row.

**Block 5 — T+16 to T+20 (Fri 10:00–14:00): Polish + release-artifact assembly**
- `app.leger.run` stays on `wrangler dev` (CF Pages production deploy deferred as 2h post-sprint task).
- `leger.run/docs` goes live via team C's Starlight deploy.
- Copy LP-02-v2 sample into `leger-labs/leger-docs/` so it's browseable.
- Record a screen capture of the end-to-end flow as a release artifact (not a pitch video — a build-log style capture).
- Claude teams: broken-link scan, emergency polish.
- **Checkpoint T+20:** release artifact captured; all 5 core repos deployable.

**Block 6 — T+20 to T+24 (Fri 14:00–18:00): Blueprint-2 seed + release tag**
- T+20–T+21: Blueprint-2 repo seeds: push `leger-labs/leger-infra` + `leger-labs/leger-dotfiles` with ~3-paragraph READMEs referencing the Leger-Labs-as-customer trajectory.
- T+21–T+22: second release-artifact capture (final pass).
- T+22–T+24: tag `v0.1.0-sprint` across the 7 repos. Open the post-sprint issue list (§F + §G items below) so it's publicly trackable.

### Cut list (19 explicit items)

Shipped cuts accepted as known-deferred:
1. `leger bootstrap` verb group (cloudflare / vault / workspace / github / mail / accounts / tailscale / leger-run)
2. `leger rotate` verb
3. `leger models` subcommand group
4. `leger changelog render` as post-apply hook
5. `leger report` with canonical queries
6. Cosign signature verification anywhere
7. Multi-repo release-please fan-out
8. Real Vaultwarden quadlet on FD + paper envelope
9. Real Google Workspace super-admin provisioning
10. F43 Anaconda `bootc` kickstart (use F42 + mutate-in-place)
11. Full Claude Code skills bundle mirroring all LPs (ship ONE demo skill reference)
12. Blueprint 3 (insurance finetune) repos
13. Datasette front-end on `run_ledger`
14. `38-vault` scope flip + `leger-extract` marker-pass follow-ups in Sodimo manual
15. FR translation drift on Sodimo manual post-renumber
16. Compilation-verification pipeline integration
17. Full 100-row evidence map (30-row prefix ships)
18. CF Pages production deploy of `app.leger.run` (stays on `wrangler dev`)
19. **`leger doctor --map-row` exec (devil's fix)** — ships print-only; exec in `leger-cli#12`

### 7 new repos `git init`'d this sprint

| Repo | Purpose |
|---|---|
| `leger-labs/leger-release-manifest` | Version-tuple source of truth; cosign-signed when signing lands |
| `leger-labs/leger-docs` | Starlight pack: 14 LPs + `0.1-controls-evidence-map.md` + 5 companion artefacts |
| `leger-labs/leger-cli` | Go CLI, 8 verbs this sprint |
| `leger-labs/app.leger.run` | Web configurator (lifted from `leger-labs/leger-legacy/frontend/`) |
| `leger-labs/leger-schema` | JSON-Schema envelope per `02-schema-envelope.md` |
| `leger-labs/leger-infra` | Blueprint-2 placeholder (README only — Leger-Labs-as-customer) |
| `leger-labs/leger-dotfiles` | Blueprint-2 placeholder (README only — Leger-Labs chezmoi) |

Deferred: `leger-labs/leger-skills`, `leger-labs/blueprint-fmcg`, `leger-labs/template-changelog`, `leger-labs/template-infra`, `leger-labs/template-dotfiles`, `leger-labs/harness`, `leger-labs/template-mcp`, `leger-labs/template-etl`, `leger-labs/template-mail`, `<customer>-compliance`. The missing `leger-labs/harness` is intentional — it's the target of the §D.1 extraction sequence, not a sprint deliverable.

### External-dependency blockers (timeline-ordered)

Must pre-stage Thursday evening before the sleep block: GitHub App registration (`leger-configurator`), GH App install on `leger-labs/test-sandbox`, throwaway Google account for OAuth, Anthropic API key for both devs, CF Pages project creation. Cannot close in 24h and are acceptable to skip: RPM GPG key, cosign key pair, real Google Workspace super-admin, real Vaultwarden deployment, real DNS records beyond sprint-specific subdomains.

### Top sprint risks

1. `bootc switch` fails on FD (read-only FS bug from F42) — pre-test in a Fedora VM Thursday evening; stock F42 is the fallback.
2. OAuth device-grant flow flaky on localhost — pre-stage token in `auth.toml`; ship `--test-mode` flag that skips OAuth.
3. GH App PR-emit 403s because App isn't installed on the target repo — pre-install + `curl`-loop test Thursday night.
4. Cofounder time sink on hardware driver issues — hard hour-budget at block boundaries; if still fighting drivers at T+6, swap to a markdown task.
5. Claude overnight agent teams regress to nonsense PRs — tight per-team scope (lift these components / draft these rows), not design tasks.

### Monday-morning states

- **80%:** end-to-end works live; release artifacts captured; 7 repos tagged `v0.1.0-sprint`; `app.leger.run` on CF Pages post-sprint (2h pass); `leger.run/docs` live; FD is the primary dev machine; `leger doctor --map-row` prints rows + issue open for exec.
- **50%:** 3 of 5 demo beats land; `app.leger.run` on `wrangler dev`; CLI has 4 verbs not 8; map prefix exists but `--map-row` half-wired. Monday = write up what shipped, defer the v0.1.0 tag, triage the gaps.
- **Floor:** live end-to-end fails but capture plays back; FD runs the Sodimo changelog dual-build at localhost; 5+ repos `git init`'d. Credible release-milestone evidence; not a v0.1.0 tag.

---

## F. Tom decisions stacked (product + release only)

Pure product/release decisions. No pitch, no price, no buyer.

1. **Anthropic Path (c).** Make `leger install-claude-code` optional at install time, never load-bearing for `login / pull / plan / apply / doctor`. ~3h implementation cost in **Block 2** (moved from Block 4 per close-out fix — Block 4 was over-budget). Document in `0.0-overview.md`: "Claude Code accelerates runbook narration; it is not required for stack operation." Partnership is upside; dependency is avoidable. **Recommendation: yes, in sprint.**
2. **LP-15 Symmetric-Application fate.** Demote to an invariant marker referenced by LP-04/LP-05/LP-12 (cleaner) OR keep as standalone LP so CI has a named file to point at (more enforceable). **Recommendation: keep standalone.**
3. **LP-02-v2 `status:` on sprint day.** Ship as `status: stub` (honest — citation-resolution CI is not built) OR `status: shipped` (aspirational — CI would need to exist). **Recommendation: `stub`, promote post-sprint when CI lands.**
4. **Multi-site / DR / scale-up scope boundary.** Add to `0.0-overview.md` scope section: "Leger v1 is single-site. Multi-site is an unrealized blueprint. DR is a scoped re-engagement. Scale-up ≤2× uses `leger apply`; >2× is scoped re-engagement." Architectural honesty.
5. **Sodimo-manual renumbering ripple** — open decisions 1–6 from `99-WAVE-OVERVIEW.md` remain (`scope` enum name, `38-vault` flip to `both`, `51-the-repos` re-tag, release-fan-out picker monorepo-vs-manifest, `leger.run` signup model, `leger-labs/leger-docs` monorepo-vs-polyrepo). Signup model (#5) touches `leger login` UX and is sprint-relevant; rest are post-sprint.
6. **Externalization sequence ownership.** Each of the four sequences in §D needs an owner and a target completion week. Not decided today; blocking post-sprint release `v0.2.0` (the "Leger-generic-harness" release).

---

## G. What's still to produce (the gap Tom flagged)

Not deliverables of this document. Named here so the gap is explicit and so the sprint doesn't paper over it.

### The four externalization sequences (§D, restated for stacking)
1. Harness extraction: `sodimo/harness` → `leger-labs/harness`.
2. Dotfiles extraction: `sodimo/dotfiles` decomposition.
3. LP-XX citation rewrite: from Sodimo-manual line references to Leger-generic citation grammar.
4. `leger.config.yaml` envelope lock.

### The `leger deploy` behavior specification
What `leger deploy` does end-to-end against a Leger-generic harness:
- Takes a `leger.config.yaml` + a pinned `leger-release-manifest` version.
- Produces a signed bootc image (harness) + chezmoi tree + quadlet set + Caddy/DNS/Postfix config.
- Applies them to any Framework Desktop (DMI-detected hardware profile).
- Emits the initial `run_ledger` bootstrap row with `surface='leger_deploy'`, `change_type='install'`, `after_digest=<release version ID>`.
- Customer-agnostic: no Sodimo-specific string anywhere in the Leger-generic side of the tree.

### The per-engagement-vs-generic split
For each Sodimo-specific repo, the split between "Leger template" and "customer residual" has not been produced:
- `sodimo/mcp` → `leger-labs/template-mcp` (generic Worker pattern) + `sodimo/mcp` (Sodimo-specific tool manifest)
- `sodimo/etl` → `leger-labs/template-etl` (generic SFTP-NAS-D1 flow) + `sodimo/etl` (Sodiwin-specific schema)
- `sodimo/mail` → `leger-labs/template-mail` (generic Postfix/Dovecot/rspamd/Piler) + `sodimo/mail` (four-domain CNIL-specific)
- `sodimo/brand`, `sodimo/changelog`, `sodimo/sodimo-repo-template` — per-engagement by definition

### The release-manifest promotion pipeline
- How a `leger-release-manifest` version advances from draft to signed-and-published.
- Who merges what, when, against what test gates.
- The CI that validates citation resolution (§C) before a release can promote.

---

## H. File / repo map

### Produced by the `leger-scaffolding` wave
- `notes-product-doctrine.md`, `notes-policy-format.md`, `notes-delivery-sprint.md` — rolling thinking traces
- `01-product-doctrine-v1.md` → `01-product-doctrine-final.md` (contains pitch/price material from the pre-clean-up framing — read as historical thinking trace, not as prescription)
- `02-policy-format-v1.md` → `02-policy-format-final.md`
- `03-delivery-sprint-v1.md` → `03-delivery-sprint-final.md`
- `98-devils-advocate.md`
- `99-consolidated.md` — this file (product + release only)

### Already-extant inputs
- `~/sodimo/secondweek/thursday/noon-leger-progress.md` — 4-layer model + Appendix A
- `~/sodimo/secondweek/thursday/noon-research.md` — bootc/Anaconda/hardware research
- `~/sodimo/secondweek/thursday/morning-investigating/FINAL-SYNTHESIS.md`
- `~/sodimo/secondweek/thursday/midday-conceptualizing-leger/` — 5 findings + `99-WAVE-OVERVIEW.md`
- `~/leger-new/` — architecture/, leger-docs/ (17 LPs in old shape), app-leger-run/, leger-cli/, manual-split/
- `~/sodimo-dev/changelog/` — Sodimo customer site, `scope` field wired, build green
- `~/leger-labs/` — legacy reference (read-only unless explicitly authorized)
- `~/Downloads/forumvc-material/` — policy-pack format inspiration only

### 7 new repos to `git init` in the sprint
`leger-labs/{leger-release-manifest, leger-docs, leger-cli, app.leger.run, leger-schema, leger-infra, leger-dotfiles}`

### Repos NOT initialized this sprint (deferred work)
`leger-labs/{harness, leger-skills, blueprint-fmcg, template-dotfiles, template-changelog, template-infra, template-mcp, template-etl, template-mail}` + per-customer `<customer>-compliance`. `leger-labs/harness` is the target of §D.1 and its non-creation is a deliberate signal that the extraction sequence is real work, not a `git init`.

---

*End. The 24 hours start now. §E is the sprint block plan. §F is decisions-only-Tom-can-make within sprint scope. §G is the work that falls outside the sprint and has not yet been produced.*
