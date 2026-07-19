---
schema: "1.0"
name: storytelling
version: "1.0.0"
description: Story creation: novel writing, comic scripts, video planning, narrative structure, character development
domain: creative
triggers:
  keywords:
    primary: [novel, story, writing, screenplay, narrative]
    secondary: [manga, comic, character, plot, worldbuilding, dialogue]
  context_boost: [creative, imagination, literature]
  context_penalty: [code, api, database, technical]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Storytelling

> Craft compelling stories that readers/viewers can't put down

## Use Cases

- Novel/short story writing
- Comic scripts and panel layout
- Video/short film screenplays
- Game narrative design
- Brand storytelling marketing

## Story Structure

### Three-Act Structure

```
┌─────────────────────────────────────────────────────────────────┐
│  Three-Act Structure                                             │
│                                                                 │
│  Act One (25%)       Act Two (50%)        Act Three (25%)       │
│  Setup              Confrontation        Resolution            │
│  ┌─────────┐       ┌─────────────┐       ┌─────────┐           │
│  │ Establish│       │  Conflict   │       │ Resolve │           │
│  │ ordinary │       │  Escalating │       │ Climax  │           │
│  │ world    │       │  difficulty │       │         │           │
│  │ Inciting │       │  Midpoint   │       │ New     │           │
│  │ incident │       │  turn       │       │ balance │           │
│  └────┬────┘       └──────┬──────┘       └────┬────┘           │
│       │                   │                   │                 │
│  ─────●───────────────────●───────────────────●─────→           │
│    Inciting          Midpoint            Climax                │
│    Event             Turning Point       Climax                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Hero's Journey

```markdown
## The 12-Stage Hero's Journey

### Departure
1. **Ordinary World** - Everyday life
2. **Call to Adventure** - Inciting incident
3. **Refusal of the Call** - Hesitation and fear
4. **Meeting the Mentor** - Gaining guidance
5. **Crossing the Threshold** - Entering the new world

### Initiation
6. **Tests, Allies, Enemies** - Making allies, facing challenges
7. **Approach to the Inmost Cave** - Nearing the core crisis
8. **Ordeal** - The greatest trial
9. **Reward** - Gaining a treasure/power

### Return
10. **The Road Back** - Returning on the way home
11. **Resurrection** - The final test
12. **Return with the Elixir** - A transformed new life
```

### Story Spine

```markdown
## Pixar Story Spine

Once upon a time there was ___.
Every day, ___.
One day ___.
Because of that, ___.
Because of that, ___.
Because of that, ___.
Until finally ___.
And ever since then ___.

Example:
Once upon a time there was a shy robot named WALL-E.
Every day, he cleaned up trash all alone.
One day, a robot named EVE arrived.
Because of that, WALL-E fell in love with her and followed her onto a spaceship.
Because of that, he accidentally activated a plan to return to Earth.
Because of that, humanity finally returned to Earth.
Until finally, WALL-E and EVE were together.
And ever since then, Earth began to recover.
```

## Character Development

### Character-Building Elements

```
┌─────────────────────────────────────────────────────────────────┐
│  Character Iceberg Model                                        │
│                                                                 │
│  ~~~~~~~~~~~~ Waterline ~~~~~~~~~~~~                             │
│                                                                 │
│  Visible Layer (external presentation)                          │
│  ├─ Appearance, clothing, way of speaking                       │
│  ├─ Behavioral habits, skills                                   │
│  └─ Occupation, social status                                   │
│                                                                 │
│  ~~~~~~~~~~~~ Below the Surface ~~~~~~~~~~~~                    │
│                                                                 │
│  Hidden Layer (internal motivation)                             │
│  ├─ Fears and desires                                          │
│  ├─ Past trauma                                                 │
│  ├─ Core beliefs                                                │
│  └─ Fatal Flaw                                                  │
│                                                                 │
│  A good character = external goal + internal need + obstacle    │
└─────────────────────────────────────────────────────────────────┘
```

### Character Profile Sheet

```markdown
## Character Profile

### Basic Information
- Name:
- Age:
- Occupation:
- Physical description:

### Inner Dimensions
- **Want (external goal)**: What do they want?
- **Need (internal need)**: What do they truly need?
- **Flaw (fatal flaw)**: What holds them back?
- **Ghost (past trauma)**: What happened in the past?

### Personality
- 3 positive traits:
- 3 negative traits:
- Catchphrase:
- Habitual gesture:

### Relationship Web
- Relationship to the protagonist:
- Source of conflict:
```

## Dialogue Writing

### Traits of Good Dialogue

| Trait | Description | Example |
|------|------|------|
| **Subtext** | Meaning beneath the words | "Nice weather" (wanting to end the conversation) |
| **Conflict** | Opposing character goals | A wants to leave, B wants to keep them there |
| **Individualization** | Reflects character traits | A professor's speech vs. a child's speech |
| **Advances the plot** | Every line has a purpose | Reveals information or changes a relationship |

### Dialogue Techniques

```markdown
## Principles of Dialogue Writing

