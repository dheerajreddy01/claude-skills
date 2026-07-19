---
schema: "1.0"
name: deckbuilder-roguelike
version: "1.0.0"
description: Slay the Spire-style deckbuilding roguelike game design framework
domain: creative
triggers:
  keywords:
    primary: [deckbuilder, Slay the Spire, roguelike deckbuilding, card combat, turn-based card game]
    secondary: [energy system, relic, card pool, card balance, synergy]
  context_boost: [card, combat, turn, draw]
  context_penalty: [TCG, trading card game, physical card]
  priority: high
dependencies:
  domain-skills: [game-design, game-planner]
author: claude-domain-skills
---

# Deckbuilder Roguelike Design

> A design framework for single-player card-building roguelike games in the style of Slay the Spire

## Applicable Scenarios

- Designing Slay the Spire-style card combat systems
- Card value balancing and effect design
- Energy/resource system design
- Relic and card synergy design

## Quick Commands

```
/deckbuilder card [name]      # Design a single card
/deckbuilder character [name] # Design a character's card pool
/deckbuilder relic [rarity]   # Design a relic
/deckbuilder balance [list]   # Value balance check
```

---

## Core System Architecture

### Core Deckbuilder Roguelike Formula

```
Energy System + Deckbuilding + Roguelike = Decision Tension + Build Diversity + Replay Value
  └ Relic System + Enemy Intent → Synergy Depth + Tactical Planning
```

**Why it works:**
| Element | Function | Design Goal |
|------|------|----------|
| Energy limit | Forces trade-offs | Every card is a decision |
| Random rewards | Passive deckbuilding | No predetermined optimal strategy |
| Enemy intent | Information game | Meaningful defensive choices |
| Relic system | Permanent change | Every run feels different |
| Deck thinning | Dilution penalty | Less is more |

---

## 1. Energy System

**Base:** 3 energy per turn, can be increased via relics/cards

**Energy cost distribution:**
- 0 cost (10-15%): Support/conditional trigger
- 1 cost (40-50%): Main cards/basic effects
- 2 cost (25-30%): Strong/compound effects
- 3 cost (10-15%): Finishers/ultimate
- X cost (5%): Variable/dump resources

**Energy manipulation design:**
| Type | Effect Template | Design Consideration |
|------|----------|----------|
| Energy generation | Gain X energy | High value, needs balancing |
| Cost reduction | Next card -X cost | Combo potential |
| X-cost cards | Consumes all energy | Turn finisher |

---

## 2. Card Types

### Five Main Types

- **Attack:** Deals damage, may apply debuffs
- **Skill:** Block, draw, buff/debuff, deck manipulation
- **Power:** Permanent effect, doesn't go to discard pile after being played
- **Status:** Negative effect, removed after combat
- **Curse:** Permanent negative, requires special means to remove

### Rarity

| Rarity | Appearance Rate | Characteristics |
|--------|--------|------|
| Basic | Starting cards | Simple and direct |
| Common | 60% | Build foundation |
| Uncommon | 37% | Strategy defining |
| Rare | 3%* | Build core |

> *Pity system: +1% rare chance for each common drawn

### Keyword Quick Reference

```
Exhaust: removed after use    Ethereal: consumed at end of turn
Innate: always in starting hand   Retain: kept without discarding
Block: mitigation (resets to zero at the start of your next turn)
Vulnerable: +50% damage taken   Weak: -25% damage dealt   Frail: -25% block gained
Strength: +X attack       Dexterity: +X block
```

---

## 3. Turn Structure

```
Turn start → Gain energy(3) + Draw cards(5) + Block resets to zero
    ↓
Player action → Play cards/use potions/end turn
    ↓
Turn end → Trigger effects + Discard (Ethereal consumed) + Energy clears
    ↓
Enemy action → Executes per intent + Determines next turn's intent
```

### Enemy Intent Types

- **Attack:** Shows damage value
- **Defend:** Gains block
- **Buff/Debuff:** Buffs itself or debuffs the player
- **Unknown:** Not shown (elites/bosses)

---

## 4. Relic System

### Design Philosophy

> "Relics should change your decisions, not just add numbers"

**Good relic design:**
- Changes strategic priorities
- Creates new synergies
- Defines build direction

