# Notion configuration

Follow these steps to prepare a database that the automation can read and update.

## 1. Create a Notion integration
1. Visit https://www.notion.so/my-integrations.
2. Click **New integration** → choose a workspace.
3. Enable capabilities: `Read content`, `Update content`, `Insert content`, `Read user information (optional)`.
4. Copy the "Internal Integration Token" and store it somewhere secure. This value becomes `NOTION_ACCESS_TOKEN`.

## 2. Build the database schema
Create a database named "Vocabulary" (table layout works best) with the following properties:

| Property        | Type        | Required | Notes |
|-----------------|-------------|----------|-------|
| English         | Title       | Yes      | Enter the English headword here. |
| Category        | Select      | Optional | Pre-populate common parts of speech (noun, verb, adjective, adverb, phrasal verb). |
| Pronunciation   | Rich text   | Optional | Stores IPA string such as /ˈwɜːd/. |
| Thai Meaning    | Rich text   | Optional | Native-language gloss produced by the workflow. |
| Synonyms        | Rich text   | Optional | Comma-separated synonyms. |
| Example         | Rich text   | Optional | Example sentence using the headword. |
| AI_done         | Checkbox    | Yes      | Must default to unchecked. Used to skip already-processed rows. |

You can add additional helper properties (e.g., `Tags`, `Level`) if you want; the workflow will simply leave them untouched.

## 3. Share the database with the integration
1. Open the database page.
2. Click **Share** → **Invite**.
3. Search for your integration name and grant it **Can edit** access.

## 4. Collect IDs
- Database ID: open the database in a browser, copy the 32-character identifier from the URL (between the last `/` and `?`), and paste it into `NOTION_DATABASE_ID` in `docker/.env`.
- Token: paste the integration token into `NOTION_ACCESS_TOKEN` in `docker/.env`.

## 5. Testing tips
- Create a sample row with only the English title filled in; leave `AI_done` unchecked.
- After you activate the workflow, new rows should populate within one polling cycle (default 5 minutes).
- If nothing happens, check the n8n execution list and confirm the trigger filters (`AI_done == false`).
