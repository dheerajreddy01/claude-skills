---
schema: "1.0"
name: investment-analysis
version: "1.1.0"
description: Stock analysis, financial statement interpretation, and investment decision-making
triggers:
  keywords:
    primary: [stock, investment, financial-report, valuation]
    secondary: [fundamental, technical, PE, ROE, price-to-earnings, dividend-yield]
  context_boost: [finance, analysis, portfolio, dividend, earnings]
  context_penalty: [trading, algo, backtest]
  priority: high
keywords: [finance, investment, analysis, stock]
dependencies:
  software-skills: [python, data-analysis]
author: claude-domain-skills
---

# Investment Analysis

> Investment decision support based on financial data

## Applicable Scenarios

- Reading and analyzing financial statements
- Company valuation calculations
- Portfolio evaluation
- Industry comparison analysis

## Analysis Framework

**Process**: Industry analysis → Company analysis → Valuation analysis → Risk assessment

## Core Metrics

### Profitability

| Metric | Formula | Good Standard |
|------|------|----------|
| **ROE** | Net Income / Shareholders' Equity | > 15% |
| **ROA** | Net Income / Total Assets | > 5% (varies by industry; asset-light businesses run higher) |
| **Gross Margin** | Gross Profit / Revenue | Above industry average |
| **Net Margin** | Net Income / Revenue | Above industry average |

### Valuation Metrics

| Metric | Formula | Description |
|------|------|------|
| **P/E** | Share Price / EPS | Price-to-earnings ratio; the lower, the cheaper |
| **P/B** | Share Price / Book Value per Share | Price-to-book ratio |
| **PEG** | P/E / Earnings Growth Rate | < 1 may indicate undervaluation |
| **EV/EBITDA** | Enterprise Value / EBITDA | Valuation that accounts for debt |

## The Three Financial Statements

| Income Statement | Balance Sheet | Cash Flow Statement |
|--------|------------|------------|
| Revenue - Cost = Gross Profit | Assets = Liabilities + Equity | Operating + Investing + Financing |
| Gross Profit - Expenses = Net Income | Current vs Non-current | = Net Cash Flow |

## Best Practices

1. **Look at long-term trends** - Review at least 5 years of data
2. **Peer comparison** - Compare metrics with competitors
3. **Qualitative analysis** - Don't just look at numbers; understand the business model
4. **Margin of safety** - Use conservative valuations, leave room for error
5. **Diversify investments** - Don't put all your eggs in one basket

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Buying just because P/E looks low | Judge using multiple combined metrics |
| Chasing hot stocks | Independently analyze fundamentals |
| Ignoring cash flow | Cash flow matters more than reported profit |
| Panicking over short-term volatility | Focus on long-term value |

## Sharp Edges

### SE-1: Value Trap
- **Severity**: critical
- **Scenario**: A stock looks cheap (low P/E, low P/B), but the price keeps falling
- **Cause**: The company's fundamentals are deteriorating; the "cheap" price reflects the market's pessimistic outlook
- **Symptoms**: P/E 50%+ below peers, consecutive revenue declines, industry disruption, loss of major customers
- **Fix**: Check the 5-year revenue trend, analyze the moat, assess the industry outlook

### SE-2: Over-Reliance on a Single Metric
- **Severity**: high
- **Scenario**: Making investment decisions based solely on P/E or ROE
- **Minimum requirement**: Profitability + valuation metrics + cash flow + financial health + qualitative analysis

### SE-3: Anchoring Bias in Holding Decisions
- **Severity**: high
- **Scenario**: Refusing to sell at $80 because "I bought at $100"
- **Fix**: Ask yourself: "If I didn't own this stock, would I buy it at today's price?"

### SE-4: Confirmation Bias
- **Severity**: medium
- **Scenario**: Only seeking information that supports your own view while ignoring negative news
- **Fix**: Proactively search for "[Company Name] risks," list failure scenarios, review periodically

### SE-5: Earnings Quality Trap
- **Severity**: high
- **Scenario**: The company reports high net income, but it isn't actually sustainable
- **Red flags**: Operating cash flow << net income, large non-recurring gains/losses, accounts receivable growth > revenue growth
- **Check**: Operating cash flow / net income > 80%, accounts receivable growth <= revenue growth

## DCF Valuation Model

**Formula**: Intrinsic Value = Σₙ [FCFₙ / (1 + Discount Rate)ⁿ] + [Terminal Value / (1 + Discount Rate)ᴺ]

