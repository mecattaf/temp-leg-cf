# Release fan-out — one logical Leger release, many repos

**Status:** design doc, 2026-04-23 midday
**Author:** `changelog-architect`
**Scope:** how a single "Leger v2026.04.23" release lands in `leger-labs/harness`, `leger-labs/leger-schema`, `leger-labs/leger-docs`, `leger-labs/leger-cli`, `leger-labs/leger-skills`, `leger-labs/template-*` as coordinated releases, and then shows up as one entry on `leger.run/changelog` + one entry on each paying tenant's `<customer>.leger.run/changelog` when that tenant pulls it.

---

## 1. The problem

A Leger release is not one repo — it's a set of repos that ship together:

| Repo | What might change in one release |
|---|---|
| `leger-labs/harness` | New bootc image digest; new kargs; new preloaded packages |
| `leger-labs/leger-schema` | Schema version bump; new envelope field; new service module |
| `leger-labs/leger-docs` | New LP-XX policy; policy rewrite; new pattern doc |
| `leger-labs/leger-cli` | New CLI subcommand; behavior change |
| `leger-labs/leger-skills` | New Claude Code skill; skill prompt rewrite |
| `leger-labs/template-dotfiles` | New quadlet in catalog; pinned-tag bump |
| `leger-labs/template-infra` | New `leger.config.yaml` field, backfilled defaults |
| `leger-labs/template-changelog` | Astro components / site-config updates |
| `leger-labs/template-mail` | Postfix/rspamd config drift |
| `leger-labs/cloud-pricing` | Monthly rate refresh |
| `leger-labs/model-store` | New model added to catalog |

Nine-to-eleven repos, each with its own release cadence, some of which are customer-installed (templates — customers fork) and some of which are Leger-consumed artifacts (harness image is pulled by bootc, skills are baked in).

The question: **when is something "released"**, and how does the tenant layer's `leger apply` know *which version-tuple of all these repos* is the thing it's converging to?

## 2. Decision: release-please in a manifest repo, templates fan out via PR bot

Not `changesets` (npm-ecosystem bias, poor cross-language fit).
Not `release-please-monorepo` (actual monorepo — not what we have).
**Yes:** `release-please` running in *each* repo on conventional commits, PLUS a small `leger-labs/leger-release-manifest` repo that pins a version-tuple and is the *only* source the tenant CLI trusts.

### 2.0 Research-preferred option vs. our deviation

The Claude.ai research thread on release fan-out (`06-claude-web-research-findings.md` RQ6) taxonomized the options as:

