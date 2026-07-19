# Quant Trading Templates and Checklists

> This document contains the backtest report template, live-trading checklist, and more

## Backtest Report Template

```markdown
## Strategy Backtest Report

### Basic Information
- Strategy name:
- Backtest period:
- Trading instrument:
- Initial capital:

### Performance Metrics
| Metric | Value |
|------|------|
| Annualized return | |
| Sharpe ratio | |
| Max drawdown | |
| Calmar ratio | |
| Win rate | |
| Profit/loss ratio | |

### Risk Analysis
- Longest consecutive losing streak (days):
- Largest single-trade loss:
- VaR (95%):

### Trading Statistics
- Total number of trades:
- Average trades per year:
- Average holding period:

### Conclusions and Recommendations
[Analysis of the strategy's strengths and weaknesses]
```

## Live Trading Go-Live Checklist

### Backtest Validation
- [ ] Out-of-sample test passed
- [ ] Trading costs included
- [ ] No look-ahead bias
- [ ] Reasonable slippage estimate

### Risk Control Setup
- [ ] Per-trade stop-loss configured
- [ ] Strategy-level max-drawdown stop configured
- [ ] Daily loss limit configured
- [ ] Position size cap configured

### System Readiness
- [ ] Capital is in place
- [ ] API permissions correct
- [ ] Backup systems ready
- [ ] Monitoring/alerting configured

### Psychological Readiness
- [ ] Accept that early losses are possible
- [ ] Won't change the strategy over a single loss
- [ ] Set a review cadence (e.g., monthly)
- [ ] Have clear criteria for discontinuing the strategy

## Overfitting Diagnostic Table

| Signal | Description | Solution |
|------|----------|----------|
| Backtest Sharpe > 3 | Very hard to achieve in reality | Check for data leakage |
| Parameter sensitivity | Small parameter changes cause big result swings | Use a robust parameter range |
| Few trades | Insufficient sample for statistical significance | Extend the backtest period |
| Out-of-sample collapse | Large gap between train/test performance | Re-examine the strategy logic |

### Methods to Prevent Overfitting

1. **Walk-Forward Analysis** - Rolling-window training and testing that simulates real-world usage
2. **Cross-Validation** - K-fold cross-validation (be careful about order with time series)
3. **Parameter robustness testing** - Test performance around the chosen parameters to avoid picking extreme values
4. **Multi-market validation** - Test across different markets; the strategy logic should generalize

## Considerations for ML Strategy Development

### Viable Applications
- Factor combination optimization
- Risk forecasting (volatility prediction)
- Trade execution optimization
- Alternative data processing (text, images)

### Common Pitfalls

| Pitfall | Description |
|------|------|
| **Data Leakage** | Unintentionally using future information during training |
| **Sample Bias** | Only using data from a specific market regime |
| **Overfitting** | A complex model memorizes noise |
| **Non-Stationarity** | The distribution of financial data changes over time |

### Feature Engineering Recommendations

**Good features:**
- Technical indicators (RSI, MACD, Bollinger %B)
- Fundamental ratios (Z-scores of P/E, P/B)
- Momentum features (returns across multiple timeframes)
- Volatility features (realized volatility, implied volatility)

**Features to avoid:**
- Raw prices (non-stationary)
- Unnormalized values
- Lagged variables highly correlated with the target (leakage)