1. **Less is more**
   ❌ "I went to the supermarket this morning and bought milk and bread."
   ✅ "The fridge is empty."

2. **Action instead of explanation**
   ❌ "I'm so angry!" he said angrily.
   ✅ He slammed the door.

3. **Interruption and silence**
   - Interruption = assertive/urgent
   - Silence = thinking/resistance

4. **Indirect answers**
   "Do you love me?"
   ❌ "Yes, I love you."
   ✅ "Your coffee's gone cold."
```

## Comic Scripts

### Panel Design

```
┌─────────────────────────────────────────────────────────────────┐
│  Basic Comic Panel Format                                        │
│                                                                 │
│  Page: P.XX                                                     │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Panel 1 (large)                                          │   │
│  │ Shot: Long shot                                          │   │
│  │ Description: City skyline, sunset                        │   │
│  │ Dialogue: (none)                                         │   │
│  │ Effect: Establishes mood                                 │   │
│  ├─────────────────────┬───────────────────────────────────┤   │
│  │ Panel 2              │ Panel 3                           │   │
│  │ Shot: Medium shot     │ Shot: Close-up                    │   │
│  │ Description: Protagonist walking on the street │ Description: Protagonist's eyes │   │
│  │ Dialogue: ...        │ Dialogue: "Finally here."          │   │
│  └─────────────────────┴───────────────────────────────────┘   │
│                                                                 │
│  Shot type reference:                                           │
│  Long shot → Establishes the setting                           │
│  Full shot → Shows a character's full body                     │
│  Medium shot → Dialogue scenes                                  │
│  Close-up → Emphasizes emotion                                  │
│  Extreme close-up → Object or expression detail                 │
└─────────────────────────────────────────────────────────────────┘
```

### Comic Pacing

```markdown
## Pacing Control

### Speeding Up (tension/action)
- Panels become smaller and more numerous
- Diagonal compositions
- Speed lines
- Minimal dialogue

### Slowing Down (emotion/reflection)
- Panels become larger
- Negative space
- Close-ups on expressions
- Internal monologue

### Page-Turn Surprises
- Place the key image right after a page turn
- Place suspense in the last panel of the right-hand page
```

## Video Scripts

### Script Format

```markdown
## Short Film Script Template

Scene 1 - Interior/Coffee Shop/Day

[Description]
Sunlight pours in through the floor-to-ceiling windows. Mei (25) sits alone in the corner,
staring at her phone screen.

[Action]
Mei scrolls through her phone, her expression shifting from anticipation to disappointment.

[Dialogue]
Mei: (to herself) Read and ignored again...

[Transition]
Close-up on the phone screen, showing "typing..."

---

Scene 2 - Exterior/Street/Day

...
```

### Video Structure

| Type | Duration | Structure |
|------|------|------|
| **Short-form video** | 15-60s | Hook → Content → CTA |
| **YouTube** | 8-15min | Hook → Problem → Solution → CTA |
| **Short film** | 5-15min | Condensed three-act structure |
| **Micro-film** | 15-40min | Full three-act structure |

## Worldbuilding

### Worldbuilding Elements

```markdown
## Worldbuilding Checklist

### Foundational Setup
- [ ] Time period (past/present/future)
- [ ] Geographic environment
- [ ] Level of technology
- [ ] Magic/superpower system (if any)

### Social Structure
- [ ] Political system
- [ ] Economic system
- [ ] Social hierarchy
- [ ] Cultural customs

### Rules and Limitations
- [ ] What is possible?
- [ ] What is forbidden?
- [ ] Consequences of breaking the rules?

### History
- [ ] Major historical events
- [ ] How do they affect the present?
```

## Common Problems

| Problem | Solution |
|------|----------|
| Slow opening | Start with action/conflict |
| Boring middle | Add twists and obstacles |
| Flat characters | Add flaws and contradictions |
| Weak ending | Echo the opening, complete the character arc |
| Stiff dialogue | Add subtext, reduce on-the-nose statements |

## Recommended Tools

- **Writing**: Scrivener, Ulysses, Yuque
- **Outlining**: Notion, Workflowy, Mubu
- **Comics**: Clip Studio Paint, Procreate
- **Scripts**: Final Draft, WriterSolo
- **Characters**: Campfire, World Anvil

## Related Resources

- [Story - Robert McKee](https://mckeestory.com/)
- [Save the Cat - Blake Snyder](https://savethecat.com/)
- [The Writer's Journey - Christopher Vogler](https://www.thewritersjourney.com/)
- [Understanding Comics - Scott McCloud](https://en.wikipedia.org/wiki/Understanding_Comics)
