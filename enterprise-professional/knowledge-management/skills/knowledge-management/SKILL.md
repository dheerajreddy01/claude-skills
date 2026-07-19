---
schema: "1.0"
name: knowledge-management
version: "1.0.0"
description: Knowledge management: second brain, note-taking systems, information organization, digital tool workflows
domain: professional
triggers:
  keywords:
    primary: [knowledge management, second brain, note-taking, PKM, zettelkasten, card box]
    secondary: [information organization, digital garden, workflow, PARA, Obsidian, Notion, Roam, Logseq]
  context_boost: [productivity, learning, output, linking]
  context_penalty: [code, api, database, technical]
  priority: medium
dependencies:
  software-skills: [documentation]
author: claude-domain-skills
---

# Knowledge Management

> Build a personal knowledge system so information becomes a compounding asset

## Applicable Scenarios

- Building a personal note-taking system
- Designing an information organization workflow
- Building a second brain
- Choosing and integrating digital tools
- Knowledge output and sharing

## The CODE Method (Building a Second Brain)

**Capture → Organize → Distill → Express**

| Stage | Action | Focus |
|------|------|------|
| Capture | Collect valuable information | Only collect what you'll actually use |
| Organize | Categorize using the PARA system | Categorize by actionability |
| Distill | Extract key points and summarize | Progressive summarization |
| Express | Output and create | Apply what you've learned |

**Core idea: it's not about collecting more, it's about being able to use it when you need it**

## The PARA Organization System

| Category | Definition | Example |
|------|------|------|
| **P**rojects | Tasks with a clear deadline | Product launch, writing a report |
| **A**reas | Ongoing areas of responsibility | Health, finance, career |
| **R**esources | Reference material on topics of interest | Investing, writing, AI |
| **A**rchives | Completed or no longer active items | Old projects, expired data |

**Organizing principle: by "actionability," not by "topic"**

## Zettelkasten (Slip-Box) Method

### Core Principles
1. **Atomicity** - One card, one concept
2. **Connectivity** - Cards linked to each other
3. **Your own words** - Rewrite in your own understanding
4. **Permanent notes** - Notes that have been thought through

### Note Types

| Type | Description | Retention |
|------|------|----------|
| Fleeting notes | Quickly capture ideas | Temporary |
| Literature notes | Excerpts while reading | For reference |
| Permanent notes | Your own digested understanding | Permanent |
| Project notes | Related to a specific project | For the project's duration |

## Progressive Summarization

Layer 0: Original content → Layer 1: Bold key sentences → Layer 2: Highlight keywords → Layer 3: Summary at top → Layer 4: Original insight

**Principle: only go to the next layer when needed — go deep on demand**

## Tool Selection Matrix

| Need | Recommended tools | Characteristics |
|------|----------|------|
| Bidirectionally linked notes | Obsidian, Logseq | Local, Markdown |
| All-in-one | Notion | Cloud, team collaboration |
| Quick capture | Apple Notes, Drafts | Speed first |
| Read later | Readwise, Instapaper | Highlight sync |
| Task management | Todoist, Things | GTD workflow |
| Long-term storage | DEVONthink, Evernote | Powerful search |

## Workflow Design

```
Information flow: Browser → Read later → Highlight → Note app → Permanent note
Task flow: Inbox → Process → Project/To-do → Execute → Archive
```

### Daily Routine
1. Morning: review today's tasks
2. Work: focused execution
3. Anytime: quick-capture ideas
4. Evening: process inbox, plan tomorrow

## Balancing Input and Output

**Knowledge compounding cycle: Input → Connect → Create → Share → (feedback becomes new input)**

| Input | Output |
|------|------|
| Reading, courses, podcasts, conversations | Writing, teaching, creating, applying |

**Suggested ratio: input : processing : output = 3 : 2 : 5**

**Key: output drives input — apply what you learn**

## Search-First Principle

**Modern mindset: fast capture + powerful search > spending lots of time categorizing**

### Search Optimization Tips
- **Title naming**: `2024-01-15-product-strategy-meeting-Q1-goals-discussion.md`
- **Embedded keywords**: `tags: product, strategy, OKR, Q1`
- **Aliases and synonyms**: `aliases: [PKM, personal knowledge management]`
- **Special markers**: `TODO:`, `IDEA:`, `QUESTION:`

## Linking Strategy

| Link type | Description | Example |
|----------|------|------|
| Definition link | Explains a concept | What is [[Feynman Technique]] |
| Evidence link | Supports an argument | According to [[Research Report]] |
| Application link | Use case | Applied in [[Project X]] |
| MOC link | Index page | Part of [[Productivity MOC]] |

**When to create a link? When you notice "this reminds me of..."**

## Common Pitfalls

| Pitfall | Solution |
|------|------|
| Compulsive collecting | Only collect what you'll actually use |
| Tool obsession | Tools serve the purpose, they aren't the purpose |
| Over-categorizing | Search matters more than categorization |
| Only input, no output | Force yourself to output — apply what you learn |
| Chasing a perfect system | Starting matters more than perfection |

## Extended Resources

Full templates in `extended/templates.md`:
- Meeting notes template
- Reading notes template
- Project notes template
- Weekly review template
- Daily/weekly/monthly/quarterly workflow checklists

Examples and structures in `extended/examples.md`:
- Digital garden architecture
- Personal wiki structure
- MOC examples
- Knowledge management maturity model
- AI-assisted knowledge management

## Recommended Tools

- **Notes**: Obsidian, Notion, Logseq, Roam
- **Reading**: Readwise, Kindle, Pocket
- **Tasks**: Todoist, Things 3, TickTick
- **Automation**: Zapier, Make, Shortcuts
- **Writing**: Ulysses, iA Writer, Typora

## Related Resources

- [Building a Second Brain - Tiago Forte](https://www.buildingasecondbrain.com/)
- [How to Take Smart Notes - Sönke Ahrens](https://www.soenkeahrens.de/en/takesmartnotes)
- [Obsidian](https://obsidian.md/)
- [Zettelkasten Method](https://zettelkasten.de/)
