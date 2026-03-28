# Discord Bot — n8n Workflow

A Discord slash command bot built entirely in [n8n](https://n8n.io). No custom code needed to add new commands — just create a subworkflow and register it in a lookup table.

## How it works

The main workflow receives Discord interactions via webhook, immediately sends a deferred response (`type: 5`) to satisfy Discord's 3-second timeout, then looks up the command name in an n8n DataTable to find the matching subworkflow. It executes that subworkflow, waits for the result, and posts the reply back to Discord.

```
Discord → Webhook → ACK (type:5) → DataTable lookup → Execute Subworkflow → Reply to Discord
```

## Setup

### Prerequisites

- A running n8n instance (self-hosted or n8n Cloud)
- A Discord application with slash commands registered in the [Discord Developer Portal](https://discord.com/developers/applications)

### 1. Import the workflows

Import all three JSON files into your n8n instance:

- Via UI: **Settings → Import Workflow**
- Via CLI: `n8n import:workflow --input=<file.json>`

After import, activate each workflow manually.

### 2. Create the DataTable

Create an n8n **DataTable** named `Discord Bot - Matching` with two columns:

| Column | Type | Description |
|---|---|---|
| `name` | String | The slash command name (e.g. `wownews`) |
| `subworkflow_id` | String | The n8n workflow ID of the subworkflow to run |

Open the main workflow (`Discord Bot Command`), go to the **Get row(s)** node, and point it to your newly created DataTable.

### 3. Connect Discord

In the Discord Developer Portal, set your bot's **Interactions Endpoint URL** to:

```
https://<your-n8n-instance>/webhook/<your-webhook-id>
```

The webhook ID is shown in the **Webhook** node of the main workflow after import.

### 4. Register your subworkflows

For each slash command, add one row to the DataTable with the command name and the n8n workflow ID of its subworkflow.

## Adding a new command

1. Create a new n8n workflow that starts with a **When Executed by Another Workflow** trigger
2. Do whatever you want — call an API, run some code, etc.
3. End with an **Edit Fields** node that maps your result to a field called `data` (this is the message text posted to Discord)
4. Add a row to the DataTable: `name` = your slash command name, `subworkflow_id` = the workflow ID

That's it. No changes to the main workflow needed.

## Example subworkflows

| File | What it does |
|---|---|
| `example subworkflows/Discord Bot - Subworkflow - No as a Service.json` | Returns a random witty "no" reason from [naas.daniilmira.com](https://naas.daniilmira.com/no) |
| `example subworkflows/Discord Bot - Subworkflow - WoW RSS Reader.json` | Fetches [Wowhead](https://www.wowhead.com) news and returns articles from the last 12 hours |

## Subworkflow contract

Every subworkflow must return a `data` field containing the message string to post to Discord:

```json
{ "data": "Your message here" }
```
