# Galgame Creation Examples

> Detailed character type explanations, dialogue generation examples, and art direction examples

## Complete Character Type Definitions

### Tsundere

```json
{
  "type_id": "tsundere",
  "core_trait": "Says one thing but means another; the more she likes you, the more awkward she gets",
  "surface_vs_private": {
    "public": "Sharp-tongued, cold, \"It's not like I did it for you\"",
    "private": "Secretly caring, preparing things, heart racing"
  },
  "speech_pattern": {
    "signature_phrases": ["I-it's not like that!", "Don't get the wrong idea!", "Hmph, do what you want", "Idiot......"],
    "progression": {
      "lv0": "\"Who are you? Go away.\"",
      "lv3": "\"It's not like I was worried about you...... I just happened to be passing by!\"",
      "lv5": "\"......Only you get to see me like this. Idiot.\""
    }
  },
  "gap_moe_triggers": ["Caught secretly caring", "Flustered upon receiving a gift", "Jealous but won't admit it"],
  "bond_reactions": {
    "praise": "\"Hmph, of course I can do this much!\" (ears turning red)",
    "jealousy": "\"You seem pretty close with that guy...... whatever, do what you want!\"",
    "affection": "\"......You can stay a little longer today. It doesn't mean anything!\""
  }
}
```

### Kuudere

```json
{
  "type_id": "kuudere",
  "core_trait": "Icy beauty, clumsy at expressing emotion",
  "surface_vs_private": {
    "public": "Expressionless, few words, seems disinterested",
    "private": "A lot going on inside, doesn't know how to express it"
  },
  "speech_pattern": {
    "signature_phrases": ["......Mm", "......Whatever", "(silence)"],
    "progression": {
      "lv0": "\"......\" (glances at you and walks off)",
      "lv3": "\"......You came.\" (actually been waiting a long time)",
      "lv5": "\"......I wanted to see you.\" (turns away after saying it, ears red)"
    }
  },
  "design_notes": "The charm lies in her 'rare changes in expression' — design a clear melting process"
}
```

### Koakuma (Little Devil)

```json
{
  "type_id": "koakuma",
  "core_trait": "Deliberately teasing, enjoys the thrill of the chase",
  "surface_vs_private": {
    "public": "Actively flirtatious, keeps you at arm's length",
    "private": "Actually shy when things get serious, afraid of being taken seriously"
  },
  "speech_pattern": {
    "signature_phrases": ["Missed me?", "Blushing already~", "Gotcha"],
    "progression": {
      "lv0": "\"Ehh~ you look interesting~\"",
      "lv3": "\"Strange...... why is my heart racing now?\"",
      "lv5": "\"Don't look at me with those serious eyes...... it's embarrassing......\""
    }
  },
  "design_notes": "The twist is that 'the hunter becomes the hunted'"
}
```

### Yandere

```json
{
  "type_id": "yandere",
  "core_trait": "Loves to an extreme, intensely possessive",
  "surface_vs_private": {
    "public": "Gentle, perfect, thoughtful",
    "private": "Intense possessiveness, can't tolerate a third party"
  },
  "speech_pattern": {
    "signature_phrases": ["As long as I have you, that's enough", "We'll be together forever", "Only look at me"],
    "progression": {
      "lv0": "\"Hi~ I'm your new classmate.\" (smiling)",
      "lv3": "\"Who did you talk to today? Tell me.\"",
      "lv5": "\"From now on, everything about you belongs to me.\""
    }
  },
  "design_notes": "Balance love and madness — avoid turning her into a purely horror character"
}
```

---

## Situational Dialogue Examples

### Jealousy Scene

```
[Tsundere] "You seem pretty close with that person...... whatever, do what you want!" (turns to leave)
[Kuudere] "......That person." (silence, but the mood turns cold)
[Natural Airhead] "......You two seem close." (doesn't understand why she feels sad)
[Yandere] "Who is that person?" (smiling, but her eyes aren't)
[Genki] "Hey hey! I want in too!" (masking her unease)
[Haraguro] "How nice~" (secretly making a mental note of that person)
```

### Confession Scene

