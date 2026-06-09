# Codebase Concerns

**Analysis Date:** 2026-06-09

## Tech Debt

**No `src/` directory despite `tsconfig.json` referencing it:**
- Issue: `tsconfig.json` declares `"rootDir": "src"` and `"include": ["src/**/*.ts", "src/**/*.tsx"]`, but no `src/` directory exists in the repo. This means tsc would emit TS18003 "No inputs were found" and fail if ever run standalone. The CI workflow paper-covers this by detecting "no tracked TS files" and skipping typecheck for content-only extensions — but the config is misleading and will cause confusion if someone tries to add TypeScript code.
- Files: `tsconfig.json`, `.github/workflows/ci.yml` (lines 104–112)
- Impact: Any contributor adding `.ts` files may hit unexpected tsc errors because the `tsconfig.json` settings (e.g., `"jsx": "react-jsx"`, `"noEmit": false`) were copied from a different kind of extension (UI/component), not a pure skill.
- Fix approach: Either remove `tsconfig.json` entirely (this is a content-only skill with no TypeScript) or replace its contents with a minimal "no-src" placeholder config.

**`tsconfig.json` settings are mismatched for a skill extension:**
- Issue: `"jsx": "react-jsx"`, `"lib": ["DOM", "DOM.Iterable"]`, and `"declaration": true` are all React/UI settings. The `drupal-widget-chat` skill is a plain-text SKILL.md prompt; it has no TypeScript, no React, and no build output.
- Files: `tsconfig.json`
- Impact: Misleads contributors into thinking this repo produces compiled output or has React components.
- Fix approach: Remove `tsconfig.json` or replace with a comment-only placeholder noting it is intentionally empty for this skill kind.

**`noImplicitAny: false` overrides `strict: true`:**
- Issue: `tsconfig.json` sets both `"strict": true` and `"noImplicitAny": false`. The second silently negates a key strict-mode check. If TypeScript sources are ever added, this will allow untyped parameters to slip through.
- Files: `tsconfig.json`
- Impact: Low now (no TS sources), but creates a type-safety hole if sources are added later.
- Fix approach: Remove the explicit `noImplicitAny: false` override so `strict` is internally consistent.

**`package.json` version is `0.1.0` with no published baseline:**
- Issue: Version `0.1.0` signals pre-release maturity. The release workflow is explicitly "dormant until the org infra exists". There is no lockfile and no published artifact yet.
- Files: `package.json`, `.github/workflows/release.yml`
- Impact: Release pipeline will silently no-op (reusable workflow not yet available) until `cinatra-ai/.github` org infra is wired. Any attempt to publish will fail with no clear error surfaced locally.
- Fix approach: Document the dormant state in README.md; add a check in `release.yml` or a comment listing the org-level prerequisites.

## Known Bugs

**No known bugs detected in the current codebase.**
- The only functional artifact is `skills/drupal-widget-chat/SKILL.md` (a plain-text system prompt). Bugs in the prompt's behavior (e.g., incorrect tool-call decisions) would only manifest at runtime in the Cinatra platform and are not detectable statically.

## Security Considerations

**System prompt exposes tool call contract:**
- Risk: `skills/drupal-widget-chat/SKILL.md` explicitly documents that `instanceId` and `nodeId` are "forcibly overridden server-side." If a malicious user reads this prompt (e.g., via a jailbreak or prompt-leak attack), they learn the server trusts the context injection and may try to manipulate the `instructions` parameter to achieve unintended edits.
- Files: `skills/drupal-widget-chat/SKILL.md` (line 18)
- Current mitigation: The prompt explicitly instructs the model never to include `nodeId` or `instanceId` in replies. Server-side override of those parameters is the actual defense.
- Recommendations: Ensure the server-side Drupal connector validates and sanitizes the `instructions` field, not just the ID parameters. Prompt-injection via crafted node content (e.g., a body field containing adversarial instructions) is not addressed anywhere in the skill.

**No `.env` files present** — not applicable.

**`.npmrc` disables peer auto-install:**
- Risk: `auto-install-peers=false` means peer dependency mismatches are silently skipped during install. If a consumer installs this package standalone, missing optional peers will not raise warnings.
- Files: `.npmrc`
- Current mitigation: Low impact because the package has no dependencies at all currently.
- Recommendations: Not applicable until dependencies are added.

