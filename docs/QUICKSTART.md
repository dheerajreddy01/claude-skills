# Claude Skills Quick Start

> Get started with 16 domain skills in 3 minutes

## What is Claude Skills?

This is a **non-technical domain** skill library designed for Claude Code, covering business, creative, finance, lifestyle, and professional domains.

| Category | # of Skills | Examples |
|------|--------|------|
| Business | 5 | marketing, sales, strategy, product-management |
| Creative | 5 | game-design, storytelling, ui-ux-design |
| Finance | 2 | investment-analysis, quant-trading |
| Lifestyle | 2 | personal-growth, side-income |
| Professional | 2 | research-analysis, knowledge-management |

---

## Step 1: Install the Full Skill Library (1 minute)

### Method A: Inside Claude Code

```
Install the claude-skills skill library
```

### Method B: Using the CLI

```bash
npx skillpkg-cli install dheerajreddy01/claude-skills
```

---

## Step 2: Install a Single Skill (30 seconds)

Only need a specific skill?

```bash
# Install the marketing skill
npx skillpkg-cli install github:dheerajreddy01/claude-skills#enterprise-business/marketing

# Install the game-design skill
npx skillpkg-cli install github:dheerajreddy01/claude-skills#enterprise-creative/game-design

# Install the quant-trading skill
npx skillpkg-cli install github:dheerajreddy01/claude-skills#enterprise-finance/quant-trading
```

---

## Step 3: Use the Skill (1 minute)

Once installed, Claude will automatically apply the relevant knowledge.

### Example Conversations

```
# Using the marketing skill
Help me plan a marketing strategy for a new product

# Using the game-design skill
/evolve Design the economy system for an RPG

# Using the investment-analysis skill
Analyze this company's financial statements and determine if it's worth investing in

# Using the storytelling skill
Help me write an outline for a science fiction short story
```

---

## Skill Catalog

### Business
`marketing` `sales` `strategy` `product-management` `project-management`

### Creative
`game-design` `storytelling` `brainstorming` `ui-ux-design` `visual-media`

### Finance
`investment-analysis` `quant-trading`

### Lifestyle
`personal-growth` `side-income`

### Professional
`research-analysis` `knowledge-management`

---

## Combining with Software Skills

Domain Skills can be combined with Software Skills:

| Combination | Effect |
|------|------|
| `game-design` + `frontend` | Game UI development |
| `quant-trading` + `python` + `database` | Quantitative trading system |
| `marketing` + `data-analysis` | Marketing data analysis |
| `ui-ux-design` + `react-ecosystem` | Design system implementation |

---

## FAQ

### Q: What's the difference between Domain Skills and Software Skills?

**Software Skills** focus on technical implementation (like Python, React, API Design).
**Domain Skills** focus on domain knowledge (like marketing strategy, game design, investment analysis).

The two can be combined for better results.

### Q: Are skills loaded automatically?

Yes. The Self-Evolving Agent automatically identifies and loads relevant skills based on the task description.

For example, a task like "design a mobile game's monetization mechanics" will automatically load the `game-design` skill.

### Q: How do I see which skills are loaded?

```bash
npx skillpkg-cli list
```

Or in Claude Code, say:
```
List loaded skills
```

---

## Next Steps

| Goal | Command |
|------|------|
| View the full skill list | [README.md](../README.md) |
| Contribute a new skill | [CONTRIBUTING.md](../CONTRIBUTING.md) |

---

## Success!

```
✅ Domain Skills installed
✅ Claude can now use domain knowledge
✅ Combine with Software Skills for even stronger capabilities

/evolve [your goal]
```
