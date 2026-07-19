---
schema: "1.0"
name: project-management
version: "1.0.0"
description: Project management: agile development, Scrum, Gantt charts, risk management, and team collaboration
domain: business
triggers:
  keywords:
    primary: [project management, PM, Scrum, agile, sprint, JIRA]
    secondary: [gantt, milestone, schedule, risk, kanban, backlog]
  context_boost: [team, collaboration, software, iteration]
  context_penalty: [personal, marketing, finance]
  priority: high
dependencies:
  software-skills: [git-workflows, documentation]
author: claude-domain-skills
---

# Project Management

> Ensure projects finish on time, on quality, and on budget

## Applicable Scenarios

- Project planning and scheduling
- Agile/Scrum process planning
- Risk identification and management
- Team collaboration and communication
- Project progress tracking

## Project Management Framework

```
┌─────────────────────────────────────────────────────────────────┐
│  Project Lifecycle                                              │
│                                                                 │
│  Initiate → Plan → Execute → Monitor → Close                    │
│                                                                 │
│  ┌─────────┐                                                   │
│  │Initiate │ Define goals, confirm scope, form the team         │
│  └────┬────┘                                                   │
│       ↓                                                         │
│  ┌─────────┐                                                   │
│  │  Plan   │ Break down tasks, estimate time, allocate resources│
│  └────┬────┘                                                   │
│       ↓                                                         │
│  ┌─────────┐                                                   │
│  │ Execute │ Carry out the plan, manage the team, coordinate    │
│  └────┬────┘                                                   │
│       ↓                                                         │
│  ┌─────────┐                                                   │
│  │ Monitor │ Track progress, manage changes, control risk       │
│  └────┬────┘                                                   │
│       ↓                                                         │
│  ┌─────────┐                                                   │
│  │  Close  │ Accept deliverables, capture lessons, disband team │
│  └─────────┘                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Agile/Scrum Framework

### Scrum Roles

| Role | Responsibilities |
|------|------|
| **Product Owner** | Define requirement priorities, maintain the Product Backlog |
| **Scrum Master** | Facilitate the process, remove impediments, protect the team |
| **Development Team** | Deliver increments, self-organize, cross-functional |

### Scrum Events

```
Sprint (2-4 weeks)
│
├── Sprint Planning (4-8 hours)
│   ├─ What: what will this sprint accomplish?
│   └─ How: how will it get done?
│
├── Daily Standup (15 minutes)
│   ├─ What did you do yesterday?
│   ├─ What will you do today?
│   └─ Any blockers?
│
├── Sprint Review (2-4 hours)
│   └─ Demo completed features to stakeholders
│
└── Sprint Retrospective (1.5-3 hours)
    ├─ What went well?
    ├─ What could be improved?
    └─ What will we try next sprint?
```

### User Story Format

```markdown
## Standard Format
As a [role]
I want [feature]
So that [value]

## Story Point Estimation (Fibonacci)
1 - Very small, done within hours
2 - Small, half a day to a day
3 - Medium, 1-2 days
5 - Larger, 2-3 days
8 - Large, nearly a week
13 - Very large, consider splitting
```

## Project Plan Template

```markdown
# Project Plan

## 1. Project Overview
- **Project name**:
- **Project manager**:
- **Start date**:
- **Target completion date**:

## 2. Project Goals
### Business Goals
- [specific, measurable goals]

### Success Criteria
- [how success will be judged]

## 3. Scope Definition
### In Scope
- [feature/deliverable 1]
- [feature/deliverable 2]

### Out of Scope
- [explicitly excluded items]

## 4. Milestones
| Milestone | Date | Deliverable |
|--------|------|--------|
| M1: Requirements confirmed | MM/DD | Requirements doc |
| M2: Design complete | MM/DD | Design doc |
| M3: Development complete | MM/DD | Testable build |
| M4: Launch | MM/DD | Production release |

## 5. Team Composition
| Role | Name | Responsibility |
|------|------|------|
| PM | | |
| Development | | |
| QA | | |

