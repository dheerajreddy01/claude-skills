---
schema: "1.0"
name: business-strategy
version: "1.0.0"
description: Business strategy: blue ocean strategy, differentiation, competitive advantage, value innovation, and business model design
domain: business
triggers:
  keywords:
    primary: [strategy, blue ocean, business model, competitive advantage, positioning]
    secondary: [differentiation, value proposition, moat, B2B, B2C, D2C, platform]
  context_boost: [innovation, market, analysis, insight]
  context_penalty: [technology, code, frontend]
  priority: high
dependencies:
  software-skills: [documentation]
author: claude-domain-skills
---

# Business Strategy

> Create uncontested market space and make competition irrelevant

## Applicable Scenarios

- New venture/product strategy planning
- Market positioning and differentiation
- Business model design and innovation
- Competitive strategy analysis
- Value proposition design

## Blue Ocean Strategy Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  Red Ocean vs. Blue Ocean                                        │
│                                                                 │
│  Red Ocean                    Blue Ocean                         │
│  ─────────────────           ─────────────────                  │
│  Compete in existing markets  Create new market space            │
│  Beat the competition         Make competition irrelevant        │
│  Exploit existing demand      Create and capture new demand      │
│  Value-cost trade-off         Break the value-cost trade-off     │
│  Choose differentiation       Pursue differentiation and low     │
│  OR low cost                  cost simultaneously                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### The Four Actions Framework (ERRC)

| Action | Question | Purpose |
|------|------|------|
| **Eliminate** | Which factors that the industry takes for granted should be eliminated? | Reduce cost |
| **Reduce** | Which factors should be reduced well below the industry standard? | Reduce cost |
| **Raise** | Which factors should be raised well above the industry standard? | Increase value |
| **Create** | Which factors should be created that the industry has never offered? | Increase value |

### Strategy Canvas

```
Value
  ↑
High │           ●───●
   │          /     \
   │    ●────●       ●
Mid │   /              \
   │  ●                ●───●
Low │ ●                      ●
   └─┴──┴──┴──┴──┴──┴──┴──┴──→ Competing Factors
     A  B  C  D  E  F  G  H

── Industry standard
── Our strategy
── Competitors

Goal: create a differentiated value curve
```

## Competitive Strategy

### Porter's Five Forces (Industry Analysis)

| Force | Key Question |
|------|------|
| Threat of New Entrants | How easily can new competitors enter the market? |
| Bargaining Power of Suppliers | Can suppliers dictate prices or terms? |
| Bargaining Power of Buyers | Can customers demand lower prices or better quality? |
| Threat of Substitutes | Are there alternative ways to meet the same need? |
| Rivalry Among Existing Competitors | How intense is competition within the industry? |

**Use case**: analyze industry attractiveness before choosing a competitive strategy below

### Porter's Generic Strategies

```
┌─────────────────────────────────────────────────────────────────┐
│                     Source of Competitive Advantage             │
│              ┌────────────────┬────────────────┐               │
│              │   Low Cost     │  Differentiation│               │
│  ┌───────────┼────────────────┼────────────────┤               │
│  │  Broad    │  Cost Leader   │  Differentiation│              │
│  │  Market   │                │                │              │
│  ├───────────┼────────────────┼────────────────┤               │
│  │  Niche    │  Cost Focus    │  Diff. Focus   │               │
│  │  Market   │                │                │              │
│  └───────────┴────────────────┴────────────────┘               │
│                                                                 │
│  ⚠️ Avoid getting "stuck in the middle": no clear strategic     │
│  position                                                        │
└─────────────────────────────────────────────────────────────────┘
```

### Sources of Competitive Advantage

| Type | Description | Examples |
|------|------|------|
| **Cost advantage** | A lower cost structure than competitors | Walmart, Costco |
| **Differentiation advantage** | Unique value that lets customers pay a premium | Apple, Tesla |
| **Network effects** | Value increases as more users join | Facebook, Uber |
| **Switching costs** | High cost to leave | Adobe, SAP |
| **Brand moat** | Strong brand recognition | Coca-Cola, Nike |
| **Patents/exclusivity** | Legal or exclusive protection | Pharmaceutical patents |

## Business Model Design

### Business Model Canvas

