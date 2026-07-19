---
schema: "1.0"
name: claude-skills
version: "2.0.0"
description: A collection of domain expertise covering business, finance, creative, professional services, lifestyle, methodology, programming languages, and cloud platforms - 26 domains in total
triggers: [domain, professional, knowledge, business, finance, creative, lifestyle, language, cloud]
keywords: [domain, skills, knowledge, business, finance, creative, professional, lifestyle, languages, cloud]
author: dheerajreddy01
---

# Claude Skills — Enterprise Edition

> A collection of skills that give AI professional knowledge across many domains

## Overview

This is a library of domain expertise — spanning both non-technical fields and technical domains like programming languages and cloud platforms — designed to:
- Work together with the `/evolve` skill to automatically identify the domain a task needs
- Load specific domain knowledge on demand via skillpkg
- Provide frameworks, methodologies, and best practices for each domain

## Available Domains (26)

### 💰 Finance
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-finance/quant-trading` | quant, backtest, strategy | Quantitative trading strategy development |
| `enterprise-finance/investment-analysis` | financial statements, investment, valuation, ROE | Investment analysis and valuation |
| `enterprise-finance/strategy-optimization` | strategy optimization, win rate, returns, backtest | Trading strategy optimization methodology |

### 💼 Business
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-business/marketing` | marketing, SEO, funnel, CAC | Digital marketing strategy |
| `enterprise-business/sales` | sales, e-commerce, CRM | Sales and e-commerce operations |
| `enterprise-business/product-management` | PRD, OKR, roadmap | Product management |
| `enterprise-business/project-management` | Scrum, Sprint, Gantt chart | Project management |
| `enterprise-business/strategy` | strategy, blue ocean, differentiation, business model | Business strategy |

### 🎨 Creative
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-creative/game-design` | game, level, balance, GDD | Game design |
| `enterprise-creative/ui-ux-design` | UI, UX, accessibility, WCAG | Interface and experience design |
| `enterprise-creative/brainstorming` | inspiration, brainstorming, creativity, SCAMPER | Creative ideation |
| `enterprise-creative/storytelling` | novel, screenplay, character, three-act structure | Story creation |
| `enterprise-creative/visual-media` | photography, video, animation, storyboard | Visual media production |

### 🔬 Professional
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-professional/research-analysis` | research, competitive analysis, market research, analysis | Research analysis |
| `enterprise-professional/knowledge-management` | notes, PKM, second brain, Zettelkasten | Knowledge management |

### 🌱 Lifestyle
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-lifestyle/personal-growth` | life planning, personal brand, time management | Personal growth |
| `enterprise-lifestyle/side-income` | side hustle, passive income, financial freedom | Side income and investing |

### 🧠 Methodology
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-methodology/knowledge-acquisition-4c` | learning, research, new domain, knowledge acquisition | Systematic learning from novice to expert |

### 🔧 Languages
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-languages/python` | python, django, fastapi, pandas | Python language and production practices |
| `enterprise-languages/javascript` | javascript, typescript, node, react | JavaScript/TypeScript and Node.js production practices |
| `enterprise-languages/go` | golang, go, goroutine, channel | Go language and concurrency practices |
| `enterprise-languages/rust` | rust, cargo, ownership, borrow checker | Rust ownership model and systems practices |
| `enterprise-languages/java` | java, jvm, spring, maven | Java, JVM, and Spring ecosystem practices |

### ☁️ Cloud
| Domain | Triggers | Description |
|------|--------|------|
| `enterprise-cloud/aws` | aws, ec2, s3, lambda, iam | AWS architecture and security practices |
| `enterprise-cloud/azure` | azure, azure functions, app service, entra id | Azure architecture and security practices |
| `enterprise-cloud/gcp` | gcp, google cloud, cloud run, bigquery | Google Cloud architecture and security practices |

## Usage

### Method 1: Automatic Detection (recommended)

When used together with the `/evolve` skill, the matching domain is automatically loaded based on the task's keywords:

```
User: /evolve analyze Apple's financial statements and assess investment value

Agent:
🔍 Auto Domain Detection
→ Keywords: financial statements, analysis, investment
→ Loaded: enterprise-finance/investment-analysis
→ Executing the task using the investment analysis framework
```

### Method 2: Manual Installation

```python
# Install a specific domain
mcp__skillpkg__install_skill({
    "source": "github:dheerajreddy01/claude-skills#enterprise-finance/quant-trading"
})

# Load it for use
mcp__skillpkg__load_skill({ "id": "quant-trading" })
```

### Method 3: Using claude-starter-kit

```bash
npx claude-starter-kit
# Interactively select the domains you need
```

## Domain Skill Structure

Each domain skill contains:

```markdown
## Applicable Scenarios
When to use this domain knowledge

## Core Framework
The domain's standard methodologies and mental models

## Best Practices
Proven approaches

## Common Mistakes
Guidance for avoiding common pitfalls

## Recommended Tools
Practical tools relevant to the domain

## Reference Resources
Resources for further learning
```

## Contributing

Contributions of new domains or translations are welcome! Please see [CONTRIBUTING.md](./CONTRIBUTING.md)

