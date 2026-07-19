---
schema: "1.0"
name: quant-trading
version: "1.1.0"
description: Quantitative trading strategy development, backtesting, and risk management
triggers:
  keywords:
    primary: [quant, trading, backtest, algo-trading]
    secondary: [strategy, factor, arbitrage, hedge]
  context_boost: [python, pandas, numpy, finance, investment, stock, futures]
  context_penalty: [design, marketing, frontend]
  priority: high
keywords: [finance, trading, quantitative, investment]
dependencies:
  software-skills: [python, database, data-analysis]
author: claude-domain-skills
---

# Quant Trading

> Systematic, data-driven trading strategy development

## Applicable Scenarios

- Developing trading strategies (trend following, mean reversion, arbitrage)
- Strategy backtesting and performance analysis
- Risk management and position sizing
- Factor research and alpha discovery

## Strategy Development Process

```
Hypothesis formation → Data preparation → Strategy coding → Backtest validation → Risk control → Live monitoring
```

## Core Knowledge

### Strategy Types

| Type | Description | Risk |
|------|------|------|
| **Trend Following** | Go with the trend, buy strength/sell weakness | Losses in choppy markets |
| **Mean Reversion** | Price reverts after deviating | Losses in trending markets |
| **Statistical Arbitrage** | Pairs trading, spread convergence | Correlation breakdown |
| **High-Frequency Trading** | Capturing microsecond-level spreads | High technical risk |

### Risk Metrics

| Metric | Formula | Good Standard |
|------|------|----------|
| **Sharpe Ratio** | (Return - Risk-Free Rate) / Standard Deviation | > 1.5 |
| **Sortino Ratio** | (Return - Risk-Free Rate) / Downside Deviation | > 2.0 (only penalizes downside volatility, so it's typically higher than Sharpe) |
| **Max Drawdown** | Largest decline from peak to trough | < 20% |
| **Calmar Ratio** | Annualized Return / Max Drawdown | > 1.0 |
| **Win Rate** | Winning Trades / Total Trades | > 50% |

> Win rate alone doesn't determine profitability — a strategy with a 35% win rate can still be profitable if the average win is large enough relative to the average loss. Always pair win rate with the profit/loss (risk-reward) ratio, i.e., **Expectancy = (Win Rate × Avg Win) - (Loss Rate × Avg Loss)**; see `enterprise-finance/strategy-optimization` for the full expectancy framework.

### Common Factors

| Factor | Description | Logic |
|------|------|------|
| **Value** | Low P/E, P/B | Cheap stocks perform well long-term |
| **Momentum** | Past winners keep winning | Trend persistence |
| **Quality** | High ROE, low debt | Good companies command a premium |
| **Size** | Small-cap premium | Liquidity compensation |
| **Volatility** | Low-volatility anomaly | Low risk, high return |

### Backtest Checklist

- **Data quality**: Is there survivorship bias?
- **Look-ahead bias**: Was future data used?
- **Overfitting**: Are parameters over-optimized?
- **Trading costs**: Are commissions and slippage included?
- **Out-of-sample testing**: Was a test set held out?

## Best Practices

1. **Start simple, then add complexity** - Begin with simple strategies and gradually increase complexity
2. **Out-of-sample validation** - Always hold out a portion of data for final testing
3. **Account for trading costs** - Include realistic commissions and slippage in backtests
4. **Diversify risk** - Don't put all your capital into a single strategy
5. **Continuous monitoring** - Monitor strategy performance in live trading and set stop-loss conditions

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Going live because backtest returns look amazing | Check for overfitting |
| Ignoring trading costs | Include realistic commissions and slippage |
| Training on all available data | Split into train/validation/test sets |
| Going all-in on a single strategy | Diversify risk across multiple strategies |

## Sharp Edges

### SE-1: Backtest Overfitting
- **Severity**: critical
- **Scenario**: The strategy performs excellently in backtests (Sharpe > 3) but loses money live
- **Cause**: Parameters were over-optimized on historical data, capturing noise rather than signal
- **Symptoms**: Backtest Sharpe > 3, live performance < 50% of backtest, parameter-sensitive, too few trades
- **Detection**: `sharpe.*[3-9]\.|sharpe.*\d{2,}`
- **Fix**: Use walk-forward validation with 5+ rolling test segments

### SE-2: Look-ahead Bias
- **Severity**: critical
- **Scenario**: Future data is unintentionally used during backtesting
- **Cause**: Time ordering wasn't respected during data processing
- **Symptoms**: Backtest returns are unrealistically perfect, using the current day's closing price as the current day's signal
- **Detection**: `shift\(-|iloc\[-1\].*today`
- **Fix**: Signals must be lagged by one day: `signal = prices.shift(1) > ma.shift(1)`

### SE-3: Survivorship Bias
- **Severity**: high
- **Scenario**: Backtesting only uses stocks that currently exist
- **Cause**: Delisted stocks are excluded, inflating apparent strategy performance
- **Symptoms**: Backtest returns are noticeably higher than reality, small-cap strategies look especially good
- **Fix**: Use a complete database that includes delisted stocks; use point-in-time data

### SE-4: Ignoring Trading Costs and Slippage
- **Severity**: high
- **Scenario**: A high-turnover strategy looks very profitable
- **Cause**: Commissions and slippage weren't accounted for; real costs eat up all the profit
- **Symptoms**: Annual turnover > 1000%, per-trade profit < 0.5%, backtest fills at closing price
- **Fix**: Include commission (0.1%) + slippage (0.2%) = 0.3% per trade

### SE-5: Overconfidence in In-Sample Performance
- **Severity**: medium
- **Scenario**: Deciding to go live based only on in-sample backtest results
- **Cause**: No independent test set was held out
- **Symptoms**: No out-of-sample testing, validation set reused repeatedly, test set too small
- **Fix**: Split data 60/20/20 (train/validation/test); use the test set only once

## Risk Management Framework

**Layer 1 - Trade level**: 2% stop-loss per trade, volatility-based take-profit, trailing stops to protect gains
**Layer 2 - Strategy level**: No single strategy exceeds 20% of total capital, halve position size if drawdown exceeds 15%
**Layer 3 - Portfolio level**: Daily VaR under 2%, stress testing, maintain 20% cash

### Kelly Formula

```
f* = (p × b - q) / b
f* = optimal betting fraction, p = win rate, q = loss rate, b = odds
Example: win rate 55%, odds 1.5 → f* = 25% (in practice, use Half-Kelly at 12.5%)
```

### Execution Risk

| Risk | Description | Countermeasure |
|------|------|------|
| **Slippage** | Difference between fill price and expected price | Limit orders, staged execution |
| **Liquidity** | Unable to fill the desired quantity | Avoid small-cap stocks, set position caps |
| **System Failure** | Software/network outage | Backup systems, manual monitoring |
| **Black Swan** | Extreme events | Position caps, stop-loss mechanisms |

## Recommended Tools

- **Backtrader** - Python backtesting framework
- **QuantConnect** - Cloud-based quantitative platform
- **TradingView** - Chart analysis and strategy testing
- **Zipline** - Quantopian's open-source backtesting engine

## Extended Resources

- Code examples: [extended/code-examples.md](extended/code-examples.md)
- Templates and checklists: [extended/templates.md](extended/templates.md)

## Reference Resources

- Quantitative Trading - Ernest Chan
- Advances in Financial Machine Learning - Marcos Lopez de Prado

## Disclaimer

Quantitative trading involves high risk; past performance does not guarantee future results. This content is for educational reference only and does not constitute investment advice.
