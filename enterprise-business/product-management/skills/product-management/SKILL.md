---
schema: "1.0"
name: product-management
version: "1.0.0"
description: Product management: writing PRDs, user stories, OKR setting, and roadmap planning
domain: business
triggers:
  keywords:
    primary: [PRD, product requirements, user story, OKR, roadmap, product manager]
    secondary: [roadmap, product spec, spec, feature planning, feature, backlog, RICE, MoSCoW]
  context_boost: [requirement, planning, priority]
  context_penalty: [code, database, api]
  priority: high
dependencies:
  software-skills: [documentation, api-design]
author: claude-domain-skills
---

# Product Management

> The full product development process, from requirements to delivery

## Applicable Scenarios

- Writing product requirements documents (PRDs)
- Defining user stories and acceptance criteria
- Setting OKRs and success metrics
- Planning product roadmaps
- Prioritizing features

## PRD Structure Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  Standard PRD Structure                                          │
│                                                                 │
│  1. Overview                                                    │
│     ├─ Problem statement (why)                                 │
│     ├─ Target users (for whom)                                 │
│     └─ Success metrics (how to measure)                        │
│                                                                 │
│  2. Requirements                                                │
│     ├─ Functional requirements (User Stories)                  │
│     ├─ Non-functional requirements (performance, security)     │
│     └─ Design constraints                                      │
│                                                                 │
│  3. Design                                                      │
│     ├─ User flows                                               │
│     ├─ Wireframe/prototype links                                │
│     └─ Edge case handling                                       │
│                                                                 │
│  4. Technical Considerations                                    │
│     ├─ Architectural impact                                     │
│     ├─ API requirements                                         │
│     └─ Dependencies                                             │
│                                                                 │
│  5. Launch Plan                                                 │
│     ├─ Milestones                                                │
│     ├─ Risk assessment                                          │
│     └─ Acceptance criteria                                      │
└─────────────────────────────────────────────────────────────────┘
```

## User Story Format

```markdown
### Standard Format
As a [user role]
I want [feature/goal]
So that [value/reason]

### Acceptance Criteria
Given [precondition]
When [triggering action]
Then [expected result]

### Example
As an e-commerce shopper
I want to save items to a wishlist
So that I can purchase them later

Acceptance Criteria:
- Given the user is logged in
- When they click "Add to Wishlist"
- Then the item appears in the wishlist
- And a success message is shown
```

## OKR Setting Framework

| Element | Description | Example |
|------|------|------|
| **Objective** | Qualitative, inspiring goal | Become the most trusted payment platform |
| **Key Result 1** | Quantifiable outcome | Raise NPS from 45 to 60 |
| **Key Result 2** | Quantifiable outcome | Achieve 99.9% transaction success rate |
| **Key Result 3** | Quantifiable outcome | Complaint handling time < 2 hours |

### OKR Checklist

```
□ Is the Objective challenging yet achievable?
□ Are the Key Results quantifiable?
□ Do the Key Results have a clear baseline and target value?
□ Are there 3-5 Key Results?
□ Is a check-in cadence defined?
```

## Prioritization Frameworks

### RICE Framework

| Dimension | Description | Scoring |
|------|------|------|
| **R**each | How many users are affected | Users/quarter |
| **I**mpact | Degree of impact | 0.25/0.5/1/2/3 |
| **C**onfidence | Level of confidence | 50%/80%/100% |
| **E**ffort | Development cost | Person-months |

**RICE Score = (Reach × Impact × Confidence) / Effort**

### MoSCoW Classification

| Category | Description |
|------|------|
| **Must have** | Core functionality; can't launch without it |
| **Should have** | Important, but can be iterated on later |
| **Could have** | Nice to have, if there's time |
| **Won't have** | Explicitly excluded from this iteration |

### Kano Model

| Category | Effect on Satisfaction | Example |
|------|------|------|
| **Must-be** | Absence causes dissatisfaction; presence isn't noticed | App doesn't crash |
| **Performance** | Satisfaction scales linearly with how well it's done | Faster load times |
| **Attractive** | Absence isn't missed; presence delights | Unexpected personalization |
| **Indifferent** | No real effect on satisfaction either way | Rarely-used settings |

**Use case**: classify features by the type of satisfaction they drive, complementing RICE/MoSCoW's effort-vs-value view

## Roadmap Template

```markdown
## Q1 2025

### Theme: Infrastructure
- [ ] User authentication system refactor
- [ ] API Gateway upgrade
- [ ] Monitoring system build-out

### Milestones
- 1/15: Technical design complete
- 2/28: Development complete
- 3/15: Launch

---

## Q2 2025

