---
schema: "1.0"
name: strategy-optimization
version: "1.0.0"
description: A methodology for trading strategy optimization, an iterative process from diagnosis to reaching target performance
triggers:
  keywords:
    primary: [strategy optimization, improve strategy, improve win rate, improve returns]
    secondary: [win rate, backtest, parameter tuning, performance improvement]
  context_boost: [trading, quant, return, profit]
  context_penalty: [design, marketing, frontend]
  priority: high
keywords: [finance, trading, optimization, strategy, backtest]
dependencies:
  domain-skills: [quant-trading]
author: claude-domain-skills
---

# Strategy Optimization

> A systematic strategy optimization process, from diagnosing problems to reaching your goal

## Applicable Scenarios

- Win rate too low (< 40%)
- Returns not meeting target
- Strategy performs poorly in certain market regimes
- Need to hit a specific return target (e.g., 15%+)

## Core Process

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                Strategy Optimization Iteration Loop            │
│                                                                 │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐         │
│  │ ANALYZE │ → │ DIAGNOSE│ → │ RESEARCH│ → │IMPLEMENT│         │
│  │ Analyze │   │ Diagnose│   │ Research│   │Implement│         │
│  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘         │
│       │             │             │             │               │
│       ▼             ▼             ▼             ▼               │
│  Collect metrics  Find root cause  Search solutions  Implement changes │
│                                                                 │
│       ┌─────────────────────────────────────────┐               │
│       │                                         │               │
│       ▼                                         │               │
│  ┌─────────┐   ┌─────────┐                     │               │
│  │VALIDATE │ → │ ITERATE │ ────────────────────┘               │
│  │Validate │   │ Iterate │   Not met → back to ANALYZE          │
│  └────┬────┘   └─────────┘                                      │
│       │                                                         │
│       ▼                                                         │
│   Target met → document and deploy                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 1: ANALYZE

Collect current performance metrics:

| Metric | Description | Healthy Standard |
|------|------|----------|
| **Win Rate** | Proportion of profitable trades | > 40% |
| **Return** | Total P&L | Positive |
| **Drawdown** | Largest loss from peak | < 20% |
| **Number of Trades** | Statistical significance | > 100 trades |
| **Expectancy** | Expected profit per trade | Positive |

### Expectancy Formula

```
Expectancy = (WinRate × AvgWin) - ((1-WinRate) × AvgLoss)

Example:
Win rate: 54.5%, average win: $7.42, average loss: $8.26
Expectancy: (0.545 × 7.42) - (0.455 × 8.26) = +$0.29/trade

Positive expectancy × large number of trades = stable profit
```

## Phase 2: DIAGNOSE

**Key: find the root cause, not the symptom**

### Common Root Causes

| Symptom | Possible Root Cause | Diagnostic Method |
|------|---------|---------|
| Low win rate | Strategy doesn't fit the market regime | Check ADX / market regime |
| Low return | Poor SL/TP ratio | Analyze the profit/loss distribution |
| Large drawdown | Excessive leverage | Calculate risk exposure |
| Few trades | Entry conditions too strict | Loosen entry conditions |

### Market Regime Analysis

```
┌─────────────────────────────────────────────────────────────────┐
│  Market Regime vs Strategy Fit                                 │
│                                                                 │
│  ADX < 25 (ranging)   → Grid Trading, Mean Reversion            │
│  ADX > 25 (trending)  → Trend Following, Momentum               │
│  High volatility       → Scalping, reduce position size         │
│  Low volatility        → Grid Trading, prepare for breakout     │
│                                                                 │
│  ⚠️ Markets range 90% of the time — trend strategies will keep losing! │
└─────────────────────────────────────────────────────────────────┘
```

### Diagnostic Example

```
Symptom: win rate is only 25%
  ↓
Analysis: using an RSI+MACD trend-following strategy
  ↓
Check: average ADX < 25 (ranging 90% of the time)
  ↓
Root cause: a trend-following strategy applied in a ranging market
  ↓
Direction: need a ranging-market strategy (Grid Trading)
```

## Phase 3: RESEARCH

### Strategy-to-Market-Regime Mapping

| Market Regime | Recommended Strategy | Reason |
|---------|---------|------|
| **Ranging** (ADX < 25) | Grid Trading | Exploits price oscillation |
| | Mean Reversion | Reverts to the mean |
| | Scalping | Fast in, fast out |
| **Trending** (ADX > 25) | Trend Following | Go with the trend |
| | Momentum | Momentum persistence |
| | Breakout | Follow through on breakouts |
| **High Volatility** | Scalping | Quick profit-taking |
| | Reduce position size | Control risk |

