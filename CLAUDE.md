# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development

Start local server (opens browser automatically on port 8080):
```
serve.bat
```
Or directly: `python -m http.server 8080`

No build step, no package manager, no transpilation — this is a zero-dependency static site.

## Architecture

This is a bilingual (Lithuanian/English) FIFA World Cup 2026 news portal built with vanilla HTML/CSS/JS.

**Core files:**
- [index.html](index.html) — Home page: fetches `data/reports.json`, renders a filterable card grid
- [data/reports.json](data/reports.json) — Single source of truth for report metadata; add entries here when adding a new report
- [reports/](reports/) — Individual report HTML files, manually authored as complete HTML pages
- [css/style.css](css/style.css) — Home page styles (dark theme, card grid)
- [css/report.css](css/report.css) — Shared report page styles; defines the official FIFA color palette as CSS custom properties (`--red`, `--blue`, `--green`)

## Adding a New Report

1. Create `reports/<date>-<slug>.html`, include `../css/report.css`
2. Add an entry to `data/reports.json`:
   ```json
   { "date": "YYYY-MM-DD", "title": "LT title", "titleEN": "EN title", "author": "Name", "file": "reports/<filename>.html" }
   ```
   Reports are sorted newest-first by date string comparison.

## Internationalization

- Language persists via `localStorage` key `lang` (`"lt"` or `"en"`)
- `index.html` contains an `i18n` object with both language keys for all UI labels
- Lithuanian plural rules are implemented as `pluralLT(n)` (3-form)
- Each report object carries `title` (LT) and `titleEN` (EN) for card display
- Report pages themselves are fully bilingual HTML — both LT and EN content is typically included inline