### Theme: UX Optimization
- [ ] New homepage
- [ ] Enhanced search
- [ ] Personalized recommendations V1
```

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Writing a PRD so detailed it reads like a design spec | Focus on What & Why, leave How to the engineering team |
| User Stories without acceptance criteria | Every story must have clear AC |
| Setting too many OKRs | Focus on the 3-5 most important |
| Locking the roadmap to fixed dates | Organize by "theme" rather than a "feature list" |

## Communication Template

### Feature Spec Brief

```markdown
# [Feature Name]

## One-Line Description
[Explain what this feature is in one sentence]

## Why We're Building It
- User pain point: [description]
- Business value: [description]

## What We're Building
- [Feature point 1]
- [Feature point 2]

## What We're Not Building
- [Explicitly excluded scope]

## Success Metrics
- [Metric 1]: from X to Y
- [Metric 2]: from X to Y

## Timeline
- Design: X weeks
- Development: X weeks
- Testing: X weeks
```

## Related Resources

- [Inspired - Marty Cagan](https://www.amazon.com/INSPIRED-Create-Tech-Products-Customers/dp/1119387507)
- [Shape Up - Basecamp](https://basecamp.com/shapeup)
- [The Mom Test - Rob Fitzpatrick](http://momtestbook.com/)

## Product Discovery

### User Interview Techniques

```
┌─────────────────────────────────────────────────────────────────┐
│  The Mom Test Principles                                        │
│                                                                 │
│  ❌ Don't ask: "Do you think this feature is good?"             │
│  ✅ Do ask: "When was the last time you ran into this problem?  │
│     How did you solve it?"                                      │
│                                                                 │
│  ❌ Don't ask: "Would you use this product?"                    │
│  ✅ Do ask: "How do you handle this today? How much time/money  │
│     does it cost you?"                                          │
│                                                                 │
│  Core principles:                                                │
│  1. Talk about past behavior, not future intentions             │
│  2. Talk about specific instances, not generalities              │
│  3. Let the user talk — you just ask questions                  │
└─────────────────────────────────────────────────────────────────┘
```

### Sample Interview Questions

| Stage | Question |
|------|------|
| **Opening** | Can you tell me about your work/daily routine? |
| **Uncovering pain points** | What's the most frustrating problem you've run into recently? |
| **Understanding behavior** | How do you currently handle it? |
| **Quantifying impact** | How much time/money does this problem cost you? |
| **Alternatives** | Have you tried other solutions? |
| **Validating demand** | If a tool could solve this problem, how much would you pay for it? |

## Data-Driven Decision Making

### Product Metrics Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  North Star Metric + Supporting Metrics                          │
│                                                                 │
│                    ┌──────────────┐                             │
│                    │  North Star  │  ← Core value metric        │
│                    │   Metric     │     e.g. Weekly Active Users│
│                    └──────┬───────┘                             │
│           ┌───────────────┼───────────────┐                     │
│           ↓               ↓               ↓                     │
│     ┌─────────┐     ┌─────────┐     ┌─────────┐                │
│     │Acquire  │     │ Engage  │     │Monetize │                │
│     │new users│     │ retention rate│ ARPU    │                │
│     └─────────┘     └─────────┘     └─────────┘                │
│                                                                 │
│  Principle: the North Star should represent the core value      │
│  users receive                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### A/B Testing Framework

```markdown
## A/B Test Plan

### 1. Hypothesis
"We believe [change] will cause [result], because [reason]"

