# Game Design Examples

## growth-curve-formulas

### Common Growth Formulas

| Type | Formula | Suitable For |
|------|------|------|
| Linear | `value = base + (level x growth)` | Simple increments |
| Exponential | `value = base x (multiplier ^ level)` | Leveling experience |
| Logarithmic | `value = base x log(level + offset)` | Player attributes |
| S-curve | `value = max / (1 + e^(-k(level - mid)))` | Content unlocks |

### Recommended Usage
- **Player attributes**: logarithmic curve (avoid late-game explosion)
- **Leveling experience**: exponential curve (extend game longevity)
- **Content unlocks**: S-curve (control pacing)

## damage-calculation

### Damage Calculation Example

```
Base damage = attack power x (100 / (100 + defense))
DPS = (base damage x crit bonus) / attack interval
Crit bonus = 1 + (crit rate x crit multiplier)
```

### Balance Checkpoints
- Is TTK (Time To Kill) reasonable?
- Is the gap between high and low levels too large?
- Is there a risk of numeric overflow?
- Are all classes/characters balanced?

## number-overflow-fix

### Numeric Overflow Solutions

```python
# Wrong approach - pure exponential growth
damage = base * (1.5 ** level)  # Level 100 = 4e17

# Correct approach - logarithmic or soft cap
damage = base * log(level + 1) * level  # controllable growth

# Or use an asymptote
damage = base * (max_multiplier * level / (level + k))
```

## economy-resource-types

| Resource Type | Acquisition Difficulty | Purpose | Example |
|----------|----------|------|------|
| Soft currency | Easy | Daily consumption | Gold |
| Hard currency | Hard | Rare items | Diamonds |
| Stamina | Time-gated | Limits play time | Energy |
| Materials | Medium | Crafting and upgrades | Materials |

## dialogue-variables

```yaml
# Variables the dialogue system needs to track
player_choices:
  helped_merchant: true
  sided_with_rebels: false

relationship_values:
  companion_a: 75  # -100 to 100
  faction_b: -20

story_flags:
  act1_completed: true
  secret_discovered: false
```

## pcg-applications

| Element | Method | Example |
|------|------|------|
| Maps | Wave Function Collapse | Bad North, Caves of Qud |
| Levels | Rule system + random seed | Spelunky, Diablo |
| Enemy composition | Difficulty curve + random selection | Left 4 Dead |
| Dialogue | LLM generation | AI Dungeon |
| Quests | Templates + variable substitution | Radiant Quests |
| Names/descriptions | Markov chain / LLM | Dwarf Fortress |

### PCG Design Principles
1. Set reasonable constraints (avoid impossible generation)
2. Guarantee a minimum quality bar (filter out unreasonable results)
3. Preserve key hand-crafted content
4. Seed system (reproducible randomness)

## npc-ai-patterns

### Behavior Tree
Traditional AI, predictable, suitable for enemy AI

```
Selector -> Sequence -> Action
   |
[Attack] -> [Chase] -> [Patrol]
```

### Finite State Machine (FSM)
Simple and intuitive, suitable for basic NPCs

```
[Idle] <-> [Alert] <-> [Attack]
              ^          |
              +-- [Flee] <+
```

### LLM-Driven NPCs (Experimental)
Dynamic dialogue, personalized reactions. Challenges: cost, latency, consistency

## ui-types

### Game UI Layers

| Type | Description | Example | Pros/Cons |
|------|------|------|--------|
| Diegetic UI | Exists within the game world | Dead Space's back-mounted health bar | Strong immersion but harder to read |
| Non-Diegetic UI | Pure interface elements (HUD) | Health bar, minimap | Clear information but may break immersion |
| Spatial UI | Exists in 3D space but not part of the world | Enemy health bar above their head | Somewhere in between |

### Design Points
- Place important information in the visual focus area
- Dynamic elements should have animated transitions
- Colors should have high contrast (red = danger, green = safe)
- Support colorblind modes
