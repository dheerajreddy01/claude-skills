# Game Design Templates

## GDD-template

```markdown
# Game Design Document

## 1. Overview
- Game title
- Genre, platform
- Target audience
- Core experience (one sentence)

## 2. Core Mechanics
- Main gameplay
- Controls
- Core loop

## 3. Progression System
- Growth curve
- Unlock mechanics
- Long-term goals

## 4. Numerical Systems
- Resource types
- Balance formulas
- Economy model

## 5. Content Planning
- Level design
- Enemies/NPCs
- Items/equipment
```

## RPG-system-design

```markdown
## RPG System Design Checklist

### Character Progression
- [ ] Level system (balanced experience acquisition)
- [ ] Skill tree design (meaningful choices)
- [ ] Equipment system (stats vs. appearance)
- [ ] Class/job change system

### Combat System
- [ ] Turn-based vs. real-time vs. ATB
- [ ] Elemental/type advantage system
- [ ] Skill combos/chains
- [ ] Status effect design

### Worldbuilding
- [ ] Main story structure
- [ ] Side quest design
- [ ] Depth of NPC interaction
- [ ] Exploration reward mechanics

### Numerical Balance
| Stage | Player HP | Enemy HP | Combat Duration |
|------|---------|---------|----------|
| Early | 100 | 30-50 | 10-30 sec |
| Mid | 500 | 200-400 | 30-60 sec |
| Late | 2000 | 1000+ | 1-3 min |
| Boss | - | 5-10x player | 3-10 min |
```

## roguelike-design

```markdown
## Roguelike Core Design

### Randomness Balance
| Element | Degree of Randomness | Description |
|------|----------|------|
| Map layout | High | Procedurally generated |
| Enemy composition | Medium | Increasing difficulty |
| Item drops | Medium-High | Build diversity |
| Boss mechanics | Low | Learnable patterns |
| Core mechanics | None | Player-masterable |

### Meta Progression Design
| Within a Run | Across Runs (Permanent) |
|--------|-------------|
| Gain power-ups | Unlock new characters/weapons |
| Temporary ability boosts | Permanent stat bonuses |
| Resource accumulation | Increased starting resources |
| In-run sense of growth | Long-term goals + variety |

### Build Design
- Each item is independently useful
- Item combinations create synergy
- Extreme builds should be viable but risky
- Avoid "must-pick" items (all options should have value)
```

## action-game-design

### Core Elements of Action Games

**Controller feel**: 60fps, input latency < 100ms

**Hit feedback formula**:
1. Attack lands -> hitstop for 2-4 frames
2. Play hit sound effect + particle effects simultaneously
3. Enemy plays hurt animation + knockback
4. Slight camera shake (optional)

**Enemy design**: Clear attack telegraphs, readability

## puzzle-game-design

### Puzzle Design Principles

1. **Progressive complexity**: simple -> variation -> combination -> inversion
2. **Fairness**: all clues are provided before the puzzle, the solution follows logic, avoid "pixel hunting"
3. **"Aha" moment**: difficulty = degree of information hidden x number of logical steps

### Hint System
- Level 1: directional hint, "pay attention to that corner"
- Level 2: specific hint, "try moving that box"
- Level 3: direct answer, "push the box onto the switch"

## mobile-game-design

### Mobile Game Design Considerations

**Input constraints**: prioritize one-handed play, avoid overly complex virtual joysticks, tapping preferred over swiping

**Session length**: 3-5 minutes per session (fragmented), can pause/save anytime, offline reward mechanics

### Monetization Models
| Model | Description |
|------|------|
| Premium/buy-to-play | One-time payment |
| In-app purchases (IAP) | Items, cosmetics, VIP |
| Ads | Rewarded ads, interstitial ads |
| Subscription | Monthly membership perks |
| Battle Pass | Seasonal pass |

**Monetization design principles**: paying != winning, free players should also have a good experience, paid content should have clear perceived value

## narrative-structure

### Types of Narrative Structure

1. **Linear narrative**: A -> B -> C -> D -> ending (tight story, low development cost, low replay value)
2. **Branching narrative**: multiple paths to multiple endings (players feel a sense of choice, large content development volume)
3. **Convergent branching**: branches converge back to the main line (sense of choice but convergence, players may feel choices are meaningless)
4. **Open-world narrative**: main line + multiple parallel side quests, players freely choose the order

### Dialogue System Design

**Good options**: distinct personality, clear consequences, reflect player intent

**Avoid**: three options saying the same thing, false choices, option text not matching the actual dialogue

**Dialogue pacing**: single line of dialogue <= 2 lines, important information should have visual emphasis, allow skipping but preserve important information

## multiplayer-design

### Network Model Comparison

| Model | Description | Suitable For | Latency Tolerance |
|------|------|------|----------|
| P2P | Direct player connection | Fighting games, racing | Low |
| Client-Server | Central server | MMO, FPS | Medium-High |
| Rollback | Prediction + rollback | Fighting games | High |
| Lockstep | Synchronized input | RTS | Low |

### Latency Compensation Techniques
- Client-side prediction (execute locally first)
- Server authoritative
- Interpolation
- Lag compensation (hit detection rollback)

### Matchmaking System (Elo/MMR)

**Formula**: new rating = old rating + K x (actual result - expected result)

**K value**: new players K=32 (fast adjustment), veteran players K=16 (stable)

**Protection mechanisms**: new player protection, losing streak protection, rank protection

## live-service-design

### Content Update Cadence

| Cycle | Content |
|------|------|
| Weekly update | Daily quest refresh, weekly events, shop rotation |
| Monthly update | New characters/weapons, limited-time events, Battle Pass season |
| Quarterly update | Major version updates, new game modes, main storyline |

### Operational Metrics (KPIs)

**User metrics**:
- DAU/MAU ratio: healthy range 20-30%
- D1/D7/D30 retention: 40%/20%/10%
- Stickiness: DAU/MAU > 20%

**Revenue metrics**:
- ARPU = total revenue / total users
- ARPPU = total revenue / paying users
- LTV = ARPU x lifetime

**Health formula**: LTV > CPI x 3

## development-workflow

### Game Development Lifecycle

| Phase | Duration | Output |
|------|------|------|
| Concept phase | 1-2 weeks | Game concept document, market research, feasibility assessment |
| Prototype phase | 2-4 weeks | Core mechanic prototype, gameplay validation, technical risk assessment |
| Pre-production | 1-3 months | Complete GDD, art style, vertical slice |
| Production | 6-18 months | Content production, iterative refinement, Alpha/Beta |
| Post-production | 1-2 months | Debugging, optimization, certification |
| Post-launch | Ongoing | Live operations, content updates, community management |

### Agile Development (Scrum)

**Sprint cycle**: 2 weeks recommended, each sprint should produce a playable output

**DoD (Definition of Done)**: feature complete, no known bugs, passed QA, code review, documentation updated
