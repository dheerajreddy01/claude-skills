---
schema: "1.0"
name: skill-optimizer
version: "1.0.0"
description: Skill slimming optimizer — reduce token usage while preserving full functionality
triggers:
  - skill optimization
  - skill slimming
  - reduce tokens
keywords: [methodology, optimization, skill-development, token-efficiency]
author: claude-domain-skills
---

# Skill Optimizer v1.0.0

> Optimize the token efficiency of SKILL.md while preserving full functionality

## Quick Start

```bash
# Analyze a single skill
/skill-optimizer analyze [skill-path]

# Optimize a single skill
/skill-optimizer optimize [skill-path]

# Batch-analyze an entire repo
/skill-optimizer audit [repo-path]
```

---

## 1. Optimization Principles

### 1.1 Token Efficiency Pyramid

```
               ┌─────────┐
               │Essential│ ← Keep: triggers, core knowledge
               ├─────────┤
               │ Useful  │ ← Trim: best practices, common mistakes
               ├─────────┤
               │Cosmetic │ ← Remove: ASCII art, decorative diagrams
               ├─────────┤
               │Examples │ ← Link out: templates, full examples, checklists
               └─────────┘
```

### 1.2 Core Strategies

| Strategy | Description | Estimated savings |
|------|------|----------|
| **Layered loading** | Separate core content from extended content | 40-60% |
| **External references** | Move large examples to separate files | 20-30% |
| **Concise wording** | Remove redundant modifiers | 10-15% |
| **Diagram simplification** | ASCII → concise description | 5-10% |

---

## 2. Layered Structure

### 2.1 Recommended Skill Structure

```
skill-name/
├── SKILL.md           # Core layer (< 300 lines)
│   ├── frontmatter    # triggers, keywords
│   ├── core knowledge # the most important 20%
│   └── quick reference # frequently used items
│
├── extended/          # Extended layer (loaded on demand)
│   ├── templates.md   # Template collection
│   ├── examples.md    # Full examples
│   └── checklists.md  # Checklists
│
└── references/        # Reference layer (external links)
    └── links.md       # External resource links
```

### 2.2 Core Layer Content Standards

**Must include** (< 300 lines):
- Frontmatter (triggers, keywords)
- Applicable scenarios (5-10 items)
- Core concepts (3-5)
- Decision frameworks (1-2)
- Sharp Edges (3-5)

**Should be linked externally**:
- Full examples (> 20 lines)
- Templates (> 10 lines)
- Detailed checklists
- ASCII diagrams (> 10 lines)

---

## 3. Optimization Process

### 3.1 Analysis Phase

```
Step 1: Gather statistics
────────────────────────────────────────
wc -l SKILL.md                    # total lines
grep -c "^#" SKILL.md             # number of sections
grep -c "```" SKILL.md            # number of code blocks
grep -E "┌|└|│|─" SKILL.md | wc -l  # ASCII diagram lines

Step 2: Classify
────────────────────────────────────────
Identify the type of each section:
□ Core knowledge (essential)
□ Best practices (useful)
□ Examples/templates (can be linked out)
□ ASCII diagrams (can be simplified)
□ Decorative content (can be removed)

Step 3: Score
────────────────────────────────────────
Token efficiency score = core content lines / total lines × 100

Excellent: > 70%
Good: 50-70%
Needs optimization: < 50%
```

### 3.2 Performing the Optimization

**A. Diagram Simplification**

Before (8 lines, ~200 tokens):
```
┌─────────────────────────────────────┐
│  MDA Framework                       │
│                                      │
│  Mechanics → Dynamics → Aesthetics   │
│  Rules       Behavior     Feeling    │
│                                      │
│  Designer's perspective ──→ Player's perspective │
└─────────────────────────────────────┘
```

After (2 lines, ~30 tokens):
```
MDA: Mechanics(rules) → Dynamics(behavior) → Aesthetics(feeling)
     Designer's perspective ───────────────────────→ Player's perspective
```

**B. Table Condensing**

Before (large numeric table):
```markdown
| Stage | Player HP | Enemy HP | Battle Duration | XP | Gold |
|------|---------|---------|----------|--------|------|
| Early | 100 | 30-50 | 10-30 sec | 10-20 | 5-10 |
| Mid | 500 | 200-400 | 30-60 sec | 50-100 | 20-50 |
| Late | 2000 | 1000+ | 1-3 min | 200+ | 100+ |
```

After (linked out):
```markdown
Balance value reference → [extended/balance-tables.md]
```

**C. Linking Out Examples**

Before (50-line inline template):
```markdown
## GDD Template

