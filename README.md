# DiFifa World Cup News

Bilingual (Lithuanian/English) FIFA World Cup 2026 daily news and match analysis portal.

## Local Setup

**Requirement:** Python 3 must be installed and available in your PATH.

Check if Python is installed:
```
python --version
```

## Running Locally

Double-click `serve.bat` or run it from the terminal:
```
serve.bat
```

This starts a local server at [http://localhost:8080](http://localhost:8080) and opens the browser automatically.

To stop the server press `Ctrl+C` in the terminal.

> **Why a server?** The site fetches `data/reports.json` via `fetch()`, which browsers block on `file://` URLs. A local HTTP server is required.

## Project Structure

```
dififa-world-cup-news/
├── index.html          # Home page — report listing with date filter and LT/EN toggle
├── css/
│   ├── style.css       # Home page styles
│   └── report.css      # Report page styles + FIFA color palette
├── data/
│   └── reports.json    # Report metadata index (date, title, author, file path)
└── reports/
    └── *.html          # Individual report pages
```

## Adding a New Report

1. Create a new file in `reports/`, e.g. `reports/2026-06-20-dienos-ataskaita.html`
2. Add an entry to `data/reports.json`:

```json
{
  "date": "2026-06-20",
  "title": "Dienos ataskaita – Grupių etapas, 6 diena",
  "titleEN": "Daily Report – Group Stage, Day 6",
  "author": "Your Name",
  "file": "reports/2026-06-20-dienos-ataskaita.html"
}
```

Reports are displayed newest-first (sorted by `date`).

## Pushing Code to GitHub

Repository: [https://github.com/marraz1/di-fifa-world-cup-news](https://github.com/marraz1/di-fifa-world-cup-news)

**First time — clone the repo:**
```
git clone https://github.com/marraz1/di-fifa-world-cup-news.git
cd di-fifa-world-cup-news
```

**Everyday workflow:**

```bash
# 1. Check what changed
git status

# 2. Stage your changes
git add .

# 3. Commit with a message
git commit -m "Add Day 6 group stage report"

# 4. Push to GitHub
git push
```

**If the push is rejected** (someone else pushed first):
```
git pull --rebase
git push
```
