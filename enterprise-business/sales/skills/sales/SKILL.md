---
schema: "1.0"
name: sales
version: "1.0.0"
description: Sales techniques: B2B/B2C sales, customer development, e-commerce operations, and closing strategies
domain: business
triggers:
  keywords:
    primary: [sales, CRM, ecommerce, B2B, B2C]
    secondary: [closing, prospecting, quote, negotiation, AOV, average order value, cart]
  context_boost: [customer, revenue, conversion]
  context_penalty: [code, api, database]
  priority: high
dependencies:
  software-skills: [e-commerce]
author: claude-domain-skills
---

# Sales & E-Commerce

> The complete sales process, from prospecting to closing

## Applicable Scenarios

- B2B enterprise sales strategy
- B2C retail and e-commerce operations
- Customer development and business expansion
- Quoting, negotiation, and closing techniques
- E-commerce data analysis and optimization

## Sales Process

```
┌─────────────────────────────────────────────────────────────────┐
│  B2B Sales Funnel                                                │
│                                                                 │
│  ┌─────────────────────────────────────────────────────┐       │
│  │                     Leads                             │       │
│  └─────────────────────────────────────────────────────┘       │
│              ↓  Initial contact, qualification                  │
│        ┌───────────────────────────────────────┐               │
│        │       Qualified Lead (MQL → SQL)       │               │
│        └───────────────────────────────────────┘               │
│              ↓  Needs discovery, solution demo                  │
│           ┌─────────────────────────────────┐                  │
│           │            Opportunity           │                  │
│           └─────────────────────────────────┘                  │
│              ↓  Quoting, negotiation, objection handling        │
│              ┌───────────────────────────┐                     │
│              │        Closed Won          │                     │
│              └───────────────────────────┘                     │
│                                                                 │
│  Key metrics: conversion rate, sales cycle, average deal size,  │
│  win rate                                                        │
└─────────────────────────────────────────────────────────────────┘
```

## B2B Sales Techniques

### SPIN Selling

| Stage | Question Type | Purpose |
|------|----------|------|
| **S** Situation | Situational questions | Understand the customer's current state |
| **P** Problem | Problem questions | Uncover pain points |
| **I** Implication | Implication questions | Expand the impact of the pain point |
| **N** Need-payoff | Need-payoff questions | Lead toward the solution |

### Handling Sales Objections

```markdown
## LAER Framework

1. **Listen** - Let the customer fully express themselves
2. **Acknowledge** - Show understanding
3. **Explore** - Dig into their real concerns
4. **Respond** - Address it specifically

## Common Objection Types

| Objection | Real Meaning | How to Handle |
|------|----------|----------|
| "It's too expensive" | Value hasn't been communicated | Emphasize ROI |
| "I need to think about it" | There's an unresolved concern | Explore the real issue |
| "No budget" | Priority isn't high enough | Quantify the cost of the pain point |
| "We'll just keep using what we have" | Fear of change risk | Offer a transition plan |
```

### Quoting and Negotiation

```
┌─────────────────────────────────────────────────────────────────┐
│  Quoting Strategy                                                │
│                                                                 │
│  1. Anchoring Effect                                            │
│     • Present the higher-tier plan first, then the middle tier │
│     • Three-tier pricing: Basic / Pro / Enterprise              │
│                                                                 │
│  2. Value First                                                 │
│     • Discuss value before price                                │
│     • Use an ROI framework: input vs. output                    │
│                                                                 │
│  3. Scarcity                                                    │
│     • Limited-time offers                                       │
│     • Limited spots                                             │
│                                                                 │
│  Negotiation principles:                                         │
│  • Always have a fallback (BATNA)                               │
│  • Don't reveal your bottom line first                          │
│  • Trade concessions, don't just give them away                 │
└─────────────────────────────────────────────────────────────────┘
```

## E-Commerce Operations

### Core Metrics

