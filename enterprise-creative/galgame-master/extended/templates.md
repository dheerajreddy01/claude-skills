# Galgame Creation Templates

> Detailed templates and specification-sheet formats

## Complete Character Setup Template

```json
{
  "character_id": "unique_id",
  "name": "Character Name",
  "age": 17,

  "type_combination": {
    "dere": "type ID",
    "relationship": "type ID",
    "identity": "type ID",
    "special": ["attribute 1", "attribute 2"]
  },

  "appearance": {
    "height": "165cm",
    "hair": "long black hair",
    "eyes": "dark brown",
    "features": ["feature 1", "feature 2"],
    "usual_outfit": "outfit description"
  },

  "personality": {
    "surface": "description of surface impression",
    "private": "true personality in private",
    "gap_moe": ["gap moe point 1", "gap moe point 2"]
  },

  "speech_pattern": {
    "tone": "tone characteristics",
    "signature_phrases": ["catchphrase 1", "catchphrase 2"],
    "progression": {
      "lv0": "typical dialogue at first meeting",
      "lv3": "dialogue once familiar",
      "lv5": "dialogue when intimate"
    }
  },

  "bond_reactions": {
    "praise": "reaction to being praised",
    "jealousy": "reaction when jealous",
    "affection": "way of being affectionate"
  },

  "route_themes": ["route theme 1", "route theme 2"],
  "conflict_source": "source of the character's inner conflict"
}
```

---

## CG Specification Template

```yaml
cg_spec:
  scene_id: unique_id
  title: CG name

  composition:
    type: [close-up/half-body/full-body/two-character/group]
    focus: [focal character or element]
    angle: [angle description]

  characters:
    - id: character ID
      pose: pose description
      expression: expression
      action: action

  environment:
    location: setting
    time: time of day
    weather: weather (if applicable)

  atmosphere:
    mood: [romantic/tense/warm/...]
    lighting: lighting description
    color_tone: color tone

  key_elements:
    - key visual element 1
    - key visual element 2
```

---

## Standard Expression Sheet List

| ID | Expression | Applicable Situation |
|----|------|----------|
| neutral | neutral | everyday dialogue |
| smile | smile | friendly interaction |
| laugh | laugh | comedic scene |
| sad | sad | sentimental moment |
| angry | angry | conflict scene |
| embarrassed | embarrassed | flirtatious interaction |
| surprised | surprised | sudden event |
| crying | crying | moving/sad moment |
| love | heart eyes | romance scene |
| special_1 | character-specific 1 | designed per character |
| special_2 | character-specific 2 | designed per character |

---

## Script Structure Template

### Common Route Chapters

```yaml
common_route:
  prologue:
    title: Prologue
    duration: about 30 minutes
    goals:
      - introduce the protagonist and the setting
      - first-impression event (for each character)
      - establish the everyday tone

  chapter_1:
    title: Everyday Life Arc
    duration: about 2 hours
    goals:
      - deepen impressions of each character
      - establish the foundation of relationships
      - plant seeds for individual routes

  chapter_2:
    title: Incident Arc
    duration: about 2 hours
    goals:
      - trigger each character's key event
      - build up affinity/favorability
      - prepare for the branch choice

  branch_point:
    title: Branch Point
    timing: at the end of the common route
    mechanism: affinity level + key choice
```

### Individual Route Chapters

```yaml
personal_route:
  character_id: character ID

  deep_chapter:
    title: Deep Dive Arc
    goals:
      - reveal the character's background
      - show her inner world
      - establish exclusive interactions

  conflict_chapter:
    title: Conflict Arc
    goals:
      - core conflict erupts
      - relationship crisis
      - opportunity for character growth

  resolution_chapter:
    title: Resolution Arc
    goals:
      - overcome the difficulty together
      - relationship deepens
      - prepare for the ending

  endings:
    true_end:
      condition: max affinity + correct choices
      theme: the most perfect ending
    good_end:
      condition: high affinity
      theme: happy but with some regret
    normal_end:
      condition: moderate affinity
      theme: a quiet, understated ending
    bad_end:
      condition: wrong choices
      theme: a regretful ending
```

---

## Design Checklist

### Character Design
- [ ] Type combination selected (Dere + relationship + identity + special)
- [ ] Basic information complete
- [ ] Has a "surface vs. private" contrast
- [ ] Has Lv0-Lv5 affinity progression
- [ ] Has a distinct speech_pattern
- [ ] Has gap_moe_triggers
- [ ] Has bond_reactions

### Script Structure
- [ ] Common route complete
- [ ] Individual routes for each character
- [ ] Multiple endings designed
- [ ] Key branch points clearly defined

### Art Assets
- [ ] Character sprite direction
- [ ] Expression sheet list
- [ ] Key CG specification sheets
- [ ] Background list

### Multi-Character Balance
- [ ] Has contrasting pairs
- [ ] Balanced screen time
- [ ] Love-triangle dynamics (if applicable)
