# Cinatra Drupal Skills

The conversational skill behind the Cinatra chat widget that lives inside the Drupal node editor. It decides on every message whether you are asking for a content change to the current node or just having a conversation, calls the Drupal content editor when an edit is intended, and reports the result in plain English instead of pasting raw tool output back into the chat.

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
