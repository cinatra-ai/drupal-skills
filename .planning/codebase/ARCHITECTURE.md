<!-- refreshed: 2026-06-09 -->
# Architecture

**Analysis Date:** 2026-06-09

## System Overview

```text
┌─────────────────────────────────────────────────────────────┐
│              Cinatra Drupal Widget Chat (Skill)              │
│            skills/drupal-widget-chat/SKILL.md                │
└────────────────────────┬────────────────────────────────────┘
                         │ System prompt injected at runtime
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              Cinatra Runtime / Agent Engine                  │
│     (host monorepo — not present in this repo)               │
└──────────────┬──────────────────────────┬───────────────────┘
               │                          │
               ▼                          ▼
┌──────────────────────┐    ┌─────────────────────────────────┐
│  Conversational path │    │  drupal_content_editor_run tool  │
│  (no tool call)      │    │  (Cinatra Drupal connector)      │
└──────────────────────┘    └─────────────────────────────────┘
                                          │
                                          ▼
                              ┌───────────────────────┐
                              │  Drupal node (CMS)    │
                              │  field edits applied  │
                              └───────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| Skill definition | System prompt that governs LLM behavior inside the Drupal widget | `skills/drupal-widget-chat/SKILL.md` |
| Package manifest | Declares Cinatra skill kind, version, license, no runtime deps | `package.json` |
| TypeScript config | Compile config for future TypeScript sources (currently no `src/`) | `tsconfig.json` |
| CI workflow | Validates package shape, typechecks, runs tests, dry-packs | `.github/workflows/ci.yml` |
| Release workflow | Publishes to Cinatra Marketplace via reusable org workflow | `.github/workflows/release.yml` |

## Pattern Overview

**Overall:** Skill-as-content — a single SKILL.md file that is a structured natural-language system prompt consumed by the Cinatra agent runtime. There is no compiled application code in this repository.

**Key Characteristics:**
- The entire behavioral logic is encoded in `skills/drupal-widget-chat/SKILL.md` as a system prompt
- The repo is a "source mirror" pattern: it declares host-internal `@cinatra-ai/*` packages as optional `peerDependencies`, and the Cinatra monorepo provides/builds/typechecks/tests them
- No direct runtime code executes from this repo; it is registered into the skills catalog at workspace level
- `cinatra.kind = "skill"` declared in `package.json` under the `cinatra` manifest block

## Layers

**Skill prompt layer:**
- Purpose: Define LLM decision logic — when to call `drupal_content_editor_run` vs. answer conversationally
- Location: `skills/drupal-widget-chat/SKILL.md`
- Contains: Decision rules, tool-call instructions, response formatting rules, error handling rules
- Depends on: Cinatra runtime (injects the system prompt) and the `drupal_content_editor_run` tool provided by the Cinatra Drupal connector
- Used by: Cinatra agent engine at chat widget runtime

**Package/manifest layer:**
- Purpose: Identify this repo as a Cinatra skill extension and declare its metadata
- Location: `package.json`
- Contains: `cinatra.apiVersion`, `cinatra.kind`, `cinatra.dependencies`, npm metadata
- Depends on: Nothing (no runtime dependencies)
- Used by: Cinatra Marketplace submission pipeline and CI

**CI/CD layer:**
- Purpose: Gate correctness of extension shape and publish to Marketplace on release
- Location: `.github/workflows/ci.yml`, `.github/workflows/release.yml`
- Contains: First-party dependency shape checks, typecheck/test orchestration, dry-pack, marketplace release
- Depends on: `cinatra-ai/.github` reusable workflows (org-level)
- Used by: GitHub Actions on push/PR and GitHub Release events

## Data Flow

### Chat Widget Request Path

1. User sends a message in the Drupal node editor sidebar widget
2. Cinatra runtime injects `skills/drupal-widget-chat/SKILL.md` as the system prompt
3. LLM evaluates the message against decision rules in the skill prompt
4. If edit intent detected → calls `drupal_content_editor_run` with `instructions` only (runtime overrides `instanceId` and `nodeId` server-side)
5. Tool returns `{ nodeId, changes: [...] }` or `{ result: "<text>" }` or an error
6. LLM summarizes result in plain English per rules in `SKILL.md` — never pastes raw JSON

### Conversational Path

1. User sends a greeting, question, or ambiguous message
2. LLM determines no edit intent from rules in `SKILL.md`
3. LLM replies directly without calling any tool

### Release/Publish Path

1. GitHub Release is published with tag `v<version>`
2. `.github/workflows/release.yml` triggers
3. Delegates to `cinatra-ai/.github/.github/workflows/reusable-extension-release.yml@main`
4. Extension submitted to Cinatra Marketplace via MCP proxy review/approval saga

**State Management:**
- No client-side state. `instanceId` and `nodeId` context are pinned server-side by the Cinatra Drupal connector and forcibly injected into every tool call — the skill prompt never manages these values.

## Key Abstractions

**Skill (SKILL.md):**
- Purpose: A structured system-prompt document that encodes agent behavioral rules for a specific Cinatra widget context
- Examples: `skills/drupal-widget-chat/SKILL.md`
- Pattern: YAML front-matter (`name`, `description`) followed by Markdown sections defining decision trees, tool-call contracts, and response formatting rules

**Cinatra Extension Manifest (`cinatra` block in package.json):**
- Purpose: Identifies the npm package as a Cinatra extension of a specific kind (`skill`, `agent`, `workflow`, etc.)
- Examples: `package.json` → `cinatra.kind = "skill"`, `cinatra.apiVersion = "cinatra.ai/v1"`
- Pattern: Embedded in `package.json` under the `cinatra` key; parsed by CI and Marketplace pipelines

## Entry Points

**Skill injection:**
- Location: `skills/drupal-widget-chat/SKILL.md`
- Triggers: Every chat message in the Drupal widget sidebar; injected by Cinatra runtime as system prompt
- Responsibilities: Govern all LLM decisions — edit vs. converse, tool arguments, result summarization, error messaging

**CI entry point:**
- Location: `.github/workflows/ci.yml`
- Triggers: Push or PR to `main`
- Responsibilities: Classify repo (source mirror vs. standalone), validate first-party dep shape, typecheck, test, dry-pack

## Architectural Constraints

- **No src/ today:** `tsconfig.json` references `src/` as `rootDir` but no TypeScript sources exist yet; the CI correctly skips typecheck for content-only extensions with no tracked `.ts` files
- **Source mirror pattern:** First-party `@cinatra-ai/*` peers must appear in `peerDependencies` with `peerDependenciesMeta.optional = true` — CI enforces this and fails (exit 2) on violations
- **Tool argument restriction:** The skill prompt hard-rules that `instanceId` and `nodeId` must NEVER be included in tool call arguments; the runtime overrides them server-side
- **No direct Verdaccio publish:** Release goes through the Cinatra Marketplace MCP proxy saga, not a direct npm registry publish

## Anti-Patterns

### Passing instanceId/nodeId in tool arguments

**What happens:** Skill prompt explicitly forbids passing `instanceId` or `nodeId` to `drupal_content_editor_run`
**Why it's wrong:** The server forcibly overrides these values; including them is redundant and could surface internal IDs to users
**Do this instead:** Pass only `instructions` (the natural-language edit request) to the tool call

### Pasting raw tool-result JSON into replies

**What happens:** Tool returns structured JSON (`{ nodeId, changes: [...] }`)
**Why it's wrong:** The widget shows a diff panel separately; pasting JSON confuses users and breaks UX
**Do this instead:** Summarize in plain English per the examples in `skills/drupal-widget-chat/SKILL.md`

### Adding first-party deps to dependencies/devDependencies

**What happens:** CI `classify` step scans for `@cinatra-ai/*` or `@cinatra/*` in non-peer dep fields
**Why it's wrong:** These packages are not published to any registry; they live only in the monorepo
**Do this instead:** Declare them as `peerDependencies` with `peerDependenciesMeta[pkg].optional = true`

## Error Handling

**Strategy:** Skill prompt defines user-facing error messaging; no programmatic error handling exists in this repo (no runtime code).

**Patterns:**
- Tool call failure → reply: "I couldn't apply that edit — please try again, and let me know if it keeps failing."
- Internal error messages, stack traces, and HTTP status codes must never be surfaced to the user

## Cross-Cutting Concerns

**Logging:** Not applicable — no runtime code in this repo
**Validation:** CI validates extension package shape (dep classification script in `.github/workflows/ci.yml`)
**Authentication:** Not applicable at skill level; handled by Cinatra runtime and Drupal connector

---

*Architecture analysis: 2026-06-09*
