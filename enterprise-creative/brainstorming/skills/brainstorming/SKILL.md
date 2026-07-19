---
schema: "1.0"
name: brainstorming
version: "1.0.0"
description: Creative ideation: brainstorming, inspiration, creative thinking frameworks, divergent and convergent thinking
domain: creative
triggers:
  keywords:
    primary: [inspiration, brainstorm, ideation, idea, creativity]
    secondary: [innovation, thinking, divergent, convergent, SCAMPER]
  context_boost: [thoughts, create, inspire]
  context_penalty: [code, implementation]
  priority: medium
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Creative Ideation: Brainstorming & Ideation

> A systematic approach to creative thinking, from inspiration to actionable plans

## Use Cases

- New product/feature ideation
- Solution brainstorming
- Breaking through creative blocks
- Team creative workshops
- Personal inspiration

## Creative Thinking Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  Creative Thinking Process                                      │
│                                                                 │
│  Divergent Thinking → Explore → Convergent Thinking             │
│                                                                 │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐       │
│  │  Diverge    │     │  Explore    │     │  Converge   │       │
│  │   ────      │     │   ────      │     │   ────      │       │
│  │ • Quantity  │ ──→ │ • Combine   │ ──→ │ • Evaluate  │       │
│  │   first     │     │   ideas     │     │   & filter  │       │
│  │ • No        │     │ • Analogical│     │ • Prioritize│       │
│  │   judgment  │     │   thinking  │     │             │       │
│  │ • Seek      │     │ • Reverse   │     │ • Feasibility│      │
│  │   novelty   │     │   thinking  │     │   analysis  │       │
│  └─────────────┘     └─────────────┘     └─────────────┘       │
│                                                                 │
│  Rule: Diverge first, then converge — never both at once!       │
└─────────────────────────────────────────────────────────────────┘
```

## Brainstorming Methods

### Classic Brainstorming

```markdown
## Brainstorming Rules

1. **Defer judgment** - Don't criticize any idea
2. **Quantity first** - Aim for as many ideas as possible
3. **Welcome wild ideas** - The bolder, the better
4. **Combine and improve** - Build on others' ideas

## Process

1. Clearly define the problem (5 min)
2. Individual silent ideation (5 min)
3. Take turns sharing without commentary (15 min)
4. Free association and extension (10 min)
5. Categorize and organize (5 min)
6. Vote and select (5 min)
```

### SCAMPER Technique

| Letter | Technique | Question |
|------|------|------|
| **S** | Substitute | What materials/people/processes could be substituted? |
| **C** | Combine | What could this be combined with? |
| **A** | Adapt | What could be borrowed from elsewhere? |
| **M** | Modify | What could be magnified/minimized/changed? |
| **P** | Put to other uses | Could this be used elsewhere? |
| **E** | Eliminate | What could be removed? |
| **R** | Reverse | Could this be reversed/rearranged? |

### Six Thinking Hats

```
┌─────────────────────────────────────────────────────────────────┐
│  Six Thinking Hats                                              │
│                                                                 │
│  🎩 White Hat - Facts & data       "What do we know?"           │
│  🔴 Red Hat - Emotions & intuition "My feeling is..."           │
│  ⚫ Black Hat - Critical risks     "The possible problems are..."│
│  🟡 Yellow Hat - Optimism & value  "The benefits are..."        │
│  🟢 Green Hat - Creative possibility "What if...?"              │
│  🔵 Blue Hat - Process control     "Our goal is..."              │
│                                                                 │
│  Usage: Switch perspectives in order to avoid mental clutter    │
└─────────────────────────────────────────────────────────────────┘
```

## Inspiration Techniques

### Random Stimulus Method

```markdown
1. Randomly pick a word (nouns work best)
2. List that word's characteristics
3. Force a connection to the problem
4. Generate new ideas

Example:
Problem: How to increase user engagement?
Random word: Tree
Tree characteristics: growth, branching, seasonal change, provides shade
Connected ideas:
- Grows like a tree → user leveling system
- Branching → diverse interaction methods
- Seasonal change → limited-time events
- Shade → mutual-support community
```

### Analogical Thinking

| Domain | Analogy Question |
|------|----------|
| **Nature** | How does nature solve this problem? |
| **Other industries** | How do hotels/airlines/gaming handle this? |
| **History** | Have there been similar challenges in history? |
| **Art** | How would an artist approach this? |

### Reverse Thinking

```markdown
Forward question: How do we make users more satisfied?
Reverse question: How do we make users as dissatisfied as possible?

List ways to make users dissatisfied:
1. Slow responses
2. Complicated interface
3. Hidden features
4. No help documentation

Flip into solutions:
1. Real-time response system
2. Clean interface
3. Prominent features
4. Comprehensive help center
```

## Creative Evaluation Matrix

### Feasibility vs Impact

```
Impact
   ↑
