# Consistency Checker - Examples and Templates

## Code Consistency Check Examples

```python
# 1. Naming consistency check
Grep(
    pattern="(function|const|class|interface) [a-z_]+[A-Z]",  # mixed snake_case and camelCase
    path="src/",
    output_mode="content"
)

# 2. Error handling consistency
Grep(
    pattern="throw new (Error|.*Error)\\(",
    path="src/",
    output_mode="content"
)
# Confirm whether the project's Error class is used consistently

# 3. API style consistency
Grep(
    pattern="(get|fetch|retrieve|load)[A-Z]",  # check if the verb used to fetch data is consistent
    path="src/",
    output_mode="files_with_matches"
)
```

## Documentation Consistency Check Examples

```python
# 1. Version number consistency
# Check whether the versions in SKILL.md, package.json, and CHANGELOG match
version_skill = Grep(pattern="^version:", path="SKILL.md")
version_pkg = Read(file_path="package.json")  # get the version field
# Compare whether the two match

# 2. Function documentation sync
# Check whether JSDoc/docstrings match the actual parameters
Grep(
    pattern="@param|:param",
    path="src/",
    output_mode="content",
    C=5
)

# 3. README feature list
# Compare the features listed in the README against the actually exported functions
```

## Cross-Repo Sync Examples

```bash
# 1. Version comparison
grep "^version:" /path/to/repo1/SKILL.md
grep "^version:" /path/to/repo2/SKILL.md

# 2. File diff
diff -rq /path/to/repo1/skills/ /path/to/repo2/skills/

# 3. Detailed comparison
diff /path/to/repo1/skills/SKILL.md /path/to/repo2/skills/SKILL.md
```

## AI Output Validation Examples

```python
# 1. Command validation - verify before suggesting a command
# Bad example: directly saying "please run /install-plugin"
# Good example: verify the command exists first

# Query official documentation via context7
mcp__context7__query-docs(
    libraryId="/anthropics/claude-code",
    query="plugin install command"
)

# 2. Path validation
Bash(command="ls -la /suggested/path 2>/dev/null || echo 'Path not found'")

# 3. Version validation
Bash(command="curl -s https://api.github.com/repos/owner/repo/releases/latest | jq -r '.tag_name'")
```

## Check Report Format Template

```
┌─────────────────────────────────────────────────────────────────┐
│  🔍 Consistency Check Report                                    │
│                                                                 │
│  Project: [project name]                                        │
│  Time: [timestamp]                                               │
│  Scope: [current changes | whole project | cross-repo]          │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  📝 Code consistency: [✅ Passed | ⚠️ Warnings | ❌ Errors]     │
│     [Details]                                                    │
│                                                                 │
│  📄 Documentation consistency: [✅ Passed | ⚠️ Warnings | ❌ Errors] │
│     [Details]                                                    │
│                                                                 │
│  🔄 Cross-repo sync: [✅ In sync | ⚠️ Differences found | ❌ Out of sync] │
│     [Details]                                                    │
│                                                                 │
│  ═══════════════════════════════════════════════════════════   │
│                                                                 │
│  📋 Recommended fixes:                                           │
│     1. [Recommendation 1]                                        │
│     2. [Recommendation 2]                                        │
│                                                                 │
│  Summary: [X/Y items passed]                                     │
└─────────────────────────────────────────────────────────────────┘
```

## CLAUDE.md Configuration Example

```yaml
consistency-checker:
  # Enable/disable modules
  modules:
    code: true
    doc: true
    cross-repo: true
    ai-output: true

  # Auto-check timing
  auto-check:
    on-save: false        # check on save
    pre-commit: true      # check before commit
    pre-push: false       # check before push

  # Cross-repo sync settings
  sync:
    repos:
      - ~/self-evolving-agent
      - ~/evolve-plugin
    auto-sync: false      # auto-sync (dangerous)

  # Ignore rules
  ignore:
    - "*.test.ts"
    - "node_modules/"
    - ".git/"
```
