# Contributing to Claude Skills

Thanks for your interest in contributing! Here's the contribution guide.

## How to Contribute a New Domain

### 1. Fork and Clone

```bash
git clone https://github.com/YOUR_USERNAME/claude-skills.git
cd claude-skills
```

### 2. Choose the Right Category

| Category | Applicable Content |
|------|----------|
| `enterprise-business/` | Business operations, sales, marketing, product |
| `enterprise-finance/` | Finance, investing, trading |
| `enterprise-creative/` | Design, creativity, content creation |
| `enterprise-professional/` | Professional services, research, consulting |
| `enterprise-lifestyle/` | Personal growth, quality of life |

### 3. Create the Skill Directory

```bash
mkdir -p category/your-domain
```

### 4. Create SKILL.md

Use the following template:

```markdown
---
schema: "1.0"
name: your-domain
version: "1.0.0"
description: Short description (one line)
triggers: [keyword1, keyword2, common-term]
keywords: [category, subcategory]
author: your-name
---

# Domain Name

> One sentence explaining the value of this domain

## Applicable Scenarios

- Scenario 1
- Scenario 2
- Scenario 3

## Core Knowledge

### Topic 1

[content...]

### Topic 2

[content...]

## Best Practices

- Practice 1
- Practice 2

## Recommended Tools

- Tool 1
- Tool 2

## Related Resources

- [Resource name](URL)
```

### 5. Trigger Design Principles

```yaml
triggers:
  # ✅ Good triggers
  - quantitative   # Domain-specific term
  - backtest       # Common synonym
  - strategy testing  # Common phrasing

  # ❌ Triggers to avoid
  - analysis   # Too broad
  - report     # Too generic
```

### 6. Submit a Pull Request

```bash
git checkout -b feat/add-your-domain
git add .
git commit -m "feat: add category/your-domain skill"
git push origin feat/add-your-domain
```

Then open a Pull Request on GitHub.

## Quality Checklist

- [ ] SKILL.md frontmatter is complete (schema, name, version, triggers, keywords)
- [ ] Triggers include relevant keywords
- [ ] Content has practical value (frameworks, methodologies, best practices)
- [ ] No copyright issues with the content
- [ ] README.md updated (if needed)

## Reporting Issues

If you find a problem, please report it on [Issues](https://github.com/dheerajreddy01/claude-skills/issues).

## License

Contributed content will be released under the MIT license.
