# Tech Spec Generator - Templates

> Full template collection for `/tech-spec-gen` to reference when generating spec documents

---

## 1. Main Spec Document Template (tech-spec.md)

```markdown
# [Project Name] Technical Specification

> **Version**: v1.0 | **Generated**: YYYY-MM-DD | **Source Document Version**: vX.X

## Table of Contents

1. [Technical Overview](#1-technical-overview)
2. [System Architecture](#2-system-architecture)
3. [Data Model](#3-data-model)
4. [API/Interface Definitions](#4-apiinterface-definitions)
5. [State Management](#5-state-management)
6. [Task Breakdown](#6-task-breakdown)
7. [Acceptance Criteria](#7-acceptance-criteria)
8. [Technical Decision Records](#8-technical-decision-records)

---

## 1. Technical Overview

### 1.1 Tech Stack Confirmation

| Layer | Technology choice | Version | Rationale |
|------|----------|------|----------|
| Language | | | |
| Framework | | | |
| Database | | | |
| Test framework | | | |
| Deployment environment | | | |

### 1.2 Project Structure

```
[project root]/
├── src/                    # Source code
│   ├── core/               # Core logic
│   ├── domain/             # Domain layer
│   ├── data/                # Data layer
│   ├── presentation/       # Presentation layer
│   └── infrastructure/     # Infrastructure
├── tests/                  # Tests
├── docs/                   # Documentation
└── config/                 # Configuration
```

### 1.3 External Dependencies

| Dependency | Purpose | Version constraint | Alternative |
|------|------|----------|----------|
| | | | |

---

## 2. System Architecture

### 2.1 High-Level Architecture Diagram

```
[architecture diagram - ASCII or Mermaid]
```

### 2.2 Module Responsibilities

| Module | Responsibility | Dependencies | Corresponding design doc section |
|------|------|------|------------------|
| | | | |

### 2.3 Core Flow

[Sequence diagram or flowchart of the main process]

---

## 3. Data Model

### 3.1 Core Entities

```[language]
// Entity definitions, including fields, types, constraints, validation rules
```

### 3.2 Data Relationships

```
[ER diagram or relationship description]
```

### 3.3 Constants and Configuration

```[language]
// Numeric constants extracted from the design document
// Each constant annotated with its source section
```

---

## 4. API/Interface Definitions

### 4.1 Public API

```[language]
// Signature, parameters, return value, and error handling for each API
// Includes usage examples
```

### 4.2 Event System

| Event name | Payload | Trigger timing | Subscribers |
|----------|---------|----------|--------|
| | | | |

---

## 5. State Management

### 5.1 Global State

```[language]
// State structure, initial values, update rules
```

### 5.2 State Machine Definition

```
[state]─trigger─>[state]
```

---

## 6. Task Breakdown

### 6.1 Task Overview

| Task ID | Task name | Priority | Dependencies | Complexity | Status |
|---------|----------|--------|------|--------|------|
| T001 | | P0/P1/P2 | | S/M/L/XL | pending |

### 6.2 Phase Grouping

```
Phase 1: Foundation
├── T001: ...
└── T002: ...

Phase 2: Core
├── T003: ...
└── T004: ...
```

> See the [tasks/](./tasks/) directory for detailed task specs

---

## 7. Acceptance Criteria

### 7.1 Functional Acceptance

- [ ] [Feature 1]: [testable criterion]
- [ ] [Feature 2]: [testable criterion]

### 7.2 Performance Acceptance

| Metric | Target | Test method |
|------|------|----------|
| | | |

### 7.3 Quality Acceptance

- [ ] Test coverage > X%
- [ ] No linter warnings
- [ ] Documentation complete

---

## 8. Technical Decision Records

### ADR-001: [Decision title]

**Status**: Decided

**Context**: [Why this decision is needed]

**Options**:
1. [Option A] - pros and cons
2. [Option B] - pros and cons

**Decision**: [Choice and rationale]

**Impact**: [Resulting impact]
```

---

## 2. Task Spec Template (task-spec.md)

