# competitor-intelligence — Smart Competitor Intelligence System

## Project Purpose

This n8n workflow automates weekly competitive intelligence using 5 AI agents (Groq Llama 3.1 70B, free tier).
It collects data from competitor websites and RSS feeds, runs it through a multi-agent analysis chain,
and delivers a polished strategic report to Google Drive, Discord, and Telegram.

## Key Files

- `workflow.json` — Import into n8n to deploy the complete workflow
- `README.md` — Full setup instructions (start here)
- `AGENTS.md` — Complete system prompts for all 5 agents with customization tips

## Architecture

5-agent chain: Scout → Analyst → Critic → Strategist → Writer
All LLM calls use Groq's free API (Llama 3.1 70B)
Data sources: 3 HTTP scraped sites + 2 RSS feeds + Google Sheets (historical comparison)
Delivery: Google Drive (full report) + Discord (summary) + Telegram (alert)

## When Modifying This Workflow

- Agent prompts live in Code nodes prefixed "Prepare [Agent] Request"
- All configurable values are in the "Set Weekly Config" Set node
- Credential placeholders: search for "REPLACE_WITH_CREDENTIAL_ID" after import
- Delivery nodes all have continueOnFail=true so one failure doesn't break others

## Testing

Run manually via n8n's "Execute Workflow" button.
First run will have no historical comparison data — this is expected.
Full run takes ~3-5 minutes (5 Groq API calls, each ~30-60 seconds).
