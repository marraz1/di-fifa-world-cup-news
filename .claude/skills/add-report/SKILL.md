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

## Step 4 — Confirm

Tell the user:
- What file was created
- That the new report is now the featured card for its category on the home page
- That any previous report of the same type has automatically moved to the Archive section
- Remind them to refresh the browser (the server is already running on port 8080)