```markdown
┌──────────────────┬──────────────────┬──────────────────┐
│   Key Partners   │  Key Activities  │ Value Proposition│
│                  │                  │                  │
│                  │                  │                  │
│  • Suppliers     │  • Product dev   │  • What problem  │
│  • Strategic     │  • Operations    │    does it solve?│
│    alliances     │  • Platform      │  • What need does│
│                  │    maintenance   │    it meet?      │
│                  ├──────────────────┤  • What value    │
│                  │   Key Resources  │    does it       │
│                  │                  │    deliver?      │
│                  │  • Talent        │                  │
│                  │  • Technology    │                  │
│                  │  • Capital       │                  │
├──────────────────┴──────────────────┼──────────────────┤
│          Cost Structure             │  Revenue Streams │
│                                     │                  │
│  • Fixed costs: staff, rent         │  • Sales revenue │
│  • Variable costs: marketing,       │  • Subscription  │
│    customer service                 │    revenue       │
│                                     │  • Licensing     │
│                                     │    revenue       │
└─────────────────────────────────────┴──────────────────┘
              │                              │
              │       Customer Relations     │
              ├──────────────────────────────┤
              │       Channels               │
              ├──────────────────────────────┤
              │       Customer Segments       │
              └──────────────────────────────┘
```

### Value Proposition Design

```markdown
## Value Proposition Canvas

### Customer Profile
┌─────────────────────────────────────┐
│  Jobs                               │
│  • What is the customer trying to   │
│    get done?                        │
│  • Functional, social, emotional    │
│    jobs                             │
├─────────────────────────────────────┤
│  Pains                              │
│  • What frustrates the customer?    │
│  • Risks, obstacles, undesired      │
│    outcomes                         │
├─────────────────────────────────────┤
│  Gains                              │
│  • What outcomes does the customer  │
│    want?                            │
│  • Required, expected, desired,     │
│    unexpected gains                 │
└─────────────────────────────────────┘

### Value Map
┌─────────────────────────────────────┐
│  Products & Services                │
│  • What are we offering?            │
├─────────────────────────────────────┤
│  Pain Relievers                     │
│  • How do we ease customer pains?   │
├─────────────────────────────────────┤
│  Gain Creators                      │
│  • How do we create customer gains? │
└─────────────────────────────────────┘

Goal: achieve Product-Market Fit
```

## Cultivating Insight

### Sources of Insight

| Source | Method |
|------|------|
| **Customer observation** | Observe user behavior in the field, not surveys |
| **Anomalies** | Notice data or behavior that defies expectations |
| **Extreme users** | Study heavy users and non-adopters |
| **Analogical thinking** | How do other industries solve similar problems? |
| **Reverse thinking** | What if we did the opposite? |

### Insight Note Template

```markdown
## 💡 Insight Log

**Observation**: (What did you see?)
**Why**: (What's the underlying reason?)
**So what**: (What opportunity does this represent?)
**Action**: (What can we do about it?)
```

## Differentiation Strategy

### Sources of Differentiation

```
┌─────────────────────────────────────────────────────────────────┐
│  Dimensions of Differentiation                                   │
│                                                                 │
│  Product Differentiation                                         │
│  ├─ Features: more/fewer/different features                     │
│  ├─ Quality: more durable/more refined                          │
│  ├─ Design: more attractive/more user-friendly                  │
│  └─ Technology: more advanced/simpler                           │
│                                                                 │
│  Service Differentiation                                         │
│  ├─ Speed: faster delivery                                      │
│  ├─ Convenience: easier to buy/use                               │
│  ├─ Customization: personalized service                         │
│  └─ After-sales: better support                                 │
│                                                                 │
│  Image Differentiation                                           │
│  ├─ Brand: emotional connection                                 │
│  ├─ Story: brand narrative                                      │
│  └─ Values: social responsibility                               │
└─────────────────────────────────────────────────────────────────┘
```

### Differentiation Checklist

```markdown
## Differentiation Assessment

□ Is it valuable to customers? (willing to pay)
□ Is it hard for competitors to imitate?
□ Does it match our capabilities?
□ Can it be sustained over time?
□ Can it be communicated clearly?
```

## Strategic Planning Process

### Strategic Thinking Framework

