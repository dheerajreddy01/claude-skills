---
name: consistency-checker
version: 1.0.0
description: Content consistency checker — ensures consistency across code, documentation, and multiple sources
triggers: [check, consistency, sync]
keywords: [consistency, validation, sync, documentation, code-style, cross-repo, pre-commit]
---

# Consistency Checker v1.0.0

> Ensure content consistency: code style → documentation sync → cross-repo sync → AI output validation

## Quick Start

```bash
/check                      # check the consistency of current changes
/check --pre-commit         # full check before PR/commit
/check --full               # full-project consistency review
/check --sync repo1 repo2   # cross-repo sync check
/check --ai                 # validate AI output consistency
```

## Four Check Modules

| Module | Purpose | Trigger timing |
|------|------|----------|
| **code-consistency** | Naming, style, API consistency | During development, before PR |
| **doc-consistency** | Documentation and code sync | After changes, before PR |
| **cross-repo-sync** | Content sync across multiple repos | When updating across projects |
| **output-consistency** | AI output consistency before/after | After generating content |

## Execution Flow

```
/check → determine scope → run checks → generate report → provide fix suggestions if issues found
```

---

## Module 1: Code Consistency (code-consistency)

### Check Items

| Item | Description |
|------|------|
| **Naming rules** | Whether variable, function, and class naming follow project conventions |
| **Code style** | Indentation, brackets, semicolons, and other formatting |
| **API consistency** | Whether similar features have consistent API design |
| **Error handling** | Error class and error message format |
| **Import order** | Whether import order follows convention |

### Usage

```bash
/check code                    # check current changes
/check code src/auth/*.ts      # check specific files
/check code --rule naming      # check a specific rule
```

---

## Module 2: Documentation Consistency (doc-consistency)

### Check Items

| Item | Description |
|------|------|
| **README sync** | Whether the README description matches actual functionality |
| **API documentation** | Whether API documentation matches the implementation |
| **SKILL.md** | Whether version and description are correct |
| **Comment sync** | Whether function comments match the implementation |
| **Example code** | Whether the examples in the documentation actually run |

### Usage

```bash
/check doc                     # check documentation consistency
/check doc README.md           # check a specific document
/check doc --api               # check API documentation
```

---

## Module 3: Cross-Repo Sync (cross-repo-sync)

### Check Items

| Item | Description |
|------|------|
| **Version sync** | Whether versions match across multiple repos |
| **Shared files** | Whether identical file contents are in sync |
| **Config sync** | Whether config files are consistent |
| **Dependency sync** | Whether shared dependency versions are consistent |

### Usage

```bash
/check sync repo1 repo2              # compare two repos
/check sync repo1 repo2 --path skills/  # specify comparison path
/check sync repo1 repo2 --version-only  # check version only
```

### Sync Flow

```
Specify source/target → compare versions → compare file contents → ask for sync direction if differences found → perform sync
```

---

## Module 4: AI Output Consistency (output-consistency)

### Check Items

| Item | Description |
|------|------|
| **Contradictions** | Whether statements within the same conversation contradict each other |
| **Factual consistency** | Whether cited facts are correct |
| **Command consistency** | Whether suggested commands actually exist |
| **Format consistency** | Whether output format is uniform |

### Usage

```bash
/check ai "content to verify"
/check ai --verify-command "/some-command"  # verify a command
/check ai --verify-paths                     # verify file paths
```

### Key Point: Command Validation

**Always verify that a command exists via context7's official documentation before suggesting it.**

---

## Integrated Usage Scenarios

### Scenario 1: During Development

```bash
/check
# → ✅ Naming consistency: passed
# → ⚠️ Found a similar function `formatDate` in src/utils/time.ts
#    Suggestion: consider reusing the existing function
```

### Scenario 2: Before PR

```bash
/check --pre-commit
# → ✅ Code consistency: passed
# → ⚠️ CHANGELOG.md not updated
#    Suggestion: add a description of this change
```

### Scenario 3: Cross-Repo Sync

```bash
/check --sync ~/self-evolving-agent ~/evolve-plugin
# → ❌ Versions out of sync
#    self-evolving-agent: v5.2.0
#    evolve-plugin: v5.0.0
#    Sync now? [Y/n]
```

### Scenario 4: AI Output Validation

```bash
/check --ai --verify-command "/plugin install"
# → ✅ Command validation: /plugin install exists
#    Source: Claude Code official documentation
```

---

## Extended Resources

- Full examples and templates → [extended/examples.md](./extended/examples.md)
- CLAUDE.md configuration example → [extended/examples.md#claudemd-configuration-example](./extended/examples.md#claudemd-configuration-example)

---

## Why It Matters

1. **Prevent inconsistency** — keep code, documentation, and multiple sources in sync
2. **Reduce errors** — catch potential problems early
3. **Improve quality** — unify style and conventions
4. **Build trust** — AI output is more reliable once validated
