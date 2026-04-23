# 04 — PR-to-infra output flow

The configurator's *output* is a pull request against the customer's own `<customer>/infra` repository. Not a tarball download, not an R2 write, not a deploy-to-Kubernetes. A PR, with a human-readable diff, that the customer (or Leger's FDE) reviews, merges, and pulls locally via chezmoi + bootc.

This design replaces the Beam.cloud deployment orchestrator from the legacy codebase. Reasons:
- **No long-lived customer credentials on Leger infra.** A GitHub App install ID is stored; a short-lived installation token is minted per-request.
- **Customer retains veto.** Every change is a reviewable PR; the customer can merge, modify, or reject.
- **Git is the audit log.** Every deploy is a commit on `main` of `<customer>/infra`. `git log` on the customer's box tells them exactly what Leger applied and when, without any Leger-side audit infrastructure.
- **Rollback is `git revert`.** No separate rollback UI to build.

## End-to-end flow

```
┌─────────────────┐
│ User in browser │
└────────┬────────┘
         │ 1. Edits form on app.leger.run
         │    (saves partial config to D1)
         │
         │ 2. Clicks "Apply — open PR"
         ▼
┌─────────────────────────────────────┐
│ app.leger.run CF Worker             │
│                                     │
│ POST /engagements/:id/open-pr       │
│  ├─ Validate envelope (AJV)        │
│  ├─ Resolve compat.leger_release    │
│  │   → fetch release-manifest tuple │
│  │   → verify cosign signature      │
│  │   → derive schema/image/harness  │
│  ├─ Apply x-profile-overrides       │
│  ├─ config-generator.ts             │
│  ├─ template-renderer.ts (Nunjucks) │
│  ├─ Build in-memory file tree       │
│  └─ Call pr-orchestrator.ts         │
└─────────────────┬───────────────────┘
                  │
                  │ 3. GitHub App API calls via
                  │    short-lived install token
                  ▼
┌─────────────────────────────────────┐
│ <customer>/infra on github.com      │
│                                     │
│ ├─ POST /repos/.../git/refs         │
│ │   (create branch leger/<sha>)     │
│ ├─ POST /repos/.../git/trees        │
│ │   (commit rendered tree)          │
│ ├─ POST /repos/.../git/commits      │
│ ├─ POST /repos/.../git/refs/:ref    │
│ │   (point branch at commit)        │
│ └─ POST /repos/.../pulls            │
│     (open PR with human diff body)  │
└─────────────────┬───────────────────┘
                  │
                  │ 4. PR notification
                  │    (email / Slack via GH)
                  ▼
┌─────────────────┐
│ Customer / FDE  │ reviews, merges
└────────┬────────┘
         │
         │ 5. Next chezmoi apply on the box
         ▼
┌───────────────────────┐
│ Customer Framework    │
│ Desktop (Fedora bootc)│
│                       │
│ chezmoi pulls from    │
│ <customer>/infra main │
│ applies .tmpl + env   │
│ systemctl --user      │
│ daemon-reload         │
└───────────────────────┘
```

## Step detail

### Step 1 — Incremental form edits
Every field change in the configurator saves to D1 (`engagements/:id/draft`). This is the "unfinished PR" buffer; the user can close the browser and come back. No GitHub traffic yet.

### Step 2 — User hits "Apply — open PR"
Worker receives `POST /engagements/:id/open-pr`. Validates the assembled envelope against the `leger-schema` JSON-Schema (AJV). If validation fails, the UI surfaces errors against the right fields via the existing `validation-summary.tsx` component — no PR is opened.

If validation passes, the Worker:
1. **Resolves `compat.leger_release`** against the `leger-labs/leger-release-manifest` repo — fetches the cosign-signed tuple for the slug, verifies the signature, extracts the pinned `{harness_digest, schema_version, cli_version, docs_version, quadlet_catalog_version, image_digest_set}`. If verification fails or the slug is unknown, aborts before any GH write.
2. Writes the resolved tuple back into the envelope's `compat.*` derived fields so the rendered tree carries them explicitly (and the on-box `leger apply` can cross-check without re-fetching the manifest).
3. Deep-merges `x-profile-overrides` server-side so the authoritative envelope matches what the UI displayed.
4. Runs `config-generator.ts` to produce a per-service `UserConfig` object.
5. Runs `template-renderer.ts` which walks `api/templates/services/**/*.njk` invoking Nunjucks for every service the engagement has `enabled: true`.
6. Collects the output into an in-memory map: `{"containers/systemd/openwebui.container": "<text>", "chezmoi/data.yaml": "<text>", "etc/leger/release.yaml": "<resolved tuple>", ...}`.
7. **Optionally touches `<customer>/changelog`.** When the apply adds a tenant-scope changelog entry — either because the release slug bumped, the profile changed, or a service was added/removed — the Worker also emits a changelog frontmatter file (`<customer>/changelog/src/content/changelog/<date>-<slug>.md` with the dual-publish `scope: sodimo` frontmatter per changelog-architect's schema). Per changelog-architect's `leger changelog render` design, the on-box CLI will later render this into the per-engagement manual via the env-gated single-codebase build from `<customer>/changelog`. Whether this ends up in the same PR or a sibling PR to the `<customer>/changelog` repo is an open design choice flagged below.

### Step 3 — GitHub App commits and PR

Authentication model:
- **Leger controls a GitHub App** named `leger-configurator[bot]`. Its private key is a Cloudflare Worker secret (`wrangler secret put GH_APP_PRIVATE_KEY`). The app ID is in `wrangler.toml` vars.
- **Customer installs the app** on their GitHub org during engagement setup (see "Auth flow" below). The install ID is stored in D1 per-engagement — *not* a token.
- **Per-request:** Worker mints a JWT signed with the app private key → exchanges it for an installation access token via `POST /app/installations/:install_id/access_tokens`. Token TTL is ~1 hour; Leger never persists it.

PR creation uses the GitHub REST API (can also use GraphQL; REST is simpler for the tree-and-commit pattern):

```
// simplified
const token = await mintInstallationToken(env.GH_APP_PRIVATE_KEY, env.GH_APP_ID, installId)
const gh = githubClient(token)

const baseRef = await gh.get(`/repos/${owner}/infra/git/refs/heads/main`)
const branchName = `leger/apply-${shortSha(engagement.id)}-${Date.now()}`

// Commit via tree API (no temp clone required — API-native)
const tree = await gh.post(`/repos/${owner}/infra/git/trees`, {
  base_tree: baseRef.object.sha,
  tree: Object.entries(fileTree).map(([path, content]) => ({
    path, mode: '100644', type: 'blob', content,
  })),
})
const commit = await gh.post(`/repos/${owner}/infra/git/commits`, {
  message: `leger-configurator: apply engagement ${engagement.tenant_slug}\n\n${humanSummary}`,
  tree: tree.sha,
  parents: [baseRef.object.sha],
})
await gh.post(`/repos/${owner}/infra/git/refs`, {
  ref: `refs/heads/${branchName}`,
  sha: commit.sha,
})

const pr = await gh.post(`/repos/${owner}/infra/pulls`, {
  title: `leger: apply ${engagement.tenant_slug} (${profile})`,
  head: branchName,
  base: 'main',
  body: buildHumanReadableBody(diffBefore, diffAfter, profile, lpXxEvidence),
})
```

### Step 4 — Human-readable PR body

The PR description is not a machine dump. It explains **what changed in business terms** so the customer's non-technical ops lead (Sodimo's Michel, for example — per memory) can understand.

Template:
```markdown
## Summary

This PR applies a Leger engagement configuration for **sodimo.eu** (FMCG distributor, 30 people).

**Profile:** leger-smb-baseline
**Leger release:** `2026-Q2-c` (bumped from `2026-Q2-b`) — [release notes](https://leger.run/releases/2026-Q2-c)

Resolved tuple (from leger-release-manifest, cosign-verified):
- harness: Fedora bootc @ sha256:abc123…
- schema: leger-schema@v0.3.1
- cli: leger@v0.4.2
- docs: leger-docs@v0.3.0
- quadlet-catalog: quadlet-catalog@v0.5.1
- image-digest-set: images-2026-Q2-c

## What changed since the last apply

- **Enabled** Piler mail archive (LP-12: Mail Sovereignty & Retention → indefinite retention)
- **Updated** OpenWebUI primary chat models: added `local/qwen3-30b-coder`
- **Rotated** `cf/pages-deploy-token` (LP-03: 365d cadence reached 2026-04-15)
- **Added** WhatsApp webhook endpoint for order acknowledgments (Sodimo-specific)

## Files modified

- `containers/systemd/openwebui.container` (image pin bumped, env-vars updated)
- `containers/systemd/openwebui.env` (added 2 env-vars for new models)
- `containers/systemd/piler.container` (NEW)
- `chezmoi/data.yaml` (new `cf_pages_deploy_token_ref`)
- `caddy/Caddyfile` (added `piler.sodimo.eu` route)

## Policy evidence (LP-XX cross-refs)

Every change links to the policy pack page that motivated it:
- LP-03 evidence: see `leger-docs/policies/LP-03-key-rotation.md`
- LP-12 evidence: see `leger-docs/policies/LP-12-mail-sovereignty.md`

## How to apply

1. Review this PR (look for anything unexpected — the diff should match what you entered in the configurator)
2. Merge when ready
3. On your Framework Desktop, run `chezmoi apply`. That's it.
4. If something breaks, `bootc rollback` reverts the OS layer; `git revert` on `<customer>/infra` reverts the app layer.

🤖 Generated by leger-configurator
```

### Step 5 — Customer pulls on their box

Outside the configurator's scope but worth naming: on the Framework Desktop, `chezmoi apply` (triggered manually by the FDE or by a systemd timer on a deliberate cadence — no auto-pull per LP-09) fetches from `<customer>/infra`, renders `.tmpl` files against local Vaultwarden secrets (`bw get`), writes `.env` files, and writes quadlet files. `systemctl --user daemon-reload` picks up the new quadlets. Done.

## Auth flow: GitHub App install and revocation

### Install flow (engagement setup)
1. During initial engagement setup the FDE opens `app.leger.run/engagements/new`
2. Configurator shows a "Connect GitHub" button that redirects to `https://github.com/apps/leger-configurator/installations/new?state=<engagement_id>`
3. Customer selects the `<customer>/infra` repo (or all repos in their org) and confirms install
4. GitHub redirects back to `app.leger.run/gh-app/callback?installation_id=...&state=...`
5. Worker's callback route verifies the `state` matches a pending engagement, stores `installation_id` in D1's `engagements` table

### Per-request token minting
Every time `POST /engagements/:id/open-pr` runs, the Worker:
1. Looks up the install ID for that engagement in D1
2. Signs a JWT with the app private key (10min TTL)
3. Exchanges it for an installation access token via `POST /app/installations/:install_id/access_tokens`
4. Uses that token for the PR operations
5. Discards the token after the request completes (no KV/D1 write — the token is request-scoped memory only)

### Scope limits
The GitHub App declares minimal permissions:
- **Repository: Contents — Read & Write** (to push commits)
- **Repository: Pull requests — Read & Write** (to open PRs)
- **Repository: Metadata — Read** (required by the above)

No access to: secrets, actions, packages, issues, teams, members, billing, webhooks-on-other-repos.

### Revocation
Customer can revoke at `https://github.com/organizations/<customer>/settings/installations`. Worker detects revocation on next attempt (401) and surfaces a prominent UI error: "Your GitHub App installation has been revoked. Reinstall to continue applying configurations." No Leger-side revocation endpoint needed.

### Token rotation on Leger side
The app private key itself (Worker secret) is rotated on the same 365-day cadence as customer service tokens — LP-03 applies to Leger's own credentials too, dogfooded per Blueprint 2.

## Branch protection conflict edge case

If the customer's `<customer>/infra` repo has branch protection requiring 2 reviews (which LP-XX `hipaa-adjacent` actively pushes them toward), the PR simply won't merge without those reviews. Leger's bot cannot auto-merge. That's **correct behavior** — the customer wanted the review gate; Leger respects it.

## Failure modes (and their UI surfacing)

| Failure | How it surfaces | How the user resolves |
|---|---|---|
| App not installed | 404 on install lookup → UI: "GitHub App is not installed on your org. Click to install." | Re-install via the linked flow |
| Token revoked | 401 from GH → UI: "App access was revoked. Please reinstall." | Same as above |
| Repo doesn't exist | 404 from GH on refs/heads/main → UI: "Cannot find <owner>/infra. Did you rename or archive the repo?" | Update the `engagement.infra_repo` setting or unarchive |
| Branch already exists | 422 conflict → Worker retries with a new timestamp suffix | Transparent to user |
| Validation failure | Caught before any GH traffic → inline field errors | User fixes form; no PR is attempted |
| GitHub API rate limit | 429 → UI: "GitHub is rate-limiting. Try again in N seconds." | Wait |
| Template render error | Worker-side exception → 500 with a redacted error + internal ID for support | Contact Leger; this is a Leger bug, not a customer bug |

## What is NOT part of this flow

- Leger never clones the customer's repo. All operations are GitHub-API-native.
- Leger never stores commit-scope tokens or SSH keys for the customer's repo.
- Leger never merges on the customer's behalf.
- Leger never writes directly to the customer's box. (CLI on the box does the pull, per the cli-scoper task.)

## Changelog entry in emitted PR

Per research RQ6 + changelog-architect's finalized design: Leger has **ONE product-release feed** (JSON/YAML, git-tracked, lives at `leger-labs/leger-release-manifest` or a sibling repo — single source of truth for product-level changelog entries), **ONE per-box `run_ledger` SQLite** (append-only event log on the customer's Framework Desktop), and **TWO rendering pipelines** off the shared entry schema: `leger.run/changelog` (marketing-adjacent-but-technical, internet-evaluator audience) and `<customer>.leger.run/changelog` (handoff-artifact + DR/rollback-log audience, non-technical pilot management).

