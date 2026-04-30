# Trading Journal — CLAUDE.md

## Project Overview

Single-page Trading Journal web application. Traders log trades with entry, stop loss, and take profit prices; the app automatically calculates Risk:Reward ratios and aggregates performance statistics. All data persists in the browser's localStorage — no backend, no server, no build step.

**Stack:** Vanilla HTML5 / CSS3 / JavaScript (ES6+), zero external dependencies.  
**UI language:** Thai labels, English trading terminology.  
**Theme:** Dark terminal aesthetic with monospace fonts and neon accent colors.

## Running the App

Open `index.html` directly in any modern browser. No server or install required.

```
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

For local development with live-reload, any static file server works (e.g., `python3 -m http.server`), but it is not required.

## Repository Structure

```
/
└── index.html    # Entire application (289 lines)
```

`index.html` has three logical sections:

| Lines | Section | Contents |
|-------|---------|---------|
| 9–79 | `<style>` | All CSS (minified single-line rules) |
| 82–152 | HTML body | Header, stats dashboard, filter tabs, trade list, modal form |
| 154–284 | `<script>` | All JavaScript logic |

## Global State Variables

Declared at the top of the `<script>` block:

| Variable | Type | Purpose |
|----------|------|---------|
| `trades` | `Array` | All trade objects; loaded from localStorage on init |
| `dir` | `string` | Current direction selection: `'buy'` or `'sell'` |
| `res` | `string` | Current result selection: `'win'`, `'loss'`, or `'be'` |
| `filt` | `string` | Active filter: `'all'`, `'win'`, `'loss'`, `'be'`, `'buy'`, `'sell'` |
| `eid` | `number\|null` | ID of trade being edited; `null` when adding new |

localStorage key: `'tj_v3'`

## Data Model

Each element of `trades[]` is a plain object:

```js
{
  id:     number,   // Date.now() at creation time
  symbol: string,   // e.g. "XAUUSD" (always uppercased)
  dir:    string,   // "buy" | "sell"
  type:   string,   // "market" | "limit" | "stop"
  entry:  number,   // entry price
  sl:     number,   // stop loss price
  tp:     number,   // take profit price
  date:   string,   // "YYYY-MM-DD"
  result: string,   // "win" | "loss" | "be"
  rin:    string,   // reason for entry (free text)
  rout:   string,   // reason for exit/outcome (free text)
  risk:   number,   // Math.abs(entry - sl)
  reward: number,   // Math.abs(tp - entry)
  rr:     number    // reward / risk
}
```

`risk`, `reward`, and `rr` are always derived from the prices — never set manually.

## Key JavaScript Functions

| Function | Purpose |
|----------|---------|
| `sv()` | Serialize `trades[]` to localStorage |
| `openModal(id?)` | Open modal; if `id` provided, load that trade for editing |
| `closeModal()` | Hide modal overlay, reset form via `rf()` |
| `bgC(event)` | Close modal when user clicks the dark overlay background |
| `rf()` | Reset all form fields to defaults (symbol=XAUUSD, dir=buy, today's date) |
| `sd(dir)` | Set direction; updates `dir` variable and toggles `.on` on `#dB`/`#dSe` |
| `sr(result)` | Set result; updates `res` variable and toggles `.on` on result buttons |
| `cRR()` | Live R:R calculation triggered by `oninput` on price fields |
| `saveTrade()` | Validate form → build trade object → upsert into `trades[]` → `sv()` → `ra()` |
| `delT(id)` | Confirm then remove trade from `trades[]`, call `sv()` and `ra()` |
| `sf(filter, el)` | Set `filt`, update active tab, call `ra()` |
| `ra()` | Full re-render: recalculate all stats, apply current filter, rebuild `#tList` innerHTML |

## HTML Element ID Reference

**Statistics display**

| ID | Displays |
|----|---------|
| `sWR` | Win rate percentage |
| `sT` | Total trade count |
| `sRR` | Average R:R ratio |
| `sPF` | Profit factor |
| `eqF` | Win-rate progress bar fill (`width` style) |
| `eqP` | Win-rate percentage label on the bar |
| `tC` | Trade count label above the list |
| `tList` | Trade card list container |