| Metric | Description | Formula |
|------|------|------|
| **GMV** | Gross merchandise value | Number of orders × AOV |
| **AOV** | Average order value | Revenue / number of orders |
| **Cart conversion rate** | Add-to-cart to purchase | Orders / carts created |
| **Cart abandonment rate** | Share who abandon checkout | Abandoned carts / carts created |
| **Repeat purchase rate** | Share of repeat customers | Repeat customers / total customers |

### E-Commerce Growth Strategies

```markdown
## Increasing AOV

1. **Bundle offers** - buy A get B, spend-threshold gifts
2. **Upsells** - low-friction upgrades
3. **Free shipping threshold** - set a minimum spend
4. **Recommended add-ons** - smart recommendations

## Reducing Cart Abandonment

1. **Simplify checkout** - fewer steps
2. **Diverse payment methods** - support multiple payment options
3. **Abandonment reminders** - email/SMS recovery
4. **Trust badges** - display security certifications

## Increasing Repeat Purchases

1. **Loyalty program** - points, tiers
2. **Subscriptions** - recurring purchases
3. **Birthday offers** - personalized touches
4. **Email marketing** - valuable content
```

### E-Commerce Operating Calendar

```
┌─────────────────────────────────────────────────────────────────┐
│  Annual E-Commerce Sales Calendar                                │
│                                                                 │
│  Q1                                                             │
│  ├─ Jan: New Year sales, winter clearance                       │
│  ├─ Feb: Valentine's Day                                        │
│  └─ Mar: International Women's Day, spring launch               │
│                                                                 │
│  Q2                                                             │
│  ├─ Apr: Spring refresh sales                                   │
│  ├─ May: Mother's Day, mid-year sale warm-up                    │
│  └─ Jun: Mid-year mega sale                                     │
│                                                                 │
│  Q3                                                             │
│  ├─ Jul: Summer promotions                                      │
│  ├─ Aug: Back-to-school season                                  │
│  └─ Sep: Fall launch, Labor Day sale                            │
│                                                                 │
│  Q4                                                             │
│  ├─ Oct: Halloween, early holiday teasers                       │
│  ├─ Nov: Singles' Day (11/11), Black Friday, Cyber Monday        │
│  └─ Dec: Holiday sale, Christmas, New Year's Eve                │
└─────────────────────────────────────────────────────────────────┘
```

## CRM Customer Management

### Customer Segmentation (RFM Model)

| Dimension | Description | Meaning |
|------|------|------|
| **R** Recency | Time since last purchase | Freshness |
| **F** Frequency | Purchase frequency | Loyalty |
| **M** Monetary | Amount spent | Value |

```markdown
## Customer Segmentation Strategy

| Segment | RFM Profile | Action |
|------|----------|------|
| High value | R↑ F↑ M↑ | VIP retention |
| Growth potential | R↑ F↓ M↑ | Increase purchase frequency |
| At-risk high value | R↓ F↑ M↑ | Re-engagement campaigns |
| Needs win-back | R↓ F↓ M↑ | Win back churned customers |
| New customers | R↑ F↓ M↓ | Nurture toward higher value |
```

## Sales Team Management

### Performance Management

```markdown
## Setting Sales Targets (SMART)

- **S** pecific - concrete numbers
- **M** easurable - measurable
- **A** chievable - attainable
- **R** elevant - relevant
- **T** ime-bound - time-bound

## Breaking Down Targets

Annual target
├─ Quarterly targets (accounting for seasonality)
├─ Monthly targets
├─ Weekly targets
└─ Daily targets (activity metrics)

Activity metrics:
• Daily calls made
• Weekly customer visits
• Monthly proposals sent
```

## Recommended Tools

- **CRM**: Salesforce, HubSpot, Pipedrive
- **E-commerce platforms**: Shopify, Shopline, 91APP
- **Payment processing**: ECPay, NewebPay, Stripe
- **Data analytics**: Google Analytics, Mixpanel
- **Email**: Mailchimp, Klaviyo

## Advanced Negotiation Techniques

