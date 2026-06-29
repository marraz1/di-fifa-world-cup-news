---
description: Add a new report to the FIFA World Cup portal. Asks for report details and content, creates the HTML file, updates data/reports.json. The previous report of the same type automatically moves to the Archive section — no extra steps needed.
---

# Skill: Add New Report

You are helping the user publish a new report to the FIFA World Cup 2026 news portal.

## Step 1 — Collect metadata

Ask the user for the following (ask everything in one message):

1. **Report type** — one of: `naujienos` (📰 News), `rezultatai` (⚽ Results), `rungtynes` (📅 Fixtures), `media` (📸 Media)
2. **Date** — YYYY-MM-DD format. Default: today's date from context (currentDate in system prompt).
3. **Lithuanian title** — short title for the card (e.g. "Naujienos Trumpai")
4. **English title** — short title for the card (e.g. "Brief News")
5. **Author** — default is `FIFA analitikas`
6. **Report HTML content** — ask the user to paste the full HTML body content, or attach the file. The content should be a complete HTML page (with `<!DOCTYPE html>`, `<head>`, `<body>`, etc.) following the same style as existing reports.

If the user already provided some of this in their message, use those values and only ask for what's missing.

## Step 2 — Create the report HTML file

File path: `reports/<type>_<date>.html`

Example: `reports/naujienos_2026-06-22.html`

- If the user pasted complete HTML, write it as-is.
- If the user only pasted body content (no `<!DOCTYPE>`), wrap it using the standard template below.

### Standard template (use only if user didn't provide full HTML):

```html
<!DOCTYPE html>
<html lang="lt">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FIFA 2026 – {LT_TITLE} · {DATE}</title>
<style>
  :root {
    --dark: #0d1b2a; --gold: #c9a227; --mid: #1e3a5f; --light: #f0f4f8;
    --red: #c0392b; --blue: #2980b9; --green: #27ae60; --orange: #e67e22; --purple: #8e44ad;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: var(--dark); color: var(--light); font-family: 'Segoe UI', Arial, sans-serif; line-height: 1.5; }
  a { color: var(--gold); text-decoration: none; }
  .header {
    background: linear-gradient(135deg, #0a1628 0%, #1e3a5f 50%, #0a1628 100%);
    border-bottom: 3px solid var(--gold); padding: 1.5rem 1rem; text-align: center;
  }
  .header h1 { font-size: 1.8rem; color: var(--gold); letter-spacing: 1px; }
  .header .sub { color: #aac4e0; font-size: 0.9rem; margin-top: 0.3rem; }
  .section { max-width: 1200px; margin: 0 auto; padding: 1.4rem 1rem; }
  .section-title {
    font-size: 1.25rem; font-weight: 700; color: var(--gold);
    border-bottom: 2px solid var(--gold); padding-bottom: 0.4rem; margin-bottom: 1.1rem;
  }
  footer { background: #070f1a; border-top: 2px solid var(--gold); padding: 1rem; text-align: center; color: #708090; font-size: 0.75rem; margin-top: 2rem; }
</style>
</head>
<body>

<div class="header">
  <div style="font-size:2rem;">{ICON}</div>
  <h1>FIFA 2026 – {LT_TITLE}</h1>
  <div class="sub">{DATE_FORMATTED}</div>
</div>

{BODY_CONTENT}

<footer>
  <p>FIFA Pasaulio Čempionatas 2026™ – {LT_TITLE} · {DATE_FORMATTED}</p>
</footer>

</body>
</html>
```

Icons by type: `naujienos` → 📰, `rezultatai` → ⚽, `rungtynes` → 📅, `media` → 📸

## Step 3 — Update data/reports.json

Read the current `data/reports.json` and prepend a new entry at the top of the array:

```json
{
  "date": "<DATE>",
  "type": "<TYPE>",
  "title": "<LT_TITLE>",
  "titleEN": "<EN_TITLE>",
  "author": "<AUTHOR>",
  "file": "reports/<TYPE>_<DATE>.html"
}
```

The file is sorted newest-first; prepending keeps the order correct as long as the new date is the latest.

## Step 3.5 — Auto-update knockout bracket from the uploaded report

After updating `reports.json`, **automatically** read both the just-created report HTML and `data/bracket.json` in parallel. Extract bracket-relevant information from the HTML without asking the user first.

### How to extract results from HTML

Read the full text content of the report HTML. For each match in `bracket.json` whose `status` is not `"finished"`:

1. Check whether **both team names** (Lithuanian `name` field, or common shortened forms) appear in the HTML near each other.
2. Look for a **score pattern** nearby: `X–Y`, `X:Y`, `X-Y`, `X — Y` (where X and Y are digits). Also look for phrases like "X–Y … nugalėjo" (won), "laimėjo" (won), "įveikė" (defeated).
3. If a clear winner + score is found, that match has a confirmed result.

Be flexible with name matching — e.g. "P. Afrika", "Pietų Afrika", "South Africa" all refer to the same team. Match against both `name` (Lithuanian) and `nameEN` (English) fields.

### For `rezultatai` (results) or `naujienos` (news) reports

1. Scan the HTML and identify all knockout matches with confirmed scores.
2. For each confirmed match result, find it in `bracket.json` by matching team names to a bracket slot:
   - Set `score1`, `score2` (integers), `winner` (Lithuanian `name`), `status: "finished"`
3. Advance the winner to the next round:
   - Find which R16/QF/SF match has `feedsFrom` containing this match's ID.
   - Fill `team1` if this match is the lower-indexed entry in `feedsFrom`, else `team2`.
   - If both team slots in the next-round match are now filled, set its `status` to `"scheduled"`.
4. If finishing an SF: loser → `bronze.match` (team1 or team2 by same lower/higher index rule), winner → `final.match`.
5. Write the updated `data/bracket.json`.

### For `rungtynes` (fixtures) reports

1. Scan the HTML for upcoming knockout matches: team names, date, time, venue, prediction percentages.
2. For each match found:
   - If `date`, `time`, or `venue` in `bracket.json` differ from what's in the report, update them.
   - If probability bars are shown (e.g. "72%"), update `prediction.team1pct` / `prediction.team2pct` and `prediction.winner`.
3. Write the updated `data/bracket.json` if anything changed.

### For `media` reports

Skip bracket update — media reports don't contain match data.

### After updating (or if nothing changed)

Do **not** ask the user for manual input. Simply report in Step 4 what was found and updated (or "no bracket changes detected" if nothing matched).

## Step 4 — Confirm

Tell the user:
- What file was created
- That the new report is now the featured card for its category on the home page
- That any previous report of the same type has automatically moved to the Archive section
- Bracket update summary (one of):
  - Which matches were marked finished, scores, and which next-round slots were filled
  - Which upcoming match details (dates/venues/predictions) were updated
  - "No knockout changes detected in this report" — if nothing matched
- Remind them to hard-refresh the browser (`Ctrl+Shift+R`) — the server is already running on port 8080