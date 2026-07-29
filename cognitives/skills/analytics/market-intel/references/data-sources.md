# Data Sources Guide

How to fetch, validate, and prioritize real-time financial data. Read this when you need to go deeper than the basic search queries in SKILL.md.

---

## Source Hierarchy

Not all financial data sources are equal. Use this priority order:

### Tier 1 — Authoritative (prefer these)
- **SEC EDGAR** (`sec.gov/cgi-bin/browse-edgar`) — 10-K, 10-Q, 8-K filings, insider transactions (Form 4)
- **Company investor relations pages** — Earnings releases, guidance, presentations
- **Federal Reserve** (`fred.stlouisfed.org`) — Macro data (rates, CPI, employment)

### Tier 2 — Reliable aggregators
- **Yahoo Finance** (`finance.yahoo.com`) — Price data, key statistics, financials, analyst estimates
- **Finviz** (`finviz.com`) — Screener snapshots, technical indicators, ownership data
- **MarketWatch** (`marketwatch.com`) — News, financials, analyst consensus
- **Seeking Alpha** (`seekingalpha.com`) — Earnings transcripts, analyst opinions

### Tier 3 — Supplementary
- **TradingView** (`tradingview.com`) — Technical analysis data (moving averages, RSI, volume)
- **Unusual Whales** / **Barchart** — Options flow, unusual activity
- **Reuters** / **Bloomberg** (free articles) — Breaking news, institutional analysis
- **Reddit** (`reddit.com/r/wallstreetbets`, `/r/stocks`, `/r/investing`) — Retail sentiment gauge

### Tier 4 — Use with caution
- **Social media** (Twitter/X finance accounts) — Sentiment only, never for facts
- **Stock tip sites** — High noise, potential conflicts of interest

---

## Search Strategies by Data Type

### Real-Time Price & Technical Data

```
Primary:   WebFetch "https://finance.yahoo.com/quote/{TICKER}"
Fallback:  WebSearch "{TICKER} stock price today"
Deep:      WebFetch "https://finviz.com/quote.ashx?t={TICKER}"
```

Extract: current price, day change %, 52-week range, volume vs average, SMA 20/50/200, RSI, beta.

### Financial Statements

```
Primary:   WebSearch "{TICKER} income statement revenue earnings {latest_quarter}"
Deep:      WebSearch "{TICKER} 10-Q SEC filing {year}"
Compare:   WebSearch "{TICKER} vs {COMPETITOR} revenue growth margin comparison"
```

Extract: revenue, EPS (GAAP and non-GAAP), margins, FCF, guidance.

### News & Catalysts

```
Breaking:  WebSearch "{TICKER} news today"
Depth:     WebSearch "{TICKER} earnings analysis {quarter} {year}"
Hidden:    WebSearch "{TICKER} insider buying OR insider selling SEC Form 4"
Industry:  WebSearch "{SECTOR} sector outlook {current_month} {year}"
```

Extract: key events, analyst reactions, insider moves, sector trends.

### Sentiment & Flows

```
Analyst:   WebSearch "{TICKER} analyst upgrade downgrade price target {month}"
Short:     WebSearch "{TICKER} short interest short squeeze"
Options:   WebSearch "{TICKER} unusual options activity call put"
Retail:    WebSearch "{TICKER} reddit wallstreetbets sentiment"
Inst:      WebSearch "{TICKER} 13F institutional ownership top holders"
```

Extract: consensus rating, price target range, short interest %, put/call ratio, institutional changes.

### Macro Context

```
Rates:     WebSearch "federal reserve interest rate decision {month} {year}"
Economy:   WebSearch "US GDP CPI employment data latest"
Sector:    WebSearch "sector rotation {month} {year} defensive cyclical"
VIX:       WebSearch "VIX volatility index today"
Global:    WebSearch "global market risks geopolitical {month} {year}"
```

---

## Handling Missing Data

Not all data will be available for every stock. When data is missing:

1. **Price data unavailable**: State it explicitly. Do not estimate. "Current price data could not be fetched — analysis is based on last known price of $X as of {date}."
2. **Financials outdated**: Note the staleness. "Most recent earnings data is from Q{X} {year} — {N} months old. Treat profitability metrics as directional, not precise."
3. **No analyst coverage**: Common for small-caps. Say so and weight fundamental analysis more heavily.
4. **Insider data sparse**: Not all companies have active insider trading. Note the absence rather than inferring meaning from silence.

The principle: transparency about data gaps is more valuable than filling them with assumptions.

---

## Cross-Referencing Protocol

When a key metric appears in multiple sources with different values:

1. Check the date of each source — use the most recent
2. Check GAAP vs non-GAAP — earnings figures often differ because of this
3. Check currency — international companies may report in local currency
4. When genuinely conflicting, note both values and which source reported each

Example: "Yahoo Finance reports TTM EPS of $6.42 (likely GAAP), while Seeking Alpha shows $7.15 (likely adjusted/non-GAAP). The difference is primarily stock-based compensation of ~$0.73/share."

---

## Rate Limiting & Fallbacks

If a WebFetch fails (403, timeout, captcha):
1. Try the WebSearch equivalent instead
2. Try an alternative source from the same tier
3. If all sources fail for a data point, note it as unavailable and proceed with what you have

A report with clearly flagged data gaps is more honest and useful than one that silently fills gaps with stale training data.
