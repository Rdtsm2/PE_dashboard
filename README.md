# ValuScope — P/E Ratio Dashboard

> A static, single-file intelligence dashboard tracking Price-to-Earnings ratios across five global industry sectors: Banks, Fintechs, Telecom Integrated, Telecom Mobile, and Cybersecurity.

---

## Live URL

```
https://www.perplexity.ai/computer/a/valuscope-p-e-ratio-dashboard-zY9FtCqUTfC_J1UM6QyJlw
```

Asset ID: `cd8f45b4-2a94-4df0-bf27-550ce90c8997`

---

## Purpose

Built to answer the question: **how do P/E valuations compare across banks, fintechs, telecoms, and cybersecurity — historically, regionally, and at the individual company level?**

Covers:
- TTM (trailing 12-month) P/E
- Forward P/E (where available)
- Historical P/E trends (as far back as 2005 for some companies)
- Global regional splits (Americas, Europe, APAC, Africa/EM)
- Individual company profiles including private company valuations

---

## Architecture

| Property | Detail |
|---|---|
| Stack | Pure static HTML — no backend, no build step |
| Charts | Chart.js 4.4.0 via CDN |
| Fonts | Inter (UI) + JetBrains Mono (data) via Google Fonts |
| Theming | CSS variables; light/dark toggle in header |
| Default theme | **Light mode** (v4+) |
| Data storage | All data embedded in JS objects inside `index.html` |
| Deployment | Perplexity Computer `deploy_website()` → S3 static hosting |

The entire dashboard lives in a single file: `index.html`.

---

## Sectors & Data Coverage

### 🏦 Banks
- **Companies:** 10 public (JPM, BAC, GS, WFC, C, HSBC, DB, BCS, UBS, Maybank)
- **History:** 2005–2025 for major US banks
- **Median TTM P/E:** 13.5x | 10yr avg: 12.4x
- **Regions:** Americas, Europe, APAC

### 💳 Fintechs
- **Companies:** 27 public + 4 private
- **Includes:** PYPL, COIN, INTU, SQ(XYZ), NU, HOOD, SE, ADYEY, WISE, ETOR, GLBE, GRAB, DLO, UPST, LC, FOUR, WEX, DAVE, OPFI, NRDS + Klarna, Chime + 2 others
- **Exclusions:** Visa and Mastercard (payments infrastructure, not fintech)
- **Median TTM P/E:** 32.7x (profitable cohort) | Loss-making: 4
- **Regions:** Americas, Europe, APAC

### 📡 Telecom Integrated
- **Companies:** 20 public (T, VZ, CMCSA, BCE, RCI, AMX, DTEGY, BTGOF, SCMN, TELNY, NTT, KDDI, SKM, Singtel, VOD, TEF, SKM, VEON, et al.)
- **Median TTM P/E:** 14.6x | 10yr avg: 14.2x
- **Note:** Several operators (VOD, TEF, Orange) show distorted/negative P/E due to impairment charges
- **Regions:** Americas, Europe, APAC

### 📱 Telecom Mobile
- **Companies:** 9 public (TMUS, TIGO, Airtel, Maxis, Indosat, True, PLDT, TKC, MTN)
- **Median TTM P/E:** 16.3x | Fwd P/E TMUS: 7.9x
- **Regions:** Americas, APAC, Africa/EM

### 🔐 Cybersecurity *(added v4)*
- **Companies:** 15 public + 5 private
- **Profitable public (9):** CHKP, FTNT, PANW, OKTA, QLYS, GEN, AKAM, VRSN, RPD
- **Loss-making public (6):** CRWD, NET, ZS, S, TENB, CYBR
- **Private (5):** Lacework (~$8.3B), Netskope (~$7.5B), Snyk (~$7.4B, UK), 1Password (~$6.8B, Canada), Arctic Wolf (~$4.3B)
- **Median TTM P/E:** 31.1x | Range (profitable): 14.3–93.8x | Fwd P/E FTNT: 30x
- **10yr proxy avg:** ~20x (CHKP) | vs avg: +56%
- **Regions:** Americas, Europe/Israel

---

## Dashboard Structure

```
Overview tab
├── 5-column KPI cards (one per sector)
├── Sector Median P/E bar chart (side-by-side)
├── P/E Distribution box plot (dot chart, capped 100x)
├── Historical sector median trend lines
└── Valuation Premium/Discount comparison table

Banks tab
Fintechs tab
Telecom Integrated tab
Telecom Mobile tab
Cybersecurity tab
  ├── Overview sub-view
  │   ├── KPI cards (sector median, regional, loss-making, 10yr avg)
  │   ├── Historical P/E trend chart (key players)
  │   ├── Current P/E snapshot bar chart
  │   └── Americas vs Europe regional bar chart
  ├── Regional sub-view
  │   ├── Americas company bar chart
  │   ├── Europe / Israel bar chart
  │   └── Private companies valuation reference bar chart
  └── Companies sub-view
      └── Full company table (search + filter by region)
```

Each sector tab follows the same three sub-view pattern: **Overview / Regional / Companies**.

---

## Design System

| Token | Value |
|---|---|
| Banks color | `#388bfd` (blue) |
| Fintechs color | `#3fb950` (green) |
| Telecom Integrated color | `#d29922` (amber) |
| Telecom Mobile color | `#a371f7` (purple) |
| Cybersecurity color | `#e05d44` (red-orange) |
| P/E normal | green text |
| P/E > 40x | yellow text |
| P/E > 80x | orange text |
| Loss-making | red text |
| Font (UI) | Inter |
| Font (data) | JetBrains Mono |

