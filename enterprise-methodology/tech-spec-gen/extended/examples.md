# Tech Spec Generator - Examples

> Full usage examples showing the conversion process for different types of design documents

---

## 1. GDD (Game Design Document) Conversion Example

### Input

```bash
/tech-spec-gen docs/design/GDD-MASTER.md
```

### Execution Process

```
Step 1: Document parsing
├── Detected document type: GDD (Game Design Document)
├── Identified Feature Documents: 5
│   ├── card-system.md
│   ├── battle-system.md
│   ├── progression.md
│   ├── deck-building.md
│   └── meta-game.md
└── Identified design pillars: 3

Step 2: Technical analysis
├── Existing tech stack: Flutter 3.38+ / Flame / Dart
├── Identified system modules: 12
└── Technical decision points: 5

Step 3: Architecture design
├── Core architecture: ECS + State Machine
├── Data model: 8 entities
└── Core APIs: 24

Step 4: Task breakdown
├── Phase 1 (Foundation): 6 tasks
├── Phase 2 (Core): 8 tasks
├── Phase 3 (Features): 9 tasks
└── Phase 4 (Integration): 4 tasks

Step 5: Output
├── tech-spec.md (main spec)
├── tasks/*.md (27 task specs)
└── adr/*.md (5 decision records)
```

### GDD Section Mapping Table

| GDD section | Technical spec equivalent |
|----------|-------------|
| Design pillars | ADR (architecture constraints) |
| Core gameplay | System architecture + state machine |
| Game mechanics | API design + business logic |
| Numerical design | Constant definitions |
| Character design | Entity Schema |
| UI/UX | Screen state machine |

---

## 2. PRD (Product Requirements Document) Conversion Example

### Input

```bash
/tech-spec-gen docs/PRD-v1.md
```

### Execution Process

```
Step 1: Document parsing
├── Detected document type: PRD (Product Requirements Document)
├── Identified user stories: 15
├── Identified feature list: 8 modules
└── Identified success metrics: 5 KPIs

Step 2: Technical analysis
├── Suggested tech stack: Next.js / TypeScript / PostgreSQL
├── Identified feature modules: 8
└── Technical decision points: 3

Step 3: Architecture design
├── Core architecture: Clean Architecture
├── Data model: 12 entities
└── Core APIs: 36 endpoints

Step 4: Task breakdown
├── Phase 1 (Foundation): 5 tasks
├── Phase 2 (Core): 12 tasks
├── Phase 3 (Features): 10 tasks
└── Phase 4 (Polish): 5 tasks

Step 5: Output
└── Technical spec generated at docs/tech-spec/
```

### PRD Section Mapping Table

| PRD section | Technical spec equivalent |
|----------|-------------|
| Problem definition | Project goals (acceptance criteria) |
| User stories | Feature modules + API |
| Feature list | Task breakdown |
| Success metrics | Performance acceptance criteria |
| Constraints | ADR |

---

## 3. FRD (Feature Requirements Document) Conversion Example

### Input

```bash
/tech-spec-gen docs/api-requirements.md --type frd
```

### Execution Process

```
Step 1: Document parsing
├── Document type: FRD (Feature Requirements Document)
├── Identified API endpoints: 24
├── Identified business rules: 18
└── Identified data requirements: 6 entities

Step 2: Technical analysis
├── Suggested tech stack: FastAPI / Python / MongoDB
├── Identified service modules: 6
└── Technical decision points: 2

Step 3-5: [omitted]
```

### FRD Section Mapping Table

| FRD section | Technical spec equivalent |
|----------|-------------|
| Feature description | API design |
| Business rules | Business logic + validation rules |
| Data requirements | Data model |
| Interface requirements | External integration |
| Exception handling | Error handling strategy |

---

## 4. Task Management Example

### View Task List

```bash
/tech-spec-gen tasks

# Output:
Task ID | Name              | Priority | Status    | Dependencies
--------|-------------------|----------|-----------|-------------
T001    | Project setup      | P0       | completed | -
T002    | Core data model    | P0       | completed | T001
T003    | Auth module        | P1       | in-progress | T002
T004    | Card system        | P1       | pending   | T002
T005    | Battle system       | P1       | pending   | T002, T004
```

### View a Specific Task

```bash
/tech-spec-gen tasks T004

# Outputs detailed task spec...
```

### Create a Worktree

```bash
/tech-spec-gen worktree T004

# Actions performed:
# 1. git worktree add ../myproject-T004 -b feature/T004-card-system
# 2. cd ../myproject-T004
# 3. Copy TASK-SPEC.md into the worktree
# 4. Create .claude/context.md

# Output:
Worktree created at: ../myproject-T004
Branch: feature/T004-card-system
Task spec: TASK-SPEC.md
Ready to implement!
```

### Update Task Status

```bash
/tech-spec-gen tasks T004 --status completed

# Output:
Task T004 marked as completed.
Next available tasks:
- T005: Battle system (dependencies: T002, T004 ✓)
```

---

## 5. Full Worktree Workflow Example

```bash
# 1. View tasks ready to be executed
/tech-spec-gen tasks --available

# 2. Choose a task and create a worktree
/tech-spec-gen worktree T006

# 3. Enter the worktree directory
cd ../myproject-T006

# 4. View the task spec
cat TASK-SPEC.md

# 5. [AI Agent implements within the worktree]
# ... implementation process ...

# 6. Verify completion
/tech-spec-gen verify T006

# 7. Merge back into the main branch
git checkout main
git merge feature/T006-xxx
git worktree remove ../myproject-T006

# 8. Update task status
/tech-spec-gen tasks T006 --status completed
```

---

## 6. Output File Structure Example

```
docs/
├── design/                  # Original design documents
│   ├── GDD-MASTER.md
│   └── features/
│       ├── card-system.md
│       └── battle-system.md
└── tech-spec/               # Generated technical spec
    ├── tech-spec.md         # Main spec document
    ├── architecture.md      # Detailed architecture (optional)
    ├── data-models.md       # Detailed data models (optional)
    ├── api-reference.md     # API reference (optional)
    ├── tasks/               # Task specs
    │   ├── README.md        # Task index and dependency graph
    │   ├── T001-project-setup.md
    │   ├── T002-core-models.md
    │   ├── T003-auth-module.md
    │   └── ...
    └── adr/                 # Architecture Decision Records
        ├── ADR-001-ecs-pattern.md
        ├── ADR-002-state-management.md
        └── ...
```
