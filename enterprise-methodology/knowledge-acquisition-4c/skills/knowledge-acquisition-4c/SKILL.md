---
schema: "1.0"
name: knowledge-acquisition-4c
version: "1.0.0"
description: A systematic learning methodology for going from an unknown domain to expertise
triggers:
  keywords:
    primary: [learning, research, new domain, knowledge acquisition]
    secondary: [getting started, professional, mastery, starting from scratch, beginner, expert, mastery]
  context_boost: [unfamiliar, unknown, novice]
  context_penalty: [implement, code]
  priority: high
keywords: [methodology, learning, knowledge, acquisition, expertise]
dependencies: []
author: claude-domain-skills
---

# Knowledge Acquisition 4C Methodology

> A systematic process for going from an unknown domain to expertise

## Applicable Scenarios

- Entering a completely unfamiliar domain
- Needing to rapidly build expert knowledge
- Learning a new skill from scratch
- Researching an unfamiliar topic

## Core Framework

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                    4C Knowledge Acquisition                     │
│                                                                 │
│   COLLECT        CURATE         CONNECT        CREATE          │
│                                                                 │
│     │              │               │              │             │
│     ▼              ▼               ▼              ▼             │
│  ┌──────┐      ┌──────┐       ┌──────┐      ┌──────┐          │
│  │Diverse│      │Verify │       │Build  │      │Produce│          │
│  │sources│  →   │& test │   →   │system │  →   │& teach│          │
│  │Broad  │      │Filter │       │Connect│      │Iterate│          │
│  └──────┘      └──────┘       └──────┘      └──────┘          │
│                                                                 │
│ Dreyfus: Novice → Adv.Beginner → Competent → Proficient → Expert│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Phase 1: COLLECT

**Goal**: Gather broadly, without preconceptions

**Dreyfus stage**: Novice

### Information Source Tiers

| Tier | Source | Credibility | Use |
|------|------|--------|------|
| L1 | Official documentation, academic papers, authoritative books | ⭐⭐⭐⭐⭐ | Core concepts |
| L2 | Expert articles, technical blogs, tutorials | ⭐⭐⭐⭐ | Practical experience |
| L3 | Community discussions, Stack Overflow, forums | ⭐⭐⭐ | Common problems |
| L4 | Social media, news reports | ⭐⭐ | Trend observation |

### Collection Checklist

```markdown
□ Core terminology of the domain (build a glossary)
□ Main concepts and principles
□ Key figures/authoritative sources
□ Common problems and solutions
□ Tools and resources
□ Success stories and failure cases
```

### Output

- Raw notes (unorganized, just recorded)
- List of source links
- List of hypotheses to be verified

---

## Phase 2: CURATE

**Goal**: Verify information, filter out the noise

**Dreyfus stage**: Advanced Beginner

### Three Verification Steps

```
1. Cross-verification: Does the same information appear across multiple independent sources?
2. Practical testing: Does theoretical data match actual results?
3. Timeliness check: Is the information outdated?
```

### Sharp Edge: Theory ≠ Practice

```
┌─────────────────────────────────────────────────────────────────┐
│  ⚠️ SE-1: Theoretical data ≠ actual results                     │
│                                                                 │
│  Severity: critical                                             │
│                                                                 │
│  Real case:                                                     │
│  - A chart pattern study claimed an 80-84% success rate        │
│  - Actual backtest: made the strategy perform worse            │
│  - Reason: stock market research data doesn't apply to crypto  │
│                                                                 │
│  Lesson: always verify research conclusions with real testing  │
│                                                                 │
│  Solution:                                                      │
│  ❌ Believing research data at face value                       │
│  ✅ Verifying with real-data backtests/experiments               │
└─────────────────────────────────────────────────────────────────┘
```

### Filtering Criteria

| Keep | Mark as unverified | Discard |
|------|-----------|------|
| Consistent across sources | Single source but testable | Cannot be verified |
| Proven effective in testing | Theoretically sound but untested | Already refuted |
| Authoritative source | Community consensus but no evidence | Outdated information |

### Output

- List of verified knowledge
- List of unverified hypotheses
- Excluded misconceptions

---

## Phase 3: CONNECT

