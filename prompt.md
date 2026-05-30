# Weekly Outbound Pipeline — Batch 1

A Claude Code prompt that runs a weekly batch of cold outbound for a CPG client. Uses the LinkedIn MCP (Zevari) for research + saving leads, Instantly for the email send, Pipedrive for CRM dedupe, Google Sheets as the source of truth, and Slack for run notifications.

This is the prompt I run every Wednesday morning. Copy it, swap the placeholders for your IDs, and you have a working pipeline.

---

## Overview
You are running the first of two weekly outbound pipeline batches for `[CLIENT_NAME]` (a CPG brand). Your job: research up to 10 uncontacted prospects from the priority sheet, qualify them against the ICP, write personalized hooks, enroll them in the Instantly cold email campaign, save them to the LinkedIn MCP (Zevari), update the tracking sheet, and send a Slack summary.

This task is intentionally capped at 10 leads to stay well under LinkedIn's ~40 lookups/hour rate limit. Batch 2 runs 90 minutes later and picks up the next 10.

---

## ERROR & APPROVAL NOTIFICATIONS
In two situations, send a Slack notification to `[ALERT_CHANNEL]`:
1. An error occurs that stops the task — send notification, then stop.
2. Approval is required — send notification at the same time the approval request appears.

How to send (bash sandbox blocks hooks.slack.com — use Chrome only):
1. Call `mcp__Claude_in_Chrome__tabs_context_mcp` with `createIfEmpty: true`
2. If the tab is on a `chrome://` URL, navigate to `https://google.com` first
3. Run via `mcp__Claude_in_Chrome__javascript_tool`:

```js
fetch('[SLACK_WEBHOOK_URL]', {
  method: 'POST',
  mode: 'no-cors',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    text: "<@[YOUR_SLACK_USER_ID]> ⚠️ *Weekly Pipeline Batch 1 — Action Needed*\n\n<describe the error or what approval is needed>"
  })
}).then(r => ({status: r.status, type: r.type})).catch(e => ({error: e.message}))
```

`{status: 0, type: "opaque"}` = success.

---

## CONNECTORS
- **Zevari (LinkedIn MCP):** `mcp__[ZEVARI_MCP_ID]` — research, ICP scoring, lead save
- **Pipedrive:** `mcp__pipedrive__` — CRM dedupe
- **Instantly:** `mcp__[INSTANTLY_MCP_ID]` — email campaign enrollment
- **Google Drive (read-only):** `mcp__[GDRIVE_MCP_ID]` — read the priority sheet
- **Chrome:** `mcp__Claude_in_Chrome__` — Sheet writes + Slack webhook
- **WebSearch:** for company news + signal hunting

See `connectors.md` for setup links.

---

## STEP 1 — LOAD SHEET & IDENTIFY LEADS

Google Sheet:
- ID: `[GOOGLE_SHEET_ID]`
- Tab: `"Contact - Prioritized"` (gid=`[TAB_GID]`)
- URL: `https://docs.google.com/spreadsheets/d/[GOOGLE_SHEET_ID]/edit#gid=[TAB_GID]`

Column layout:
- A = Contacted (checkbox — TRUE = already contacted, skip)
- B = Date Contacted (YYYY-MM-DD)
- C = Priority Rank (numeric, ascending = higher priority)
- D = In Pipedrive? (TRUE/FALSE)
- E = In Zevari? (TRUE = existed in Zevari before this run, was skipped)
- F = Priority Tier
- G = ICP Score
- H = Email
- I = First Name
- J = Last Name
- K = Company Name
- L = Company Website URL
- M = Current Job Title
- N = LinkedIn URL

Row = Priority Rank + 1 (Rank 1 = Row 2, Rank 5 = Row 6, etc.)

Read the sheet via Google Drive MCP. Collect rows where column A = FALSE. Sort by column C ascending. Take the first 10.

### Step 1b — Zevari pre-check
For each lead, call `leads_lookup_by_url` with their LinkedIn URL. If found in Zevari already:
- Mark column E = TRUE in sheet
- Skip — do not research or enroll

### Step 1c — Instantly blocklist check
Search the Instantly campaign for each lead's email using `list_leads` (campaign ID: `[INSTANTLY_CAMPAIGN_ID]`). If already present in any status, skip.

### Step 1d — Pipedrive CRM check
Search Pipedrive via `search_contacts` and `search_organizations` using the contact's name and company.
- `won_deals_count > 0` → flag "Won Client", mark D=TRUE, exclude from cold outreach
- Found but no won deals → mark D=TRUE, note as "Previously Contacted"
- Not found → mark D=FALSE, proceed

If Pipedrive is unavailable, flag it in the Slack summary and continue.

### Step 1e — ICP scoring
Call `profile_get` to load all current ICP criteria, hard disqualifiers, employee thresholds, company type rules, and company age rules. Never rely on hardcoded values — always apply what `profile_get` returns.

Call `agents_icp_score` for each remaining lead. If `agents_icp_score` returns `server_side_ai_unavailable`, apply ICP criteria manually using `profile_get` data — do not pause.

Apply all hard disqualifiers from `profile_get`. Remove any failing leads from the batch, do not enroll them, and log them in Slack.

If a lead fails ICP: leave the sheet row completely untouched (do not check the box, do not write a date).

---

## STEP 2 — RESEARCH
Call `agents_research_person` for each qualifying lead using their LinkedIn URL and company data.

- Process in batches of no more than 3 concurrent calls to respect LinkedIn rate limits
- This task is capped at 10 leads to stay under the ~40/hour cap
- If the cap is hit anyway: use website + WebSearch to write the best hook possible; flag the affected lead in the Slack summary — do not silently fall back to the warm opener

