# Git — Extended Checklists

## History Safety Checklist

- [ ] No `git push --force` used on `main` or any branch other people have pulled
- [ ] `--force-with-lease` used instead of plain `--force` on personal feature branches
- [ ] Rebase only applied to branches that are exclusively local/personal and unpulled by anyone else
- [ ] Any committed secret triggers immediate credential rotation, independent of whether history is rewritten
- [ ] History rewrites (`filter-repo`) coordinated with the full team before force-pushing, since every downstream clone needs to re-sync
- [ ] `.gitignore` configured before the first commit that would need it (build artifacts, `.env`, `node_modules`, etc.)
- [ ] Large binary assets routed through Git LFS rather than committed directly
- [ ] `git status`/stash checked before any destructive command (`reset --hard`, `checkout .`, `clean -f`)

## Commit & Branch Hygiene Checklist

- [ ] Commits are atomic — one logical change per commit, not a mixed bag
- [ ] Commit messages explain *why*, not just *what* (the diff already shows what)
- [ ] Feature branches merge/rebase from `main` at least every few days to avoid conflict pileup
- [ ] PR-ready branches have interactive-rebase-cleaned history (squashed WIP commits) before review, where the team's convention calls for it
- [ ] Secret scanning (gitleaks/trufflehog or platform-native) runs in CI on every push