**Goal**: Build a knowledge system, form a mental model

**Dreyfus stage**: Competent → Proficient

### Connection Methods

| Method | Description | Example |
|------|------|------|
| Concept mapping | Draw a diagram of relationships between concepts | Mind maps, knowledge graphs |
| Analogical transfer | Connect to similar concepts in known domains | "This is like..." |
| Hierarchical organization | Arrange from basic to advanced | Pyramid structure |
| Causal tracing | Understand "why," not just "what" | 5 Whys |

### The Feynman Technique

```
┌─────────────────────────────────────────────────────────────────┐
│  Feynman Technique                                               │
│                                                                 │
│  1. Choose a concept                                            │
│  2. Explain it in simple terms to a layperson                   │
│  3. Find the gaps in your explanation → go back and study        │
│  4. Simplify, then simplify again                                │
│                                                                 │
│  Core idea: being able to explain simply = true understanding   │
│             an unclear explanation = not yet understood          │
└─────────────────────────────────────────────────────────────────┘
```

### Knowledge Structure Template

```markdown
## [Domain Name] Knowledge System

### Core Concepts
- Concept A: [Definition]
  - Relationship to B: [Explanation]
  - Practical application: [Case]

### Key Principles
1. Principle 1: [Explanation]
   - Why it matters: [Reason]
   - Common misconceptions: [Correction]

### Decision Framework
When [situation] → choose [approach]
Because [reason]
```

### Output

- Structured knowledge map
- Decision framework/mental model
- A simple, teachable explanation

---

## Phase 4: CREATE

**Goal**: Produce value, iterate continuously

**Dreyfus stage**: Expert

### Forms of Creation

| Form | Description | Verification method |
|------|------|---------|
| Teaching | Teach others (the strongest form of learning) | Do students understand? |
| Implementation | Apply the knowledge in practice | Do results meet expectations? |
| Documentation | Write it up as a Skill/tutorial | Can others use it? |
| Innovation | Propose a new method/improvement | Is it better than existing methods? |

### Iteration Loop

```
CREATE → discover gaps → back to COLLECT
   ↑                    ↓
   └──── CURATE ← CONNECT ←┘
```

### Self-Assessment of Expertise

```markdown
□ Can you explain the core concept in one sentence?
□ Can you predict common problems and their solutions?
□ Can you identify and correct misconceptions?
□ Can you apply the knowledge to new situations?
□ Can you teach it to someone else?
```

### Output

- Reusable Skill documentation
- Actual results/work product
- Teaching materials

---

## Best Practices

1. **Don't skip CURATE** - Verification is the most important phase
2. **Build a glossary** - Terminology is the key to entering a domain
3. **Find authoritative sources** - Identify the experts and classics in the field
4. **Actually do the work** - Doing it once beats reading about it ten times
5. **Teach others** - Teaching is the best form of learning

## Common Mistakes

| Mistake | Consequence | Fix |
|------|------|------|
| ❌ Collecting without verifying | Incorrect knowledge | Enforce the CURATE phase |
| ❌ Verifying without connecting | Fragmented knowledge | Draw a concept map |
| ❌ Learning without applying | All theory, no practice | Enforce CREATE |
| ❌ Skipping verification | Theory ≠ practice | Test every hypothesis in reality |
| ❌ Information overload | Can't digest it all | Limit daily input |

## Suggested Time Allocation

```
COLLECT: 20% (don't over-collect)
CURATE:  30% (verification matters most)
CONNECT: 30% (build the system)
CREATE:  20% (produce and verify)
```

## Integration with evolve

This Skill is automatically detected and loaded by `/evolve`:

```
User: /evolve learn blockchain development

Agent:
🔍 Auto Domain Detection
→ Keywords: learning, blockchain
→ Loading: enterprise-methodology/knowledge-acquisition-4c
→ Using the 4C methodology to execute the learning task
```

## References

- [Dreyfus Model of Skill Acquisition](https://en.wikipedia.org/wiki/Dreyfus_model_of_skill_acquisition)
- [The Feynman Technique](https://fs.blog/feynman-technique/)
- [Knowledge Management Guide 2025](https://knowmax.ai/knowledge-management/)
