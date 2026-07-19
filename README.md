# Claude Skills — Enterprise Edition

> A governed library of domain-expertise packs that turns Claude into a specialist across business, finance, creative, professional services, lifestyle, and methodology work.

[![Skills](https://img.shields.io/badge/skills-24-blue)](./README.md)
[![Domains](https://img.shields.io/badge/domains-6-green)](./README.md)
[![Version](https://img.shields.io/badge/version-2.0.0-informational)](./SKILL.md)
[![Plugin](https://img.shields.io/badge/Claude_Code-Plugin-orange)](https://code.claude.com/docs/en/discover-plugins)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](./LICENSE)

```
┌─────────────────────────────────────────────────────────────────┐
│                  Claude Skills — Enterprise Edition              │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │   Business   │  │   Finance    │  │   Creative   │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐         │
│   │ Professional │  │  Lifestyle   │  │  Methodology │         │
│   └──────────────┘  └──────────────┘  └──────────────┘         │
│                                                                 │
│         Claude auto-detects the task → loads the matching      │
│                    domain knowledge pack                        │
└─────────────────────────────────────────────────────────────────┘
```

## Table of Contents

- [Overview](#overview)
- [Why This Exists](#why-this-exists)
- [Domain Catalog](#domain-catalog)
- [Installation](#installation)
- [Automatic Domain Detection](#automatic-domain-detection)
- [Skill Architecture](#skill-architecture)
- [Creating a New Domain Skill](#creating-a-new-domain-skill)
- [Directory Structure](#directory-structure)
- [Contributing](#contributing)
- [Governance & Quality Bar](#governance--quality-bar)
- [License](#license)

## Overview

This repository is a structured library of **24 non-technical domain-expertise skills**, organized into **6 categories**. Each skill packages the frameworks, formulas, checklists, and pitfalls a specialist in that field would bring to a task — so that Claude can apply them on demand instead of answering from generic knowledge alone.

Every skill is designed to:
- Be **auto-loaded** based on task keywords (via the `/evolve` skill or the `triggers` mechanism)
- Provide **verified frameworks and methodologies**, not just prose — every formula and benchmark in this library has been checked for correctness
- Stay **self-contained**, so a skill can be installed individually without pulling in the rest of the library
- Follow a **consistent schema** (`SKILL.md` + optional `extended/` reference material) so new domains are easy to author and review

## Why This Exists

Generic LLM responses to domain questions ("how do I value this stock," "how do I structure this sprint," "is this game balance fair") tend to be shallow and inconsistent from one conversation to the next. This library exists to give Claude a **repeatable, reviewed baseline** for each domain — the same DCF formula, the same RFM segmentation logic, the same WCAG contrast reference — every time, rather than reinventing it per conversation.

## Domain Catalog

| Category | Sub-domains | Description |
|------|--------|------|
| **enterprise-business/** | marketing, sales, product-management, project-management, strategy | Business operations |
| **enterprise-finance/** | quant-trading, investment-analysis, strategy-optimization | Finance expertise |
| **enterprise-creative/** | game-design, game-planner, galgame-master, deckbuilder-roguelike, ui-ux-design, brainstorming, storytelling, visual-media | Creative work |
| **enterprise-professional/** | research-analysis, knowledge-management | Professional services |
| **enterprise-lifestyle/** | personal-growth, side-income | Lifestyle |
| **enterprise-methodology/** | knowledge-acquisition-4c, tech-spec-gen, skill-optimizer, consistency-checker | Development methodology |

### Full Skill Index

| Domain | Example Triggers | Description |
|------|------------|------|
| `enterprise-finance/quant-trading` | quant, backtest, strategy | Quantitative trading strategy development |
| `enterprise-finance/investment-analysis` | financial statements, investment, valuation | Investment analysis and valuation |
| `enterprise-finance/strategy-optimization` | strategy optimization, improve win rate, parameter tuning | Trading strategy optimization methodology |
| `enterprise-business/product-management` | PRD, OKR, roadmap | Product management |
| `enterprise-business/project-management` | Scrum, sprint, Gantt chart | Project management |
| `enterprise-business/marketing` | marketing, CAC, funnel | Marketing strategy |
| `enterprise-business/sales` | sales, e-commerce, CRM | Sales and e-commerce operations |
| `enterprise-business/strategy` | strategy, blue ocean, differentiation | Business strategy and competitive advantage |
| `enterprise-creative/game-design` | game, level, balance | Game design |
| `enterprise-creative/game-planner` | GDD, game planning, design pillars | Game design documents |
| `enterprise-creative/galgame-master` | galgame, bishoujo game, visual novel, romance route | Galgame creation |
| `enterprise-creative/deckbuilder-roguelike` | deckbuilding, deckbuilder, roguelike | Card roguelike design |
| `enterprise-creative/ui-ux-design` | UI, UX, accessibility | Interface and experience design |
| `enterprise-creative/brainstorming` | inspiration, brainstorming, creativity | Creative ideation methodology |
| `enterprise-creative/storytelling` | novel, comic, screenplay, character | Story creation and narrative |
| `enterprise-creative/visual-media` | photography, video, animation, film | Visual media production |
| `enterprise-professional/research-analysis` | research, competitive analysis, market research | Research analysis methodology |
| `enterprise-professional/knowledge-management` | knowledge management, notes, PKM | Personal knowledge systems |
| `enterprise-lifestyle/personal-growth` | life planning, personal brand, time management | Personal growth and career development |
| `enterprise-lifestyle/side-income` | side hustle, passive income, investing | Side income and financial freedom |
| `enterprise-methodology/knowledge-acquisition-4c` | learning, research, entering a new field | Systematic learning methodology (4C) |
| `enterprise-methodology/tech-spec-gen` | tech-spec, technical spec, design-to-spec, PRD | Design document → technical spec conversion |
| `enterprise-methodology/skill-optimizer` | skill optimization, reduce tokens, skill trimming | Skill optimization and token efficiency |
| `enterprise-methodology/consistency-checker` | check, consistency, verify, sync | Content consistency checker |

## Installation

### Option A: Plugin Marketplace (recommended)

```bash
# 1. Add the marketplace (GitHub format: owner/repo)
/plugin marketplace add dheerajreddy01/claude-skills

# 2. Open the plugin management UI and browse available plugins on the Discover tab
/plugin

# 3. Install specific skills (pick what you need)
/plugin install marketing@claude-skills
/plugin install quant-trading@claude-skills
/plugin install game-design@claude-skills

# Or just mention a skill's subject in conversation — Claude will load it automatically
```

**Supported marketplace formats:**
```bash
# Short format (recommended)
/plugin marketplace add dheerajreddy01/claude-skills

# HTTPS URL
/plugin marketplace add https://github.com/dheerajreddy01/claude-skills.git

# Specific branch or tag
/plugin marketplace add dheerajreddy01/claude-skills#main
```

**Plugin commands:**
| Command | Description |
|------|------|
| `/plugin` | Open the interactive plugin management UI |
| `/plugin install <name>@<marketplace>` | Install a specific plugin |
| `/plugin disable <name>@<marketplace>` | Temporarily disable a plugin |
| `/plugin uninstall <name>@<marketplace>` | Completely remove a plugin |

### Option B: Clone into your Skills directory

```bash
git clone https://github.com/dheerajreddy01/claude-skills.git ~/.claude/skills/domain-skills
```

Claude will automatically discover and use the relevant skills as needed.

### Option C: claude-starter-kit

```bash
npx claude-starter-kit
# Interactively select the domains you need
```

## Automatic Domain Detection

When used together with the `/evolve` skill, Claude will automatically:

1. Analyze the task's keywords
2. Match them against each skill's `triggers`
3. Load the relevant domain knowledge before executing the task

```
User: /evolve analyze Apple's financial statements and assess investment value

Agent:
🔍 Auto Domain Detection
→ Keywords: financial statements, analysis, investment
→ Loaded: enterprise-finance/investment-analysis
→ Executing the task using the investment analysis framework
```

## Skill Architecture

Each skill is a self-contained package:

```
enterprise-finance/quant-trading/
├── .claude-plugin/
│   └── plugin.json            # Plugin manifest (name, author, version)
├── skills/quant-trading/
│   └── SKILL.md                # Core instructions, frontmatter, triggers
└── extended/                   # Optional deep-dive reference material
    ├── code-examples.md
    └── templates.md
```

### Triggers vs. Keywords

`SKILL.md` frontmatter follows the skillpkg-compatible schema:

```yaml
---
schema: "1.0"
name: quant-trading
triggers: [quant, trading, backtest, strategy, factor]
keywords: [finance, trading, quantitative]
---
```

| Field | Purpose | Example |
|------|------|------|
| `triggers` | **Task matching** — fires based on what the user says | `[financial statements, investment, stock]` |
| `keywords` | **Category search** — lookup by domain | `[finance, business]` |

**Writing good triggers:**
```yaml
triggers:
  # ✅ Include synonyms
  - quantitative
  - quant

  # ✅ Include common phrasing
  - backtest
  - strategy testing

  # ❌ Avoid overly generic terms
  # - analysis (too broad)
  # - report (too broad)
```

### Skill Enhancement Features

| Feature | Description | Purpose |
|------|------|------|
| **Sharp Edges** | Documented warnings about common domain pitfalls, with severity ratings | Proactively steers users away from known failure modes |
| **Validations** | Quality-check rules embedded in the skill | Checks output against domain standards before it's considered done |
| **Collaboration** | Cross-domain prerequisite/delegation rules | Enables smart handoff between skills (e.g. quant-trading → python) |

```yaml
# SKILL.md frontmatter example
---
schema: "1.0"
name: quant-trading
collaboration:
  prerequisites:
    - skill: python
      reason: Foundation for quant development
  delegation_triggers:
    - trigger: database design
      delegate_to: database
---

# Sharp Edges
- id: backtest-overfitting
  severity: critical
  situation: The strategy backtest shows suspiciously high returns
  solution: Use walk-forward validation and out-of-sample testing
```

## Creating a New Domain Skill

```bash
# 1. Create the directory
mkdir -p enterprise-finance/my-domain

# 2. Create SKILL.md
cat > enterprise-finance/my-domain/SKILL.md << 'EOF'
---
schema: "1.0"
name: my-domain
version: "1.0.0"
description: Domain description
triggers: [keyword1, keyword2]
keywords: [category1, category2]
---

# Domain Name

## Applicable Scenarios
- Scenario 1
- Scenario 2

## Core Knowledge
...

## Best Practices
...
EOF

# 3. Register it as an installable plugin
# Add the following entry to the "plugins" array in .claude-plugin/marketplace.json:
{
  "name": "my-domain",
  "description": "Domain description",
  "source": "./enterprise-finance/my-domain",
  "version": "1.0.0",
  "strict": true
}

# 4. Verify
# Mention the trigger keyword in a Claude Code conversation — it should load automatically
```

Before submitting a new skill, run it against the [Governance & Quality Bar](#governance--quality-bar) checklist below.

## Directory Structure

```
claude-skills/
├── .claude-plugin/              # Plugin marketplace configuration
│   └── marketplace.json         # Lists all 24 skills as standalone plugins
├── enterprise-business/         # Business operations (5 skills)
├── enterprise-finance/          # Finance (3 skills)
├── enterprise-creative/         # Creative work (8 skills)
├── enterprise-professional/     # Professional services (2 skills)
├── enterprise-lifestyle/        # Lifestyle (2 skills)
├── enterprise-methodology/      # Methodology (4 skills)
├── interfaces/                  # Cross-domain → technical handoff definitions
├── docs/                        # Quickstart and supporting documentation
├── SKILL-TEMPLATE.md            # Skill creation template
├── SKILL.md                     # Library-level manifest and usage guide
├── CONTRIBUTING.md              # Contributing guide
├── OPTIMIZATION-REPORT.md       # Token-efficiency audit of existing skills
├── README.md
└── LICENSE
```

## Contributing

Contributions of new domains, corrections, or translations are welcome — see [CONTRIBUTING.md](./CONTRIBUTING.md) for the skill schema, style conventions, and review checklist.

## Governance & Quality Bar

Every skill in this library is expected to meet the same bar before merge:

- [ ] `SKILL.md` frontmatter is complete (`schema`, `name`, `version`, `triggers`, `keywords`)
- [ ] Every formula and numeric benchmark is verified against a recognized standard or textbook definition
- [ ] Tables and checklists are internally consistent (no contradicting rows across sections)
- [ ] Triggers include realistic phrasing a user would actually type, not just the domain's formal name
- [ ] No fabricated statistics, invented frameworks, or unverifiable citations
- [ ] `extended/` reference material (if present) is linked correctly and doesn't duplicate the core `SKILL.md`

## License

Released under the [MIT License](./LICENSE).