**Avoid:**
- Pure stat boosts (no impact)
- Overly complex (hard to evaluate)
- Unconditionally powerful (mandatory pick, boring)

### Rarity Sources

| Rarity | Source | Design Goal |
|--------|------|----------|
| Starting | Character default | Defines style |
| Common | Elite/Shop | General power boost |
| Uncommon | Elite/Shop | Build defining |
| Rare/Boss | Boss | Game changer |

---

## 5. Builds and Synergy

### Common Build Archetypes

| Build | Core | Synergy |
|-------|------|---------|
| Strength flow | Stack Strength buff | Strength + multi-hit attacks |
| Block flow | Excessive blocking/retention | Barricade + stacking block |
| Exhaust flow | Exhaust triggers effects | Each exhaust +X effect |
| Draw flow | Massive draw/small deck | Thin deck + infinite loop |
| Poison/DOT flow | Stack damage over time | Poison doubling + survival |

### Synergy Design Principles

- Each Build needs 2-3 "core build" cards
- "Safe pick" cards maintain baseline playability
- "High risk, high reward" cards create excitement

---

## 6. Value Baselines

**1 Energy ≈**
- 6-8 damage / 5-6 block
- 1 card draw / 1 energy generation
- Light debuff (1 turn)

**Keyword value:**
- Exhaust = -0.5 energy
- Ethereal = -0.3 energy
- Innate = +0.3 energy
- Retain = +0.2 energy
- Upgrade = +0.5~1.0 energy

> See `extended/balance-tables.md` for detailed value tables

---

## 7. Map and Progression

### Node Types

- **Normal combat:** Grants card rewards
- **Elite combat:** Grants relic + card rewards
- **Unknown event:** Random event
- **Shop:** Buy/remove cards
- **Rest:** Heal or upgrade a card
- **Boss:** Defeat to advance to the next act

### Three-Act Structure

| Act | Enemy HP | Enemy Damage | Goal |
|-----|--------|----------|------|
| 1 | 30-50 | 8-15 | Build formation |
| 2 | 50-100 | 15-25 | Refine synergy |
| 3 | 100-200 | 25-40 | Prove the build |

---

## 8. Sharp Edges

### SE-1: Deck Bloat

**Problem:** Taking too many cards → can't draw key cards

**Solution:**
- Design a deck size cap or penalty
- Provide opportunities to remove cards
- Teach that "not picking is also a choice"

### SE-2: Mandatory Cards/Relics

**Problem:** Overpowered options → mindless auto-pick → reduced diversity

**Solution:**
- All options have trade-offs
- Powerful cards require build support
- Pick rate >80% needs a nerf

### SE-3: Defense is Boring

**Problem:** Players find playing block cards boring

**Solution:**
- Defensive cards should also feel satisfying
- Design counterattacks/block-to-attack conversion
- Reward excess block

### SE-4: Luck vs. Skill

**Problem:** Players complain "luck determines the outcome"

**Solution:**
- Guarantee baseline playability
- Offer "choices" rather than pure randomness
- Rare card pity mechanism
- Multiple build paths available

---

## 9. Balance Tracking

### Key Metrics

| Metric | Warning Threshold | Action |
|------|--------|------|
| Pick rate > 80% | Too strong | Nerf |
| Pick rate < 5% | Too weak/unclear role | Buff/redesign |
| Win rate delta > 10% | Needs adjustment | Analyze cause |

### Adjustment Principles

1. Fix bugs/unintended interactions — highest priority
2. Nerf "mandatory" cards/relics — high priority
3. Buff "never picked" options — medium priority
4. Fine-tune values — low priority

---

## Extended Resources

- `extended/templates.md` — Card/relic/character/enemy design templates
- `extended/examples.md` — Build examples/case study analysis
- `extended/balance-tables.md` — Detailed value balance tables

## Related Resources

- [Slay the Spire Wiki](https://slaythespire.wiki.gg/)
- [How StS devs use data](https://www.gamedeveloper.com/design/how-i-slay-the-spire-i-s-devs-use-data-to-balance-their-roguelike-deck-builder)

## Related Domains

- [[game-design]] — General game design theory
- [[game-planner]] — GDD writing