```markdown
# Task [ID]: [Task name]

> **Priority**: P0/P1/P2 | **Complexity**: S/M/L/XL | **Dependencies**: [dependent tasks]

## Goal

[One sentence describing what this task should accomplish]

## Background

### Related Design Document Sections
- [Section name and summary]

### Related Technical Spec
- [Link to tech-spec.md section]

## Implementation Spec

### Files to Create

| File path | Purpose |
|----------|------|
| | |

### Code Spec

```[language]
// Interfaces/classes/functions to implement
// Include complete signatures and descriptions
```

## Acceptance Criteria

### Functional Acceptance

- [ ] [Specific testable criterion 1]
- [ ] [Specific testable criterion 2]

### Test Cases

```
Given: [preconditions]
When: [action performed]
Then: [expected result]
```

### Edge Cases

| Scenario | Expected behavior |
|------|----------|
| | |

## Implementation Guidance

### Suggested Implementation Order

1. [Step 1]
2. [Step 2]

### Notes

- [Important reminder]

## Worktree Setup

```bash
git worktree add ../[project]-[task-id] -b feature/[task-id]-[name]
```
```

---

## 3. ADR Template (Architecture Decision Record)

```markdown
# ADR-[number]: [Decision title]

**Status**: Proposed / Decided / Superseded / Deprecated

**Date**: YYYY-MM-DD

**Decision maker**: [Who made this decision]

## Background

[Describe the context and problem prompting this decision]

## Decision Drivers

- [Driver 1]
- [Driver 2]

## Options Considered

### Option A: [Name]

**Pros**:
- ...

**Cons**:
- ...

### Option B: [Name]

**Pros**:
- ...

**Cons**:
- ...

## Decision

Chose [option], because:

- [Reason 1]
- [Reason 2]

## Impact

### Positive Impact
- ...

### Negative Impact
- ...

### Required Follow-up Actions
- ...

## Related

- [Related ADR link]
- [Related design document link]
```

---

## 4. API Design Template

```markdown
# API Reference

## Overview

| Endpoint | Method | Purpose |
|------|------|------|
| `/api/v1/xxx` | GET | ... |

---

## Endpoint Details

### GET /api/v1/[resource]

**Description**: [feature description]

**Authentication**: Required / Not required

**Request Parameters**:

| Parameter | Type | Required | Description |
|------|------|------|------|
| | | | |

**Response Format**:

```json
{
  "status": "success",
  "data": { ... }
}
```

**Error Codes**:

| Code | Description | Recommended handling |
|------|------|----------|
| 400 | Bad Request | Check parameters |
| 401 | Unauthorized | Re-authenticate |
| 404 | Not Found | Confirm the resource exists |

**Usage Example**:

```bash
curl -X GET "https://api.example.com/api/v1/[resource]" \
  -H "Authorization: Bearer TOKEN"
```
```

---

## 5. Data Model Template

```markdown
# Data Models

## Entity: [Entity name]

### Schema

```typescript
interface [Entity] {
  id: string;           // unique identifier
  [field]: [type];      // [description]
  createdAt: Date;      // creation time
  updatedAt: Date;      // last update time
}
```

### Constraints

- `[field]`: [constraint description]

### Relationships

```
[Entity] --< [RelatedEntity]  // one-to-many
[Entity] >--< [RelatedEntity] // many-to-many
```

### Indexes

| Field | Type | Purpose |
|------|------|------|
| id | Primary | Primary key |
| [field] | Index | Query optimization |

### Validation Rules

| Field | Rule |
|------|------|
| | |
```

---

## 6. Worktree Context Template

```markdown
# Task Context for AI Agent

## Task Info

- **Task ID**: [ID]
- **Task name**: [name]
- **Branch**: feature/[task-id]-[name]

## What You Need to Do

[Clearly describe the task goal]

## Completed Prerequisite Tasks

- [x] T00X: [task name] - [key output]

## Related Files

| File | Purpose | Notes |
|------|------|------|
| | | |

## Acceptance Criteria

- [ ] [Criterion 1]
- [ ] [Criterion 2]

## After Completion

1. Run tests to confirm they pass
2. Commit and describe the changes
3. Prepare to merge back into the main branch
```