### Sharp Edge: Validate Research Data

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️ SE-1: Research findings must be validated with real testing │
│                                                                 │
│  Severity: critical                                             │
│                                                                 │
│  Case:                                                          │
│  - Research claimed an 80-84% success rate for a chart pattern  │
│  - Actual backtest: made the strategy perform worse             │
│  - Reason: stock market data doesn't apply to crypto            │
│                                                                 │
│  Lesson:                                                        │
│  ❌ Implementing as soon as you see research data                │
│  ✅ Validate with real-data backtesting first                    │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 4: IMPLEMENT

### Parameter Tuning Strategy

| Parameter | Adjustment Direction | Impact |
|------|---------|------|
| **SL/TP Ratio** | Symmetric (1:1) vs asymmetric | Win rate vs profit/loss ratio |
| **Leverage** | 1x → 10x | Return vs risk |
| **Entry Threshold** | Loose vs strict | Number of trades vs quality |

### Key Findings

```
┌─────────────────────────────────────────────────────────────────┐
│  Optimization Findings                                          │
│                                                                 │
│  1. Symmetric SL/TP works better                                 │
│     - 1.8%/1.8% produces a ~55% win rate                        │
│     - Performs better than 1%/3%                                 │
│                                                                 │
│  2. Grid Trading in ranging markets                              │
│     - Win rate improved from 25% to 55%                         │
│     - Key: enable it when ADX < 25                               │
│                                                                 │
│  3. More trades + positive expectancy                            │
│     - Small profits accumulate into large returns                │
│     - 191 trades × $0.29/trade = +$55                            │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 5: VALIDATE

### Backtest Requirements

| Item | Minimum Requirement | Recommended |
|------|---------|------|
| Backtest period | 60 days | 180 days |
| Number of trades | 50 trades | 100+ trades |
| Out-of-sample test | Present | Required |
| Trading costs | Included | Include slippage |

### Validation Checklist

```markdown
□ Did the win rate improve?
□ Did the return hit the target?
□ Is the drawdown acceptable?
□ Is the expectancy positive?
□ How is out-of-sample performance?
□ No look-ahead bias (every signal only uses data that would actually have been available at that point in time)
```

> **Look-ahead bias warning**: after re-tuning parameters or adding new features, re-check that no future information leaked into signal generation (e.g., using a day's closing price to generate that same day's signal, or recalculating an indicator over the full series instead of a rolling window). A sudden, suspiciously large jump in backtest performance after a change is the classic symptom — see `enterprise-finance/quant-trading` SE-2 for the full diagnostic.

## Phase 6: ITERATE

```
Target not met → back to ANALYZE
  │
  ├── Analyze new results
  ├── Adjust parameters or strategy
  ├── Re-validate
  └── Repeat until target is met
```

## Best Practices

1. **Diagnose before you act** - Don't blindly tune parameters
2. **Change one variable at a time** - So you know what actually worked
3. **Validate research conclusions** - Theoretical data isn't trustworthy
4. **Sufficient number of trades** - For statistical significance
5. **Document successful approaches** - Write them up as ADRs

## Common Mistakes

| Mistake | Consequence | Fix |
|------|------|------|
| ❌ Only tuning parameters, never changing strategy | Fails to solve the root problem | Diagnose the root cause first |
| ❌ Backtest period too short | Overfitting | Use 180+ days |
| ❌ Ignoring market regime | Strategy-market mismatch | Use regime detection |
| ❌ Chasing a high win rate | Sacrifices the profit/loss ratio | Focus on expectancy |

## Success Case

```
Before optimization:
- Return: 0.46%
- Win rate: 23.5%
- Strategy: RSI+MACD (a trend-following strategy used in a ranging market)

Diagnosis:
- ADX < 25 (ranging) 90% of the time
- Trend-following strategy was unsuitable

After optimization:
- Return: +19.93%
- Win rate: 54.5%
- Strategy: Grid Trading + Regime Detection

Key changes:
1. Added a Grid Trading strategy
2. Used regime detection to dynamically switch strategies
3. Symmetric SL/TP (1.8%/1.8%)
4. 10x leverage + 191 trades
```

## Risk Warning

```
⚠️ High returns come with high risk

- The above case had a maximum drawdown of 48.84%
- Consider lowering leverage (5x) in exchange for lower risk
- Continuously monitor changes in market regime
- Set up a stop-loss mechanism
```

## Reference Configuration

```python
# A validated configuration
config = {
    "leverage": 10.0,          # or 5.0 to lower risk
    "stop_loss_pct": 0.018,    # 1.8%
    "take_profit_pct": 0.018,  # 1.8%
    "top_n_signals": 3,
    "min_confidence": 0.5,
    "use_regime_selector": True,
}
```

## Related Skills

- `enterprise-finance/quant-trading` - Quantitative trading fundamentals
- `enterprise-finance/investment-analysis` - Investment analysis
- `enterprise-methodology/knowledge-acquisition-4c` - Learning new domains
