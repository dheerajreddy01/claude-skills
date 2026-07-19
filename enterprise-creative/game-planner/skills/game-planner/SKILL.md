---
schema: "1.0"
name: game-planner
version: "1.0.0"
description: Professional game planning framework for producing industry-standard Game Design Documents (GDD)
domain: creative
triggers:
  keywords:
    primary: [GDD, game planning, game design document, design pillars]
    secondary: [feature document, GDD audit, document consolidation, consolidate, game mechanics design]
  context_boost: [planning, document, architecture, design]
  context_penalty: [coding, art]
  priority: high
dependencies:
  domain-skills: [game-design, storytelling]
author: claude-domain-skills
---

# Game Planner

> Professional game planning framework — producing industry-standard Game Design Documents (GDD)

## Use Cases

- GDD initialization and architecture design
- Consolidating scattered design documents
- GDD completeness audit
- Feature Document writing
- Numerical balance design framework

## Quick Commands

```bash
/game-planner gdd init          # Initialize a GDD for a new project
/game-planner gdd audit         # Audit the completeness of an existing GDD
/game-planner gdd consolidate   # Consolidate scattered design documents into a single GDD
/game-planner feature [name]    # Generate a Feature Document
/game-planner mechanic [name]   # Design a game mechanic
/game-planner balance [system]  # Numerical balance analysis
```

---

## Core Philosophy

### The Modern Philosophy of GDDs

Industry consensus: **there is no single standard format**, but there are shared principles:

| Principle | Description |
|------|------|
| **Living Document** | Continuously updated living document, not a one-time deliverable |
| **Agile & Lightweight** | Agile and lightweight, avoiding hundred-page documents nobody reads |
| **Findable** | Information can be found quickly |
| **Visual** | Games are a visual medium, so the document should be visual too |

### Two-Tier Architecture Design

```
┌─────────────────────────────────────────────────────────────┐
│  Master Document                                              │
│  ├── Executive summary, vision, design pillars                │
│  ├── Core gameplay overview                                   │
│  └── Links to each Feature Document                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  Feature Documents                                             │
│  ├── Combat system                                             │
│  ├── Character system                                          │
│  ├── Economy system                                             │
│  └── ...                                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## Standard GDD Structure

### Master Document Required Sections

| Section | Content | Priority |
|------|------|--------|
| **1. Executive Summary** | One-page overview: game title, genre, platform, target audience, elevator pitch | P0 Required |
| **2. Design Pillars** | 3-5 core design principles that all decisions are based on | P0 Required |
| **3. Core Gameplay** | Gameplay loop, win/loss conditions, controls | P0 Required |
| **4. Game Mechanics** | Overview of each system (links to Feature Docs) | P0 Required |
| **5. Narrative & World** | Story outline, setting overview | P1 Important |
| **6. Character Design** | Overview of main characters | P1 Important |
| **7. Level Design** | Level structure, progression design | P1 Important |
| **8. UI/UX Design** | Interface flow, HUD design | P1 Important |
| **9. Music & Sound** | Audio style guidelines | P2 Supplementary |
| **10. Business Model** | Pricing, monetization strategy | P1 Important |
| **11. Development Plan** | Milestones, timeline | P1 Important |
| **12. Risk Assessment** | Potential risks and mitigation | P2 Supplementary |

### Feature Document Template

```markdown
# [System Name] Feature Document

## Design Goals
> What problem does this system solve? What experience does it deliver?

## Core Mechanics
### Basic Rules
### Numerical Design
### Special Cases

## Player Experience
### Intended Feel
### Risks and Negative Experiences

## Relationship With Other Systems
### Dependent Systems
### Affected Systems

## Implementation Priority (LoQ)
| Tier | Content | Status |
|------|------|------|
| Core | Minimum playable version | |
| Expected | Features players expect | |
| Desired | Nice-to-have features | |
| Aspirational | Ideal state | |

