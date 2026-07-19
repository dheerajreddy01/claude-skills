# Business → Tech Interface

> Mapping business domain requirements to technical implementation

## Domain Skills Covered

- `business-strategy` - Business strategy
- `product-management` - Product management
- `project-management` - Project management
- `marketing` - Digital marketing
- `sales` - Sales and e-commerce

## Requirement → Technology Mapping

| Domain Requirement | Technical Implementation | Software Skills |
|-------------------|-------------------------|-----------------|
| PRD writing | Markdown + Notion/Confluence | `documentation` |
| API spec design | OpenAPI/GraphQL Schema | `api-design` |
| Data tracking | Analytics + ETL Pipeline | `data-analysis`, `database` |
| E-commerce system | E-commerce Platform | `e-commerce`, `backend` |
| Project tracking | JIRA/Linear + Git | `git-workflows`, `documentation` |
| Marketing automation | Marketing Automation Tools | `automation-scripts` |
| A/B testing | Feature Flags + Analytics | `testing-strategies`, `data-analysis` |
| CRM integration | API Integration | `api-design`, `backend` |

## Common Combination Patterns

### Pattern 1: Tech Product Manager

**Focus**: Technical product specs, API design, cross-team coordination

```yaml
domain_skills:
  - product-management (deep)
  - project-management (basic)

software_skills:
  - documentation (required)
  - api-design (required)
  - git-workflows (recommended)
```

**Use Case**: SaaS product development, platform API planning

### Pattern 2: Growth Marketer

**Focus**: Data-driven marketing, conversion optimization, user growth

```yaml
domain_skills:
  - marketing (deep)
  - sales (basic)

software_skills:
  - data-analysis (required)
  - automation-scripts (recommended)
```

**Use Case**: User acquisition, conversion funnel optimization, A/B testing

### Pattern 3: E-commerce Operator

**Focus**: E-commerce operations, order management, inventory systems

```yaml
domain_skills:
  - sales (deep)
  - marketing (basic)

software_skills:
  - e-commerce (required)
  - database (recommended)
  - backend (recommended)
```

**Use Case**: E-commerce websites, shopping carts, payment integration

### Pattern 4: Scrum Master / Tech Lead

**Focus**: Agile development, team collaboration, technical management

```yaml
domain_skills:
  - project-management (deep)
  - product-management (basic)

software_skills:
  - git-workflows (required)
  - documentation (required)
  - devops-cicd (recommended)
```

**Use Case**: Software project management, sprint planning, technical decisions

## Technology Stack Recommendations

### Documentation & Collaboration

| Use Case | Recommended Stack |
|----------|------------------|
| PRD/Spec | Notion / Confluence / Markdown |
| API Docs | OpenAPI + Stoplight / Readme |
| Project management | JIRA / Linear / GitHub Projects |

### Analytics & Data

| Use Case | Recommended Stack |
|----------|------------------|
| Website analytics | Google Analytics / Mixpanel |
| Product analytics | Amplitude / PostHog |
| Data warehouse | BigQuery / Snowflake |

### Marketing Tech

| Use Case | Recommended Stack |
|----------|------------------|
| Email marketing | SendGrid / Mailchimp |
| CRM | HubSpot / Salesforce |
| A/B testing | Optimizely / LaunchDarkly |

### E-commerce

| Use Case | Recommended Stack |
|----------|------------------|
| Platform | Shopify / WooCommerce / custom build |
| Payments | Stripe / ECPay / TapPay |
| Inventory | Custom + ERP Integration |

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Better Approach |
|--------------|---------|-----------------|
| No data tracking | Can't measure results | Set up tracking before making decisions |
| Vague PRDs | Gaps between spec and implementation | Use concrete user stories + acceptance criteria |
| Accumulating tech debt | Long-term dev velocity drops | Fold refactoring into the backlog |
| Over-engineering | MVP delays | Ship something usable first, refine later |
