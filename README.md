# Smart Competitor Intelligence System

An automated weekly competitor intelligence system built on n8n with 5 AI agents powered by Groq's free Llama 3.1 70B model.

## What It Does

Every Sunday at 8:00 PM, this workflow:

1. **Scrapes** 3 competitor websites and 2 RSS feeds (Product Hunt + Hacker News)
2. **Compares** this week's data against last week's (stored in Google Sheets)
3. **Runs 5 AI agents** in sequence, each building on the previous:
   - **Scout** — Organizes raw data into a structured intelligence brief
   - **Analyst** — Deep analysis: threats, opportunities, competitor trajectories
   - **Critic** — Quality control: challenges unsupported claims, recalibrates scores
   - **Strategist** — Actionable recommendations: immediate, short-term, medium-term
   - **Writer** — Polished Markdown report for executive stakeholders
4. **Delivers** the report to Google Drive, Discord, and Telegram

**Total cost: $0** — All tools are free tier.

---

## Architecture

```
Schedule (Sun 8PM)
    │
    ├──► Fetch Competitor A ──► HTML Extract ──┐
    ├──► Fetch Competitor B ──► HTML Extract ──┤
    ├──► Fetch Competitor C ──► HTML Extract ──┤──► Aggregate ──► Merge ──► Compare
    ├──► RSS Product Hunt ────────────────────┤               │
    ├──► RSS Industry News ───────────────────┘               │
    └──► Google Sheets Read (prev week) ──────────────────────┘
                                                              │
                                        Scout Agent [Groq] ◄─┘
                                              │
                                        Analyst Agent [Groq]
                                              │
                                        Critic Agent [Groq]
                                              │
                                        Strategist Agent [Groq]
                                              │
                                        Writer Agent [Groq]
                                              │
                              ┌───────────────┼───────────────┐
                              ▼               ▼               ▼
                        Google Drive    Discord Post    Telegram Alert
                        Google Sheets Log
```

---

## Setup Instructions

### Step 1: Prerequisites

