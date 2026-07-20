# Bash / Shell Scripting — Extended Checklists

## Quoting and Safety Checklist

- [ ] Every variable expansion is quoted (`"$VAR"`, `"$@"`) unless word splitting is explicitly intended
- [ ] `set -euo pipefail` present at the top of every non-trivial script
- [ ] Critical commands (`cd`, destructive file operations) have their exit status explicitly checked, not left to `set -e` alone
- [ ] Destructive operations (`rm -rf`, bulk moves) validate variables are non-empty first (`${VAR:?}` or explicit check)
- [ ] No `| while read` loop relies on variables persisting after the loop — process substitution used instead where needed
- [ ] File iteration uses `find -print0 | xargs -0` or arrays, not `for f in $(ls ...)`
- [ ] `trap` used for cleanup (temp files, locks) so it runs on error/early exit, not just normal completion
- [ ] ShellCheck run in CI on all scripts

## Portability Checklist

- [ ] Scripts using bash-only features are invoked via their shebang (`./script.sh`), never via `sh script.sh`
- [ ] Shebang line explicitly states `#!/bin/bash` or `#!/usr/bin/env bash` when bash features are used
- [ ] Scripts intended to run under minimal `/bin/sh` (e.g., in slim containers) are tested under `dash`, not just bash
- [ ] `checkbashisms` run if POSIX portability is a real requirement
- [ ] Exit codes from the script itself are meaningful and checked by callers (CI steps, calling scripts)