## Performance Bottlenecks

**Long-running tool calls (30–90 seconds) acknowledged but not timeout-guarded:**
- Problem: `SKILL.md` warns users that large edits "may take a minute" (30–90 seconds), but there is no documented timeout or retry policy in the skill.
- Files: `skills/drupal-widget-chat/SKILL.md` (lines 57–59)
- Cause: The Drupal content-editor agent is described as "LLM-driven blocking-mode dispatch." No maximum timeout is specified.
- Improvement path: Add a timeout guidance note to the skill (e.g., "If no response after 2 minutes, suggest the user retry"), or rely on the platform-level timeout if one exists.

## Fragile Areas

**Single skill file — no validation or schema enforcement:**
- Files: `skills/drupal-widget-chat/SKILL.md`
- Why fragile: The entire behavioral contract of the widget (when to call tools vs. converse, how to summarize results, error handling) lives in one 63-line markdown file. Any accidental edit or merge conflict in this file directly degrades the assistant's behavior with no automated guard.
- Safe modification: Changes should be reviewed against the "hard rules" section (lines 44–46) and the two-action decision tree (lines 10–27).
- Test coverage: None. There are no automated tests validating that the skill prompt produces correct behavior.

**CI skips all meaningful checks for this repo:**
- Files: `.github/workflows/ci.yml`
- Why fragile: Because `package.json` has no `@cinatra-ai/*` peer dependencies, the CI classifier sets `first_party=0` and attempts a full standalone install/typecheck/test. However, there is no `test` script, no `typecheck` script, and no `src/` directory — so all three steps effectively no-op. The only real check is `npm pack --dry-run` and the `kind-gates` job (which currently does nothing for the `skill` kind). This means regressions to `SKILL.md` or `package.json` metadata are not caught by CI.
- Safe modification: Do not rely on CI as a quality gate for prompt content changes.

## Scaling Limits

**Not applicable** — this is a pure prompt-as-code skill with no runtime infrastructure in this repo.

## Dependencies at Risk

**No runtime dependencies declared** — `package.json` has an empty `cinatra.dependencies` array and no `dependencies`, `devDependencies`, or `peerDependencies`. Not applicable.

**Release pipeline depends on `cinatra-ai/.github` reusable workflow (external):**
- Risk: `release.yml` calls `cinatra-ai/.github/.github/workflows/reusable-extension-release.yml@main`. This is a hard dependency on an org-level workflow that is explicitly documented as not yet existing.
- Files: `.github/workflows/release.yml` (line 29)
- Impact: Any `published` release event will cause the workflow to fail with a "workflow not found" error.
- Migration plan: Document prerequisites in README; pin to a tag (`@v1`) rather than `@main` once the reusable workflow is stable to prevent silent behavior changes.

## Missing Critical Features

**No automated prompt-quality tests:**
- Problem: The skill's decision logic (tool-call vs. conversational path) has no test coverage of any kind — no unit tests, no integration tests, no golden-file assertions against sample conversations.
- Blocks: Confidence that prompt changes don't regress the widget's core behavior.

**No SKILL.md schema validation in CI:**
- Problem: The `kind-gates` CI job explicitly states "No kind-specific gate for this extension kind." The SKILL.md frontmatter (name, description) is never validated against a schema, so a malformed frontmatter block would pass CI undetected.
- Files: `.github/workflows/ci.yml` (lines 129–140), `skills/drupal-widget-chat/SKILL.md` (lines 1–4)
- Blocks: Reliable marketplace submission; a malformed skill could be submitted and fail only at the registry ingestion step.

## Test Coverage Gaps

**No tests exist in this repository:**
- What's not tested: The entire behavioral contract of `skills/drupal-widget-chat/SKILL.md` — edit-vs-converse classification, tool call argument construction, result summarization, error handling phrasing, long-edit warnings.
- Files: `skills/drupal-widget-chat/SKILL.md`
- Risk: Prompt regressions introduced by edits are invisible until they surface in production widget interactions.
- Priority: Medium — the skill is simple and the prompt is short, but the lack of any regression harness means even a one-line change can silently break the intended behavior.

---

*Concerns audit: 2026-06-09*
