---
schema: "1.0"
name: bash
version: "1.0.0"
description: Bash/shell scripting quoting discipline, error handling, subshell semantics, and portability practices
domain: technology
triggers:
  keywords:
    primary: [bash, shell script, shell scripting, sh]
    secondary: [set -euo pipefail, subshell, trap, posix, dash]
  context_boost: [automation script, CI script, cron job]
  context_penalty: [python, powershell, batch file]
  priority: high
dependencies:
  software-skills: []
author: claude-domain-skills
---

# Bash / Shell Scripting

> Quoting is the whole game — nearly every shell bug traces back to word-splitting or an unchecked failure

## Applicable Scenarios

- Writing or reviewing Bash scripts for automation, CI, or deployment
- Debugging scripts that behave differently depending on filenames or input
- Deciding on error-handling strategy (`set -e`, explicit checks, traps)
- Writing scripts intended to be portable across shells (bash vs POSIX sh vs dash)
- Diagnosing why a variable set in a loop or pipeline doesn't persist

## Core Knowledge

### Quoting & Word Splitting

- Unquoted variable expansion (`$VAR` instead of `"$VAR"`) undergoes **word splitting** (on `$IFS`) and **globbing** — a variable containing a space or a filename with spaces silently becomes multiple arguments
- `"$@"` (quoted, with the `@`) is the only form that correctly preserves each positional argument as a separate word, including ones containing spaces — `$@` and `$*` unquoted both break on spaces
- Command substitution `$(...)` is also subject to word splitting unless quoted: `"$(...)"`

### Error Handling

| Setting | Effect | Limitation |
|------|------|------|
| `set -e` | Exit immediately if a command fails | Does NOT trigger inside a pipeline (except the last command) unless `set -o pipefail` is also set; does NOT trigger for commands used in `if`/`while`/`&&`/`||` conditions |
| `set -u` | Error on referencing an unset variable | Doesn't catch a variable that's set but empty |
| `set -o pipefail` | A pipeline's exit status is the last non-zero command, not just the last command | Still needs `set -e` to actually stop execution |

`set -euo pipefail` is a strong default but is not a substitute for explicitly checking the exit status of commands whose failure matters most (e.g., `cd`, critical file operations).

### Subshells & Scope

- A pipeline (`cmd1 | cmd2`) runs each stage in its own subshell in most shells — variables set *inside* a `| while read` loop do not persist after the pipeline ends, because the loop body ran in a subshell with its own copy of the environment
- Command substitution `$(...)` and `(...)` (explicit subshell) also isolate variable changes from the parent shell

### Portability

- `#!/bin/bash` scripts may use bash-only features (arrays, `[[ ]]`, `local`, string manipulation operators) that don't exist in POSIX `sh`/`dash` — a script that works when run as `bash script.sh` can fail when invoked as `sh script.sh` or on a system where `/bin/sh` is dash (e.g., Debian/Ubuntu)

## Best Practices

1. **Quote every variable expansion** (`"$VAR"`, `"$@"`) unless you specifically need word splitting
2. **Use `set -euo pipefail`** as a baseline, but still explicitly check the exit status of commands whose failure has serious consequences
3. **Avoid `| while read` when the loop needs to set variables used after it** — use process substitution (`< <(...)`) instead, which doesn't create a subshell for the loop body
4. **Use `trap` for cleanup** (temp file removal, lock release) so cleanup runs even on early exit/error
5. **Prefer `[[ ]]` over `[ ]`** in bash scripts — safer word-splitting behavior and more operators, at the cost of bash-only portability
6. **Validate destructive variables before use** — never run `rm -rf "$VAR"/...` without confirming `$VAR` is non-empty and set

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `rm -rf $DIR/*` with `$DIR` possibly empty/unset | Quote it and validate: `[[ -n "$DIR" ]] && rm -rf "${DIR:?}"/*` |
| `for f in $(ls *.txt)` | Use a glob directly: `for f in *.txt` |
| Assuming `set -e` catches a failed command in a pipeline | Add `set -o pipefail`, and still check specific critical commands explicitly |
| Setting a variable inside `cmd | while read line; do var=$line; done` and using `$var` after | Use `while read line; do var=$line; done < <(cmd)` to avoid the subshell |
| `cd /some/path && do_thing` without checking `cd` succeeded when not using `&&` | Always chain with `&&` or explicitly check `cd`'s exit status before proceeding |

## Sharp Edges

### SE-1: Unquoted Expansion Breaking on Filenames With Spaces
- **Severity**: high
- **Situation**: A script that iterates over files or passes variables to commands works fine in testing (filenames without spaces) but silently mishandles or deletes the wrong things once a filename with a space (or glob character) is encountered
- **Cause**: Unquoted `$VAR` undergoes word splitting and globbing — a single filename with a space becomes two separate "words" to whatever command receives it
- **Symptoms**:
  - A loop processes "half" of a filename as if it were two separate arguments
  - A command that should operate on one file mysteriously operates on multiple, or fails with a "file not found" for a file that clearly exists
