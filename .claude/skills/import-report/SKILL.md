---
description: Import an already-created report HTML file into the portal. Asks which file(s) to import, auto-detects type and date from the filename, reads titles from the HTML, then updates data/reports.json. Use this when the report HTML already exists in reports/.
---

# Skill: Import Report File

You are helping the user register an existing report HTML file into the FIFA World Cup 2026 portal.

## Step 1 — Ask for the file(s) to import

Ask the user:

> "Which report file(s) do you want to import? You can name one or several files from the `reports/` folder (e.g. `naujienos_2026-06-22.html`). I'll detect the type and date automatically from the filename."

List the files currently in `reports/` that are NOT yet registered in `data/reports.json` — those are the candidates. To find them:
1. Read `data/reports.json` to get already-registered `file` values.
2. Glob `reports/*.html` to get all files on disk.
3. Show the user the unregistered ones as a numbered list so they can pick easily.

If there are no unregistered files, tell the user and stop.

## Step 2 — Auto-detect metadata from each filename

Report filenames follow the pattern `<type>_<date>.html`, e.g. `naujienos_2026-06-22.html`.

- **type** — the part before the first `_`: one of `naujienos`, `rezultatai`, `rungtynes`, `media`
- **date** — the part after the first `_`, without `.html`: format `YYYY-MM-DD`

If the filename does not match this pattern, ask the user to provide the type and date manually.

## Step 3 — Read titles from the HTML file

Open each report HTML file and extract the titles automatically:

- Look for the `<title>` tag — strip emoji, "FIFA Pasaulio Taurė 2026 –", the date suffix, and other boilerplate. What remains is a reasonable LT title candidate.
- Also look for the `<h1>` inside `.header` for the LT title.

Then ask the user to confirm or correct:

> "I found these details — please confirm or edit:
>
> | File | Type | Date | LT Title | EN Title |
> |------|------|------|----------|----------|
> | naujienos_2026-06-22.html | naujienos | 2026-06-22 | Dienos Naujienos | *(please provide)* |
>
> Provide the EN title for each, or press Enter to accept suggestions."

Default EN titles by type:
- `naujienos` → "Daily News"
- `rezultatai` → "Yesterday's Results"
- `rungtynes` → "Today's Fixtures"
- `media` → "Best Moments of the Day"

Author defaults to `FIFA analitikas` unless the user says otherwise.

## Step 4 — Update data/reports.json

Read the current `data/reports.json`.

For each imported file, prepend a new entry at the **top** of the array:

```json
{
  "date": "<DATE>",
  "type": "<TYPE>",
  "title": "<LT_TITLE>",
  "titleEN": "<EN_TITLE>",
  "author": "<AUTHOR>",
  "file": "reports/<FILENAME>"
}
```

If importing multiple files for the same date, add them all at the top, ordered: `naujienos`, `rezultatai`, `rungtynes`, `media` (matching the dashboard tile order).

Write the updated array back to `data/reports.json`.

## Step 5 — Confirm

Tell the user:
- Which files were registered and their types
- That they are now the featured cards for their categories on the home page
- That any previous report of the same type has automatically moved to the Archive section
- Remind them to do a hard refresh (`Ctrl+Shift+R`) if the browser cached the old `reports.json`