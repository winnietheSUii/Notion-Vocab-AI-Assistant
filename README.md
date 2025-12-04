# Notion Vocab AI Assistant

Self-hosted automation that enriches a Notion vocabulary database using n8n and a local Ollama model. It focuses on learning from category (part of speech), synonyms, antonyms, and usage examples – not from translations.

## What it does
- Watches a Notion vocabulary database (based on the **Vocabulary Book (English)** template) for rows where `AI_done` is unchecked.
- Sends the English word to a local LLM (default: `phi3:mini` via Ollama) with a strict JSON-only prompt.
- Writes back:
  - **Category** (POS label like `Verb`, `Noun`, etc.)
  - **Pronunciation** (IPA string)
  - **Synonyms** (comma-separated list)
  - **Antonyms** (comma-separated list)
  - **Example** (one natural sentence)
- Marks the row as `AI_done = true` so it is not processed again.

## Quick start

1. Clone the repo:
```powershell
cd C:\path\to\workspace
git clone https://github.com/winnietheSUii/Notion-Vocab-AI-Assistant.git
cd Notion-Vocab-AI-Assistant
```

2. Configure env:
```powershell
Copy-Item docker\env.example docker\.env
notepad docker\.env
```
Fill in:
- `NOTION_ACCESS_TOKEN`
- `NOTION_DATABASE_ID`
- `N8N_ENCRYPTION_KEY`, `N8N_BASIC_AUTH_USER`, `N8N_BASIC_AUTH_PASSWORD`

3. Start Docker stack:
```powershell
docker compose -f docker/docker-compose.yml --env-file docker/.env up -d
```

4. Import `n8n-workflow/workflow.json` into n8n, attach your Notion credential to the Notion nodes, and activate the workflow.

## Notion database
This project is designed around the public Notion template:

> Vocabulary Book (English) – https://www.notion.com/templates/vocabulary-book-english

Credit: all database structure inspiration and property names come from that template.

Required / used properties:
- `English` (title)
- `Category` (multi-select, used as POS target)
- `Synonyms` (text)
- `Antonyms` (text)
- `Example` (text)
- `Pronunciation` (text)
- `AI_done` (checkbox)

## Prompt + model behavior
The main Function node builds a prompt that:
- Restricts the model to JSON only (no markdown).
- Asks for `pos`, `ipa`, `synonyms`, `antonyms`, `example`.
- Explicitly forbids Thai or any other translation.

The Transform node parses the streamed NDJSON from Ollama, extracts the single JSON object, and maps it into Notion properties.

## License and credits
- Database template: **Vocabulary Book (English)** on Notion.
- Built for personal learning, running entirely on your own machine with open models via Ollama.
