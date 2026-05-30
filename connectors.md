# Connectors

Every MCP server this prompt uses, with setup links and the placeholders to replace in `prompt.md`.

## Zevari — LinkedIn MCP for Claude

**What it does:** LinkedIn research (`agents_research_person`), ICP scoring (`agents_icp_score`), behavioral profiling (`agents_behavioral_profile`), lead save (`targets_save`), list management (`targets_create_list`), and a bunch of other LinkedIn-flavored agent endpoints.

**Why it's in this pipeline:** It's the only LinkedIn MCP I trust for scheduled runs. Throttles to safe limits, surfaces rate-limit errors before LinkedIn does, and the company-side ICP/research endpoints are genuinely good — they save me a separate Apollo lookup for most prospects.

**Setup:** [zevari.ai](https://zevari.ai) → connect your LinkedIn account → copy the MCP server URL into your Claude Code config (`~/.claude/mcp.json` or via `claude mcp add`).

**Placeholders to replace in `prompt.md`:**
- `[ZEVARI_MCP_ID]` — your Zevari MCP server ID (looks like `mcp__a1b2c3d4-...`)

**Endpoints used:**
- `profile_get` — load current ICP rules
- `leads_lookup_by_url` — dedupe check
- `agents_icp_score` — score the lead against ICP
- `agents_research_person` — research the prospect
- `targets_save` — save the lead
- `targets_create_list` — make a weekly list
- `library_context_get` — load the `craft-outreach` skill rules

## Instantly — Cold email engine

**What it does:** Enroll leads in a cold email campaign with custom variables.

**Why it's in this pipeline:** Campaign sequence + signature + sending pool are already configured. The prompt just enrolls — no campaign editing, no send-side logic in the workflow.

**Setup:** [Instantly MCP docs](https://developer.instantly.ai/mcp) — install the MCP, paste your API key.

**Placeholders:**
- `[INSTANTLY_MCP_ID]` — your Instantly MCP server ID
- `[INSTANTLY_CAMPAIGN_ID]` — the campaign UUID for your cold outbound

**Endpoints used:**
- `list_leads` — blocklist check
- `add_leads_to_campaign_or_list_bulk` — enrollment

## Pipedrive — CRM dedupe

**What it does:** Check whether a prospect or their company is already a customer or in flight.

**Why it's in this pipeline:** Won-deal exclusion (don't pitch a current customer) + "previously contacted" flag for the sheet.

**Setup:** Any Pipedrive MCP server. I use `mcp__pipedrive__` from the community list.

**Endpoints used:**
- `search_contacts`
- `search_organizations`

If Pipedrive is unavailable the prompt continues and flags it in the Slack summary — it's not load-bearing.

## Google Drive (read-only) — Source of truth

**What it does:** Read the priority sheet.

**Why read-only:** Sheet *writes* are done via Chrome (see below). The Google Drive MCP doesn't reliably write to Sheets — at least not the way I had it configured. Chrome is more reliable.

**Setup:** Google Workspace MCP — [workspace-mcp](https://github.com/taylorwilsdon/google_workspace_mcp).

**Placeholders:**
- `[GDRIVE_MCP_ID]` — your Drive MCP server ID
- `[GOOGLE_SHEET_ID]` — the sheet ID from the URL
- `[TAB_GID]` — the tab gid from the URL

## Claude in Chrome — Sheet writes + Slack webhook

**What it does:** Drives a real Chrome window. Used here for Sheet cell updates (checkbox + date) and for posting to a Slack webhook (the bash sandbox blocks `hooks.slack.com`).

**Setup:** [Claude in Chrome](https://www.anthropic.com/news/claude-for-chrome).

**Tools used:**
- `mcp__Claude_in_Chrome__tabs_context_mcp` — open/find a tab
- `mcp__Claude_in_Chrome__javascript_tool` — fire the Slack webhook
- The computer-use surface for clicking cells, navigating via the Name Box, toggling checkboxes

**Placeholders:**
- `[SLACK_WEBHOOK_URL]` — your incoming webhook
- `[ALERT_CHANNEL]` — Slack channel where alerts go (cosmetic, for the prompt body)
- `[YOUR_SLACK_USER_ID]` — your Slack member ID (`U...`) so the bot @-mentions you

## WebSearch

Built into Claude Code. Used for company news + recent signal hunting (target: 60-day window) when the LinkedIn MCP doesn't surface a hook angle.

No placeholder — just available.