### 1. Overview
- Game name
- Genre, platform
... (50 lines)
```

After (linked out):
```markdown
## GDD Template

Full template → [extended/templates.md#gdd]

Quick version (three core elements):
1. Core experience: describe in one sentence
2. Core loop: what the player repeatedly does
3. Unique selling point: why play this game
```

---

## 4. Optimization Checklist

### 4.1 Structure Check

- [ ] Total lines < 300
- [ ] Core knowledge proportion > 70%
- [ ] No examples > 20 lines
- [ ] No ASCII diagrams > 10 lines
- [ ] Has an extended/ directory for extended content

### 4.2 Content Check

- [ ] Frontmatter is complete (triggers, keywords)
- [ ] Applicable scenarios are clear
- [ ] Core concepts are concise
- [ ] Sharp Edges are retained
- [ ] Large examples are linked out

### 4.3 Link Check

- [ ] External link paths are correct
- [ ] extended/ files exist
- [ ] Reference resources are accessible

---

## 5. Example: Before/After Optimization

### Before: game-design (934 lines)

```
Section breakdown:
├── Core knowledge: 200 lines (21%)
├── Examples/templates: 450 lines (48%) ← can be linked out
├── ASCII diagrams: 180 lines (19%) ← can be simplified
└── Best practices: 104 lines (11%)

Token efficiency: 21% (needs optimization)
```

### After: game-design (280 lines)

```
Section breakdown:
├── Core knowledge: 180 lines (64%)
├── Simplified diagrams: 30 lines (11%)
├── Best practices: 50 lines (18%)
└── External link guidance: 20 lines (7%)

Token efficiency: 64% (good)
Savings: 70% tokens
```

---

## 6. Automation Scripts

### 6.1 Analysis Script

```bash
#!/bin/bash
# skill-analyzer.sh

SKILL_PATH="$1"

echo "=== Skill Analysis ==="
echo "Total lines: $(wc -l < "$SKILL_PATH")"
echo "Sections: $(grep -c "^#" "$SKILL_PATH")"
echo "Code blocks: $(grep -c '```' "$SKILL_PATH" | awk '{print $1/2}')"
echo "ASCII art lines: $(grep -cE '┌|└|│|─|╭|╰' "$SKILL_PATH")"

# Calculate efficiency
TOTAL=$(wc -l < "$SKILL_PATH")
ASCII=$(grep -cE '┌|└|│|─|╭|╰' "$SKILL_PATH")
EFFICIENCY=$((100 - ASCII * 100 / TOTAL))
echo "Estimated efficiency: ${EFFICIENCY}%"
```

### 6.2 Batch Audit

```bash
#!/bin/bash
# skill-audit.sh

REPO_PATH="$1"

echo "=== Skill Repository Audit ==="
echo ""
printf "%-40s %8s %10s\n" "Skill" "Lines" "Efficiency"
echo "────────────────────────────────────────────────────────────"

find "$REPO_PATH" -name "SKILL.md" -not -path "*/\.*" | while read skill; do
  NAME=$(dirname "$skill" | xargs basename)
  LINES=$(wc -l < "$skill")
  ASCII=$(grep -cE '┌|└|│|─' "$skill" 2>/dev/null || echo 0)
  EFF=$((100 - ASCII * 100 / LINES))

  if [ $LINES -gt 500 ]; then
    STATUS="⚠️"
  elif [ $LINES -gt 300 ]; then
    STATUS="🟡"
  else
    STATUS="✅"
  fi

  printf "%-40s %8d %9d%% %s\n" "$NAME" "$LINES" "$EFF" "$STATUS"
done

echo ""
echo "Legend: ✅ Good (<300) | 🟡 Review (300-500) | ⚠️ Optimize (>500)"
```

---

## 7. Frequently Asked Questions

### Q: How do you ensure full functionality after layering?

A: The core layer covers 80% of common use cases. When a full example is needed, the AI will automatically prompt:
```
"Need the full GDD template? I can load extended/templates.md"
```

### Q: Doesn't linking out files add complexity?

A: Yes, but:
- Most of the time it doesn't need to be loaded
- Loading on demand saves tokens
- The structure is clearer

### Q: How do you decide whether content should be linked out?

A: Use the "rule of 3":
- If this content is only used in 1 out of 3 conversations → link it out
- If it's used every time → keep it in the core

---

## Version History

| Version | Date | Changes |
|------|------|------|
| 1.0.0 | 2026-01-15 | Initial version: Skill optimization methodology |