## 6. Risk Management
| Risk | Probability | Impact | Response Strategy |
|------|------|------|----------|
| | High/Medium/Low | High/Medium/Low | |

## 7. Communication Plan
| Meeting | Frequency | Participants |
|------|------|--------|
| Standup | Daily | Dev team |
| Weekly sync | Weekly | Everyone |
```

## Risk Management

### Risk Matrix

```
Impact
  ↑
High │ 3 │ 6 │ 9 │
  │───┼───┼───│
Med  │ 2 │ 4 │ 6 │
  │───┼───┼───│
Low  │ 1 │ 2 │ 3 │
  └───┴───┴───┘→
   Low  Med  High  Probability

Score = Impact × Probability (1/2/3 each)
Zones by score: 1-2 = low risk (green), 3-4 = medium risk (yellow), 6-9 = high risk (red)
```

### Risk Response Strategies

| Strategy | Description | When to Use |
|------|------|----------|
| **Avoid** | Change the plan to eliminate the risk | High impact, avoidable |
| **Transfer** | Shift the risk to a third party | Specialized-domain risk |
| **Mitigate** | Reduce probability or impact | Can't be fully avoided |
| **Accept** | Prepare a contingency plan | Low impact or unmanageable |

## Progress Tracking

### Burndown Chart

```
Story Points
    │
 50 ┤ ●
    │   ╲
 40 ┤     ╲ Ideal line
    │       ╲
 30 ┤ ●       ╲
    │   ●       ╲
 20 ┤     ●       ╲
    │       ● Actual progress
 10 ┤         ●
    │           ●
  0 ┼───┬───┬───┬───┬───→ Sprint day
    1   2   3   4   5
```

### Status Report Template

```markdown
## Weekly Report - Week N

### Completed This Week
- ✅ [completed item 1]
- ✅ [completed item 2]

### In Progress
- 🔄 [item in progress] (XX% complete)

### Risks and Issues
- ⚠️ [risk/issue description]
  - Impact: [impact description]
  - Action: [response measures]

### Next Week's Plan
- [ ] [planned item 1]
- [ ] [planned item 2]

### Metrics
- Progress: on schedule / X days behind
- Budget: on budget / X% over budget
```

## Estimation Techniques

### Planning Poker

```
1. Walk through the User Story
2. Team members estimate individually
3. Reveal estimates simultaneously
4. Discuss the biggest outliers
5. Re-estimate until consensus is reached
```

### Three-Point Estimation

```
Expected time = (Optimistic + 4×Most Likely + Pessimistic) / 6

Example:
- Optimistic: 3 days
- Most likely: 5 days
- Pessimistic: 10 days
- Expected = (3 + 4×5 + 10) / 6 = 5.5 days
```

## Common Tools

| Purpose | Tools |
|------|------|
| **Task management** | JIRA, Trello, Asana |
| **Gantt charts** | MS Project, Monday.com |
| **Collaboration** | Notion, Confluence |
| **Kanban** | JIRA, Trello, Miro |
| **Documentation** | Google Docs, Notion |

## Meeting Efficiency

### Meeting Checklist

```markdown
## Before the Meeting
- [ ] Meeting purpose is clear
- [ ] Agenda has been sent
- [ ] Necessary attendees invited
- [ ] Materials prepared

## During the Meeting
- [ ] Start on time
- [ ] Someone takes notes
- [ ] Keep to time
- [ ] Confirm action items

## After the Meeting
- [ ] Meeting notes sent
- [ ] Action items assigned
- [ ] Deadlines confirmed
```

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| Letting scope creep go unmanaged | Follow a change process, assess impact |
| Reporting only good news, not bad | Surface problems early, ask for help |
| Overly optimistic estimates | Add buffer, use historical data |
| Ignoring dependencies | Identify and manage dependencies |

## Related Resources

- [Scrum Guide](https://scrumguides.org/)
- [PMBOK Guide](https://www.pmi.org/pmbok-guide-standards)
- [Atlassian Agile Coach](https://www.atlassian.com/agile)