Note: the terminal value is a lump sum at year N and must itself be discounted back to present value — it is not simply added on top of the discounted cash flows.

**Steps**: Forecast FCF (5-10 years) → Calculate terminal value → Choose WACC → Discount all cash flows (including terminal value) to present value → Divide by share count

**Free Cash Flow**: FCF = Operating Cash Flow - Capital Expenditures

## Relative Valuation

**Decision logic**:
- P/E < industry average AND ROE > industry average → possibly undervalued
- P/E > industry average AND ROE < industry average → possibly overvalued

> Note: Being undervalued doesn't mean you should buy — understand the underlying reason first

## Financial Statement Red Flags

| Signal | Possible Problem |
|------|------|
| Revenue growing but cash flow declining | Accounts receivable issues |
| Falling inventory turnover | Slow-moving products |
| Continuously declining gross margin | Intensifying competition |
| Rising days sales outstanding | Poor customer payment ability |
| Rapidly rising debt ratio | Increasing financial risk |
| Frequent related-party transactions | Risk of improper benefit transfer |

## Systematic vs Unsystematic Risk

| Type | Description | Examples | Can Diversification Remove It? |
|------|------|------|------|
| **Systematic (market) risk** | Affects the entire market; also called non-diversifiable risk | Interest rate changes, recessions, inflation, geopolitical shocks | No — can only be hedged or reduced with asset allocation across asset classes |
| **Unsystematic (specific) risk** | Affects a single company or industry; also called diversifiable risk | Management missteps, product recalls, lawsuits, a single competitor's disruption | Yes — largely reduced by holding a diversified portfolio (~20-30 uncorrelated stocks) |

> This distinction is why "diversify investments" (Best Practices #5) works: it eliminates unsystematic risk, but systematic risk remains and is instead managed through asset allocation (see Asset Allocation Model in `extended/examples.md`) and position sizing.

## DuPont Analysis

**Formula**: ROE = Net Margin × Asset Turnover × Financial Leverage

**Analysis focus**: Which factor contributes the most? Is the high ROE driven by profitability or by leverage?

> See `extended/examples.md` for detailed examples

## Industry Analysis Framework

### Industry Life Cycle
- **Introduction stage**: High-risk, venture-capital-style investing
- **Growth stage**: Growth stock investing (high P/E acceptable)
- **Maturity stage**: Value stock investing (stable dividends)
- **Decline stage**: Avoid, or invest only in special situations

### Porter's Five Forces
Assess: threat of new entrants, supplier bargaining power, buyer bargaining power, threat of substitutes, existing competitive rivalry

> See `extended/examples.md` for a detailed assessment framework

## Moat Analysis

| Type | Description | Example |
|------|------|------|
| **Brand** | Consumers willing to pay a premium | Coca-Cola, Hermès |
| **Switching Costs** | High cost to switch products | SAP, Adobe |
| **Network Effects** | Value increases as more users join | Facebook, Visa |
| **Cost Advantage** | Low-cost production | Walmart, GEICO |
| **Intangible Assets** | Patents, licenses | Pfizer, franchising |

> See `extended/examples.md` for a detailed evaluation checklist

## Recommended Tools

- **Financial statement lookup**: SEC EDGAR (10-K/10-Q filings), Yahoo Finance, Simply Wall St
- **Valuation tools**: Excel DCF templates, Finviz
- **Research reports**: Broker research reports, Seeking Alpha, Morningstar
- **Tracking tools**: TradingView, Stock Portfolio Tracker

## Filing Cadence (U.S. Public Companies)

| Filing | Frequency | Contains |
|------|------|------|
| **10-K** | Annual | Audited financials, full risk factors, MD&A |
| **10-Q** | Quarterly | Unaudited financials, interim updates |
| **8-K** | As needed | Material events (M&A, leadership changes, guidance changes) |
| **DEF 14A (Proxy)** | Annual | Executive compensation, board structure, shareholder votes |

> U.S. GAAP governs how these statements are prepared; note where a company reports non-GAAP adjusted figures alongside GAAP results, and reconcile the two before trusting the adjusted number.

## Extended Resources

- `extended/templates.md` - Investment checklist, financial statement reading guide, quantitative screening criteria
- `extended/examples.md` - DuPont analysis examples, industry life cycle diagram, moat evaluation checklist

## Disclaimer

This content is for educational reference only and does not constitute investment advice. Investing involves risk; consult a professional advisor before making decisions.