### 2. Metrics
- Primary metric: [the core metric to optimize]
- Guardrail metric: [the metric to ensure isn't harmed]

### 3. Sample Size Calculation
- Minimum detectable effect (MDE): X%
- Statistical significance: 95%
- Required sample size: N users
- Estimated experiment duration: X days

### 4. Experiment Design
- Control group: current version
- Treatment group: new version
- Traffic split: 50/50

### 5. Decision Criteria
- p-value < 0.05 and effect > MDE → ship it
- p-value < 0.05 but effect < MDE → consider it
- p-value > 0.05 → don't ship
```

## Product Documentation Best Practices

### PRD Writing Tips

| Tip | Description |
|------|------|
| **One-page summary** | Open with a one-page summary so readers grasp it quickly |
| **User perspective** | Describe from the user experience angle, not the technical implementation |
| **Visuals first** | Use flowcharts and wireframes instead of long blocks of text |
| **Version control** | Track revision history and version numbers |
| **Link, don't copy** | Reference links to design files/technical docs |

### Documentation Update Cadence

| Document Type | Update Frequency | Owner |
|----------|----------|--------|
| Product vision | Quarterly | PM Lead |
| Roadmap | Monthly | PM |
| PRD | As needed | PM |
| Release notes | Each release | PM |

## Cross-Team Collaboration

### RACI Matrix

| Task | PM | Design | Engineering | QA |
|------|-----|------|------|-----|
| Requirements definition | **R** | C | C | I |
| UI design | A | **R** | C | I |
| Technical design | C | I | **R** | I |
| Implementation | I | C | **R** | I |
| Testing/acceptance | A | I | C | **R** |

> R=Responsible, A=Accountable, C=Consulted, I=Informed

### Common Cross-Team Conflicts

| Conflict | Resolution |
|------|----------|
| Misaligned understanding of requirements | Use User Stories + AC to ensure alignment |
| Disagreement on time estimates | Engineering leads the estimate, PM coordinates priority |
| Design vs. technical constraints | Involve engineering early in design review |
| Scope creep | Strictly enforce MoSCoW, route changes through a formal process |

## Recommended Tools

- **Requirements management**: Jira, Linear, Notion
- **Design collaboration**: Figma, Sketch
- **User research**: Hotjar, FullStory, Maze
- **Data analytics**: Amplitude, Mixpanel, GA4
- **Roadmapping**: ProductBoard, Aha!, Notion

---

## Sharp Edges (Common Pitfalls)

> These are the most common and costliest mistakes in product management

### SE-1: The Feature Factory Trap
- **Severity**: critical
- **Situation**: The team focuses on "output" rather than "outcome," constantly shipping new features without validating their value
- **Cause**: Success is measured by completed story points or feature count rather than user value
- **Symptoms**:
  - New features ship every iteration, but core metrics don't grow
  - The product gets more complex while users get more confused
  - The team is busy but doesn't feel a sense of accomplishment
- **Detection**: `velocity|story.?points|feature count|features.?delivered`
- **Fix**: Validate the hypothesis behind every feature 2-4 weeks after launch; use an outcome-based roadmap

### SE-2: Stakeholder-Driven Prioritization
- **Severity**: high
- **Situation**: Priorities are decided by the boss, sales, or a big customer, rather than by user data and product strategy
- **Cause**: The organization lacks a data culture, so decisions are driven by "whoever is loudest" or "whoever has the highest title"
- **Symptoms**:
  - The roadmap frequently changes because "an important person asked for it"
  - Most feature requests come from "the customer said they want it" rather than user research
  - The PM becomes an "order taker" instead of a decision maker
- **Detection**: `boss said|sales requested|big customer needs|customer feedback|urgent request`
- **Fix**: Establish a prioritization framework (RICE), let data drive the conversation, learn to say no

### SE-3: Metric Gaming
- **Severity**: critical
- **Situation**: The team optimizes for "the metric" rather than "user value," so the metric looks great while the product experience gets worse
- **Cause**: Goodhart's Law — once a metric becomes a target, it stops being a good metric
- **Symptoms**:
  - DAU rises but retention drops (achieved by bombarding users with push notifications)
  - Conversion rate rises but so does the return rate (achieved with dark patterns)
  - NPS rises but it's all from "invite a friend to fill out the survey"
- **Detection**: `increase.*DAU|boost.*conversion|optimize.*metric`
- **Fix**: Set guardrail metrics to ensure users aren't being harmed

### SE-4: The PRD Waterfall
- **Severity**: medium
- **Situation**: The PRD is written in such detail it becomes a 100-page spec — engineers follow it literally but never understand the "why"
- **Cause**: The PM tries to "write it all up front" to avoid future discussion, but ignores inherent uncertainty
- **Symptoms**:
  - The PRD exceeds 10 pages
  - Engineers say "I built it exactly per the PRD" but the result is wrong
  - Updating the PRD takes longer than updating the code when requirements change
- **Detection**: `PRD.*pages|requirements doc.*detailed|spec.*complete`
- **Fix**: Focus the PRD on What & Why, leave How to the team; start with a one-pager and refine progressively

### SE-5: Roadmap-as-Commitment
- **Severity**: high
- **Situation**: The roadmap turns from a "plan" into a "commitment," and the team loses room to adapt
- **Cause**: Sales/marketing sells the roadmap to customers, or the boss treats the roadmap as a KPI
- **Symptoms**:
  - "Because we already promised the customer" becomes the reason for prioritization
  - The team won't adjust course even after discovering the direction is wrong
  - The team rushes to ship features with no time to do the right thing
- **Detection**: `Q[1-4].*must ship|already committed|deadline.*can't change`
- **Fix**: Organize the roadmap by "theme," not a "feature list"; keep a 30% buffer
