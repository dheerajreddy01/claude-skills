---
schema: "1.0"
name: galgame-master
version: "1.0.0"
description: Galgame Creation Master — character type library, script structure, dialogue generation, art direction
domain: creative
triggers:
  keywords:
    primary: [galgame, visual novel, dating sim, romance game, galge]
    secondary: [script, dialogue, character, romance target, heroine]
  context_boost: [game, romance, story, narrative, scenario]
  context_penalty: [marketing, finance, 3D, FPS]
  priority: high
dependencies:
  software-skills: [game-development]
keywords: [visual-novel, game-writing, character-design, narrative, romance, dating-sim]
author: claude-domain-skills
---

# Galgame Creation Master

> A general-purpose galgame creation framework — applicable to visual novel / dating sim development in any setting

## Applicable Scenarios

- Galgame / visual novel project planning and design
- Romanceable heroine design and type combinations
- Script structure planning and branch design
- Dialogue generation and character voice/tone crafting
- CG / character sprite art direction writing

---

## Core Framework

### Three-Layer Character Design

| Layer | Content | Tsundere Example |
|------|------|----------|
| Layer 1: Surface | How others see her | Cold and sharp-tongued, "It's not like I did it for you" |
| Layer 2: Private | Known only to those close to her | Secretly caring, preparing things |
| Layer 3: Affinity | Reactions that change with the relationship | Lv0 cold → Lv3 flustered → Lv5 affectionate |

### Character Formula

```
Complete character = Dere personality + relationship type + identity type + [special attributes]
```

**Example**: Classic tsundere = `tsundere` + `osananajimi` + `twintails`

---

## Character Type Library (34 Types)

### Dere Personalities (8 Types)

| ID | Type | Core Trait | Surface → Private |
|----|------|------|---------------|
| `tsundere` | Tsundere | Says one thing but means another | Sharp-tongued → caring |
| `yandere` | Yandere | Loves to an extreme | Perfect → possessive |
| `kuudere` | Kuudere | Icy beauty | Expressionless → emotionally rich |
| `tennen` | Natural Airhead | Unintentionally flirtatious | Oblivious → devoted |
| `genki` | Genki | Full of energy | Cheerful → fragile |
| `mukuchi` | Silent Type | Reticent, few words | Silent → shows through action |
| `haraguro` | Haraguro (Black-Hearted) | Smiles while scheming | Smiling → calculating |
| `koakuma` | Koakuma (Little Devil) | Deliberately flirtatious | Teasing → shy |

### Relationship Types (7 Types)

| ID | Type | Distinctive Tension |
|----|------|----------|
| `osananajimi` | Childhood Friend | Too familiar to cross the line |
| `senpai` | Upperclassman | Looking up to and relying on her |
| `kouhai` | Underclassman | Wants to be treated as an adult |
| `gimai` | Step-sister | Forbidden yet close |
| `sensei` | Teacher | Gap in status |
| `classmate` | Classmate | Starting point for pure romance |
| `tonari` | Neighbor | Everyday flirtation |

### Identity Types (9 Types)

| ID | Type | Gap-Moe Focus |
|----|------|-----------|
| `ojousama` | Young Lady | Longs for an ordinary life |
| `kaichou` | Student Council President | Wants to be selfish just once |
| `idol` | Idol | Her true self beneath the public persona |
| `sports_girl` | Sporty Girl | Wants to be seen as a girl |
| `bungaku` | Literary Girl | Rich inner fantasy world |
| `yankee` | Delinquent Girl | Soft-hearted underneath |
| `meganekko` | Glasses Girl | The contrast of taking off her glasses |
| `maid` | Maid | Wants to be seen as a woman |
| `miko` | Shrine Maiden | Curious about romance |

### Special Attributes (10 Types)

| ID | Type | Design Focus |
|----|------|-----------|
| `twintails` | Twintails | The contrast of letting her hair down |
| `kurokami` | Long Black Hair | Traditional beauty |
| `byoujaku` | Sickly | Strength within fragility |
| `kemonomimi` | Animal Ears | Ears that give away her feelings |
| `vampire` | Vampire | The intimacy of drawing blood |
| `queen_s` | Queen S | Sometimes wants to be pampered too |
| `m_type` | M-Type | Genuine feelings run deeper than just submissiveness |
| `chuunibyou` | Chuunibyou | Adorable delusions |
| `denpa` | Denpa (Erratic) | Sudden bursts of bluntness |
| `otokonoko` | Trap / Crossdressing Boy | Loves the person, not the gender |

