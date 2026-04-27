# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ValuScope** is a single-file static dashboard (`index.html`) tracking P/E ratios across 5 global sectors: Banks, Fintechs, Telecom Integrated, Telecom Mobile, and Cybersecurity. No build step, no backend, no package manager.

Live at: `https://rdtsm2.github.io/PE_dashboard/`

## Local Development

```bash
python -m http.server 3000
# → http://localhost:3000
```

Deploy: commit and push to `main` — GitHub Pages rebuilds automatically within ~1 minute.

## Architecture

Everything lives in `index.html`:
- **CSS** — design tokens via CSS variables, light/dark theming via `[data-theme]` attribute on `<html>`
- **Data** — embedded JS arrays (e.g. `BANKS_DATA`, `FINTECH_DATA`, `CYBERSEC_DATA`)
- **Logic** — vanilla JS at the bottom of the file; no frameworks

**Tab/view system:**
- Sector tabs: `<button class="sector-tab" data-sector="banks">` → shows `<section id="banks-section">`
- Sub-views within each sector: `<button class="view-btn" data-view="overview" data-target="banks">` → activates `<div id="banks-overview" class="view-panel active">`
- Active panel uses `class="view-panel active"`; CSS `.view-panel:not(.active)` handles hiding — never use inline `display:none` on panels

**Chart.js 4.4.0** loaded via CDN. Key helper functions:
- `buildHistSeries(companies, years)` — maps `hist{}` to year array with `null` gaps
- `pctileCap(datasets, years, pct)` — smart Y-axis ceiling by percentile
- `currentMedian(companies, maxPe)` — median of current P/E filtered by cap
- `sectorMedian(companies, year, maxPe)` — historical median for a given year

**Scroll container:** The page scrolls via `main.main`, not `window`. Use `document.querySelector('main.main').scrollTop` for any programmatic scroll.

## Data Object Shape

```js
{ name, ticker, region, segment, mktCap, pe, fwdPe, hist: { 2009: 34, 2010: null, ... } }
// Private companies: pe: null, fwdPe: null, valuation: "$7.5B"
```

## Updating Data

All data is edited directly in `index.html` JS arrays.

- **Update a P/E:** find the company in `*_DATA` and edit `pe`
- **Add a company:** append to `*_DATA`; `buildCompaniesTable()` picks it up automatically; update KPI card counts in HTML if needed
- **Add a sector:** add nav tab, section block (3 view-panels), sector color to `SECTOR_COLORS`, position to `SECTOR_X`, and update all Overview tab components (KPI card, bar chart, range chart, trend line, dot chart, comparison table)

## Design Tokens

| Sector | Color |
|---|---|
| Banks | `#388bfd` (blue) |
| Fintechs | `#3fb950` (green) |
| Telecom Integrated | `#d29922` (amber) |
| Telecom Mobile | `#a371f7` (purple) |
| Cybersecurity | `#e05d44` (red-orange) |

P/E coloring: normal → green, >40x → yellow, >80x → orange, loss-making → red. Fonts: Inter (UI), JetBrains Mono (data).

## Known Quirks

- **FTNT hist P/E 2014–2017:** Extremely high (200–600x); percentile cap suppresses visual distortion
- **VOD, TEF, Orange:** P/E often distorted/negative due to impairment charges
- **Private companies:** No P/E shown; Regional view uses last-known VC round valuation
- **Exclusions from medians:** Negative P/E and values >150x
