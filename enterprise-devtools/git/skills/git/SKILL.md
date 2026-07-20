---
schema: "1.0"
name: git
version: "1.0.0"
description: Git branching models, merge/rebase discipline, and history hygiene for teams
domain: technology
triggers:
  keywords:
    primary: [git, merge, rebase, branch, commit]
    secondary: [git history, force push, cherry-pick, bisect, submodule]
  context_boost: [version control, pull request, repository]
  context_penalty: [github actions, ci/cd pipeline config]
  priority: high
dependencies:
  software-skills: [git-workflows]
author: claude-domain-skills
---

# Git

> History as a tool for understanding, not just a backup

## Applicable Scenarios

- Choosing or reviewing a team's branching model
- Deciding between merge and rebase for a given situation
- Recovering from a bad rebase, force-push, or lost commit
- Cleaning up commit history before a PR merge
- Removing accidentally committed secrets from history

## Core Knowledge

### Branching Models

| Model | Approach | Best For |
|------|------|------|
| **Trunk-based** | Short-lived branches, frequent merges to `main` | Teams with strong CI/CD and feature flags |
| **GitHub Flow** | Feature branch → PR → merge to `main`, deploy from `main` | Most web/SaaS teams |
| **GitFlow** | `main` + `develop` + release/hotfix branches | Software with versioned releases (desktop apps, libraries) |

Trunk-based development scales better with strong CI and feature flags; GitFlow's overhead pays off mainly when you genuinely need to maintain multiple release versions concurrently.

### Merge vs. Rebase

| Operation | Effect | Use When |
|------|------|------|
| **Merge** | Creates a merge commit, preserves both histories exactly as they happened | Integrating a shared/public branch, preserving true chronological record |
| **Rebase** | Replays commits onto a new base, linear history, rewrites commit hashes | Cleaning up a local/feature branch before it's shared or merged |

**The rule that matters**: never rebase a branch that other people have already pulled/based work on — rewritten history diverges from what they have, and reconciling it is painful and error-prone.

### Commit Hygiene

- Atomic commits (one logical change per commit) make `git bisect`, `git revert`, and code review meaningfully easier
- Commit messages: imperative mood, explain *why* not just *what* (the diff already shows what)
- `git commit --amend` and interactive rebase (`git rebase -i`) are for cleaning up **local, unpushed** history — not for rewriting shared history

### Recovering From Mistakes

- `git reflog` retains a record of where `HEAD` has pointed, even after a "lost" commit from a reset or rebase — almost nothing is unrecoverable until garbage collection actually runs (weeks by default)
- `git revert` creates a new commit undoing a previous one, safe for shared history; `git reset` rewrites history, unsafe once pushed and pulled by others

## Best Practices

1. **Rebase local branches, merge shared ones** — never rewrite history others have already pulled
2. **Keep commits atomic and messages explanatory** — future `git blame`/`bisect` depend on this
3. **Use `git revert`, not `git reset`, on shared branches** — reset rewrites history, revert doesn't
4. **Never force-push to `main`/shared branches** — use `--force-with-lease` at most, and only on your own feature branches
5. **Treat committed secrets as compromised immediately** — rotate the credential, don't just remove it from a future commit
6. **Use `.gitignore` proactively**, not reactively — configure it before the first commit that would need it

## Common Mistakes

| Mistake | Correct Approach |
|------|----------|
| `git push --force` on a shared branch | Use `--force-with-lease` on your own feature branch only, never on `main` |
| Rebasing a branch other people have already pulled | Merge instead, or coordinate explicitly before any rebase of shared work |
| Removing a secret in a later commit, assuming it's gone | Rotate the credential immediately; treat any committed secret as compromised regardless of later removal |
| One giant commit for an entire feature | Split into atomic, reviewable commits as you go |
| `git reset --hard` without checking `git status`/stash first | Check for uncommitted work and stash it before any destructive reset |

## Sharp Edges

### SE-1: Force-Push Overwriting Shared History
- **Severity**: critical
- **Situation**: A force-push to a shared branch (`main` or a branch others have already pulled) silently discards commits other people had already based work on
- **Cause**: `git push --force` unconditionally overwrites the remote branch's history with the local one, with no check for whether someone else's work would be lost
- **Symptoms**:
  - Teammates' next `git pull` reports diverged history or, worse, their local commits appear to vanish
  - Commits that existed yesterday are missing from the remote branch today
