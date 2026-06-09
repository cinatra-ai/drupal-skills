# External Integrations

**Analysis Date:** 2026-06-09

## APIs & External Services

**Cinatra Platform Tool — drupal_content_editor_run:**
- This skill calls the `drupal_content_editor_run` tool at runtime, dispatched by the Cinatra platform
- Parameters passed by the skill: `instructions` (natural language edit instruction only)
- Parameters forcibly overridden server-side: `instanceId`, `nodeId` (pinned from request context — the skill MUST NOT pass these)
- Behavior: LLM-driven WayFlow content editor; blocking-mode dispatch; may take 30–90 seconds for large/multi-field edits
- Defined in: `skills/drupal-widget-chat/SKILL.md`

**Cinatra Marketplace / Registry:**
- Submission endpoint: `registry.cinatra.ai` (via marketplace MCP proxy)
- Flow: extension-submit-for-review → approve → promotion saga → registry publish
- Auth: `CINATRA_MARKETPLACE_VENDOR_TOKEN` org secret (inherited in `.github/workflows/release.yml`)
- Reusable release workflow: `cinatra-ai/.github/.github/workflows/reusable-extension-release.yml@main`

## Data Storage

**Databases:**
- Not applicable — this is a stateless skill/system-prompt extension; no database access

**File Storage:**
- Not applicable

**Caching:**
- Not applicable

## Authentication & Identity

**Auth Provider:**
- Not applicable at the skill level
- GitHub OIDC (`id-token: write` permission) used by the release workflow for build-provenance attestation (`.github/workflows/release.yml`)

## Monitoring & Observability

**Error Tracking:**
- Not detected — error handling is behavioral: the skill instructs the LLM to surface errors in plain English without leaking stack traces or HTTP status codes (`skills/drupal-widget-chat/SKILL.md`)

**Logs:**
- Not applicable at the skill layer; logging is handled by the Cinatra platform runtime

## CI/CD & Deployment

**Hosting:**
- Cinatra Marketplace (`registry.cinatra.ai`)

**CI Pipeline:**
- GitHub Actions — `.github/workflows/ci.yml` (build, typecheck, test, pack dry-run, kind-gates)
- GitHub Actions — `.github/workflows/release.yml` (publish on GitHub Release)
- Node.js 24 runner on `ubuntu-latest`
- Triggers: push/PR to `main` (CI); GitHub Release published or `workflow_dispatch` on a tag ref (release)

## Environment Configuration

**Required env vars:**
- None required at dev/build time
- `CINATRA_MARKETPLACE_VENDOR_TOKEN` — org-level GitHub secret, consumed only during release workflow (never in source)

**Secrets location:**
- GitHub org secrets (not committed)

## Webhooks & Callbacks

**Incoming:**
- Not applicable

**Outgoing:**
- Not applicable — the skill triggers `drupal_content_editor_run` as a platform tool call (not an outbound HTTP webhook from this repo)

---

*Integration audit: 2026-06-09*