**The configurator's role:** every apply-PR includes a one-line changelog entry in a format **compatible with the product-release feed's schema**, so `leger apply` on-box can later interleave it into the tenant changelog via a JOIN of product-feed entries (filtered to "applied to this box" via the run_ledger's apply-events) + box-local run_ledger events.

### Entry schema (6 required fields + 1 optional)

```yaml
# Emitted as a YAML block in the PR body AND as a frontmatter file
# (see "Cross-repo touch" section below for where the frontmatter file lands)
id: "2026-Q2-c-sodimo-piler-enable"     # stable slug, unique within tenant
applied_at: null                          # nullable; filled by on-box CLI when run_ledger records apply
title: "Enable Piler mail archive"
summary: "Activates LP-12 (Mail Sovereignty & Retention) for sodimo.eu with indefinite retention."
category: "service-add"                   # enum: release-bump | profile-change | service-add | service-remove | policy-change | config-tune | rotation | other
affected_services:                        # array of service keys from the envelope
  - piler
  - postfix
```

Field notes:
- `id` — built from `<leger_release>-<tenant_slug>-<short-kebab-title>`. Must be stable: the same apply, replayed, produces the same id. Used for deduplication during the JOIN.
- `applied_at` — intentionally null at PR-emission time. The on-box `leger apply` fills it when the customer's box runs the post-merge apply (pulled from the run_ledger's corresponding row). This is the hand-off point between SaaS-authored and box-applied state.
- `category` — enum matches the product-release feed's categories so the two pipelines render with matching filters / color codes / sort orders.
- `affected_services` — used by the per-tenant renderer to scope-filter entries (e.g., "show me only mail-related changes to this engagement").