---

## Classic Combinations

| Combination | Formula | Effect |
|------|------|------|
| Classic Tsundere | Tsundere + Childhood Friend + Twintails | The quintessential "It's not like I like you" |
| Icy Upperclassman | Kuudere + Upperclassman + Student Council President + Long Black Hair | An unattainable beauty who melts |
| Koakuma Underclassman | Koakuma + Underclassman + Idol | Flirts with you, ends up flustered herself |
| Sickly Angel | Natural Airhead + Sickly | Treasuring pure love in the present moment |

**Combination pitfalls to avoid**: Yandere + Sickly (too heavy), Silent Type + Denpa (communication too difficult), Tsundere + Queen S (overlapping traits)

---

## Script Structure

### Standard Route Structure

**Common route**: Everyday Life Arc (character introductions) → Incident Arc (key events) → branch point

**Individual route**: Deep Dive Arc (background revealed) → Conflict Arc (conflict erupts) → Resolution Arc → ending

**Ending types**: True End / Good End / Normal End / Bad End

### Affinity System

| Level | Relationship | Triggerable Events |
|------|------|------------|
| Lv0-1 | Stranger → Acquaintance | First meeting, everyday conversation |
| Lv2-3 | Friend → Budding romance | Alone together, dates |
| Lv4-5 | Lovers → Bonded | Confession, intimacy, True End |

---

## Dialogue Design Principles

### Type-Based Reaction Differences

Same situation, different types react differently:

**When jealous**:
- Tsundere: "Whatever, do what you want!" (turns away)
- Kuudere: "......" (mood turns cold)
- Yandere: "Who is that person?" (smiling, but her eyes aren't)

**When confessing**:
- Tsundere: "It's not a confession! I just...... don't dislike you......"
- Kuudere: "......I like you." (turns away, ears red)
- Natural Airhead: "I like you the most! Is that okay?"

### Using Catchphrases

- Use them at key moments, not in every line
- Silence is also dialogue (the charm of the kuudere and the silent type)

---

## Art Direction Essentials

### CG Specification Sheet Elements

- **Composition**: type, focus, angle
- **Characters**: pose, expression, action
- **Environment**: setting, time, weather
- **Atmosphere**: mood, lighting, color tone
- **Key elements**: visual highlights to emphasize

### Standard Expression Sheet

neutral / smile / laugh / sad / angry / embarrassed / surprised / crying / love / special

---

## Content Rating

| Rating | Permitted Content | Presentation Method |
|------|----------|----------|
| G | Everyday life, hand-holding, hugging | Depicted directly |
| R15 | Kissing, romantic implications | Implied + fade to black |
| R15+ | Overnight scenes | Fade to black + aftermath only |

**R15 principles**: Implication over explicitness, sensory detail, character reactions, leaving room for the reader's imagination

---

## Sharp Edges

| ID | Problem | Solution |
|----|------|----------|
| SE-1 | Characters feel too similar | Use contrast design to ensure differentiation |
| SE-2 | Affinity progression feels disjointed | Design smooth Lv1-Lv4 transition stages |
| SE-3 | Type feels like a shallow label | First design the backstory for "why this type" |

---

## Best Practices

**Character**: Choose the core conflict first → design the three-layer contrast → give her one memorable trait → design the affinity curve

**Script**: The common route needs a "sense of choice," the individual route needs a "sense of exclusivity," the ending needs an "emotional peak"

**Dialogue**: Design multiple type-based reactions to the same line; use catchphrases at key moments

---

## Extended Resources

- `extended/templates.md` — Complete templates for character setup, CG direction, and script structure
- `extended/examples.md` — Detailed type definitions, situational dialogue examples, CG direction examples

---

## Version History

| Version | Date | Changes |
|------|------|------|
| 1.0.0 | 2026-01 | Initial release: 34-type character library, script structure, dialogue generation, art direction |
