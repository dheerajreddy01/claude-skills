---
schema: "1.0"
name: game-design
version: "1.0.0"
description: Game design theory, mechanics design, and player experience
domain: creative
triggers:
  keywords:
    primary: [game design, game planning, GDD, level design, player experience]
    secondary: [balance, MDA, game loop, roguelike, RPG]
  context_boost: [game, gameplay, mechanics]
  context_penalty: [marketing, finance]
  priority: high
dependencies:
  software-skills: [game-development]
author: claude-domain-skills
---

# Game Design

> Create fun and immersive game experiences

## Applicable Scenarios

- Game concept design and GDD writing
- Core mechanics design and iteration
- Numerical system design and balancing
- Level design and pacing control
- Player experience optimization

## MDA Framework

**Mechanics -> Dynamics -> Aesthetics** (designer's perspective -> player's perspective)

- Jump + platform -> timing challenge -> sense of accomplishment, challenge
- Resources + trading -> market dynamics -> strategy, social interaction

## Core Knowledge

### 8 Kinds of Aesthetic Experience

| Aesthetic | Description | Game Examples |
|------|------|----------|
| Sensation | Sensory stimulation | Music games, VR |
| Fantasy | Fantasy role-playing | RPG, The Sims |
| Narrative | Story experience | Visual novels, adventure games |
| Challenge | Overcoming challenges | Dark Souls |
| Fellowship | Social interaction | MMO, party games |
| Discovery | Exploration and discovery | Open world |
| Expression | Self-expression | Minecraft |
| Submission | Relaxation and pastime | Farming games |

### Core Loop

**Goal -> Challenge (difficulty curve) -> Action (player input) -> Feedback (reward system) -> Repeat**

### Difficulty Curve

Wave-like difficulty: breathing room after each peak, practice time after new mechanics are introduced, checkpoints before bosses

## Best Practices

1. **Fun first, complete later** - The core mechanic must be validated as fun first
2. **Rapid prototyping** - Build a playable version as early as possible for testing
3. **Playtesting** - Observe player behavior, don't just listen to opinions
4. **Subtractive design** - Cut unnecessary features
5. **Flow design** - Match challenge to ability

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Piling on features | Focus on the core experience |
| Only listening to what players say | Observe what players do |
| Building the complete game from the start | Rapid prototyping and iteration |
| Linear difficulty increase | Wave-like difficulty curve |

## Sharp Edges

### SE-1: Economy Inflation
- **Severity**: critical
- **Situation**: In-game currency loses value over time, players become indifferent to rewards
- **Cause**: Currency inflow > outflow, no effective consumption mechanism (Sink)
- **Symptoms**: Veteran players hoard large amounts of currency, new players struggle to catch up, reward values keep inflating
- **Solution**: Design effective sinks (enhancement failure, repair costs), limit daily acquisition caps, target a 1:1 inflow/outflow ratio
- Full checklist -> [extended/checklists.md#economy-balance-checklist]

### SE-2: Feature Creep
- **Severity**: high
- **Situation**: More and more features are added to the game, but the core experience doesn't improve
- **Cause**: Each feature looks "good" in isolation, but together they cause the game to lose focus
- **Symptoms**: Players don't know "what this game is about," onboarding takes too long, every system feels shallow
- **Solution**: Define the core experience (in one sentence), ask "how does this enhance the core experience?" for every new feature, learn to say "no"

### SE-3: Pay-to-Win Design
- **Severity**: critical
- **Situation**: Paying players gain a clear competitive advantage, harming the free player experience
- **Cause**: Short-term revenue prioritized over long-term retention
- **Symptoms**: Massive free player churn, extremely low game ratings, only "whales" remain
- **Solution**: Limit paid content to cosmetics/convenience, paid acceleration but no skipping, standardize numbers in competitive modes

### SE-4: Numeric Overflow Risk
- **Severity**: high
- **Situation**: Numbers explode in the late game, resulting in "billions of damage" or calculation errors
- **Cause**: Poorly designed growth curve (usually exponential growth)
- **Symptoms**: Damage numbers become unreadable, integer overflow causes bugs, balancing gets harder and harder to tune
- **Solution**: Use logarithmic or soft-cap curves, avoid pure exponential growth
- Code example -> [extended/examples.md#number-overflow-fix]

### SE-5: Ignoring Player Behavior, Only Listening to Player Opinions
- **Severity**: medium
- **Situation**: Design changes are made based on player surveys/forum opinions, but the results are worse
- **Cause**: What players say and what they do are often inconsistent
- **Symptoms**: Players say it's "too hard" but data shows a normal clear rate; requested features get added but nobody uses them
- **Solution**: Observing what players "do" matters more than what they "say," validate hypotheses with data (A/B testing), understand the real problem behind the complaint

## Numerical Design Quick Reference

**Growth curves**: logarithmic for player attributes, exponential for leveling experience, S-curve for content unlocks
- Full formulas -> [extended/examples.md#growth-curve-formulas]

**Damage calculation**: base damage = attack power x (100 / (100 + defense))
- Detailed example -> [extended/examples.md#damage-calculation]

## Player Psychology

### Behavioral Incentives

| Incentive Type | Description | Application |
|----------|------|------|
| Extrinsic reward | Material payoff | Gold, equipment, experience |
| Intrinsic reward | Sense of accomplishment | Overcoming challenges, skill improvement |
| Social reward | Sense of recognition | Leaderboards, achievement showcases |
| Random reward | Sense of surprise | Gacha, drops |

### Hook Model

**Trigger -> Action -> Variable Reward -> Investment -> (loop)**

Example (mobile game): push notification -> log into game -> daily lottery -> accumulated days

## Genre Design Points

| Genre | Core Elements | Detailed Design |
|------|----------|----------|
| Action | Controller feel, hit feedback, enemy readability | [extended/templates.md#action-game-design] |
| RPG | Character progression, combat system, worldbuilding | [extended/templates.md#RPG-system-design] |
| Roguelike | Randomness balance, meta progression, build design | [extended/templates.md#roguelike-design] |
| Puzzle | Progressive complexity, fairness, hint system | [extended/templates.md#puzzle-game-design] |
| Mobile | Input constraints, session fragmentation, monetization design | [extended/templates.md#mobile-game-design] |

## Level Design and Narrative

- Level design checklist -> [extended/checklists.md#level-design-checklist]
- Tutorial design pyramid -> [extended/checklists.md#tutorial-pyramid]
- Narrative structure types -> [extended/templates.md#narrative-structure]

## Multiplayer Games

- Network model comparison -> [extended/templates.md#multiplayer-design]
- Anti-cheat checklist -> [extended/checklists.md#anti-cheat-checklist]

## Testing and Operations

- Playtest questionnaire -> [extended/checklists.md#playtest-questionnaire]
- Observation points -> [extended/checklists.md#playtest-observation]
- Live service design -> [extended/templates.md#live-service-design]
- Development workflow -> [extended/templates.md#development-workflow]

## Templates and Tools

- GDD template -> [extended/templates.md#GDD-template]
- **Engine**: Unity / Unreal
- **Economy simulation**: Google Sheets / Excel
- **UI/UX**: Figma
- **Documentation management**: Notion

## Related Resources

- [GDC Vault](https://www.gdcvault.com/)
- [Game Developer](https://www.gamedeveloper.com/)
- [Game Maker's Toolkit](https://www.youtube.com/@GMTK)

## Related Domains

- [[storytelling]] - Game narrative and script
- [[visual-media]] - Game art and animation
- [[ui-ux-design]] - Game UI/UX
