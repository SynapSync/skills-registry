# Report Template

Use this template for all market intelligence reports. Adapt section depth based on the stock and available data, but maintain the structure.

---

## Single Stock Report

```markdown
# {TICKER} — Market Intelligence Report

**Date:** {YYYY-MM-DD}
**Price at analysis:** ${current_price} ({day_change}%)
**52-Week Range:** ${low} — ${high}
**Verdict:** {STRONG BUY | BUY | HOLD | WAIT | AVOID}
**Confidence:** {1-5}/5

---

## Executive Summary

{2-3 sentences maximum. The thesis, the verdict, and the single biggest risk. A portfolio manager should be able to read only this section and make a decision.}

---

## 1. News Intelligence

### Key Developments (Last 72 Hours)

| Event | Bullish Implication | Bearish Implication | Net Signal |
|-------|--------------------|--------------------|------------|
| {event_1} | {bull} | {bear} | {Bullish/Bearish/Neutral} |
| {event_2} | {bull} | {bear} | {Bullish/Bearish/Neutral} |

### Signal vs Noise Assessment
{Which of the above actually matters for the next 30-90 days, and which is market noise.}

### Narrative Shift
{Is the market story changing? From what to what?}

---

## 2. Fundamental Analysis

### Growth Scorecard

| Metric | Latest | YoY Change | vs Sector | Assessment |
|--------|--------|-----------|-----------|------------|
| Revenue | ${X}B | +X% | {above/below/inline} | {accelerating/stable/decelerating} |
| EPS | ${X} | +X% | {above/below/inline} | {expanding/stable/compressing} |
| Guidance | {raised/maintained/lowered} | — | — | {positive/neutral/negative} |

### Profitability

| Metric | Latest | Trend | Peer Comparison |
|--------|--------|-------|----------------|
| Gross Margin | X% | {expanding/stable/contracting} | {above/below/inline} |
| Operating Margin | X% | {expanding/stable/contracting} | {above/below/inline} |
| FCF Yield | X% | {expanding/stable/contracting} | vs 10Y Treasury: {X%} |

### Competitive Position

**Moat type:** {network effects / switching costs / scale / brand / IP / none}
**Moat durability:** {strengthening / stable / eroding}
**Key risk:** {single biggest competitive threat}

### Valuation Context

| Multiple | Current | 5Y Average | Sector Median | Implied by Growth |
|----------|---------|-----------|--------------|------------------|
| P/E (fwd) | Xx | Xx | Xx | Xx |
| P/S | Xx | Xx | Xx | — |
| EV/EBITDA | Xx | Xx | Xx | — |

**Assessment:** {overvalued / fairly valued / undervalued} relative to {growth / peers / history}

---

## 3. Technical Structure

### Trend Assessment

| Timeframe | Trend | Key Level | Signal |
|-----------|-------|-----------|--------|
| Short (1-4 weeks) | {up/down/sideways} | ${level} | {bullish/bearish/neutral} |
| Medium (1-3 months) | {up/down/sideways} | ${level} | {bullish/bearish/neutral} |
| Long (6-12 months) | {up/down/sideways} | ${level} | {bullish/bearish/neutral} |

### Key Levels

- **Resistance:** ${R1}, ${R2} (52W high: ${high})
- **Support:** ${S1}, ${S2} (52W low: ${low})
- **200-day MA:** ${ma200}
- **50-day MA:** ${ma50}

### Market Phase: **{Accumulation / Markup / Distribution / Markdown}**
{Evidence for this classification in 2-3 sentences.}

### Momentum
- **RSI:** {value} ({overbought/neutral/oversold})
- **Volume:** {above/below} 20-day average ({interpretation})
- **Relative Strength vs S&P 500:** {outperforming/underperforming} over {timeframe}

---

## 4. Sentiment Decoding

### Institutional Activity
{Recent 13F changes, analyst upgrades/downgrades, notable fund positions.}

### Short Interest
{Current SI%, trend, and what it implies.}

### Retail Sentiment
{Social media tone, Reddit/X activity, retail flow data if available.}

### Classification: **{Overhyped / Quiet Accumulation / Consensus Neglect / Crowded Short}**
{Why this classification, in 2-3 sentences.}

---

## 5. Macro Context

{One paragraph connecting the macro regime to this specific stock. Interest rate environment, sector cycle position, geopolitical factors — but only what's relevant to THIS stock.}

**Macro tailwind/headwind assessment:** {Strong tailwind / Mild tailwind / Neutral / Mild headwind / Strong headwind}

---

## 6. Scenario Modeling

| Scenario | Probability | Target Price | Return | Key Driver |
|----------|------------|-------------|--------|-----------|
| Bull | {X}% | ${target} | +{X}% | {primary catalyst} |
| Base | {X}% | ${target} | {+/-X}% | {primary assumption} |
| Bear | {X}% | ${target} | -{X}% | {primary risk} |

**Expected Value: {+/-X.X}%**
**Asymmetry Ratio: {upside:downside}** (e.g., 2.5:1)

### Bull Case ({X}%)
{What drives this outcome. Be specific about catalysts and multiple expansion logic.}

### Base Case ({X}%)
{Most likely path. What assumptions must hold.}

### Bear Case ({X}%)
{What breaks the thesis. The "I was wrong" signal that triggers an exit.}

---

## 7. Verdict & Capital Allocation

### Verdict: **{VERDICT}**
### Confidence: **{X}/5**

{2-3 sentences explaining the rating. Why this rating and not one level higher or lower.}

### Position Guidance

| Parameter | Recommendation |
|-----------|---------------|
| Position type | {Core / Tactical / Speculative} |
| Suggested allocation | {X-Y}% of portfolio |
| Entry zone | ${low} — ${high} |
| Stop-loss | ${level} ({X}% below entry) |
| Take-profit targets | ${T1}, ${T2} |
| Time horizon | {X weeks/months} |
| Monitor for | {key metric or event that changes the thesis} |

---

## Hidden Opportunities

{1-3 stocks not in the analysis that are showing interesting setups. Brief format:}

### {TICKER_1} — {One-line thesis}
**Price:** ${X} | **Why now:** {catalyst or setup} | **Risk:** {low/medium/high}

---

*Sources consulted: {list of URLs}*
*Analysis generated: {timestamp}*
*Disclaimer: This is analytical output from an AI system, not financial advice. All investing carries risk of loss. Past performance does not guarantee future results. Always do your own due diligence.*
```

---

## Multi-Stock Portfolio Summary

When analyzing 3+ stocks, append this after the individual reports:

```markdown
# Portfolio Summary — {date}

## Comparative Overview

| Ticker | Price | Verdict | Confidence | Position Type | EV | Asymmetry | Top Catalyst |
|--------|-------|---------|-----------|--------------|-----|-----------|-------------|
| {T1} | ${X} | {V} | {C}/5 | {Type} | {X}% | {X:1} | {catalyst} |
| {T2} | ${X} | {V} | {C}/5 | {Type} | {X}% | {X:1} | {catalyst} |

## Relative Ranking (by risk-adjusted EV)
1. **{TICKER}** — {one-line rationale}
2. **{TICKER}** — {one-line rationale}

## Suggested Allocation
{How to distribute capital across these positions, considering correlation and diversification.}

## Portfolio-Level Risks
{Risks that affect multiple positions simultaneously — e.g., rate hikes affecting all growth names.}
```