```markdown
1. Where to Play
   - Target market?
   - Target customers?
   - Geographic scope?
   - Product/service scope?

2. How to Win
   - Differentiation or cost leadership?
   - Unique value proposition?
   - Core competencies?

3. What Capabilities
   - Capabilities we must have?
   - Capabilities we need to build?
   - Capabilities we can outsource?

4. What Management Systems
   - Metrics?
   - Incentive systems?
   - Decision-making processes?
```

## Business Model Types

### Classification by Transaction Parties

```
┌─────────────────────────────────────────────────────────────────┐
│  Business Model Matrix                                           │
│                                                                 │
│              Selling to whom?                                    │
│              ┌────────────────┬────────────────┐               │
│              │  Consumer (C)  │  Business (B)  │               │
│  ┌───────────┼────────────────┼────────────────┤               │
│  │ Business  │      B2C       │      B2B       │               │
│  │ (B) sells │  E-commerce,   │  SaaS,         │               │
│  │           │  retail        │  wholesale     │               │
│  ├───────────┼────────────────┼────────────────┤               │
│  │ Consumer  │      C2C       │      C2B       │               │
│  │ (C) sells │  Resale        │  Influencers,  │               │
│  │           │  platforms     │  freelance     │               │
│  └───────────┴────────────────┴────────────────┘               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

| Model | Description | Typical Examples |
|------|------|----------|
| **B2C** | Business to consumer | Amazon, Netflix, Starbucks |
| **B2B** | Business to business | Salesforce, AWS, Synnex |
| **C2C** | Consumer to consumer | eBay, Shopee, Airbnb |
| **C2B** | Consumer to business | Influencer marketing, Upwork, crowdfunding |
| **B2B2C** | Business to business to consumer | Brand → channel → consumer |
| **D2C** | Direct to consumer | Brand websites, Tesla's direct sales |

### E-Commerce Models

```markdown
## E-Commerce Revenue Models

### 1. Product Sales
- Self-operated inventory
- Consignment/dropshipping
- Subscriptions (recurring purchases)

### 2. Platform Commission
- Transaction fees
- Listing fees
- Advertising revenue

### 3. Membership
- Free-shipping membership
- Discount membership
- Exclusive services

## Payment Integration Considerations

| Payment Method | Pros | Cons |
|----------|------|------|
| Credit card | Instant, widely used | High fees |
| Third-party payment | Secure, convenient | Requires integration |
| Bank transfer | Low fees | Slow confirmation |
| Cash on delivery | High trust | Risk of refusal |
| Installments | Increases order value | Requires risk management |
```

### Platform Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│  Platform Business Model                                         │
│                                                                 │
│  Two-Sided Market                                                │
│                                                                 │
│  Supply Side                Platform                Demand Side  │
│  ┌─────────┐           ┌──────────┐           ┌─────────┐      │
│  │ Sellers │  ←─────→  │ Matching │  ←─────→  │ Buyers  │      │
│  │ Drivers │           │  Trust   │           │ Riders  │      │
│  │Landlords│           │ Payments │           │ Guests  │      │
│  └─────────┘           └──────────┘           └─────────┘      │
│                                                                 │
│  Key challenges:                                                 │
│  1. Chicken-and-egg problem (supply first or demand first?)     │
│  2. Cultivating network effects                                 │
│  3. Balancing supply and demand                                 │
│  4. Quality control                                             │
└─────────────────────────────────────────────────────────────────┘
```

## Common Pitfalls

| Pitfall | Solution |
|------|------|
| Copying competitors | Focus on creating unique value |
| Looking only at data, not people | Observe customer behavior in the field |
| Trying to do too much | Make clear trade-offs, stay focused |
| Assuming customers know what they want | Test and validate assumptions |
| Ignoring execution difficulty | Assess organizational capability |

## Related Resources

- [Blue Ocean Strategy](https://www.blueoceanstrategy.com/)
- [Strategyzer (Business Model Canvas)](https://www.strategyzer.com/)
- [Playing to Win - A.G. Lafley](https://rogerlmartin.com/lets-read/playing-to-win)
- [Good Strategy Bad Strategy - Richard Rumelt](https://www.amazon.com/Good-Strategy-Bad-Difference-Matters/dp/0307886239)
