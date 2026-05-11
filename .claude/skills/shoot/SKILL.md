---
name: shoot
description: Ship the current task end-to-end — push the branch, open a PR, merge it, and sync local main. Use when a task is committed locally and ready to go to main. Triggers on /shoot.
---

# /shoot — Ship a task

One-shot command that takes a committed feature branch and gets it onto `main`.
Intended for teammates who don't want to memorize git commands.

## What it does

1. Verify the working tree is clean and you are not on `main`.
2. Push the current branch to `origin` (with `-u` on first push).
3. Open a PR via `gh pr create` (or reuse an existing open one).
4. Merge the PR with `gh pr merge --merge --delete-branch` to match the project's existing merge-commit history.
5. Check out `main`, pull, and `git fetch --prune` to clean up stale remote refs.
6. Print a one-line confirmation.

## Preconditions to check before doing anything

Run these first. If any fail, STOP and tell the user what's wrong — do not try to "fix" it silently.

- `git status` must show a clean working tree. If there are unstaged or staged changes, ask the user whether to commit them first.
- `git rev-parse --abbrev-ref HEAD` must NOT return `main`. If it does, there's nothing to ship.
- `git log main..HEAD --oneline` must list at least one commit. If empty, the branch has nothing ahead of main.
- `gh auth status` should succeed. If not, tell the user to run `gh auth login`.

## Step-by-step

### 1. Push the branch

```bash
BRANCH=$(git rev-parse --abbrev-ref HEAD)
# If no upstream, push with -u; otherwise plain push.
if git rev-parse --abbrev-ref --symbolic-full-name @{u} >/dev/null 2>&1; then
    git push
else
    git push -u origin "$BRANCH"
fi
```

### 2. Open the PR (if one doesn't already exist)

Check first:
```bash
gh pr view --json url,state,number 2>/dev/null
```

If a PR exists and `state == OPEN`, skip ahead to step 3. If `state == MERGED` or `CLOSED`, tell the user and stop.

If no PR exists, create one. **Build the title from the latest commit's subject line** (truncated to ~70 chars). **Build the body from all commits in `git log main..HEAD`**, formatted as:

```bash
TITLE=$(git log -1 --pretty=%s)
COMMITS=$(git log main..HEAD --reverse --pretty='- %s')

gh pr create --title "$TITLE" --body "$(cat <<EOF
## Summary
$COMMITS

## Test plan
- [ ] Reviewed locally
- [ ] Notebook runs end-to-end (if notebook changes)

🤖 Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

Always use a HEREDOC for the body — interpolation in `--body` strings can break formatting.

### 3. Merge

```bash
gh pr merge --merge --delete-branch
```

**Do not use `--squash` or `--rebase`** unless the user explicitly asks. The project's existing PRs (#1–#7) all use merge commits, and consistency matters more than aesthetics.

`--delete-branch` removes both the remote and local branch. After this command, you may be on a deleted branch reference — the next step (`git checkout main`) handles that.

### 4. Sync main

```bash
git checkout main
git pull origin main
git fetch --prune
```

### 5. Confirm

Print:
- The merged PR URL (from step 2).
- `git status` (should be clean, on main, up to date).
- A one-line "Shipped: <task title>" confirmation.

## When to ask the user instead of proceeding

- Working tree is dirty (uncommitted changes).
- Branch has no commits ahead of `main`.
- An existing PR for this branch is `CLOSED` or already `MERGED`.
- `gh pr merge` fails (conflicts, required reviews, CI failure, branch protection).
- The current branch IS `main` — nothing to ship.

## What NOT to do

- Do not force-push.
- Do not merge to anywhere other than `main`.
- Do not skip CI / required reviews by adding `--admin` to `gh pr merge` unless the user explicitly asks.
- Do not delete other local branches automatically — only the one being shipped (`--delete-branch` handles that one).
- Do not amend or rebase commits on the feature branch — ship what's already committed.

## Example session

```
User: /shoot
Claude: Working tree clean. On branch `stage-2-target-distribution` with 1 commit ahead of main. Pushing and opening PR.
[push]
[gh pr create → https://github.com/jmzlatin/energy-gap-prediction/pull/8]
[gh pr merge --merge --delete-branch]
[git checkout main && git pull && git fetch --prune]
Claude: Shipped: Task 2.2 — target distribution and class balance. main is synced.
```
