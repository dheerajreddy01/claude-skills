---
schema: "1.0"
name: marketing
version: "1.0.0"
description: Digital marketing strategy, content marketing, and performance analysis
domain: business
triggers:
  keywords:
    primary: [marketing, SEO, advertising, social media, growth]
    secondary: [content, conversion, funnel, brand, CAC, LTV, ROAS]
  context_boost: [data, analytics, traffic, user]
  context_penalty: [code, api, backend]
  priority: high
dependencies:
  software-skills: [data-analysis]
author: claude-domain-skills
---

# Digital Marketing

> Data-driven marketing strategy and execution

## Applicable Scenarios

- Developing marketing strategy and budget allocation
- Content marketing planning and execution
- SEO optimization and keyword research
- Ad campaign management and performance optimization
- Social media management

## AARRR Marketing Funnel

**Acquisition → Activation → Retention → Revenue → Referral**

Key metrics: CAC/traffic source → sign-up rate → DAU/MAU → ARPU/LTV → NPS

## Core Knowledge

### Marketing Channels

| Channel | Best For | Cost | Time to Effect |
|------|------|------|----------|
| **SEO** | Long-term traffic | Low | 3-6 months |
| **SEM/PPC** | Immediate conversions | High | Immediate |
| **Social Media** | Brand building | Medium | 1-3 months |
| **Content Marketing** | Building trust | Medium | 3-6 months |
| **Email** | Retention/conversion | Low | Immediate |

### Key Metrics

| Metric | Calculation | Healthy Value |
|------|----------|--------|
| **CAC** | Marketing spend / new customers | Industry-dependent |
| **LTV** | ARPU × lifetime | > 3× CAC |
| **ROAS** | Ad revenue / ad spend | > 3 |
| **CTR** | Clicks / impressions | 1-5% |
| **CVR** | Conversions / clicks | 2-5% |

### 70-20-10 Budget Allocation Rule

- **70%** - Proven, effective channels
- **20%** - Emerging channels with potential
- **10%** - Experimental channels

## Best Practices

1. **Data first** - Base all decisions on data, not gut feeling
2. **A/B testing** - Continuously test and optimize, iterate in small steps
3. **User-oriented** - Understand your target users' pain points and needs
4. **Integrated marketing** - Coordinate across channels with a consistent brand message
5. **ROI mindset** - Focus on return on investment, cut underperforming channels

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Chasing vanity metrics (follower count) | Focus on conversion metrics (revenue) |
| Spreading thin across all channels | Focus deeply on high-performing channels |
| One big overhaul at a time | Continuous A/B testing and iteration |
| Only looking at short-term results | Balance short-term and long-term |

## Sharp Edges

### SE-1: The Vanity Metrics Trap
- **Severity**: critical
- **Situation**: Chasing follower counts, impressions, and other "good-looking" metrics while revenue doesn't grow
- **Cause**: Vanity metrics are easy to measure and easy to hit, but don't reflect business value
- **Symptoms**:
  - Social followers increase but conversion rate stays flat
  - Reports look great but business performance is mediocre
  - Can't answer when the boss asks "so what about revenue?"
- **Solution**: Metric priority order: revenue > conversion > behavior > vanity. For every metric, ask: how does this affect revenue?

### SE-2: The CAC > LTV Death Spiral
- **Severity**: critical
- **Situation**: Customer acquisition cost exceeds customer lifetime value — the more you market, the more you lose
- **Cause**: Blindly scaling ad spend without tracking unit economics
- **Symptoms**:
  - The more ad spend, the bigger the losses
  - Can't stop running ads (afraid of losing traffic)
  - Don't know how much each customer is worth
- **Solution**:
  - Healthy LTV/CAC ratio > 3
  - Set a CAC ceiling (1/3 of LTV)
  - Improve product retention to raise LTV

### SE-3: A/B Testing False Positives
- **Severity**: high
- **Situation**: An A/B test shows a "significant difference," but there's no real-world effect
- **Cause**: Insufficient sample size, ending the test too early, multiple comparisons problem
- **Symptoms**:
  - Test results can't be reproduced
  - Post-launch performance doesn't match the test
  - "Significant" but the lift is tiny