High │ ⭐ Quick Wins │ 🚀 Big Bets │
   │  Do first     │  Strategic  │
   │               │  investment │
   │───────────────┼─────────────│
Low  │ 🗑️ Discard   │ ❓ Needs    │
   │              │   validation │
   │              │   / prototype│
   └───────────────┴─────────────→
   Low             High     Feasibility
```

### Evaluation Criteria

| Dimension | Scoring Item |
|------|----------|
| **Novelty** | Does it offer a unique perspective? |
| **Feasibility** | Achievable with available technology/resources? |
| **Value** | Does it solve a real problem? |
| **Timeliness** | Is now a good time? |

## Creative Workshop Process

```markdown
## 90-Minute Creative Workshop

### Warm-up (10 min)
- Icebreaker activity
- Introduce the rules

### Problem Definition (10 min)
- HMW (How Might We) format
- Example: "How might we help new users onboard faster?"

### Divergent Phase (30 min)
- Individual ideation (10 min)
- SCAMPER extension (10 min)
- Analogical borrowing (10 min)

### Exploration Phase (15 min)
- Combine similar ideas
- Extend promising ones

### Convergent Phase (20 min)
- Dot voting (3 votes per person)
- Discuss the top three
- Select the plan to execute

### Action Plan (5 min)
- What's next?
- Who's responsible?
```

## Personal Inspiration Log

### Inspiration Note Template

```markdown
## 💡 Inspiration Log

**Date**:
**Trigger**: (what you saw/heard/thought of)
**Idea**:
**Possible application**:
**Related links**:
**Rating**: ⭐⭐⭐⭐⭐ (potential)
```

### Sources of Inspiration

| Type | Source |
|------|------|
| **Reading** | Books, articles, papers |
| **Observation** | Daily life, user behavior |
| **Conversation** | Colleagues, friends, customers |
| **Cross-domain** | Other industries, art, science |
| **Retrospective** | Past successes/failures |

## Breaking Through Creative Blocks

```markdown
## Emergency Kit for When You're Stuck

1. **Change environment** - Leave your desk, take a walk
2. **Change perspective** - What would a newcomer think?
3. **Change the question** - Redefine the problem
4. **Change constraints** - Adding limits can spark creativity
5. **Sleep on it** - Let your subconscious work

## Common Obstacles

| Obstacle | Solution |
|------|------|
| Fear of criticism | Defer-judgment rule |
| Perfectionism | Quantity before quality |
| Rigid thinking | Break it with random stimulus |
| Lack of information | Research first, then ideate |
```

## Advanced Creative Techniques

### Design Thinking

```
┌─────────────────────────────────────────────────────────────────┐
│  Design Thinking: Five Stages                                   │
│                                                                 │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌────────┐│
│  │ Empathize│→│ Define  │→│ Ideate  │→│Prototype│→│  Test  ││
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └────────┘│
│                                                                 │
│  • Empathize: Observe users, interview, experience               │
│  • Define: Define the problem, uncover insights                 │
│  • Ideate: Generate ideas broadly, without limits                │
│  • Prototype: Build prototypes quickly                          │
│  • Test: User testing, iterate and improve                       │
│                                                                 │
│  Traits: nonlinear, can loop back, human-centered                 │
└─────────────────────────────────────────────────────────────────┘
```

### Morphological Matrix

```markdown
## Morphological Matrix Example: Designing a New Beverage

| Dimension | Option A | Option B | Option C | Option D |
|------|--------|--------|--------|--------|
| **Base** | Tea | Coffee | Juice | Sparkling water |
| **Flavor** | Sweet | Sour | Bitter | Salty |
| **Temperature** | Iced | Room temp | Hot | Slush |
| **Container** | Cup | Bottle | Can | Pouch |
| **Occasion** | Breakfast | Afternoon tea | Post-workout | Before bed |

Random combination: Coffee + Salty + Iced + Pouch + Post-workout
→ A salty iced coffee sports drink?

Systematic generation: 4×4×4×4×4 = 1024 combinations
```

### TRIZ

```markdown
## TRIZ's 40 Inventive Principles (Selected)

| # | Principle | Applied Question |
|---|------|----------|
| 1 | Segmentation | How can a large object become flexible? |
| 2 | Extraction | What part could be separated out? |
| 10 | Prior action | How can you prepare in advance? |
| 13 | Reversal | What if you did the opposite? |
| 17 | Dimensionality change | Go from 2D to 3D? Flat to solid? |
| 25 | Self-service | How can the system maintain itself? |
| 35 | Parameter change | What if you changed a physical state? |
| 40 | Composite materials | Combine the strengths of different materials? |

