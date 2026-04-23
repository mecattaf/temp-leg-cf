# Changelog layering — product vs. tenant

**Status:** design doc, 2026-04-23 midday — revised after `06-claude-web-research-findings.md` (RQ6)
**Author:** `changelog-architect`
**Supersedes:** FINAL-SYNTHESIS §2 "Stream A / Stream B" (refines it, doesn't replace it)

---

## 1. Mental model

Two time-series of events, rendered for very different audiences, off a **shared entry schema** but with **separate storage backends**.

- Product layer — `leger.run/changelog` — vendor-owned release feed.
- Tenant layer — `<customer>.leger.run/changelog` — per-box applied-state log.

They share the *shape of an entry*. They do **not** share a row store, a database, or a render-time filter. The unification is at the schema level, not the table level.

### 1a. Product layer — `leger.run/changelog`

The public release feed for Leger itself.

- **What belongs here:**
  - A new harness bootc image digest is signed and published.
  - A new LP-XX policy is added to the pack (or an existing one rewritten).
  - A new Quadlet enters the `leger-labs/template-dotfiles` catalog.
  - A schema version bumps (`leger-schema`: 0.3.1 → 0.4.0) and what broke / what migrated.
  - A new Claude Code skill lands in `leger-labs/leger-skills`.
  - A blueprint reference repo ships (Sodimo FMCG / no-person-company / insurance-finetune).
- **What does NOT belong here:**
  - Any per-engagement customer state.
  - Any credential, token, or digest that's tenant-specific.
  - Any "we upgraded Sodimo yesterday" entry — that's the tenant layer.
- **Storage:** one YAML/JSON feed file per release at `leger-labs/leger-docs/src/content/product-releases/<version>.yaml`, git-tracked, release-please-authored (see `release-fan-out.md`).
- **Cadence:** aligned to FINAL-SYNTHESIS §2 V6 commitment (≤ 6 hr/wk, one substantial writeup bi-weekly, one flagship post monthly).
- **Render:** Astro site `leger-labs/leger-docs/` deploys to `leger.run/changelog` via CF Pages. Static build.

### 1b. Tenant layer — `<customer>.leger.run/changelog`

The private, tenant-owned applied-state log for one engagement on one box.

- **What belongs here:**
  - `2026-04-23 14:02:11 — box upgraded harness sha256:abc → sha256:def` (atomic bootc switch).
  - `2026-04-23 14:05:02 — paperclip-db migration ran, Twenty to v2.0.0`.
  - `2026-04-23 14:06:45 — LiteLLM master key rotated (class: 180d)`.
  - `2026-04-24 09:00:00 — DR drill rehearsed, <2h restore confirmed`.
  - `2026-04-30 11:00:00 — Added: Paperclip v0.5.2 activated (engagement-specific hostname allowlist set)`.
  - Auto-generated per-day rollup: "today the box ran 1.1M local tokens, saved €42.30 vs counterfactual."
- **What does NOT belong here:**
  - Marketing prose about Leger-the-product.
  - Other customers' upgrades (strict tenancy boundary).
- **Cadence:** event-driven, not editorial. Every `leger apply` produces one or more tenant-layer entries. Every vault rotation produces one entry. Every bootc switch produces one entry. Rollups are computed at render time, not stored.
- **Storage:** SQLite `run_ledger.db` at `/var/lib/leger/run_ledger.db`, append-only. Plus optional editorial MD entries in the customer's chezmoi-managed `<customer>-infra/changelog/`.
- **Render:** Astro site reads the SQLite DB via a loader (auto-events) plus the editorial MD glob (FDE-authored), JOINs product-release feed entries that were applied to this box (§5), interleaves by timestamp, deploys to the customer's own CF Pages project under `<customer>.leger.run/changelog`.

---

## 2. Audiences are not symmetric

This is the load-bearing asymmetry. Everything downstream derives from it.

### 2a. Product-layer audience — marketing-adjacent but technical

`leger.run/changelog` is read by people who do not know us yet.

- Internet evaluators scanning "is this project alive, moving, and well-operated?" — they skim the release feed as a proxy for product maturity.
- Prospective FDEs evaluating whether to build their own Leger-based practice.
- kyuz0-style LocalLLaMA lurkers checking what's new on the Strix Halo path.
- AMD Developer Program / HuggingFace partnership contacts doing due diligence before a conversation.
- Existing paying clients at renewal time, skimming "did I get my money's worth?"

Their question: **"Should I spend more of my attention on this product?"**

Implications for the render:
- Hero layout with the most recent 3 releases expanded, the rest collapsed.
- Breaking-change banners. Migration guides one click deep.
- Screenshots of new features where relevant.
- Cross-links into `leger.run/docs/` (LP-XX pack) so a reader can follow "new LP-18 Finetune Provenance landed" to the actual policy in the same session.
- Shareable per-release URLs, OpenGraph metadata, RSS feed, PRs-page-style filter by repo or type.
- Tone: **technical marketing.** Precise, but selling.

### 2b. Tenant-layer audience — handoff artifact for a non-technical operator

`<customer>.leger.run/changelog` is read by the *next engineer* and by the *business owner whose box this is*.

- The business owner / ops lead (Michel at Sodimo) who is **not technical**, who has been told "you won't be seeing us for a month" and who needs to know — at 9am on a random Tuesday — "what did the box do overnight?" or "when was the last vault rotation?" or, during an outage, "was there a change in the last 72 hours that might explain this?"
- The *next* FDE the owner hires when Tom's engagement ends — they need the full applied-state history to get context on the box without reading Tom's head.
- The auditor / regulator the customer may later route to this URL as compliance evidence (DPO request, CNIL review, insurance claim).

Their question: **"What happened to my box, when, and is it still healthy?"**

Implications for the render:
- Plain chronological timeline. Newest at the top. No editorial hierarchy.
- Every row has a timestamp, an actor, a plain-French/plain-English subject line, and a "what does this mean?" link if applicable.
- Prominent "last `leger apply`" header card showing current manifest version + date + status.
- Prominent "last backup", "last rotation", "last drill" summary strip.
- Filter strip by surface (`vault_rotation`, `leger_apply`, `bootc_switch`, …) but default view shows everything.
- Cost-saved / tokens-local rollup card at the top — the "am I getting my money's worth?" panel.
- Tone: **utilitarian.** No selling. Every entry is a statement of fact.
- Export-to-CSV and export-to-PDF from the same UI (auditor-friendly).

### 2c. What this means downstream

Because the audiences differ, these do NOT unify into one template, one component set, or one storage schema:

- **Storage**: product-layer entries are git-tracked YAML files in `leger-labs/leger-docs`. Tenant-layer entries are rows in `run_ledger.db` on one specific box. Neither lives where the other does; neither should.
- **Write path**: product-layer entries are PR-authored (release-please or hand-edited) and human-reviewed before merge. Tenant-layer entries are programmatically INSERTed by `leger` CLI subcommands or systemd units, without human review.
- **Render path**: the `leger.run` Astro build reads git-tracked YAML. The `<customer>.leger.run` Astro build reads SQLite on-box. They share Zod-validated entry types (§3) but instantiate separate collections.
- **Trust model**: product-layer entries are cosign-signed at the manifest level (see `release-fan-out.md`). Tenant-layer entries are append-only-by-convention with optional Rekor-style signed checkpoint if regulators ever demand it (§7).

A single `scope` column in a single table would hide all four of those distinctions behind a render-time filter — and the *audiences* would still be different, which means the filter would still be branching to different components, different columns, different queries. The "one table with a filter" design is strictly worse than "two storages with a shared entry schema" for this shape.

---

## 3. Shared entry schema (Zod)

Two Zod schemas, same conceptual shape, disjoint required-field sets. The product-layer schema is authored fresh (not a patch on the existing Sodimo `releaseSchema`). The tenant-layer schema is authored fresh too.

This is a **proposal** for `manual-surgeon` (product-layer schema → fresh `leger-docs` project) and for `cli-scoper` (tenant-layer schema → `leger-cli`'s render step). Neither is applied in this document.

```ts
// Shared primitives.
const digestString = z.string().regex(/^sha256:[0-9a-f]{64}$/);
const versionId = z.string().regex(/^\d{4}-\d{2}-\d{2}(-[a-z0-9-]+)?$/);

// ============================================================
// Product-layer entry — one per Leger release (manifest)
// ============================================================
// Storage: leger-labs/leger-docs/src/content/product-releases/<version>.yaml
// One file per release. Authored by release-please or by hand.

const productReleaseSchema = z.object({
  version: versionId,                 // e.g. "2026-04-23" or "2026-04-23-strix-halo"
  codename: z.string().optional(),
  published_at: z.coerce.date(),
  signed_by: z.string(),              // "cosign://leger-labs/release-signer"

  // The release manifest contents (see release-fan-out.md).
  artifacts: z.object({
    harness: z.object({
      image: z.string(),
      digest: digestString,
      fallback_digest: digestString.optional(),
    }),
    schema: z.object({ git_tag: z.string(), digest: digestString }),
    docs: z.object({ git_tag: z.string() }),
    cli: z.object({
      binary: z.string(),
      digest: digestString,
      rpm: z.string().url(),
    }),
    skills: z.object({ git_tag: z.string() }),
    templates: z.record(z.string(), z.string()),
  }),

  // Human-authored release summary.
  summary: z.string(),
  breaking: z.array(z.string()).default([]),
  migrations: z.array(z.string()).default([]),

  // Categories for the filter strip on leger.run/changelog.
  categories: z.array(
    z.enum(['harness', 'schema', 'docs', 'cli', 'skills', 'templates',
            'policy', 'catalog', 'security'])
  ).default([]),
});
```

```ts
// ============================================================
// Tenant-layer entry — synthesized from run_ledger rows + editorial MD
// ============================================================
// Storage: /var/lib/leger/run_ledger.db (auto) + <customer>-infra/changelog/*.md (editorial)
// Emitted by: leger-cli subcommands, systemd units (auto); FDE author (editorial)

const tenantEntrySchema = z.object({
  // Every row carries a wall-clock stamp; the render interleaves all sources by this.
  timestamp: z.coerce.date(),

  // For auto-events: the run_ledger rowid (stable across rebuilds).
  // For editorial entries: a slug derived from the MD filename.
  id: z.string(),

  // Auto-events inherit from run_ledger.surface; editorial entries set their own.
  surface: z.string(),

  // Human-legible description. For auto-events this is the run_ledger.subject
  // column; for editorial, the MD frontmatter title.
  subject: z.string(),

  // Optional body (MD-rendered). Editorial entries use this; auto-events
  // leave it empty and render from structured fields instead.
  body: z.string().optional(),

  // The actor attribution (non-technical reader sees "cli:thomas" or "system" here).
  actor: z.string(),

  // If this tenant event corresponds to a product-layer release applied
  // to this box, the version ID links the two. This powers the JOIN in §5.
  applied_product_release: versionId.optional(),

  // For auto-events, the prior and new observable-state digests.
  before_digest: z.string().optional(),
  after_digest: z.string().optional(),

  // Cost / token fields surfaced from run_ledger when applicable.
  cost_eur: z.number().optional(),
  cost_eur_if_cloud: z.number().optional(),
  tokens_in: z.number().optional(),
  tokens_out: z.number().optional(),

  // OpenTelemetry trace correlation (optional on all entries; see run-ledger-schema.sql).
  trace_id: z.string().optional(),
  span_id: z.string().optional(),
  event_outcome: z.enum(['success', 'failure', 'pending']).optional(),
});
```

Note: neither schema has a `scope` field, because the schema a reader is holding already tells them which layer they're in. A `productReleaseSchema` instance *is* product-layer by type. A `tenantEntrySchema` instance *is* tenant-layer by type. No runtime branching.

### 3a. What happens to the existing Sodimo `releaseSchema`?

The existing `~/sodimo-dev/changelog/src/content.config.ts` ships an 8-field `releaseSchema` used for the Sodimo per-repo release aggregator (in `releases_en.json`, `releases_fr.json`). That schema is not the product-layer and not the tenant-layer per this design — it's a legacy aggregator for the Sodimo engagement's GitHub releases across Sodimo's 7 repos.

Recommendation for `manual-surgeon`:
- Leave the existing Sodimo `releaseSchema` alone. It's fit for purpose for the Sodimo site.
- Add NEW collections to the Sodimo site for the tenant-layer render: one backed by the on-box SQLite (via a custom content loader), one backed by editorial MD globs. These use `tenantEntrySchema`, not `releaseSchema`.
- The product-layer (`leger.run/changelog`) is a separate Astro project in `leger-labs/leger-docs/` with its own `content.config.ts` using `productReleaseSchema`. The template for that project can be forked from `sodimo-dev/changelog` but the content collections are disjoint.

This kills my earlier proposal that added `scope: product | tenant | both` to one schema. Research says the split is at storage + schema, not at filter. I agree after re-reading; that earlier proposal was wrong.

---

## 4. Render pipeline — two Astro builds, shared component library

Not "one codebase with env-gated builds" as my earlier revision said. **Two Astro projects**, but they import a shared UI package for the row components (`@leger/changelog-ui` or equivalent — just a folder in `leger-labs/leger-docs/packages/ui/`).

- `leger-labs/leger-docs/apps/public/` → builds `leger.run/{changelog,docs}` — reads `productReleaseSchema` entries from YAML.
- `leger-labs/leger-docs/apps/tenant/` → builds `<customer>.leger.run/changelog` — reads `tenantEntrySchema` entries from SQLite + MD + JOINs applied product releases (§5).

Why this changed from the previous revision:
- Audience asymmetry (§2) means the hero layouts, filter bars, and header cards are materially different between the two sites. Forcing one codebase to branch at every component boundary on `RENDER_TARGET` adds complexity that disappears if you just have two projects.
- The shared work is *component-level* — `<RowTimestamp>`, `<RowSubject>`, `<DigestBadge>`, `<CostCard>` — and that deduplicates cleanly as a workspace package with no env-gating involved.
- Two projects is also what the research RQ6 says: "two rendering pipelines off the shared product-release feed."

### 4a. Build cadence

- **Public build**: on push to `leger-labs/leger-docs` main → CF Pages deploy. Fast, static, global CDN cache.
- **Tenant build**: triggered on-box by post-`leger apply` hook → `leger changelog render --tenant <slug>` → local `pnpm build` with `RUN_LEDGER_SQLITE=/var/lib/leger/run_ledger.db` and `PRODUCT_RELEASES_DIR=~/leger-infra/cached/product-releases/` → push built `dist/` to `<customer>-changelog-deploy` git repo → CF Pages deploys from that repo.

The tenant build needs the product-release YAML files to JOIN against (§5), so the CLI caches them locally from `leger-docs` at `leger apply` time. That cache is the source for the tenant build; the public `leger.run/changelog` is NOT consulted at tenant build time (offline-first).

### 4b. Why build on-box

Reasons unchanged from the earlier revision:
- SQLite file never leaves the box in readable form.
- No build-time decryption keys floating in CF Pages env vars.
- Matches LP-04 "no inbound ports; outbound-only" posture — box pushes to git, CF Pages pulls from git.
- Rebuild is fast on Strix Halo (Astro + a few hundred ledger rows).

---

## 5. The JOIN pattern — product feed × tenant ledger

This is the mechanism that lets the tenant layer show "Leger v2026.04.23 was applied to this box on 2026-04-25, here's what changed" without storing the product-release content in the tenant's storage. The cross-reference is **rendered**, not required.

### 5a. The join key

Every `leger apply` that converges the box to a manifest emits a `run_ledger` row with:
- `surface = 'leger_apply'`
- `change_type = 'upgrade'`
- `prompt_hash = <sha256 of the manifest file>`
- `after_digest = <the new manifest version ID>` (e.g. the 14-char string `2026-04-23`)
- `subject = "converged to Leger <version>"`

The `after_digest` field on those rows is the foreign key into the product-release feed.

### 5b. Renderer JOIN

At tenant-build time, the tenant Astro project's data layer does the equivalent of:

```sql
-- Conceptual; actually done in TypeScript at build time via sql.js or better-sqlite3.

SELECT
  l.rowid            AS tenant_id,
  l.timestamp        AS tenant_applied_at,
  l.after_digest     AS product_version_id,
  l.subject          AS tenant_subject,
  l.trace_id,
  l.actor
FROM run_ledger l
WHERE l.surface = 'leger_apply'
  AND l.change_type = 'upgrade'
  AND l.after_digest IS NOT NULL;
```

Each row is paired with the product-release YAML for `after_digest` from the local cache. The renderer produces a tenant-timeline entry whose *primary* data is the tenant row (timestamp = when Sodimo applied it, not when Leger published it) and whose *secondary* data is the product-release summary, rendered as a popover or expander beneath the tenant row.

### 5c. Cross-reference URL, not embedded content

The popover / expander contains a short summary (the `summary:` field from the product-release YAML, truncated) plus a link:

```
Applied Leger 2026-04-23 · strix-halo-120b-gpt-oss
  · gpt-oss-120b wired as local-heavy
  · LP-18 Finetune Provenance added
  See full release notes: leger.run/changelog/2026-04-23 ↗
```

The tenant storage never copies the full product summary. If Leger retroactively fixes a typo in the 2026-04-23 notes on `leger.run`, the tenant site's popover refreshes at the next build (and the `↗` link always points to the canonical vendor copy). This is why cross-reference is *rendered*, not required.

### 5d. When the JOIN doesn't find a product release

Two cases:
1. **Unknown version ID** — a tenant was upgraded to a Leger manifest that doesn't exist in the cached product-release feed (maybe the cache is stale, or the version is a pre-release). Render the tenant row with a "release notes not yet indexed" placeholder. Don't block the build.
2. **Tenant-only event** — most `run_ledger` rows are NOT `leger_apply` upgrades. They're vault rotations, backup runs, chezmoi applies, MCP tool calls. No JOIN needed; render them standalone.

---

## 6. Reader tooling — Datasette for on-box browse/export

Ship `datasette serve /var/lib/leger/run_ledger.db --host 127.0.0.1 --port 8001` as a user-scope quadlet, behind Caddy + CF Access, mounted at `cockpit.<domain>/datasette/` (auth gated). Customer management gets a first-class SQL-browsable and JSON/CSV-exportable view of their own box's ledger without Leger building any of the UI.

Reasons:
- Free feature. `datasette` is ~20MB, pure-Python, already maintained by Simon Willison's crew.
- Gives an auditor / regulator / accountant immediate SQL access without the FDE having to write any custom report. "Export all cost_eur_if_cloud for 2026-Q2 as CSV" becomes one URL.
- Complements the Astro tenant changelog. The Astro site is the *narrative* view (timeline, cards, friendly prose). Datasette is the *raw* view (table, SQL, export).
- Also becomes the compliance-evidence artifact: "here is `run_ledger.db` served over Datasette with a 30-day authenticated URL" is what we hand an auditor.

Quadlet spec (draft — for `cli-scoper` / template-dotfiles):
```ini
# ~/.config/containers/systemd/datasette.container
[Unit]
Description=Datasette reader for run_ledger
After=leger-apply.service

[Container]
Image=docker.io/datasetteproject/datasette:latest
Exec=serve --host 0.0.0.0 --port 8001 --immutable /data/run_ledger.db
Volume=/var/lib/leger/run_ledger.db:/data/run_ledger.db:ro,Z
PublishPort=127.0.0.1:8001:8001
User=1000:1000

[Service]
Restart=on-failure
```

Note: `--immutable` is critical — Datasette will otherwise lock the SQLite file, blocking writers. With `--immutable` it opens read-only.

Future: a tiny Datasette plugin `datasette-leger` that pre-canonicalizes the four queries from `run-ledger-queries.sql` as saved queries in the UI, so the ops lead can click "Last rotations" rather than writing SQL. Deferred to v1.1.

---

## 7. Non-repudiation (deferred, but document the pattern)

The tenant-layer ledger is append-only-by-convention (writers use INSERT; the 5-step flow's step-4 UPDATE is writer-gated; there's no DELETE except the explicit compaction-event path). This is sufficient for the audience and the pilot.

If/when a regulator requires cryptographic non-repudiation (insurance claim, CNIL audit, data-processing dispute), the pattern to adopt is **Rekor-style signed checkpoint** (Sigstore project):

- Periodically (nightly), hash the current set of `run_ledger` rowids and their content-hashes into a Merkle tree.
- Sign the Merkle root with a long-lived box-specific key (LP-02 vault item, paper break-glass).
- Publish the signed root to a tamper-evident log (could be the git repo the tenant changelog deploys from — signed commits).
- Old checkpoints are immutable; any row inserted before a checkpoint whose hash differs from what the checkpoint committed can be proven tampered-with.

This is one extra nightly systemd timer and ~50 lines of Python. Do not ship it until a real regulator asks. When they do, the pattern is already documented here.

---

## 8. What happens to the FINAL-SYNTHESIS §2 "Stream A / Stream B"?

Stream A (public leger-docs) remains, with its scope refined: the release feed is ONE file per release, YAML, under `leger-labs/leger-docs/src/content/product-releases/`. Long-form methodology writeups live separately under `leger-labs/leger-docs/src/content/essays/` or similar — they are NOT release entries, and they're rendered on a different route. Keep the release feed narrow.

Stream B (per-customer manual) was framed in FINAL-SYNTHESIS as "regenerates on every `leger apply` from the recorded state of the engagement." Under this revision that regeneration is:
- The MANUAL itself (chapters) regenerates from `leger.config.yaml` via Nunjucks (that's `configurator-scoper`'s and `manual-surgeon`'s job).
- The CHANGELOG regenerates from `run_ledger.db` via Astro (this doc's job).
- Neither touches the other's storage.

---

## 9. Acceptance criteria (revised)

The design is working when:

1. `leger-labs/leger-docs/apps/public/` builds with zero knowledge of any tenant's state and produces `leger.run/changelog` from git-tracked YAML only.
2. `leger-labs/leger-docs/apps/tenant/` builds on-box (triggered by `leger changelog render --tenant sodimo`) with `RUN_LEDGER_SQLITE=/var/lib/leger/run_ledger.db` and produces `sodimo.leger.run/changelog` — interleaving auto-events, editorial MD, and JOINed product-release popovers.
3. A `leger apply` on Sodimo's box produces a new `run_ledger` row with `surface='leger_apply', change_type='upgrade', after_digest='2026-04-23'` AND a next-build tenant changelog entry with a popover that pulls `summary` from `leger-labs/leger-docs/src/content/product-releases/2026-04-23.yaml`.
4. A PR merging a new LP-XX into `leger-labs/leger-docs` produces 0 tenant-site changes automatically. Tenants see the policy only when they next `leger apply` a manifest that includes it.
5. `datasette serve /var/lib/leger/run_ledger.db --immutable` runs as a quadlet and the Caddy route serves it authenticated. The ops lead can click "Export all rotations as CSV" and get a file without the FDE writing anything.
6. The tenant Astro project and the public Astro project share a UI package (`packages/ui/`) providing `<RowTimestamp>`, `<RowSubject>`, `<DigestBadge>`, `<CostCard>` — no duplicated component code.

---

## 10. Open questions (flagged for Tom)

1. **release-please-monorepo vs changesets vs manifest-repo.** Pre-empted by research (see `release-fan-out.md` §"Why manifest-repo over monorepo for Leger specifically"). Tom may still have a preference; flagging.
2. **Tenant build cadence.** Every `leger apply` immediately, or debounced (max one build per 5 min)? CF Pages free-tier has build quotas.
3. **Datasette auth model.** CF Access alone, or CF Access + per-visitor shareable signed URLs (for auditor "30-day read-only link" use case)?
4. **Tenant retention post-handoff** (carries from FINAL-SYNTHESIS §7). If a customer stops paying, do we freeze `<customer>.leger.run` or let it decay? Affects whether we keep the build infra live.
5. **Monorepo or polyrepo for `leger-labs/leger-docs`?** This doc assumes a single repo with `apps/public/` and `apps/tenant/` + `packages/ui/`. Works with pnpm workspace. Alternative: keep them as two separate top-level repos and use a git-submodule or npm-package for shared UI. Recommending the workspace approach but Tom may have a preference.
6. **`productReleaseSchema` categories enum.** Current draft has 9 values. Overlaps with but doesn't match the existing Sodimo `releaseCategory` enum (7 values). Deliberately separate — these are product-layer categories, not per-repo-release categories.