- **Solution**: Never force-push to shared branches; use `--force-with-lease` (which fails if the remote has commits you haven't seen) on your own feature branches only, and recover lost work via `git reflog` if it does happen
- **Details**: → [extended/checklists.md#history-safety-checklist]

### SE-2: Rebasing Public/Shared Branches
- **Severity**: high
- **Situation**: A branch already pulled by teammates gets rebased, and everyone who had it checked out now has a diverged, conflicting history with the remote
- **Cause**: Rebase rewrites commit hashes; anyone with the old commits based on top of them now has commits with no common ancestor with the new, rewritten history
- **Symptoms**:
  - Teammates hit "diverged branches" errors on pull, or accidentally create a tangled merge commit trying to reconcile
- **Solution**: Only rebase branches that are exclusively yours and haven't been pulled by anyone else; for shared branches, use merge (or coordinate an explicit, communicated rebase with everyone re-cloning)

### SE-3: Secrets Committed to History Requiring a Rewrite
- **Severity**: critical
- **Situation**: An API key or credential gets committed, then "fixed" in a later commit — but the secret remains fully recoverable in the repo's history indefinitely
- **Cause**: Git history is additive; deleting a file in a later commit doesn't remove it from earlier commits, and anyone with clone access can check out the earlier commit or diff it directly
- **Symptoms**:
  - A secret scanner (GitHub secret scanning, gitleaks, trufflehog) flags a credential in an old commit
  - The "removed" secret is still trivially found via `git log -p` or checking out the offending commit
- **Solution**: Rotate the exposed credential immediately — this is non-negotiable regardless of what happens to the repo. Then, if removal from history is required, rewrite history with `git filter-repo` (not the deprecated `filter-branch`) and force-push with full team coordination, since it rewrites every downstream commit hash

### SE-4: Large Binary Files Bloating Repository Size
- **Severity**: medium
- **Situation**: Binary assets (images, videos, compiled artifacts) committed directly into the repo cause every clone to grow, and repository size becomes a real operational problem over time
- **Cause**: Git stores every version of every committed file forever by default; binaries don't diff/compress like text, so each version adds its full size to history permanently
- **Symptoms**:
  - `git clone` takes noticeably longer over time
  - `.git` directory size is disproportionate to the current working tree size
- **Solution**: Use Git LFS (Large File Storage) for binary assets going forward, and consider a history rewrite (with team coordination) to remove already-committed large binaries if repo size has become a real problem

### SE-5: Long-Lived Feature Branches Causing Merge Conflict Pileup
- **Severity**: medium
- **Situation**: A feature branch lives for weeks without merging from `main`, and when it's finally ready, the merge/rebase conflict resolution is so large it's effectively unreviewable and risks silently reintroducing bugs
- **Cause**: The longer two branches diverge, the more conflicting changes accumulate; conflict resolution done all at once, under pressure to finally merge, gets rushed
- **Symptoms**:
  - A "simple" feature PR shows an enormous diff dominated by merge conflict resolution noise
  - Post-merge bugs trace back to a conflict that was resolved incorrectly
- **Solution**: Merge/rebase from `main` frequently (at least every few days) during long-running feature work, and prefer trunk-based development with feature flags for anything that would otherwise need a long-lived branch

## Recommended Tools

| Category | Tools |
|------|------|
| History rewriting | git filter-repo, BFG Repo-Cleaner |
| Secret scanning | gitleaks, trufflehog, GitHub secret scanning |
| Large files | Git LFS |
| Bisecting | `git bisect` (built-in) |

## History Safety

**Full checklist**: → [extended/checklists.md#history-safety-checklist]

## Related Resources

[Pro Git Book](https://git-scm.com/book/en/v2) | [Git Documentation](https://git-scm.com/doc) | [Oh Shit, Git!?!](https://ohshitgit.com/)

## Related Domains

[[github-actions]] | [[python]] | [[javascript]] | [[go]]