### Negotiation Prep Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  The 7P Negotiation Prep Framework                               │
│                                                                 │
│  1. Purpose                                                     │
│     • What's my ideal outcome?                                  │
│     • What's the minimum acceptable terms?                      │
│     • What might the other side's goals be?                     │
│                                                                 │
│  2. Product                                                     │
│     • What's the unique value of our product?                   │
│     • How do we differ from competitors?                        │
│                                                                 │
│  3. Person                                                      │
│     • Who is the decision-maker? Who are the influencers?       │
│     • What are their KPIs and pressure points?                  │
│                                                                 │
│  4. Problem                                                     │
│     • What's their core pain point?                             │
│     • What's the cost of not solving it?                        │
│                                                                 │
│  5. Power                                                       │
│     • Who needs this deal more?                                 │
│     • What leverage do I have?                                  │
│                                                                 │
│  6. Plan B                                                      │
│     • What's my BATNA?                                          │
│     • What's their BATNA?                                       │
│                                                                 │
│  7. Process                                                     │
│     • What's the negotiation agenda?                            │
│     • What's the order of concessions?                          │
└─────────────────────────────────────────────────────────────────┘
```

### Negotiation Tactics

| Tactic | Description | How to Counter |
|------|------|----------|
| **Silence** | Say your piece and wait, don't rush to fill the silence | Don't be pressured into conceding |
| **Splitting** | Break a big issue into smaller ones | Keep the overall interest in mind |
| **Trading** | "If you..., then I can..." | Make sure the trade is equal value |
| **Deadline** | Create a sense of urgency | Verify whether it's real |
| **Good cop/bad cop** | One plays nice, one plays tough | Talk directly to the decision-maker |
| **Nibbling** | Small extra asks after the deal is done | Stay alert, feel free to decline |

### Price Negotiation Scripts

```markdown
## When the Customer Says "It's Too Expensive"

❌ Wrong response:
"We can offer a discount" / "What's your budget?"

✅ Right approach:
1. "What are you comparing the price to?" (understand the anchor)
2. "Which features matter most to you?" (confirm value drivers)
3. "If this problem isn't solved, how much do you estimate you'd lose in a year?" (quantify the pain point)
4. "Our solution can save you Y within X months..." (make the ROI case)

## When the Customer Asks for a Discount

Principle: never give it away for free — always trade for something

"We do have some flexibility, but I'd need to understand..."
• "If we offer a discount, could you sign this week?"
• "If we adjust to your budget, could you go with an annual plan?"
• "If we offer a discount, could we extend the contract term?"
```

## Sales Process Automation

### Sales Automation Triggers

```
┌─────────────────────────────────────────────────────────────────┐
│  Automated Sales Process Design                                  │
│                                                                 │
│  Trigger                →  Automated Action     →  Human Follow-up│
│  ─────────────────────────────────────────────────────────────  │
│  Visitor downloads whitepaper → Send email nurture series → SDR calls│
│  Views pricing page > 3x     → Trigger live chat invite    → AE contacts│
│  3 days before trial expires → Send renewal reminder       → CSM follows up│
│  Cart abandoned for 1 hour   → Send recovery email + discount → -    │
│  High-value customer birthday → Send exclusive offer       → Sales calls│
│  Low NPS feedback            → Create urgent ticket         → Manager steps in│
└─────────────────────────────────────────────────────────────────┘
```

### Sales Email Templates

```markdown
## Cold Outreach

Subject: {Company Name}'s {pain point} problem?

Hi {Name},

Noticed that {Company Name} recently {observed event/challenge}.

We've helped {similar customers} achieve {specific results} within {timeframe}.

Do you have 15 minutes to discuss a solution for {pain point}?

## Follow-up Email

Subject: Re: {previous subject}

Hi {Name},

I sent you a note last week — wanted to check whether you saw it?

One more example: {customer name} used our solution and saw {result numbers}.

Attaching {resource name}, which might be helpful.

## After Handling an Objection

Subject: Regarding the {objection} you mentioned

Hi {Name},

Thanks for the conversation today — I understand your concern about {objection}.

After discussing with my team, here's an idea: {solution}.