## Contradiction Matrix
When improving A worsens B, consult the matrix to find a principle.
Example: increasing strength vs reducing weight → composite materials, hollow structures
```

### Mind Map Divergence

```
┌─────────────────────────────────────────────────────────────────┐
│  Mind Map Divergent Structure                                   │
│                                                                 │
│                    ┌── Feature A ─┬── Detail 1                  │
│                    │              └── Detail 2                  │
│         ┌─ Dim 1 ──┤                                            │
│         │          └── Feature B ─── Detail 3                   │
│         │                                                       │
│  Central┼─ Dim 2 ──┬── Feature C                                │
│  Problem│          └── Feature D ─┬── Detail 4                  │
│         │                         └── Detail 5                  │
│         │                                                       │
│         └─ Dim 3 ──┬── Feature E                                │
│                    └── Feature F                                │
│                                                                 │
│  Rules:                                                         │
│  • Put the core problem at the center                           │
│  • First ring holds the main dimensions                         │
│  • Keep extending until no new ideas emerge                     │
│  • Use color to distinguish different types                     │
└─────────────────────────────────────────────────────────────────┘
```

## AI-Assisted Ideation

```markdown
## Using AI as a Creative Partner

### Role-Play Method
"Suppose you're a 10-year-old — how would you solve [problem]?"
"Suppose you're Steve Jobs — how would you design this product?"
"Suppose you're an alien seeing this for the first time — what questions would you have?"

### Forced Connection Method
"Give me 5 random nouns, then force a connection to [problem]"
"Combine [problem] with [random domain] — what ideas emerge?"

### Constrained Creativity Method
"If you could only spend $100 to solve this problem, what would you do?"
"If you only had 1 hour, what would you do?"
"If you couldn't use any technology at all, what would you do?"

### Critique and Recreate
"What's the biggest weakness of this idea? How could it be overcome?"
"If a competitor wanted to attack this idea, what would they say?"
"What would the opposite of this idea look like?"
```

## Creative Warm-Up Games

### 30 Circles Exercise

```markdown
## 30 Circles Challenge

Setup: Draw 30 identically sized empty circles
Time: 3 minutes
Task: Turn each circle into something different

Purpose:
• Break perfectionism
• Train rapid ideation
• Warm up your creative muscles

Scoring:
• Number completed (fluency)
• Category diversity (flexibility)
• Uniqueness (originality)
• Level of detail (elaboration)
```

### "Yes, And..." Exercise

```markdown
## Yes, And... Improv Exercise

Rules:
1. A states an idea
2. B must say "Yes, and..." then extend it
3. No saying "no," "but," or "however"
4. Take turns for 5 minutes

Example:
A: We're opening a coffee shop
B: Yes, and the coffee shop has a cat
A: Yes, and the cat helps deliver coffee
B: Yes, and customers can adopt the cat...

Purpose: Build a culture of acceptance, practice extending ideas
```

## Facilitating Creative Sessions

```markdown
## Facilitator Checklist

### Before the Meeting
- [ ] Clearly define the problem (HMW format)
- [ ] Prepare sticky notes and a whiteboard
- [ ] Invite participants from diverse backgrounds
- [ ] Prepare a warm-up activity
- [ ] Set the time allocation

### During the Meeting
- [ ] Explain the rules (defer judgment, quantity first)
- [ ] Manage the pace
- [ ] Protect quieter voices
- [ ] Prevent premature criticism
- [ ] Record all ideas

### After the Meeting
- [ ] Organize and categorize ideas
- [ ] Send out meeting notes
- [ ] Assign follow-up actions
- [ ] Track progress

## Handling Difficult Situations

| Situation | How to Handle |
|------|----------|
| Someone keeps criticizing | Remind them of the rules, note the critique for later |
| Room is too quiet | Use round-robin sharing, individual silent ideation |
| Discussion drifts off-topic | Gently redirect, but record the tangent |
| One person dominates | "Thanks! Let's hear from others" |
| Discussion stalls | Switch techniques, or take a 5-minute break |
```

## Creative Output Format

### Creative Pitch Template

```markdown
# Creative Proposal: [Name]

## One-Line Description
[Describe this in 20 words or fewer]

## Problem It Solves
[Describe the pain point]

## Core Idea
[Describe the solution]

## Why It's Feasible
- Technical:
- Market:
- Resources:

## Expected Impact
- Short-term:
- Long-term:

## Next Steps
1. [Specific action]
2. [Specific action]
3. [Specific action]
```

## Recommended Tools

- **Whiteboard tools**: Miro, FigJam, Notion
- **Mind mapping**: XMind, MindNode
- **Inspiration collection**: Notion, Evernote
- **Voting tools**: Slido, Mentimeter

## Related Resources

- [IDEO Design Thinking](https://designthinking.ideo.com/)
- [Stanford d.school](https://dschool.stanford.edu/resources)
- [Gamestorming](https://gamestorming.com/)
