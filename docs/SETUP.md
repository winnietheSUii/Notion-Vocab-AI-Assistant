# notion-vocab-ai-automation setup

This guide walks you through running n8n and Ollama locally so the workflow can enrich Notion vocabulary entries without paid APIs.

## Prerequisites
- Docker Desktop 4.27+ with at least 6 GB RAM available
- Git (optional but recommended)
- A Notion integration token with read/write access to your vocabulary database (see `docs/NOTION_SETUP.md`)

## 1. Clone and configure
```powershell
# From any folder
cd c:\path\to\workspace
git clone https://github.com/<you>/notion-vocab-ai-automation.git
cd notion-vocab-ai-automation
```

Copy the example environment file and fill in secrets:
```powershell
Copy-Item docker\env.example docker\.env
notepad docker\.env
```
Update the following keys:
- `N8N_ENCRYPTION_KEY`: random 32+ character string
- `N8N_BASIC_AUTH_USER` / `N8N_BASIC_AUTH_PASSWORD`: credentials for the n8n UI
- `NOTION_ACCESS_TOKEN` and `NOTION_DATABASE_ID`: from your Notion integration

## 2. Start the stack
```powershell
docker compose -f docker/docker-compose.yml --env-file docker/.env up -d
```
First boot pulls the `ollama/ollama` image and the `phi3:mini` model, so it can take several minutes.

Check container status:
```powershell
docker compose -f docker/docker-compose.yml ps
```

## 3. Import the n8n workflow
1. Open http://localhost:5678 and sign in with the basic auth credentials you set.
2. Create a Notion credential inside n8n that uses the integration token.
3. Import `n8n-workflow/workflow.json` (Settings → Import from file).
4. Map the credential inside the Notion Trigger and Notion Update nodes.
5. Review the HTTP node URL and model fields if you changed the defaults.

## 4. Test the round trip
1. Duplicate a row in your Notion database with `AI_done` unchecked.
2. Click **Execute Workflow** in n8n to run once (or activate to poll every 5 minutes).
3. Confirm that the row is enriched with Part of Speech, IPA, Thai meaning, synonyms, example, and `AI_done = true`.

## 5. Useful commands
```powershell
# Tail n8n logs
docker compose -f docker/docker-compose.yml logs -f n8n

# Tail Ollama logs
docker compose -f docker/docker-compose.yml logs -f ollama

# Stop everything
docker compose -f docker/docker-compose.yml down
```

## 6. Next steps
- Adjust `prompts/vocab_enrichment_prompt.txt` if you want extra metadata.
- Swap models by setting `OLLAMA_MODEL` in `docker/.env` and restarting.
- Fork this repo and add CI to lint the workflow JSON if needed.
