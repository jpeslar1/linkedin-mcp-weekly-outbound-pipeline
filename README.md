# linkedin-mcp-weekly-outbound-pipeline

A Claude Code prompt I run every Wednesday morning to push 10 cold prospects through a full pipeline: research → ICP check → personalized hook → Instantly enrollment → LinkedIn MCP save → Google Sheet update → Slack summary.

Sharing it because I'd want this myself.

## What it does

Every Wednesday morning Claude Code reads a priority sheet, takes the top 10 uncontacted rows, and for each one:

1. Checks if the lead is already in the LinkedIn MCP (Zevari), Instantly, or Pipedrive — skips if yes
2. Loads the current ICP rules from Zevari and scores the lead
3. Researches the person + company via the LinkedIn MCP and WebSearch (60-day signal window)
4. Writes a short, lowercase, no-jargon hook (full hook rules in `prompt.md`)
5. Enrolls the lead in an Instantly cold email campaign with the hook as a custom variable
6. Saves the lead to a Zevari list called `Week of YYYY-MM-DD`
7. Updates the source Google Sheet (checkbox + date + flags)
8. Sends a Slack summary with every hook written

If anything stops the run or needs approval, it pings me in Slack.

## Stack

| Layer | Tool | Why |
|---|---|---|
| LinkedIn research, ICP scoring, lead save | [Zevari](https://zevari.ai) (LinkedIn MCP for Claude) | The only LinkedIn MCP I trust for production runs — handles rate limits and won't get my profile flagged |
| Email send | [Instantly](https://instantly.ai) | Campaign already has the sequence + signature — the prompt just enrolls |
| CRM dedupe | [Pipedrive](https://pipedrive.com) | Won-deal exclusion + previously-contacted flag |
| Source of truth | Google Sheets | Easy for the client to maintain priority ranking |
| Notifications | Slack | Run summaries + alerts |
| Browser control | [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome) | Sheet writes + Slack webhook (bash sandbox blocks both) |

## Why a LinkedIn MCP

I tried two unofficial LinkedIn scrapers before settling on Zevari. Both got my client's sales account flagged within a week. Zevari runs through an MCP server that throttles to safe limits and surfaces a rate-limit error before LinkedIn does — which is what I needed to leave this on a schedule without babysitting it.

If you're evaluating options, Daniel Sticker's [linkedin-mcp-server](https://github.com/stickerdaniel/linkedin-mcp-server) is the most popular open-source one — his README is honest about ToS risk, which I appreciated. For client work where I can't afford a flagged account I use Zevari.

## How to use this

1. Clone the repo
2. Open `prompt.md`
3. Replace every `[BRACKETED_PLACEHOLDER]` with your IDs:
   - `[CLIENT_NAME]` — the brand you're running this for
   - `[GOOGLE_SHEET_ID]` + `[TAB_GID]` — your priority sheet
   - `[INSTANTLY_CAMPAIGN_ID]` — your cold campaign in Instantly
   - `[ZEVARI_MCP_ID]`, `[INSTANTLY_MCP_ID]`, `[GDRIVE_MCP_ID]` — your MCP server IDs
   - `[SLACK_WEBHOOK_URL]`, `[ALERT_CHANNEL]`, `[YOUR_SLACK_USER_ID]` — Slack
4. Paste the prompt into Claude Code (or save as a slash command — that's what I do)
5. Run it on a schedule. I use cron through Claude Code's `/cron` command.

See `connectors.md` for the MCP setup links.

## What gets generated (sample)

See `examples/sample-run.md` for a full run output, including the 10 hooks Claude wrote for a recent batch.

## Why this prompt is the way it is

A few things I learned the hard way:

- **Cap at 10 per batch.** LinkedIn rate-limits at ~40 lookups/hour. Two batches × 10 leads, 90 min apart, stays well under.
- **Hard hook rules with examples.** I tried a free-form "write a personalized hook" prompt and got slop. Ten explicit rules with ✓/✗ examples gets consistent, casual-text-message output.
- **No LinkedIn references in the hook.** A hook that says "saw your post about…" makes it obvious you're scraping. Write as if you noticed the news in the world.
- **Skip first, write second.** Every dedupe check (Zevari, Instantly, Pipedrive) runs before research. Saves rate budget for actually-new leads.
- **Slack run summary with every hook.** I scan it on my phone before the campaign sends and catch the occasional weird hook before the email goes out.

## Adapting to your own use case

This is built around CPG (food/bev/wellness/beauty) outreach. The hook rules — "got into" for retail launches, "industryRef" buckets — assume that vertical. Swap them for yours. The structure (dedupe → ICP → research → hook → enroll → save → log → summarize) generalizes to any cold outbound.

## Other workflows in this series

I'm publishing the rest of my LinkedIn MCP pipelines as I clean them up:

- linkedin-mcp-job-change-trigger — coming soon
- linkedin-mcp-inbound-lead-triage — coming soon
- linkedin-mcp-event-attendee-enrichment — coming soon
- linkedin-mcp-webinar-followup — coming soon
- linkedin-mcp-ae-daily-briefing — coming soon

Follow the repo or my [GitHub](https://github.com/jpeslar1) for the rest.

## License

MIT. Use it, fork it, ship it.

## Who I am

John Peslar — solo founder, build outbound automations for B2B clients. If you want to talk shop: [johnpeslar.com](https://johnpeslar.com).