Attaching {case study/data} for your reference.
```

## Sales Data Analysis

### Diagnosing Sales Funnel Health

| Symptom | Likely Cause | Improvement Direction |
|----------|----------|----------|
| Insufficient lead volume | Marketing funnel issue | Add acquisition channels |
| Low MQL→SQL conversion | Poor lead quality | Refine scoring criteria |
| Low SQL→Opportunity conversion | Insufficient needs discovery | Strengthen SPIN training |
| Low Opportunity→Closed Won conversion | Weak objection handling | Negotiation skills training |
| Sales cycle too long | Process bottleneck | Identify bottleneck stages |
| Win rate declining | Increased competition | Differentiated positioning |

### Sales Forecasting Model

```markdown
## Weighted Pipeline Method

Forecasted revenue = Σ (opportunity amount × stage win probability)

| Stage | Win Probability | Example Amount | Weighted Amount |
|------|----------|----------|----------|
| Initial contact | 10% | $100K | $10K |
| Needs confirmed | 25% | $80K | $20K |
| Solution demo | 50% | $60K | $30K |
| Quote/negotiation | 75% | $40K | $30K |
| Verbal commitment | 90% | $30K | $27K |
| **Forecasted revenue** | - | $310K | **$117K** |

## Sales Velocity Formula

Sales velocity = (number of opportunities × win rate × average deal size) / sales cycle length in days

Optimization levers:
• ↑ Number of opportunities: increase lead generation
• ↑ Win rate: improve skills
• ↑ Deal size: increase AOV
• ↓ Sales cycle: streamline the process
```

## Social Selling

### LinkedIn Sales Strategy

```
┌─────────────────────────────────────────────────────────────────┐
│  LinkedIn Sales Funnel                                           │
│                                                                 │
│  Visibility Layer                                               │
│  • Post once daily (valuable content, not ads)                  │
│  • Comment on target customers' posts                           │
│  • Share industry insights                                      │
│           ↓                                                     │
│  Connection Layer                                                │
│  • Personalized connection requests (mention common ground)     │
│  • Send a welcome message after acceptance (no pitching)        │
│  • Engage with their content for 2-3 weeks                      │
│           ↓                                                     │
│  Conversion Layer                                                │
│  • Offer a valuable resource                                    │
│  • Invite them to an event/webinar                               │
│  • Propose a conversation                                       │
│                                                                 │
│  Golden ratio: 80% value content / 20% business-related          │
└─────────────────────────────────────────────────────────────────┘
```

### Social Selling Metrics

| Metric | Description | Target |
|------|------|------|
| SSI Score | LinkedIn Social Selling Index | > 70 |
| Connection growth | New connections per week | > 20 |
| Post engagement | Likes + comments + shares | > 100/post |
| InMail reply rate | Message reply percentage | > 25% |
| Conversion to meeting | Connection → first call | > 5% |

## Common Sales Mistakes

| Mistake | Correct Approach |
|------|----------|
| Pitching the product right away | Ask questions first to understand needs |
| Dropping the price at the first objection | Explore the real concern first |
| Only talking to a single point of contact | Build relationships across multiple stakeholders (economic buyer, user, influencer) |
| Letting it sit after "I'll think about it" | Set a clear next step and timeline |
| Going silent after closing the deal | Keep nurturing the relationship and generate referrals |
| Taking "I'm busy" at face value | "Busy" = priority isn't high enough — increase urgency |

## Related Resources

- [SPIN Selling](https://www.amazon.com/SPIN-Selling-Neil-Rackham/dp/0070511136)
- [The Challenger Sale](https://www.amazon.com/Challenger-Sale-Control-Customer-Conversation/dp/1591844355)
- [Never Split the Difference](https://www.amazon.com/Never-Split-Difference-Negotiating-Depended/dp/0062407805)
- [Growth Hacking for E-Commerce](https://growth.design/)
- [The Sales Acceleration Formula](https://www.amazon.com/Sales-Acceleration-Formula-Technology-Inbound/dp/1119047072)
