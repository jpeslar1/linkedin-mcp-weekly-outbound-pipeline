# Sample run

A real run of the pipeline (sanitized — names changed) from a recent Wednesday.

## Setup

- Sheet had 47 uncontacted rows
- Top 10 by priority rank pulled into the batch
- All connectors healthy

## Step 1 results

| Lead | LinkedIn check (Zevari) | Instantly blocklist | Pipedrive | ICP score | Action |
|---|---|---|---|---|---|
| A. Chen — atomo | not in Zevari | not in campaign | not found | 8/10 | proceed |
| B. Rivera — thrilling | not in Zevari | not in campaign | not found | 9/10 | proceed |
| C. Park — magic mind | already in Zevari | — | — | — | **skip** (E=TRUE) |
| D. Lopez — olipop | not in Zevari | not in campaign | found, no won deals | 8/10 | proceed (D=TRUE) |
| E. Khan — recess | not in Zevari | not in campaign | won_deals=2 | — | **skip — current customer** |
| F. Singh — ghia | not in Zevari | not in campaign | not found | 7/10 | proceed |
| G. Yamada — graza | not in Zevari | not in campaign | not found | 9/10 | proceed |
| H. Brooks — fishwife | not in Zevari | not in campaign | not found | 6/10 | **DQ — under employee threshold** |
| I. Adeyemi — goat fuel | not in Zevari | not in campaign | not found | 8/10 | proceed |
| J. Tanaka — perfy | not in Zevari | not in campaign | not found | 7/10 | proceed |

**Net:** 7 proceed, 1 skipped (Zevari), 1 skipped (current customer), 1 DQ.

## Step 3 — hooks written

```
A. Chen, atomo
hook: Saw atomo just launched a prebiotic coffee - congrats!
companyRef: Atomo
industryRef: bev brands

B. Rivera, thrilling
hook: Just saw thrilling got into whole foods in the southwest - congrats!
companyRef: Thrilling
industryRef: food brands

D. Lopez, olipop
hook: Saw olipop just dropped the cherry vanilla flavor - congrats on the launch!
companyRef: OLIPOP
industryRef: bev brands

F. Singh, ghia
hook: Just saw ghia got into erewhon - congrats!
companyRef: Ghia
industryRef: bev brands

G. Yamada, graza
hook: Saw graza just opened a new production facility in spain - congrats!
companyRef: Graza
industryRef: food brands

I. Adeyemi, goat fuel
hook: Saw goat fuel just hit sweets & snacks - congrats!
companyRef: Goat Fuel
industryRef: wellness brands

J. Tanaka, perfy
hook: I hope all is well.
companyRef: Perfy
industryRef: bev brands
```

(J. Tanaka had no signals in the 60-day window — fell back to the warm opener. Flagged in the summary.)

## Step 4-6 — enrolled, saved, sheet updated

All 7 enrolled in the Instantly campaign. All 7 saved to a Zevari list called `Week of 2026-05-27`. Sheet rows checked, dated, flagged.

## Step 7 — Slack summary posted

```
✅ Weekly Pipeline — Batch 1 Complete

📅 Run date: 2026-05-27
👥 Leads processed: 10
✅ Enrolled: 7
⏭️ Skipped (already contacted/blocklisted/in Zevari): 2
❌ Disqualified (ICP fail): 1 — H. Brooks (under employee threshold)
🔗 LinkedIn cap hit: no
📋 Pipedrive matches: D. Lopez (previously contacted), E. Khan (current customer — excluded)

Hooks written:
• A. Chen, atomo — Saw atomo just launched a prebiotic coffee - congrats!
• B. Rivera, thrilling — Just saw thrilling got into whole foods in the southwest - congrats!
• D. Lopez, olipop — Saw olipop just dropped the cherry vanilla flavor - congrats on the launch!
• F. Singh, ghia — Just saw ghia got into erewhon - congrats!
• G. Yamada, graza — Saw graza just opened a new production facility in spain - congrats!
• I. Adeyemi, goat fuel — Saw goat fuel just hit sweets & snacks - congrats!
• J. Tanaka, perfy — I hope all is well. (no signal in 60-day window)
```

## Step 8 — reflection notes

- One warm opener used (J. Tanaka). Worth checking whether perfy has news on retailers' sites — I might have missed something.
- All 6 signal-based hooks passed the lowercase/no-LinkedIn-reference rules on first pass.
- E. Khan caught by Pipedrive — good, would have been embarrassing to cold-email a current customer.
- Total run time: ~14 min.