## Changelog
| Version | Date | Change |
|------|------|------|
```

---

## GDD Audit Checklist

```
┌─────────────────────────────────────────────────────────────┐
│  GDD Audit Checklist                                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ▣ Executive Summary                                        │
│    □ Game title and tagline                                 │
│    □ Clearly defined genre                                  │
│    □ Target platform                                        │
│    □ Target audience                                        │
│    □ Elevator pitch (30-second version)                     │
│    □ Unique selling point (USP)                             │
│                                                             │
│  ▣ Design Pillars                                            │
│    □ 3-5 core design principles                              │
│    □ Each pillar has clear "do/don't" examples               │
│                                                             │
│  ▣ Core Gameplay                                             │
│    □ Gameplay loop diagram                                   │
│    □ Win/loss conditions                                     │
│    □ Controls (per platform)                                 │
│    □ Target playtime                                         │
│                                                             │
│  ▣ Game Mechanics                                             │
│    □ Every core system has a Feature Document                │
│    □ Dependencies between systems are clear                  │
│    □ Numerical design has a theoretical basis                │
│                                                             │
│  ▣ Consistency Check                                          │
│    □ Unified terminology (glossary exists)                   │
│    □ Consistent version numbers                              │
│    □ No contradictory descriptions                           │
│    □ All links are valid                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Design Pillar Examples

### What Are Design Pillars?

Design pillars are 3-5 core principles that guide all design decisions.

**Characteristics of good design pillars**:
- ✅ Can be used to reject features that don't fit
- ✅ Team members can use them to make decisions
- ✅ Unique to this project (not generic platitudes)

**Bad design pillars**:
- ❌ "Should be fun" — a platitude
- ❌ "Should look beautiful" — not measurable
- ❌ "Players will like it" — not actionable

### Example: Dead Romance Design Pillars

```markdown
### 1. Narrative and System Fusion
> Players shouldn't feel like they're "switching game genres"

- ✅ Dialogue choices = acquiring cards
- ✅ Bond level = unlocking abilities
- ❌ Separate "story mode" and "battle mode"

### 2. Short-Paced Loop
> Preserve the narrative pacing of a Galgame (visual novel)

- ✅ Battles last 60-90 seconds (3-5 turns)
- ✅ Dialogue nodes last 10-20 seconds
- ❌ Battles lasting over 5 minutes

### 3. Choices Carry Weight
> Every choice affects the build and the story

- ✅ Dialogue options have systemic effects
- ✅ Choices affect ending branches
- ❌ Purely cosmetic options

### 4. Love Is a Double-Edged Sword
> The core of the setting: love makes you stronger but also more dangerous

- ✅ Deeper bond → stronger ability → more enemies
- ✅ Higher emotion value → higher damage → possible loss of control
- ❌ Simple "affection = reward"
```

---

## Numerical Design Framework

### Basic Principles of Numerical Balance

| Principle | Description | Example |
|------|------|------|
| **Symmetry** | Resource inflow and outflow should be balanced | Gain X energy per turn, spend X-Y energy |
| **Predictability** | Players can estimate outcomes | Damage formulas are clearly visible |
| **Risk/Reward** | High risk corresponds to high reward | Emotion value 9-10: +50% damage, possible loss of control |
| **Growth Curve** | Avoid being too flat or too steep | Bond level benefits increase but have a cap |

### Combat Numbers Template

```
Expected per turn:
├── Resource income: +X (base) + Y (skills)
├── Resource expenditure: Z (playing N cards)
├── Damage output: A-B
└── Damage taken: C-D

Expected per battle (R turns):
├── Total damage output: (A-B) × R = ?
├── Total damage taken: (C-D) × R = ?
└── Net gain: remaining resources, cards acquired

Enemy design basis:
├── Regular enemy group: HP = output × 2 (cleared in 2 turns)
├── Elite single enemy: HP = output × 3-4
└── Boss: HP = output × 5-7 (including multiple phases)
```

---

## Document Consolidation Process