- **Solution**: Pre-calculate sample size, run the test for its full duration, p < 0.05, effect > 2%, validate with repeat tests
- **Details**: → [extended/checklists.md#ab-testing-checklist]

### SE-4: Channel Dependency Risk
- **Severity**: high
- **Situation**: 80% of traffic comes from a single channel, and one algorithm change tanks it
- **Cause**: Going all-in on a channel once it proves effective, ignoring diversification
- **Symptoms**:
  - A single channel contributes over 50% of traffic
  - A platform policy change causes a traffic cliff
  - No owned traffic (e.g., an email list)
- **Solution**: Use the 70-20-10 rule to spread risk, build owned channels, cap any single channel at 40%

### SE-5: Ignoring Attribution Leads to Bad Decisions
- **Severity**: medium
- **Situation**: Giving all the credit to the last-click channel, leading to investment in the wrong channels
- **Cause**: Relying only on last-click attribution, ignoring the full user journey
- **Symptoms**:
  - Content marketing "isn't working"
  - Brand advertising gets cut
  - Search ads show a "super high" ROI (stealing credit from other channels)
- **Solution**: Use multi-touch attribution, understand the user journey, use different metrics for different stages

## Content Marketing

### Content Pyramid

- **Core content**: In-depth pieces (whitepapers, research reports), 2-4 per year
- **Pillar content**: Medium-depth pieces (blog posts, tutorials), 4-8 per month
- **Distribution content**: Lightweight content (social posts), 1-3 per day

**Principle**: One piece of core content → broken down into multiple pieces of distribution content

### Content Types

| Type | Purpose | Examples |
|------|------|------|
| Educational | Build expertise | Tutorial articles, how-tos |
| Inspirational | Create resonance | Stories, case studies |
| Entertaining | Increase engagement | Memes, humorous content |
| Conversion-focused | Drive purchases | Product comparisons, testimonials |

**Content calendar template**: → [extended/templates.md#content-calendar-template]

## SEO Optimization

### Keyword Research Process

Seed keywords → expand into long-tail → assess competition (KD < 30) → intent analysis → content planning

### On-Page SEO Essentials

- **Title Tag**: Includes keyword, 50-60 characters, click-worthy
- **Meta Description**: Includes keyword, 150-160 characters, includes a CTA
- **Content**: Single H1, clear structure, keyword density 1-2%, 3-5 internal links
- **Technical**: Load time < 3 seconds, mobile-friendly, HTTPS, structured data

**Full SEO checklist**: → [extended/checklists.md#seo-optimization-checklist]

## Advertising

### Facebook/Meta Ad Structure

Campaign (objective) → Ad Set (audience, budget, placement) → Ad (creative, copy, CTA)

**Best practices**:
- 3-6 ad variants per ad set
- Budget allocation: 20% testing / 80% scaling
- Test broad audiences first, then narrow down for precision

### Google Ads Types

| Type | Best For | Advantage |
|------|----------|------|
| Search ads | Active search intent | Clear intent |
| Display ads | Brand exposure | Lower cost |
| Shopping ads | E-commerce products | Visually compelling |
| Video ads | Brand storytelling | High engagement |

**Advertising checklist**: → [extended/checklists.md#advertising-checklist]

## Email Marketing

### Email Types

| Type | Purpose | Timing |
|------|------|------|
| Welcome email | Build relationship | Immediately after sign-up |
| Newsletter | Ongoing engagement | Weekly/bi-weekly |
| Promotional email | Drive purchases | During campaigns |
| Remarketing | Win back churned users | Automatically triggered |

**Open rate optimization**: subject lines that spark curiosity, use specific numbers, and personalize; avoid ALL CAPS and excessive exclamation points

**Email checklist**: → [extended/checklists.md#email-marketing-checklist]

## Social Media

### Platform Characteristics

| Platform | Primary Audience | Best Content | Peak Times |
|------|----------|----------|----------|
| Facebook | 25-54 years old | Video, live streams | 13:00-16:00 |
| Instagram | 18-34 years old | Photos, Reels | 11:00, 19:00 |
| LinkedIn | Professionals | Articles, career content | Tue-Thu 9:00 |
| TikTok | Gen Z | Short-form video | 19:00-22:00 |
| YouTube | All ages | Long-form video, Shorts | Thu-Sun |

**Engagement rate benchmarks**: Facebook 1-3%/3-6%, Instagram 3-6%/6%+, LinkedIn 2-4%/4%+

**Social media checklist**: → [extended/checklists.md#social-media-checklist]

## Advanced Strategy

### Growth Hacking ICE Score

**Impact × Confidence × Ease = priority score**

Test the highest-scoring items first, iterate and validate quickly

**Detailed framework**: → [extended/examples.md#growth-hacking-framework]

### User Psychology

**Cialdini's six principles**: reciprocity, scarcity, authority, consistency, liking, social proof

**Behavioral economics**: anchoring effect, loss aversion, paradox of choice, framing effect

**Detailed applications**: → [extended/examples.md#user-psychology-applications]

### Conversion Rate Optimization (CRO)

**Landing page structure**: above the fold (value proposition + CTA) → pain points → solution → social proof → FAQ → final CTA

**A/B testing priority**: headline > CTA > above-the-fold image > pricing > form fields

**Detailed guide**: → [extended/examples.md#conversion-rate-optimization-cro-deep-dive]

### Marketing Automation

**Key sequences**:
- New user onboarding (D+0 to D+14)
- Cart abandonment recovery (1h / 24h / 72h)
- Re-engaging dormant users (30 days inactive)

**Detailed flows**: → [extended/examples.md#marketing-automation-flows]

## Influencer Marketing

### KOL Tiers

| Tier | Followers | Characteristics | Partnership Style |
|------|--------|------|----------|
| Mega | 1M+ | High reach, low engagement | Brand ambassador |
| Macro | 100K-1M | Influence within a professional niche | Paid partnership |
| Micro | 10K-100K | High engagement, community trust | Product exchange/paid |
| Nano | <10K | Very high engagement, authentic | Product exchange |

**Influence score** = (engagement rate × follower count × relevance) / price

**KOL evaluation checklist**: → [extended/checklists.md#kol-partnership-checklist]

## Recommended Tools

| Category | Tools |
|------|------|
| Analytics | Google Analytics, Mixpanel |
| SEO | Google Search Console, Ahrefs, SEMrush |
| Email | Mailchimp, ConvertKit |
| Social | Hootsuite, Buffer |
| Advertising | Google Ads, Meta Ads Manager |

## Report Template

**Monthly report**: executive summary → core metrics → channel performance → key insights → next month's plan

**Full template**: → [extended/templates.md#performance-report-template]

## Related Resources

[Google Digital Garage](https://learndigital.withgoogle.com/) | [HubSpot Academy](https://academy.hubspot.com/) | [Moz SEO](https://moz.com/learn/seo)

## Related Domains

[[sales]] | [[product-management]] | [[research-analysis]]
