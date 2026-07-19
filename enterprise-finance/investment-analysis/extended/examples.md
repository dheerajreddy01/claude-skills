# Investment Analysis Examples and Visualization Framework

> Examples and detailed diagrams moved out of the main SKILL.md

## DuPont Analysis

### Complete Framework Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  DuPont Analysis: Breaking ROE Into Three Drivers                │
│                                                                 │
│  ROE = Net Margin × Asset Turnover × Financial Leverage         │
│                                                                 │
│    Net Income   Revenue      Total Assets                       │
│  = ────────── × ──────── × ─────────────                       │
│    Revenue      Total Assets  Shareholders' Equity              │
│                                                                 │
│  ┌────────────┬────────────┬────────────┐                      │
│  │Profitability│ Operating  │  Financial │                      │
│  │ Net Margin  │ Efficiency │  Structure │                      │
│  │             │ Turnover   │  Leverage  │                      │
│  └────────────┴────────────┴────────────┘                      │
│                                                                 │
│  Analysis focus:                                                │
│  • Which factor contributes the most?                           │
│  • Is high ROE driven by profitability or by leverage?          │
│  • What's causing the trend to change?                          │
└─────────────────────────────────────────────────────────────────┘
```

### DuPont Analysis Comparison Example

| Company | Net Margin | Turnover | Leverage | ROE | Interpretation |
|------|--------|--------|------|-----|------|
| Company A | 20% | 0.8 | 1.5 | 24% | High-margin type: wins on high gross margin |
| Company B | 5% | 3.0 | 2.0 | 30% | High-turnover type: thin-margin, high-volume model |
| Company C | 8% | 1.2 | 3.0 | 28.8% | High-leverage type: higher financial risk |

**Investment implications**:
- Company A: Brand or technology advantage, likely a deeper moat
- Company B: Retail or fast-moving-consumer-goods characteristics, watch competition
- Company C: Beware debt risk, understand the source of leverage

---

## Industry Life Cycle Analysis

### Complete Life Cycle Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  Industry Life Cycle and Investment Strategy                    │
│                                                                 │
│  Market                                                         │
│  Size ↑                                                         │
│       │              Maturity                                   │
│       │           ┌──────────┐                                 │
│       │          /            \                                 │
│       │  Growth /              \ Decline                        │
│       │        /                \                               │
│       │  Introduction             \                              │
│       │  /                         \                            │
│       └────────────────────────────────→ Time                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Characteristics and Strategy by Stage

| Stage | Characteristics | Risk | Investment Strategy |
|------|------|------|----------|
| **Introduction** | High growth potential | Technology risk, negative cash flow | Venture-style, high risk / high reward |
| **Growth** | Rapid growth, intensifying competition | Market-share battles, capital needs | Growth stocks (high P/E acceptable) |
| **Maturity** | Stable cash flow, price competition | Slowing growth | Value stocks (stable dividends) |
| **Decline** | Shrinking market, consolidation opportunities | Structural decline | Avoid, or invest only in special situations |

---

## Moat Evaluation Checklist

### Five Types of Moats

| Type | Description | Example |
|------|------|------|
| **Brand** | Consumers willing to pay a premium | Coca-Cola, Hermès |
| **Switching Costs** | High cost to switch products | SAP, Adobe |
| **Network Effects** | Value increases as more users join | Facebook, Visa |
| **Cost Advantage** | Low-cost production | Walmart, GEICO |
| **Intangible Assets** | Patents, licenses | Pfizer, franchising |

### Brand Moat Evaluation
- [ ] Consumers can name the brand
- [ ] Willing to pay a premium for it
- [ ] High brand loyalty
- [ ] Brand value has persisted for years

### Switching-Cost Moat Evaluation
- [ ] Switching costs customers a significant amount
- [ ] Requires retraining staff
- [ ] Data migration is difficult
- [ ] Long-term contract lock-in

### Network-Effect Moat Evaluation
- [ ] More users increases product value
- [ ] Two-sided market effects exist
- [ ] Competitors struggle to replicate network scale
- [ ] High user stickiness

### Moat Durability Assessment
| Rating | Description | Investment Recommendation |
|------|------|----------|
| Wide and durable | Clear advantage, hard to replicate | Strong buy |
| Wide but under threat | Advantage exists but faces challenges | Assess carefully |
| Narrow | Weak or easily replicated advantage | Short-term opportunity |
| No moat | No differentiation | Pure price competition, avoid |

---

## Porter's Five Forces Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  Porter's Five Forces                                            │
│                                                                 │
│              Threat of New Entrants                             │
│                      ↓                                          │
│    Supplier      →  Industry   ←      Buyer                    │
│  Bargaining Power    Rivalry       Bargaining Power              │
│                 Threat of Substitutes                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Assessment Question Checklist

| Force | Assessment Question | Favorable Score (good for the industry) |
|------|----------|----------------------|
| Threat of new entrants | Are barriers to entry high? (capital, regulation, technology) | High barriers, hard to enter |
| Supplier bargaining power | Are suppliers concentrated? Do they have leverage? | Suppliers are fragmented |
| Buyer bargaining power | Do buyers have choices? Are they price-sensitive? | Buyers are fragmented, brand loyal |
| Threat of substitutes | Do substitutes exist? Is switching cost high? | No substitutes, or high switching cost |
| Existing rivalry | Is competition intense? Degree of differentiation? | High differentiation, orderly competition |

**Scoring method**: 1-5 points per item; higher total scores indicate a more attractive industry

---

## Scenario Valuation Example

### Three-Scenario Valuation

```
┌─────────────────────────────────────────────────────────────────┐
│  Three-Scenario Valuation Method                                 │
│                                                                 │
│  Bull Case                                                      │
│  ├─ Assumptions: revenue growth 15%, gross margin improves      │
│  ├─ Probability: 25%                                            │
│  └─ Valuation: $150                                             │
│                                                                 │
│  Base Case                                                      │
│  ├─ Assumptions: revenue growth 8%, status quo maintained       │
│  ├─ Probability: 50%                                            │
│  └─ Valuation: $120                                             │
│                                                                 │
│  Bear Case                                                      │
│  ├─ Assumptions: flat revenue, declining profit                 │
│  ├─ Probability: 25%                                            │
│  └─ Valuation: $80                                              │
│                                                                 │
│  Weighted valuation = 0.25×150 + 0.50×120 + 0.25×80 = $117.50   │
│                                                                 │
│  Decision: if the current price is < $94 (20% margin of safety, $117.50 × 0.8), consider buying │
└─────────────────────────────────────────────────────────────────┘
```

---

## Asset Allocation Model

### Risk Tolerance vs Asset Allocation

| Type | Stocks | Bonds | Cash | Suitable For |
|------|------|------|------|----------|
| Conservative | 30% | 50% | 20% | Retirees, low risk tolerance |
| Moderate | 50% | 35% | 15% | Middle-aged, stable income |
| Aggressive | 70% | 20% | 10% | Young, long-term investing |

### Allocation Principles
- **Age rule**: Bond allocation = 100 - age
- **Diversification principle**: No single holding exceeds 10%
- **Rebalancing**: Review and adjust annually