**Modal form inputs**

| ID | Field |
|----|-------|
| `fS` | Symbol (text) |
| `fE` | Entry price (number) |
| `fSL` | Stop loss (number) |
| `fTP` | Take profit (number) |
| `fD` | Date (date) |
| `fTy` | Order type (select) |
| `fRI` | Reason for entry (textarea) |
| `fRO` | Reason for exit/outcome (textarea) |

**Direction buttons:** `dB` (buy), `dSe` (sell)  
**Result buttons:** `rW` (win), `rL` (loss), `rB` (be)  
**R:R preview:** `rRisk`, `rRew`, `rRatio`  
**Modal:** `ov` (overlay div), `mT` (modal title span)

## CSS Conventions

**Color utility classes** (apply to any element):

| Class | Color | Use case |
|-------|-------|---------|
| `.cg` | `#00ff88` (green) | Wins, buy direction, positive metrics |
| `.cr` | `#ff3366` (red) | Losses, sell direction, risk, SL |
| `.cy` | `#ffd700` (gold) | Break-even, neutral/warning |
| `.cw` | `#e0e0f0` (white) | Default text, neutral values |

**Background palette:**

| Value | Used for |
|-------|---------|
| `#0a0a0f` | Page body |
| `#12121a` | Cards, modal panel |
| `#1a1a26` | Inputs, inner boxes, inactive buttons |
| `#2a2a3a` | Borders |

**Active state:** Add `.on` class to toggle a button/tab to its highlighted state (e.g., `.ftab.on`, `.dbt.buy.on`, `.rbt.win.on`).

**Typography:** `font-family:monospace` for all numbers, labels, headers. System sans-serif (`-apple-system,sans-serif`) for body text.

**CSS style:** All rules are written as compact single-line declarations. When adding new rules, match this minified style — do not introduce multi-line rule blocks.

## Calculated Metrics

```
risk          = Math.abs(entry - sl)
reward        = Math.abs(tp - entry)
rr            = risk > 0 ? reward / risk : 0
winRate       = (wins / total) * 100
avgRR         = sum(rr for all trades) / total
profitFactor  = totalRewardOnWins / totalRiskOnLosses   (or "∞" if no losses)
```

R:R color coding in the UI: green (≥ 2.0), gold (≥ 1.0), red (< 1.0).

## Development Guidelines

- **No build step.** Edit `index.html` and refresh the browser.
- **Test by opening the file locally.** Use browser DevTools (F12) to inspect the DOM and debug JS.
- **Inspect stored data:** In DevTools → Application → Local Storage → key `tj_v3`.
- **Reset all data:** `localStorage.removeItem('tj_v3')` in the browser console, then refresh.
- **No automated tests exist.** Verify changes manually by exercising the UI: add a trade, edit it, delete it, check all filters and stats.

## Conventions for AI Assistants

- **Single-file constraint:** Keep all code in `index.html`. Do not split into separate `.css` or `.js` files unless the user explicitly requests it.
- **No external dependencies:** Do not introduce npm packages, CDN scripts, or any external resources.
- **No frameworks:** Keep vanilla JS. Do not add React, Vue, Alpine, or similar.
- **Thai UI labels:** Preserve all Thai-language strings in the UI. Do not translate them to English.
- **CSS style:** Match the existing minified single-line CSS format. Avoid introducing formatted multi-line blocks.
- **Short function names:** Existing functions use abbreviated names (`sv`, `ra`, `rf`, `sd`, `sr`, `cRR`). Extend new functions following the same concise naming style.
- **DOM manipulation pattern:** Use `getElementById` + `textContent`/`innerHTML`/`classList`. Do not introduce `querySelector` chains or framework-style abstractions unless there is a clear reason.
- **Data integrity:** Always recompute `risk`, `reward`, and `rr` from prices — never let users set them directly.
- **localStorage migration:** The current key is `tj_v3`. If the data schema changes in a breaking way, bump the version suffix and handle migration or start fresh.