1. **Monorepo + release-please with manifest.** All leger-labs/* repos collapse into one repo; release-please-monorepo pins per-package versions in a manifest file; one CI run drives all packages.
2. **Manifest repo across polyrepos** (this document's choice). Per-repo release-please + a separate manifest repo that freezes the tuple across polyrepos.
3. **Roll your own.** Custom GH Actions composing the fan-out from scratch.

Research-preferred: **option 1 OR option 3**. Reject "multi-repo-coordination-from-scratch" — i.e., reject polyrepo setups that *don't* have a manifest tying them together.

We are picking **option 2**, which sits between 1 and 3. The research doesn't directly endorse option 2; the deviation needs justification. Bullets below.

### 2.0a Why manifest-repo is preferable to monorepo for Leger specifically

1. **Customer forks.** `leger-labs/template-dotfiles`, `template-infra`, `template-changelog`, `template-mail` are *forked* by customers into their own orgs (`<customer>-dotfiles`, `<customer>-infra`, …). A monorepo would drag the history of `harness`, `leger-schema`, `leger-cli`, and every other leger-labs artifact into every customer's fork. That's a ~200MB unnecessary carry per tenant and a subtle information leak (commit messages referencing other customers). Polyrepo lets customers fork the narrow slice they actually need.

2. **Heterogeneous build pipelines, already working.** `harness` builds a signed bootc container image (mkosi + cosign). `leger-cli` builds a signed RPM + DNF-repo update. `leger-docs` is an Astro static site. `leger-schema` is a JSON-Schema publish. Each has a mature per-repo `release-please.yaml` CI config today. A monorepo collapse would need to rewrite those pipelines as per-path matrix jobs, adding complexity without removing work.

3. **Cadence decoupling.** `cloud-pricing` updates monthly (rate refresh). `harness` updates maybe once per quarter. `leger-docs` can update daily. In a monorepo, every commit to any path churns the release-please-monorepo tooling's state machine; in polyrepo, each repo gets tagged on its own cadence and the manifest repo is the only place the cadences re-sync.

4. **Independent rollback.** If `leger-cli@2026.04.23` ships a regression, a tenant can pin `compat.leger_cli: 2026.04.20` in `leger.config.yaml` while staying on the current `harness` image. In a monorepo every artifact ships as one lockstep version tuple, so rollback means reverting all artifacts to a prior manifest even if only one is broken.

5. **Access control.** `leger-release-manifest` is the *release gate* — it's the repo Tom (eventually: a release manager) signs off on. Having it separate from `leger-cli` / `harness` / etc. means the signing identity for "this tuple is a blessed Leger release" is distinct from "this commit to leger-cli is reviewed." Fewer accidental promotions.

Accepted cost: the manifest repo is an extra piece of infrastructure the research's "option 1 or 3" would have let us skip. We pay that cost because the five reasons above are structural to our customer-forking + polyrepo + heterogeneous-build situation.

If a future fleet-of-one Leger operator runs Leger without customer-fork templates (e.g., a large Leger-Labs internal fleet), option 1 becomes more attractive; revisit then.

### 2a. Per-repo release-please

Each repo in the list above gets a `.github/workflows/release-please.yml` that:
- Reads conventional-commit history since the last tag.
- Opens / maintains a release PR ("chore: release v2026.04.23").
- When merged, cuts a tag, a GH release, and (for images) a cosign-signed container push.

This is the `sodimo-dev/changelog` pattern already (calver `YYYY-MM-DD-<slug>` tags). Keep that convention across all leger-labs repos.

### 2b. The manifest repo

`leger-labs/leger-release-manifest` is a small repo with one file:

```yaml
# leger-release-manifest/manifest.yaml
# Locked version-tuple for Leger v2026.04.23. This is THE release.
# Every tenant CLI `leger apply` trusts this file as its source of truth.

release:
  version: "2026-04-23"
  codename: "strix-halo-120b-gpt-oss"
  published_at: "2026-04-23T18:00:00Z"
  signed_by: "cosign://leger-labs/release-signer"

artifacts:
  harness:
    image: "ghcr.io/leger-labs/harness:v2026.04.23"
    digest: "sha256:abcdef0123456789…"
    fallback_digest: "sha256:fedcba9876543210…"   # the prior good for rollback
  schema:
    git_tag: "leger-schema@v0.4.0"
    digest: "sha256:…"
  docs:
    git_tag: "leger-docs@2026-04-23"
  cli:
    binary: "ghcr.io/leger-labs/leger-cli:v2026.04.23"
    digest: "sha256:…"
    rpm: "https://dnf.leger.run/leger-2026.04.23-1.fc44.x86_64.rpm"
    rpm_sig: "https://dnf.leger.run/leger-2026.04.23-1.fc44.x86_64.rpm.sig"
  skills:
    git_tag: "leger-skills@2026-04-23"
  templates:
    dotfiles:   "leger-labs/template-dotfiles@2026-04-23"
    infra:      "leger-labs/template-infra@2026-04-23"
    changelog:  "leger-labs/template-changelog@2026-04-23"
    mail:       "leger-labs/template-mail@2026-04-23"
  catalog:
    cloud_pricing_tag: "cloud-pricing@2026-04-23"
    model_store_tag:   "model-store@2026-04-23"

changes:
  # Human-authored release summary. Pulled into leger.run/changelog directly,
  # AND into the first tenant-layer entry for tenants that pull this release.
  summary: |
    - New harness image: Strix Halo kargs + kyuz0-pinned SHA 1421e870…
    - gpt-oss-120b UD-Q8_K_XL wired as local-heavy tier
    - LP-18 "Finetune Provenance" added (insurance blueprint dependency)
    - Twenty v2.0.0 pinned (short-name policy fix)
  breaking:
    - leger-schema@v0.4.0: compat.profile enum adds 'leger-smb-hipaa-adjacent'
  migrations:
    - "none; additive schema change"

compat:
  # Which previous releases a tenant may upgrade FROM with no manual migration.
  safe_upgrade_from: ["2026-04-09", "2026-04-16", "2026-04-20"]
  migration_required_from: []
```

### 2c. How a manifest is authored

A GH Action in `leger-labs/leger-release-manifest`:

1. Runs on `workflow_dispatch` (manually triggered by Tom — deliberate release gate).
2. Reads the latest-released tag of each downstream repo (via `gh release list --limit 1`).
3. Fetches the digest for container artifacts.
4. Composes a draft `manifest.yaml` and opens a PR with it.
5. Tom reviews the PR (does the combination actually work? has he tested it?), edits `changes.summary`, fills in breaking changes, and merges.
6. On merge, a second action:
   - Cosign-signs the manifest file.
   - Tags `leger-release-manifest` with the `release.version` date.
   - Pushes an entry to `leger-labs/leger-docs/src/content/releases/product/en/2026-04-23-manifest.md` (auto-generated from the manifest's `changes` section — this is what shows up on `leger.run/changelog`).
   - Posts a GH issue comment across all downstream repos ("This repo's tag X.Y.Z is included in leger-release-manifest 2026-04-23") for traceability.

The manifest is the commitment. A repo can cut 10 internal tags between manifests; only the ones pinned in a manifest are "released Leger."

### 2d. How a tenant's `leger apply` consumes it

On the customer's Framework Desktop, `leger apply` does:

1. Reads `leger.config.yaml` → looks at `compat.image_digest_set` (or new field: `compat.leger_release`).
2. Resolves that to a manifest version: `compat.leger_release: "2026-04-23"` → fetches `https://manifest.leger.run/2026-04-23/manifest.yaml` + its cosign signature. Verifies.
3. Checks current box state vs the manifest:
   - Is `/etc/bootc/current` pointing at `harness.image@digest`? If not, schedule bootc switch.
   - Is the local `leger-cli` binary the pinned version? If not, schedule RPM upgrade.
   - Is the tenant's chezmoi pulling `template-dotfiles@manifest.tag`? If not, update submodule.
   - Is the local `leger-skills/` directory checked out at the pinned tag? If not, sync.
4. For each drift, emit a pre-write `run_ledger` row with `surface=leger_apply, subject="converge to manifest 2026-04-23: <what>"`.
5. Execute the convergence.
6. Post-write: update the run_ledger rows with after_digest and duration_ms.
7. Emit one summary row with `surface=leger_apply, change_type=upgrade, subject="converged to Leger 2026-04-23", before_digest=<old manifest digest>, after_digest=<new manifest digest>, prompt_hash=<manifest sha256>`.

The tenant-layer changelog then renders:
- One "summary" card: *"Upgraded to Leger 2026-04-23 (strix-halo-120b-gpt-oss)"* with the `changes.summary` text.
- N sub-cards under it: each quadlet restart, each skill sync, each rotation triggered by the upgrade.

### 2e. How a tenant rolls back

The manifest has `artifacts.harness.fallback_digest`. If `leger apply` fails health-gates, `leger rollback` does:

1. `bootc rollback` → previous image.
2. Restore `template-*` submodule pins from the previous-manifest reference cached locally under `/var/lib/leger/manifests/`.
3. Emit `run_ledger` row `surface=leger_apply, change_type=rollback`.

The manifest-repo itself keeps every prior manifest as a git tag; the tenant CLI can always resolve a prior manifest by version.

## 3. Why not `changesets`

- `changesets` is great for npm workspace monorepos where every package publishes to npm.
- Our "packages" are a heterogeneous bag: bootc images, signed RPMs, Astro-built static sites, GH-release-hosted CLI binaries, git-tag-based template forks. `changesets` would need to be tricked into treating each of those the same way, and the build semantics don't compose.
- Conventional commits + per-repo `release-please` + a manifest repo is the simpler composition: each repo uses the tool fit for its artifact kind, and the manifest is the cross-cut.

## 4. Why not `release-please-monorepo`

- `release-please-monorepo` works when your repos are actually one monorepo with multiple published packages.
- Our repos are intentionally separate — customers fork `template-*`, Tom maintains `leger-*` — and we do not want the operational coupling of a single-repo history for release purposes. The customer should be able to fork `template-dotfiles@2026-04-23` without pulling the history of `harness` into their repo.

## 5. Alternative considered: hand-written `RELEASE.md` + GH Action

Tom's instinct: "or hand-written RELEASE.md with a release-orchestration GitHub Action."

This is functionally the same as the manifest approach with fewer guardrails. The manifest.yaml structured format buys us:
- Tenant-CLI can parse and act on it without regex-ing prose.
- The `changes.summary` + `changes.breaking` + `migrations` sections are typed; the auto-changelog entry on `leger.run/changelog` is a template render, not a copy-paste.
- `compat.safe_upgrade_from` is load-bearing for `leger apply` deciding whether it can skip a migration.

Hand-written RELEASE.md is fine for a v0 sketch but doesn't scale past the first few releases — and the tenant CLI would then be reading prose, which is the wrong interface.

## 6. Customer's infra repo PR — the output

FINAL-SYNTHESIS §2 "Emit PR to customer's infra repo" is the endpoint. Concretely:

On a new manifest, a GH Action in `leger-release-manifest` iterates all customer infra repos the FDE has registered (there's a private `leger-labs/fde-customers.yaml` with a list — renames/deletes via PR only). For each registered customer, it opens a PR against `<customer>-infra` bumping `leger.config.yaml:compat.leger_release` to the new version.

The customer's FDE (Tom, for pilot) reviews, green-lights, merges — which triggers the on-box `leger apply` flow (via the customer-infra repo's own CI → webhook → box pull).

This is the only network path between leger.run and the tenant box: a PR in the tenant's own GitHub org. leger.run never has direct access to the tenant's box.

## 7. Open questions (flagged for Tom)

1. **Is `leger-release-manifest` a separate repo or a path inside `leger-labs/leger-docs`?** Separate is cleaner (different cadence, different access controls) but adds a repo to the roster. Recommending **separate**. Tom preference?
2. **Manifest signing: cosign only, or cosign + SLSA provenance?** SLSA is more work but is the tell for the "SMB-grade compliance pack" audience. Defer to v1.
3. **`manifest.leger.run` as a subdomain** — does this need to exist, or can the manifest live at `leger.run/manifests/2026-04-23.yaml`? Either works; subdomain is trivially nicer for CORS and for the CLI's network posture.
4. **Fan-out PR bot identity.** A GH App (`leger-release-bot`)? Or Tom-the-human opens every PR manually? FINAL-SYNTHESIS §5 dropped `leger-bot` as a SaaS-of-one cut. Reintroduce narrowly, scoped to `leger-release-manifest` only? Recommending: **Tom-the-human for first 3 releases, then a GH App if volume justifies**.
5. **What happens to the per-repo release-please PRs that a human forgets to merge?** Stale release PRs are a known footgun. A weekly cron nag in `leger-release-manifest` that lists "stale release-please PRs blocking next manifest" is cheap insurance.

## 8. Migration path from current state

Currently: `sodimo/*` repos all have release-please workflows authored from `sodimo-repo-template`. The calver convention is already in place. No manifest repo exists yet.

Steps:
1. Keep `sodimo/*` as-is; the Sodimo engagement is the reference deployment, not the Leger product release channel.
2. Copy `sodimo-repo-template` into `leger-labs/template-repo` (or just fork) and use it to scaffold each `leger-labs/*` repo.
3. Scaffold `leger-labs/leger-release-manifest` from scratch — smallest repo, one YAML file + one GH Action + a signing keypair.
4. First manifest is a no-op: "Leger v0.1.0, wrapper around existing leger-labs state as of 2026-04-30." Tag it, cosign it, prove the pipeline. Then real releases start.

## 9. Acceptance criteria

Works when:
- A PR merges into `leger-labs/leger-schema` bumping schema to v0.4.1. `release-please` opens an internal release PR in that repo. Tom does NOT merge it alone; instead, he manually triggers `leger-release-manifest`'s workflow_dispatch which drafts manifest 2026-05-07. Manifest merge tags all downstream repos *and* posts a product-layer entry on `leger.run/changelog` in one atomic action.
- Running `leger apply` on the Sodimo box when `<customer>-infra/leger.config.yaml:compat.leger_release` is bumped to `2026-05-07` pulls exactly the set of artifacts named in manifest 2026-05-07 — no more, no less — and emits `run_ledger` rows under `surface=leger_apply`, yielding a single summary card + N sub-cards on `sodimo.leger.run/changelog`.
- `leger rollback` against the same tenant rolls harness back via `bootc rollback` using `artifacts.harness.fallback_digest` from the *prior* manifest and emits `run_ledger surface=leger_apply, change_type=rollback`.
