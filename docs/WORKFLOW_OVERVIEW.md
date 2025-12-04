# Workflow overview

The `n8n-workflow/workflow.json` file defines the entire automation responsible for enriching Notion vocabulary entries. This document explains each component so you can extend or troubleshoot it.

## High-level flow
1. **Notion Trigger** watches the database for pages where `AI_done` remains unchecked.
2. **Prepare Payload (Function)** extracts the English title, builds the structured prompt template, and carries forward the page ID.
3. **Call Ollama (HTTP Request)** sends the prompt to your self-hosted model (phi3:mini by default) via `http://ollama:11434/api/generate`.
4. **Transform AI JSON (Function)** parses the model response, validates the JSON structure, and maps it into Notion property objects.
5. **Update Notion Page** writes the enrichment back into Notion and toggles the `AI_done` checkbox so the row is no longer processed.

## Node details
- **Notion Trigger**
  - Poll interval: 5 minutes (change `options.pollTime`).
  - Filter: `AI_done` checkbox equals `false`.
  - Outputs raw Notion page JSON including every property.
- **Prepare Payload**
  - JavaScript function that safely reads the English title and skips empty rows.
  - Injects the `prompts/vocab_enrichment_prompt.txt` template (mirrored inline for convenience) and replaces `{{word}}` with the headword.
  - Resulting items contain `{ pageId, word, prompt }`.
- **Call Ollama**
  - POST body: `{ model, prompt, format: "json", stream: false }`.
  - URL uses `$env.OLLAMA_BASE_URL` so you can point to another host/port without editing the node.
  - Expects the Ollama response to contain a `response` string that itself is JSON.
- **Transform AI JSON**
  - Fetches the matching `Prepare Payload` item using `$items('Prepare Payload', 0, index)`.
  - Parses the response, normalizes synonyms, and constructs the property payload exactly how the Notion node expects it.
  - Throws a descriptive error if the JSON is malformed, making debugging easier in the n8n execution log.
- **Update Notion Page**
  - Runs in JSON mode so the workflow can pass a whole `properties` object.
  - Requires you to assign the same Notion credential used by the trigger.

## Error handling tips
- If Ollama returns non-JSON text, the "Transform AI JSON" node will fail fast. Inspect the execution data to see the raw `response` string.
- When the Notion Update node reports validation errors, verify that your database property names match exactly (they are case-sensitive).
- Use n8n's **Retry** feature on failed executions after resolving the issue; the trigger filter prevents duplicate updates because `AI_done` remains unchecked on failed rows.

## Customization ideas
- Change `pollTime` or set the trigger to "every X minutes" for slower hardware.
- Swap models (`mistral`, `llama3`, etc.) by editing `OLLAMA_MODEL` in `docker/.env`, then restart the stack.
- Extend the prompt and workflow to capture CEFR level, etymology, audio URLs, or spaced-repetition scheduling data.
- Add a Slack/Email node after "Update Notion Page" to notify you when new words are ready.
