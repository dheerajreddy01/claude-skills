# Game Design Checklists

## level-design-checklist

### Tutorial Design
- [ ] New mechanics have a safe environment to practice in
- [ ] Complex mechanics are introduced step by step
- [ ] Show, don't tell

### Pacing Control
- [ ] Tension and relaxation alternate
- [ ] Exploration and challenge are balanced
- [ ] Appropriate checkpoints

### Readability
- [ ] Paths are clearly visible
- [ ] Danger zones have visual cues
- [ ] Objectives are clear

### Replayability
- [ ] Explorable secret areas
- [ ] Multiple routes to completion
- [ ] Collectible elements

## playtest-questionnaire

### Basic Experience
1. Is the game fun? (1-10)
2. Would you want to keep playing?
3. Describe this game in one sentence

### Core Mechanics
4. Is the control scheme intuitive?
5. Was anything confusing?
6. What did you like most?
7. What did you like least?

### Difficulty Perception
8. Is the difficulty appropriate?
9. Were there any places you got stuck?
10. Were there any places that felt too easy or boring?

## playtest-observation

| Observation | Possible Issue |
|------|----------|
| Player pauses | Doesn't know what to do |
| Repeated attempts | Mechanic is unintuitive or too hard |
| Skips tutorial | Tutorial is too wordy |
| Leaves quickly | Core loop isn't engaging |

## economy-balance-checklist

### Economy Balance Check
- [ ] Design effective currency sinks (enhancement failure, repair costs, consumables)
- [ ] Limit daily acquisition caps
- [ ] Use bound currency to reduce circulation
- [ ] Regularly review inflow/outflow ratio (target 1:1)

## combat-balance-checklist

### Combat Numerical Balance
- [ ] Is TTK (Time To Kill) reasonable?
- [ ] Is the gap between high and low levels too large?
- [ ] Is there a risk of numeric overflow?
- [ ] Are all classes/characters balanced?

## anti-cheat-checklist

### Anti-Cheat Design
- [ ] Critical logic runs server-side
- [ ] Input validation
- [ ] Data encryption
- [ ] Anomaly detection

## tutorial-pyramid

### Tutorial Design Pyramid (Top to Bottom)

1. **Forced tutorial** - Only used for the most core mechanics
2. **Prompted guidance** - Visual cues, arrows
3. **Environmental guidance** - Level design hints
4. **Free exploration** - Players discover on their own

### Principles
- If it can be taught through level design, don't use UI to teach it
- If it can be taught through UI, don't use text to teach it
- Let players "do" rather than "watch"
- Failure is also a form of learning (but give clear feedback)

## scrum-dod

### Definition of Done (DoD)
- [ ] Feature complete
- [ ] No known bugs
- [ ] Passed QA testing
- [ ] Code review
- [ ] Documentation updated

### Special Considerations for Game Development
| Challenge | Solution |
|------|----------|
| Hard to estimate work hours | Use story points instead of hours |
| Creativity requires exploration | Reserve "spike" time |
| Cross-department dependencies | Clear deliverable specifications |
| Gameplay validation | Internal testing every sprint |