### Where the entry lives

Two placements, both required:
1. **Inline in the PR body** (the `<customer>/infra` PR): the YAML block above goes under a `## Changelog entry` heading in the PR description. Human-readable and grepable by anyone reviewing the PR, even before it merges.
2. **As a markdown file in `<customer>/changelog`** (via the sibling PR described below): a file at `<customer>/changelog/src/content/changelog/<date>-<slug>.md` with the YAML block as frontmatter + a body section the Leger-authored first-draft prose. This is what `leger changelog render` on-box reads at render time.

### Cross-reference to the product feed

If an entry's `leger_release` matches an existing entry in the product-release feed (e.g., the release bump from `2026-Q2-b` to `2026-Q2-c` is also a product-level entry at `leger.run/changelog/2026-Q2-c`), the per-tenant renderer adds a `product_feed_xref` link in the rendered output — cross-reference is **rendered, not required** (per research RQ6). The YAML doesn't carry the xref; the renderer derives it at render time by joining on `leger_release`.

---

## Cross-repo touch: `<customer>/changelog`

Per changelog-architect's design, each engagement also has a `<customer>/changelog` repo that carries the dual-scope (Leger-shared + customer-specific) frontmatter-filtered source. When an apply produces a tenant-scoped changelog entry — release-slug bump, profile change, service add/remove, LP-XX policy enforcement change — the configurator emits a new markdown file for that repo, carrying the entry schema above as frontmatter.

Two options for how the PR handles this:

1. **Sibling PR (leaning this way).** The Worker opens two PRs: one to `<customer>/infra` with the rendered tree, one to `<customer>/changelog` with the new entry. Each is independently reviewable + mergeable. The customer can hold the changelog while merging infra (common — you want to ship before you announce).

2. **Single PR touching both repos.** Not possible with the GitHub API (a PR is per-repo). A variant: only touch `<customer>/infra`, and let the on-box `leger changelog render` regenerate the rendered manual later from an entry the customer authors by hand. Simpler but loses the "Leger writes the first draft" benefit.

**Recommendation:** sibling-PR model. Requires the GitHub App to have `<customer>/changelog` installed alongside `<customer>/infra`. The install flow in §"Install flow (engagement setup)" above needs to pick up both repos when the customer selects.

**Tuple-version gate on changelog entry.** The changelog entry's frontmatter includes `leger_release: "2026-Q2-c"` so the on-box renderer can validate it matches the running harness's recorded release (at `/etc/leger/release.yaml`). If the customer has skipped a release, the CLI's render command surfaces a "stale entries" list rather than silently rendering against a mismatched tuple.