- n8n instance (self-hosted or cloud)
- Free accounts needed:
  - [Groq](https://console.groq.com) — Free API key (no credit card)
  - [Google](https://google.com) — For Sheets and Drive
  - [Telegram](https://telegram.org) — Bot for notifications
  - [Discord](https://discord.com) — Webhook for team notifications (optional)

---

### Step 2: Create Credentials in n8n

Go to **n8n → Settings → Credentials → New Credential**

#### 2a. Groq API Key

1. Go to [console.groq.com](https://console.groq.com) → API Keys → Create Key
2. In n8n, create: **HTTP Header Auth**
   - **Name**: `Groq API`
   - **Header Auth Name**: `Authorization`
   - **Header Auth Value**: `Bearer gsk_YOUR_KEY_HERE`

#### 2b. Google Sheets

1. In n8n, create: **Google Sheets OAuth2**
2. Follow OAuth flow — authorize your Google account
3. Name it: `Google Sheets OAuth2`

#### 2c. Google Drive

1. In n8n, create: **Google Drive OAuth2**
2. Follow OAuth flow
3. Name it: `Google Drive OAuth2`

#### 2d. Telegram Bot

1. Message [@BotFather](https://t.me/BotFather) → `/newbot`
2. Copy the token
3. Get your Chat ID: message [@userinfobot](https://t.me/userinfobot)
4. In n8n, create: **Telegram API**
   - **Name**: `Telegram Bot`
   - **Access Token**: your bot token

#### 2e. Discord Webhook (optional)

1. Discord channel → Settings → Integrations → Webhooks → New Webhook
2. Copy the webhook URL
3. No n8n credential needed — URL goes directly in config

---

### Step 3: Set Up Google Sheets

Create a new Google Sheet with **two tabs**:

**Tab 1: `RawData`**
| weekDate | source | competitor | title | content | url | pubDate |
|----------|--------|------------|-------|---------|-----|---------|

**Tab 2: `Reports`**
| weekDate | reportTitle | totalSourcesCollected | topThreatsCount | topOpportunitiesCount | analysisSummary | totalTokensUsed | agentsCompleted | reportUrl | generatedAt |
|----------|-------------|----------------------|-----------------|-----------------------|-----------------|-----------------|-----------------|-----------|-------------|

Copy the Google Sheets ID from the URL:
`https://docs.google.com/spreadsheets/d/`**`THIS_IS_YOUR_ID`**`/edit`

---

### Step 4: Set Up Google Drive Folder

1. Create a folder in Google Drive for reports
2. Open the folder — copy the ID from the URL:
   `https://drive.google.com/drive/folders/`**`THIS_IS_YOUR_FOLDER_ID`**

---

### Step 5: Import the Workflow

1. In n8n, go to **Workflows → Import from File**
2. Select `workflow.json` from this repository
3. The workflow will be imported in **inactive** state

---

### Step 6: Configure the `Set Weekly Config` Node

Open the workflow and click **"Set Weekly Config"**. Update these values:

| Field | Value |
|-------|-------|
| `yourCompanyName` | Your actual company name |
| `yourIndustry` | Your industry (e.g., "SaaS / HR Tech") |
| `googleSheetsId` | Your Google Sheets ID from Step 3 |
| `googleDriveFolderId` | Your Google Drive folder ID from Step 4 |
| `discordWebhookUrl` | Your Discord webhook URL (or leave placeholder) |
| `telegramChatId` | Your Telegram chat ID from Step 2d |
| `competitor1Name` | First competitor's name |
| `competitor1Url` | URL of their pricing or features page |
| `competitor2Name` | Second competitor's name |
| `competitor2Url` | Their URL |
| `competitor3Name` | Third competitor's name |
| `competitor3Url` | Their URL |
| `productHuntRssUrl` | Keep default or change to relevant RSS |
| `industryRssUrl` | Keep default (HN) or add your industry RSS |

---

### Step 7: Update Credential References

After importing, n8n will show warnings about credentials. For each Groq HTTP Request node:
1. Click the node → **Credentials** tab
2. Select your `Groq API` credential

For Google Sheets nodes, select `Google Sheets OAuth2`.
For Google Drive node, select `Google Drive OAuth2`.
For Telegram nodes, select `Telegram Bot`.

---

### Step 8: Test the Workflow

1. Click **"Execute Workflow"** (manual trigger) to test
2. Watch execution — each phase takes 15-30 seconds per Groq call
3. Total runtime: ~3-5 minutes
4. Check Google Drive for your first report
5. Check Telegram/Discord for the summary

---

### Step 9: Activate

Once tested successfully:
1. Toggle **Active** in the top-right corner
2. The workflow will now run automatically every Sunday at 8:00 PM

---

## Customization

### Adding More Competitors

Add more HTTP Request + HTML Extract node pairs, connect them to `Aggregate All Sources`, and add their names/URLs to `Set Weekly Config`.

### Using a Different LLM

Change `groqModel` in `Set Weekly Config` to any Groq-supported model:
- `llama-3.1-70b-versatile` (recommended, best quality)
- `llama-3.1-8b-instant` (faster, lower quality)
- `mixtral-8x7b-32768` (good for longer context)
- `gemma2-9b-it` (Google's model, also free)

### Changing Schedule

Click the **Schedule Trigger** node and adjust the timing. Default: every Sunday at 20:00.

### Human-in-the-Loop (Optional)

To add manual approval before delivery:
1. Add a **Wait** node after `Format Final Report`
2. Configure it to wait for a webhook
3. Send the webhook URL to Telegram
4. When you approve, hit the webhook → workflow continues to delivery

---

## Token Usage & Free Tier Notes

Each weekly run makes **5 Groq API calls**. Estimated token usage:

| Agent | Input Tokens | Output Tokens |
|-------|-------------|---------------|
| Scout | ~2,000 | ~800 |
| Analyst | ~1,500 | ~1,000 |
| Critic | ~2,500 | ~800 |
| Strategist | ~3,000 | ~1,200 |
| Writer | ~4,000 | ~2,000 |
| **Total** | **~13,000** | **~5,800** |

Groq free tier allows **14,400 tokens/minute** for Llama 3.1 70B.
One weekly run = ~19,000 total tokens — well within free tier limits.

---

## Troubleshooting

**"Groq API returned 429"** — Rate limited. Add a Wait node (60s) between agent calls.

**"Google Sheets not found"** — Double-check your Sheets ID in the config node.

**"HTML Extract returns empty"** — The competitor site likely blocks scrapers. Try different CSS selectors or add a delay.

**"First run has no comparison data"** — Normal. The comparison feature activates from week 2 onwards.

**Telegram not sending** — Verify your bot has sent at least one message to your chat first, then get the chat ID from `@userinfobot`.

---

## File Structure

```
competitor-intelligence/
├── workflow.json    ← Import this into n8n
├── README.md        ← This file
├── AGENTS.md        ← Full system prompts for all 5 agents
└── CLAUDE.md        ← Claude Code project context
```

---

## License

MIT — Free to use, modify, and distribute.