```
[Tsundere] "I-it's not a confession! I just...... don't dislike you, that's all...... idiot"
[Kuudere] "......I like you." (turns away right after, ears bright red)
[Natural Airhead] "I've been thinking about it for a long time...... I like you the most! Is that okay?"
[Genki] "I like you! ......Eh, did I just say that out loud?"
[Koakuma] "......This isn't a joke this time. I'm serious." (blushing)
[Haraguro] "You've finally fallen into my hands." (but her eyes are gentle)
```

### Late for a Date Scene

```
[Tsundere] "You're 30 minutes late...... it's not like I was waiting long!" (actually arrived an hour early)
[Kuudere] "......It's fine." (actually minds a lot but won't say so)
[Natural Airhead] "I thought you weren't coming...... I'm so glad!" (too happy to remember to be upset)
[Genki] "You're finally here! Come on, let's go!" (grabs your hand and runs)
[Yandere] "Where were you these 30 minutes? What were you doing? Who were you with?"
```

### Head Pat Scene

```
[Tsundere] "W-what are you doing......" (but doesn't pull away)
[Kuudere] (expressionless, but leans in a little closer)
[Natural Airhead] "That feels nice~ pat me again!"
[Genki] "Ehehe~" (happily nuzzling into the pat)
[Koakuma] "Usually I'm the one patting you...... the roles are reversed now" (blushing)
```

---

## CG Direction Examples

### Confession Scene CG

```yaml
cg_spec:
  scene_id: confession_sakura
  title: Confession Under the Cherry Blossoms

  composition:
    type: two-character
    focus: heroine's expression
    angle: slight low angle, emphasizing the heroine standing on higher ground

  characters:
    - id: heroine_01
      pose: hands clasped in front of chest
      expression: embarrassed
      action: looking down, avoiding eye contact

    - id: protagonist
      pose: standing, facing the heroine
      expression: earnest
      action: reaching out a hand

  environment:
    location: cherry blossom tree behind the school
    time: evening (sunset)
    weather: clear, cherry blossom petals falling

  atmosphere:
    mood: romantic, tense
    lighting: warm orange sunset side-light
    color_tone: pink-orange warm tones

  key_elements:
    - falling cherry blossom petals
    - sunset glow
    - the heroine's flushed cheeks
    - the sense of distance between the two (about to be closed)
```

### Everyday Scene CG

```yaml
cg_spec:
  scene_id: cooking_together
  title: Cooking Together

  composition:
    type: two-character
    focus: interaction
    angle: side view, eye level

  characters:
    - id: heroine_02
      pose: wearing an apron, holding a spatula
      expression: smile
      action: turning her head toward the protagonist

    - id: protagonist
      pose: standing beside her
      expression: smiling
      action: handing over ingredients

  environment:
    location: home kitchen
    time: afternoon
    weather: indoor

  atmosphere:
    mood: warm, everyday
    lighting: natural indoor light
    color_tone: warm tones

  key_elements:
    - the homely feel of the apron
    - rising steam
    - eyes meeting
    - a sense of everyday happiness
```

---

## Classic Character Combination Examples

### Standard Harem Lineup (4 Characters)

| Character | Combination Formula | Role |
|------|----------|------|
| A | Tsundere + Childhood Friend + Twintails | Standard heroine |
| B | Kuudere + Upperclassman + Student Council President + Long Black Hair | Unattainable beauty |
| C | Natural Airhead + Underclassman + Genki | Healing-type little sister figure |
| D | Haraguro + Young Lady | Contrast-type character |

**Design focus**:
- A and B form a "lively vs. calm" contrast
- C and D form a "innocent vs. calculating" contrast
- A and C compete for "everyday companionship" screen time
- B and D compete for "mysterious appeal" screen time

### Love Triangle Configuration

```
        Protagonist
       /    \
      /      \
 Character A ---- Character B
 (Childhood Friend)  (Transfer Student)

A: Tsundere, known him since childhood, too familiar to confess
B: Kuudere, mysterious transfer student, disrupts the routine

Conflict design:
- A is jealous that B can approach the protagonist naturally
- B actually envies the memories A shares with the protagonist
- The protagonist is caught between the two
```