Also use WebSearch on the company website and recent news (target signals within 60 days) to find a hook angle.

---

## STEP 3 — WRITE EMAIL VARIABLES

Call `library_context_get` with `skill_slug='craft-outreach'` and `include_examples=true`. Do not write any variables until you have read this context — all hook, `companyRef`, and `industryRef` rules live there.

For each lead, determine: `hook`, `companyRef`, `industryRef`.

### Hook rules (all must be followed):

1. **Lowercase everything after the very first letter** — including brand names, retailer names, people's names, place names, certifications. The hook reads like a casual text message.
   - ✓ "Just saw thrilling got into whole foods in the southwest - congrats!"
   - ✓ "Saw atomo just launched a prebiotic coffee - congrats!"
   - ✗ "Just saw Thrilling got into Whole Foods in the southwest - congrats!"

2. **No LinkedIn references.** Write as if you noticed the news in the world, not on their feed. Never use: "I saw your post about…", "Saw your LinkedIn post…", or any phrasing naming the platform.

3. **Retail/distribution launches:** use "got into" — never "land", "landed on shelves", or "landed in."

4. **Reactions must be brief:** only "congrats!" or "congrats on the launch!" — nothing more. No editorial commentary or compliments.
   - ✗ "that's a strong differentiator!" ✗ "nice regional build!" ✗ "great initiative!" ✗ "impressive footprint!"

5. **Never use "its" for a brand.** Use the brand name or rephrase.

6. **Events:** drop the descriptor word (Expo, Show, Conference, Summit, Convention). Use core name only.
   - ✓ "sweets & snacks" ✗ "Sweets & Snacks Expo"

7. **Cut explanatory clauses.** State the observation and stop. Never explain why it matters.
   - ✓ "Saw atomo just launched a prebiotic coffee - congrats!"
   - ✗ "Saw atomo launched a prebiotic coffee - first brewable with added prebiotic fiber - congrats on the launch!"

8. **No jargon.** Plain language only.
   - ✓ "new production facility" ✗ "SQF-certified facility"

9. **Use "just" naturally** for variety and immediacy — don't force it every time, but don't avoid it.

10. **Warm opener fallback:** Use "I hope all is well." ONLY if no qualifying signal exists within 60 days. Do not fabricate hooks.

### companyRef rules:
- Short recognizable brand name, not the full legal name
- Title case (e.g., "Goat Fuel" not "GOAT FUEL" and not "G.O.A.T. Fuel, Inc.")

### industryRef rules:
- "food brands" for food companies
- "bev brands" for beverage companies
- "wellness brands" for supplement/wellness
- "beauty brands" for beauty/personal care

---

## STEP 4 — ENROLL IN INSTANTLY
Call `add_leads_to_campaign_or_list_bulk` to enroll all qualifying leads in campaign: `[INSTANTLY_CAMPAIGN_ID]` (`"[CLIENT_NAME] - Cold Outreach (Ongoing)"`).

Required fields: `email`, `first_name`, `last_name`, `company_name`, `custom_variables: {hook, companyRef, industryRef}`
Set `skip_if_in_campaign: true`.

The campaign already has the correct email sequence and signature. Do not create a new campaign or modify the sequence.

---

## STEP 5 — SAVE TO ZEVARI (LinkedIn MCP)
For each enrolled lead:
1. Call `targets_save` to save as a Zevari target
2. Create or find a list named `"Week of [today's date YYYY-MM-DD]"` using `targets_create_list`
3. Add each lead to that list

---

## STEP 6 — UPDATE GOOGLE SHEET
For each enrolled lead, write to the sheet:
- Column A (Contacted): TRUE (check the checkbox)
- Column B (Date Contacted): today's date as YYYY-MM-DD
- Column D (In Pipedrive?): TRUE or FALSE per Step 1d
- Column E (In Zevari?): TRUE only if the lead already existed in Zevari at Step 1b and was skipped — do NOT write TRUE for newly enrolled leads

Sheet writes must use Chrome computer tool (Google Drive MCP is read-only for sheets):
1. Navigate to the sheet URL in Chrome
2. Use the Name Box (top-left cell reference box) to navigate: click it, type the cell address (e.g. `"B5"`), press Enter
3. Checkboxes: left-click directly on the checkbox cell to toggle it TRUE
4. Text/date values: type the value, press Enter
5. Verification pass: after all writes, re-read each updated cell to confirm values are correct

---

## STEP 7 — SLACK RUN SUMMARY
Send a Slack summary via Chrome `javascript_tool` (not curl or Python).

Format:
```
<@[YOUR_SLACK_USER_ID]> ✅ *Weekly Pipeline — Batch 1 Complete*

📅 Run date: [date]
👥 Leads processed: [N]
✅ Enrolled: [N]
⏭️ Skipped (already contacted/blocklisted/in Zevari): [N]
❌ Disqualified (ICP fail): [N] — [names + reasons]
🔗 LinkedIn cap hit: yes/no — [affected leads if yes]
📋 Pipedrive matches: [names if any, or "none"]

Hooks written:
• [first name, company] — [hook]
[list all enrolled leads]
```

---

## STEP 8 — REFLECTION
After sending the summary, briefly note:
- Any hook rule violations or close calls
- Any leads where no signal was found (warm opener used)
- Any process improvements to flag for next run

---

## Schedule
Every Wednesday at ~10:34 AM (Batch 1). Batch 2 runs 90 minutes later.