- **Solution**: Quote every variable expansion by default (`"$VAR"`, `"$@"`), and use `find ... -print0 | xargs -0` or arrays (`"${arr[@]}"`) for iterating over file lists safely
- **Details**: → [extended/checklists.md#quoting-and-safety-checklist]

### SE-2: `set -e` Not Catching Failures Inside Pipelines or Conditionals
- **Severity**: high
- **Situation**: A script with `set -e` is assumed to halt on any command failure, but a failing command inside a pipeline (other than the last stage) or inside an `if`/`while` condition doesn't trigger the expected exit, and the script continues in a bad state
- **Cause**: `set -e`'s exit-on-error behavior has well-documented exceptions: it doesn't apply to non-final pipeline stages (without `pipefail`), and it never applies to a command whose exit status is being tested (in `if`, `while`, `&&`, `||`, or negated with `!`)
- **Symptoms**:
  - A script continues past a command that clearly failed, producing corrupted downstream output
  - The failure is only caught much later, far from its actual origin
- **Solution**: Add `set -o pipefail` for pipeline coverage, and explicitly check the exit status of any command whose failure is consequential, rather than relying on `set -e` alone to catch everything

### SE-3: Subshell Variable Scope Loss in `| while read` Loops
- **Severity**: medium
- **Situation**: A script accumulates a count or builds a string inside `command | while read line; do ...; done`, and after the loop, the variable appears unset or unchanged, as if the loop body never ran
- **Cause**: Each stage of a pipeline typically runs in its own subshell; the `while` loop's body executes in a subshell whose variable changes are discarded once that subshell exits, leaving the parent shell's copy of the variable untouched
- **Symptoms**:
  - A counter incremented inside the loop is still 0 after the loop
  - The bug only appears with a `|`-based loop, not an equivalent `for` loop over an array
- **Solution**: Use process substitution instead of a pipe (`while read line; do ...; done < <(command)`), which runs the loop body in the current shell, not a subshell

### SE-4: Catastrophic Deletion From Unset/Empty Variable in `rm -rf`
- **Severity**: critical
- **Situation**: A script runs `rm -rf "$SOME_DIR"/*` (or similar), `$SOME_DIR` is unexpectedly empty or unset due to an earlier bug, and the command effectively becomes `rm -rf /*` or `rm -rf ./*` in the current directory, deleting far more than intended
- **Cause**: An empty or unset variable in a path expression silently collapses the path to something unintended, and bash doesn't refuse to run a destructive command just because a variable was empty
- **Symptoms**:
  - Unexpected, widespread file deletion with no corresponding intentional command
  - Root-caused after the fact to an earlier step that failed to set the variable, with no check catching it before the deletion ran
- **Solution**: Use `set -u` to error on unset variables, and for destructive operations specifically use the `${VAR:?error message}` parameter expansion to fail loudly if the variable is unset or empty before a destructive command ever runs

### SE-5: Bash-Only Syntax Breaking Under `/bin/sh`
- **Severity**: medium
- **Situation**: A script written and tested with `bash script.sh` is later invoked as `sh script.sh` (or via a shebang change, or in a minimal container image where `/bin/sh` is dash, not bash), and bash-only syntax (arrays, `[[ ]]`, `local`, `${VAR//search/replace}`) causes syntax errors or silently different behavior
- **Cause**: `/bin/sh` is not guaranteed to be bash — on many systems (Debian/Ubuntu's `dash`, Alpine's `busybox sh`) it's a smaller, POSIX-only shell that doesn't support bash extensions, and `sh script.sh` ignores the script's `#!/bin/bash` shebang entirely since it's invoked directly with `sh`
- **Symptoms**:
  - A script that "works" in CI/dev suddenly fails in a minimal container or a different base OS with syntax errors on array or `[[ ]]` usage
- **Solution**: Always invoke bash-specific scripts via their shebang (`./script.sh`, ensuring it's executable and starts with `#!/bin/bash` or `#!/usr/bin/env bash`) rather than `sh script.sh`, or write genuinely POSIX-compliant scripts (avoiding bash-only features) if the script must run under minimal `/bin/sh` environments

## Recommended Tools

| Category | Tools |
|------|------|
| Linting | ShellCheck |
| Formatting | shfmt |
| Testing | bats (Bash Automated Testing System) |
| Portability checking | `checkbashisms`, running under `dash` explicitly |

## Quoting & Safety

**Full checklist**: → [extended/checklists.md#quoting-and-safety-checklist]

## Related Resources

[Bash Manual](https://www.gnu.org/software/bash/manual/bash.html) | [ShellCheck](https://www.shellcheck.net/) | [Greg's Wiki: BashPitfalls](https://mywiki.wooledge.org/BashPitfalls)

## Related Domains

[[github-actions]] | [[docker]] | [[python]] | [[go]]
