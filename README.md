# Cinatra Drupal Skills

The conversational skill behind the Cinatra chat widget that lives inside the Drupal node editor. It decides on every message whether you are asking for a content change to the current node or just having a conversation, calls the Drupal content editor when an edit is intended, and reports the result in plain English instead of pasting raw tool output back into the chat.

**Install:** Add `@cinatra-ai/drupal-skills` as a workspace-level skill in your Cinatra instance. No extra credentials are required beyond the Drupal connector already configured on the instance.

**Usage:** Open the chat widget inside a Drupal node editor and type a plain-language instruction such as "Tighten the intro and fix the title casing". The skill calls the content-editor tool and replies with a plain-English summary of the changes applied. To ask without triggering an edit, phrase it as a question. For multi-field rewrites or translations, expect 30 to 90 seconds; the skill warns you before long operations.

**Configuration:** The skill reads `instanceId` and `nodeId` from the platform request context automatically. No message-level configuration is required. Ensure the Cinatra Drupal connector is installed and authenticated before using this skill.

**Development:** Clone the repository and run `node extension-kind-gate.mjs` to validate the manifest. The skill definition lives in `skills/drupal-widget-chat/SKILL.md`. Update that file and re-register the skill in a development Cinatra instance to test changes.

**Troubleshooting:** If the skill replies "I couldn't apply that edit", the Drupal content-editor tool returned an error — verify the connector configuration and that the Drupal connector endpoint is reachable. If edits appear to do nothing, confirm the connector has write permission for the target content type.

## Works with

- Drupal
- Cinatra Drupal connector

## Capabilities

- Edit the current Drupal node from a plain-language request — rewrite, tighten, translate, change the title, restructure
- Answer questions about the node and the editor without unnecessarily triggering an edit
- Ask one short clarifying question when an edit request is genuinely ambiguous
- Summarize editor results in human language instead of dumping raw JSON
- Warn before long multi-field edits that may take 30 to 90 seconds
- Surface errors cleanly without leaking stack traces or internal identifiers
