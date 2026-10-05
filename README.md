# Sample Portfolio Dashboard

A **static** investment dashboard for GitHub Pages: plain HTML, CSS, and vanilla JS — no build step, no framework, no backend. Chart.js loads from cdnjs. A scheduled GitHub Action refreshes prices automatically.

> **Disclaimer:** Educational content only — not personalized financial advice. The portfolio, prices, and history in this repo are a *sample* for illustration. Past performance does not guarantee future results.

## Repo structure

```
.
├── index.html                        # Dashboard page
├── styles.css                        # Light/dark theme, responsive layout
├── app.js                            # All dashboard logic (vanilla JS)
├── holdings.json                     # ★ EDIT ME: holdings, targets, theses
├── prices.json                       # Latest quotes (updated by the Action)
├── history.json                      # Daily portfolio vs S&P 500 series
├── commentary.md                     # ★ EDIT ME: "The Veteran's Desk" content
├── .github/
│   ├── workflows/update-prices.yml   # Scheduled price refresh (weekdays)
│   └── scripts/fetch_prices.py       # Stooq fetcher — no API key needed
└── README.md
```

The page reads **only** the local JSON/Markdown files. Nothing secret ever reaches the browser.

## The sample portfolio

A concentrated-but-diversified core-satellite portfolio (11 holdings):

| Sleeve | Holdings | Target |
|---|---|---|
| Core (~65%) | VTI (US total market), VXUS (international), BND (bonds) | Low-cost compounding engine |
| Satellite (~35%) | MSFT, TSM, JNJ, PG, COST, JPM, NEE, HON | 8 quality stocks, 6 sectors |

Design rules enforced by the dashboard: no single stock over 8%, no sector over 25%, ≥6 sectors, international + defensive exposure. The Risk panel scores all of this and flags any holding drifting more than 5pp from target.

## Setup — GitHub Pages (5 minutes)

1. **Create a repo** (e.g. `investment-dashboard`) and push these files to the `main` branch:
   ```bash
   git init && git add . && git commit -m "Initial dashboard"
   git branch -M main && git remote add origin https://github.com/YOU/investment-dashboard.git
   git push -u origin main
   ```
2. **Enable Pages:** repo → *Settings* → *Pages* → *Build and deployment* → Source: **Deploy from a branch** → Branch: `main`, folder `/ (root)` → *Save*.
3. **Enable the updater:** repo → *Actions* → *Enable workflows* (if prompted). The `update-prices` workflow runs weekdays at 13:05 UTC and on demand via *Run workflow*.
4. Visit `https://YOU.github.io/investment-dashboard/` — done.

### API keys (optional)

The default fetcher uses **Stooq's free CSV endpoint — no key required**. If you ever switch to Alpha Vantage or Finnhub, add the key under repo → *Settings* → *Secrets and variables* → *Actions* (e.g. `ALPHA_VANTAGE_API_KEY`) and reference it in the workflow — never hard-code keys in the front end.

## Editing

- **`holdings.json`** — change `targetWeight`, `shares`, `costBasis`, `thesis`, `risk`, or add/remove holdings. The dashboard recomputes weights, drift, sector exposure, and the diversification score automatically. Keep 10–15 holdings for the score to stay meaningful.
- **`commentary.md`** — "The Veteran's Desk" renders this file live with a tiny built-in Markdown renderer (headings, bold/italic, lists, quotes, `code`). Just commit a new version.
- **`prices.json` / `history.json`** — normally written by the Action; the committed files are sample data so the page works immediately.

## Local preview

`fetch()` doesn't work over `file://`, so serve the folder:

```bash
cd investment-dashboard
python3 -m http.server 8000
# open http://localhost:8000
```

## How the price updater works

1. `fetch_prices.py` reads tickers from `holdings.json`.
2. Fetches one Stooq CSV request for all symbols (`ticker.us` + `^spx`).
3. Rolls `prices.json` forward: new close → `price`, previous `price` → `prevClose` (this powers the day-change card).
4. Appends one point to `history.json` using the day's portfolio value change vs. the S&P 500.
5. Commits `prices.json` + `history.json` only if something changed.

Network failures exit 0 with a warning — the dashboard keeps showing the last good data.

## Diversification score (exact formula)

Starts at 100, then:

- −15 if any single **stock** exceeds 8% (ETFs excluded — they're diversified by construction)
- −15 if any **sector** exceeds 25% (broad-market ETFs and bonds excluded from this check); −5 if the top sector is 20–25%
- −3 per holding drifting more than 5pp from its target weight
- −5 if holdings count is outside 10–15
- −10 if fewer than 6 sectors are represented

Labels: 80+ Strong · 60–79 Adequate · below 60 Concentrated.

## Customizing

- **Colors/charts:** `PALETTE` in `app.js`; theme tokens in `styles.css` (`:root` / `[data-theme="dark"]`).
- **Chart.js version:** pinned in `index.html` via cdnjs (`chart.umd.min.js`).
- **Rebalance threshold:** `rebalanceThresholdPp` in `holdings.json`.
- **Schedule:** edit the `cron` line in `.github/workflows/update-prices.yml` ([crontab syntax](https://crontab.guru)).

## Limitations

- Sample prices/history are illustrative, not real quotes (until the Action runs).
- Stooq quotes are ~15 minutes delayed; this is a tracking dashboard, not a trading tool.
- No authentication — don't put personal account data in this repo.