---

## Key JS Patterns

```js
// Tab switching
data-sector="banks"  →  shows/hides  id="banks-section"

// Sub-view switching
data-view="overview"  data-target="banks"
→  activates  id="banks-overview"  (class="view-panel active")

// Chart helpers
buildHistSeries(companies, years)   // maps hist{} to year array, null gaps, spanGaps:false
pctileCap(datasets, years, pct)     // smart Y-axis ceiling by percentile
currentMedian(companies, maxPe)     // median of current P/E filtered by cap
sectorMedian(companies, year, maxPe) // historical median for a given year

// Dot chart jitter
x: SECTOR_X[sector] + (Math.random() - 0.5) * 0.4
```

### View panel conventions
- Active panel: `class="view-panel active"` — no inline `display:none`
- Inactive panels: `class="view-panel"` only — CSS `.view-panel:not(.active)` handles hiding
- View buttons: `class="view-btn"` inside a `class="view-toggle"` wrapper
- `data-target` = sector prefix (e.g. `"cybersec"`), NOT full panel ID
- Panel IDs follow pattern: `{sector}-overview`, `{sector}-regional`, `{sector}-companies`

### Data object shape
```js
const CYBERSEC_DATA = [
  {
    name: "Fortinet",
    ticker: "FTNT",
    region: "Americas",
    segment: "Network Security",
    mktCap: "$61B",
    pe: 34.15,
    fwdPe: 30,
    hist: { 2009: 34, 2010: 28, ... }  // null = no data / loss year
  },
  // Private companies use pe: null, fwdPe: null, valuation: "$7.5B"
]
```

---

## Data Methodology

- **TTM P/E:** Sourced from public market data providers; as of April 2026
- **Forward P/E:** Consensus analyst estimates; only quoted where widely available
- **Exclusions from averages:** Negative P/E (loss-making) and values >150x (distorted)
- **Private company valuations:** Last known VC/funding round valuations; no P/E calculated
- **Historical data:** Gaps represented as `null` in hist objects; `spanGaps: false` in Chart.js
- **Outlier handling:** Y-axis capped at 90th percentile (`pctileCap`) so the bulk of data remains readable

---

## Version History

| Version | Key Changes |
|---|---|
| v1 | Initial dashboard: Banks, Fintechs, Telecom Int, Telecom Mob. Company-level P/E data embedded. |
| v2 | Expanded company coverage across all 4 sectors. More fintech players. Private companies added. Visa/MC removed from Fintechs. |
| v3 | Added Overview tab (cross-sector comparison). Historical Sector Avg P/E now includes all players with percentile-capped Y-axis. |
| v4 | Default theme switched to light mode. Cybersecurity added as 5th sector (15 public + 5 private cos). Overview tab updated to "All Five Segments". |

---

## Updating the Dashboard

Since all data is embedded in `index.html`, updates are done by editing the JS data objects directly.

### To update a company's current P/E:
Find the company in the relevant `*_DATA` array and update the `pe` field.

### To add a new company:
1. Add an entry to the sector's `*_DATA` array following the existing object shape
2. If the sector tab has a companies table, the JS `buildCompaniesTable()` call will pick it up automatically
3. Update KPI card counts in the HTML if needed

### To add a new sector:
1. Add a nav tab button: `<button class="sector-tab" data-sector="newsector">...`
2. Add a `<section id="newsector-section" class="sector-section" style="display:none">` block
3. Inside the section, follow the view-panel pattern with IDs `newsector-overview`, `newsector-regional`, `newsector-companies`
4. Add the sector color to `SECTOR_COLORS`, position to `SECTOR_X` in the dot chart
5. Update Overview tab: KPI card, bar chart dataset, range chart, trend line, dot chart axis labels, and comparison table column

### To redeploy:
```
deploy_website(
  project_path="/home/user/workspace/pe-dashboard",
  site_name="ValuScope — P/E Ratio Dashboard",
  entry_point="index.html",
  name="pe-dashboard",
  should_validate=False
)
```
Use `should_validate=False` to skip the automated validator (avoids false positives on data cards).

---

## Local Development

```bash
cd /home/user/workspace/pe-dashboard
serve . -l 3000 --no-clipboard --single
# → http://localhost:3000
```

For Playwright QA:
```js
const { chromium } = await import('playwright');
const browser = await chromium.launch({ headless: true });
const context = await browser.newContext({ viewport: { width: 1600, height: 900 } });
const page = await context.newPage();
await page.goto('http://127.0.0.1:3000');
// Note: scroll via page.evaluate(() => document.querySelector('main.main').scrollTop = N)
// Tab click: await page.click('[data-sector="cybersec"]')
// Sub-view click: await page.click('[data-view="regional"][data-target="cybersec"]')
```

---

## Known Quirks

- **FTNT historical P/E 2014–2017:** Extremely high (200–600x) due to transition period. Captured in hist data but Y-axis percentile cap suppresses visual distortion.
- **PANW:** Turned GAAP-profitable in FY2023; historical P/E only meaningful from 2023 onward.
- **CYBR (CyberArk):** Listed as loss-making/acquisition pending (acquired by PANW for ~$25B; stale quote data excluded).
- **Telecom operators (VOD, TEF, Orange):** Interpret P/E cautiously — heavy capex and depreciation frequently cause distorted or negative reported earnings.
- **Private company P/E:** Not shown. Valuation bars in the Regional view use last-known VC round figures only.
- **`main.main` scroll container:** The page scrolls via `main.main`, not `window`. Use `document.querySelector('main.main').scrollTop` in any programmatic scroll.