### From Scattered Documents to a Unified GDD

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Inventory existing documents                        │
│  ├── List all design-related documents                       │
│  ├── Mark each document's topic and version                  │
│  └── Identify duplicate/contradictory content                │
├─────────────────────────────────────────────────────────────┤
│  Step 2: Define the source of truth                          │
│  ├── Each topic has only one authoritative document          │
│  ├── Mark other documents as deprecated or merge them         │
│  └── Build a terminology reference table                      │
├─────────────────────────────────────────────────────────────┤
│  Step 3: Build the master document structure                 │
│  ├── Create the Master Document                              │
│  ├── Split large systems into Feature Documents               │
│  └── Build cross-reference links                              │
├─────────────────────────────────────────────────────────────┤
│  Step 4: Content migration and consolidation                 │
│  ├── Consolidate content by priority                          │
│  ├── Resolve contradictions (prefer the newest/most complete) │
│  └── Remove redundant content                                 │
├─────────────────────────────────────────────────────────────┤
│  Step 5: Validation and cleanup                               │
│  ├── Run the GDD audit checklist                              │
│  ├── Confirm version numbers are consistent                   │
│  └── Update all cross-references                              │
└─────────────────────────────────────────────────────────────┘
```

### Consolidation Decision Matrix

When a contradiction is found, use this decision process:

```
Contradictory content found
    ↓
Which version is newer?
    ↓
    ├── Version A is newer → lean toward adopting A
    │
    └── Unsure/simultaneous → which is more complete and detailed?
                          ↓
                          ├── A is more complete → adopt A
                          │
                          └── Neither is complete → redesign
```

---

## Sharp Edges

### SE-1: GDD Over-Design
- **Severity**: high
- **Scenario**: The GDD becomes a hundred-page document nobody reads
- **Cause**: Trying to define every detail before development begins
- **Symptoms**:
  - The team doesn't read the GDD
  - Implementation and documentation are seriously out of sync
  - Updating the GDD becomes a burden
- **Solution**:
  - Adopt a two-tier architecture (Master + Feature)
  - Keep the Master Document to 10-20 pages
  - Only systems currently in development need a detailed Feature Doc
  - Use links instead of copy-pasting

### SE-2: Design Contradictions
- **Severity**: critical
- **Scenario**: The same mechanic is described differently across documents
- **Cause**: Documents are scattered with no source of truth
- **Symptoms**:
  - Developers don't know which version is correct
  - Implementation requires repeated verification
  - Discussions devolve into disagreement
- **Solution**:
  - Designate an authoritative document for each topic
  - Build a glossary to enforce consistency
  - Clearly mark outdated documents as deprecated
  - Regularly audit for consistency

### SE-3: Lack of Design Pillars
- **Severity**: high
- **Scenario**: Features pile up without direction
- **Cause**: Core design principles were never defined
- **Symptoms**:
  - Each feature looks "fine" individually but the whole loses focus
  - Hard to reject new feature requests
  - Development priorities are hard to decide
- **Solution**:
  - Define 3-5 design pillars
  - Each pillar has "do/don't" examples
  - New features must answer "How does this enhance the core experience?"

---

## Common Issues Checklist

| Issue | Symptom | Solution |
|------|------|----------|
| **Design contradictions** | The same mechanic is described differently across documents | Designate an authoritative document, unify terminology |
| **Lack of pillars** | Features pile up without direction | Define design pillars, cut features that don't fit |
| **Over-design** | Documents are too long for anyone to read | Split into Feature Docs, keep the master document concise |
| **Terminology confusion** | The same concept has multiple names | Build a glossary, enforce consistency |
| **Orphaned documents** | Documents are scattered with no index | Build a master document index |

---

## Version Control Rules

```
Version number format: vX.Y

X = Major version (significant structural changes)
Y = Minor version (content updates/corrections)

Update rules:
- New section/system added → X +1
- Content correction/addition → Y +1
- Terminology/format unification → Y +1
```

---

## Reference Resources

- [Nuclino - GDD Template](https://www.nuclino.com/articles/game-design-document-template)
- [Game Design Skills - GDD Guide](https://gamedesignskills.com/game-design/document/)
- [GitBook - How to Write a GDD](https://www.gitbook.com/blog/how-to-write-a-game-design-document)
- [Document360 - GDD Best Practices 2025](https://document360.com/blog/write-game-design-document/)
- [Unity GDD Template](https://learn.unity.com/tutorial/fill-out-a-game-design-document)

## Related Domains

- [[game-design]] - Game design theory and mechanics
- [[storytelling]] - Game narrative and scripts
- [[ui-ux-design]] - Game UI/UX
