---
name: open-pr
description: Create or update a GitHub pull request with a standardized template. Use when the user wants to open a PR, create a pull request, update a PR description, submit changes, or push a PR.
---

# Open PR

Create or update a GitHub pull request with a KISS (short, scannable) template.

## Step 1: Check for Existing PR

Check if a PR already exists for the current branch:

```bash
gh pr view --json number,title,body,url
```

- If PR exists: proceed to **update flow** (Step 5b)
- If no PR exists: proceed to **create flow** (Step 5a)

## Step 1b (Required): Identify the correct diff base

**Never assume `main`.** You must diff against the actual base branch.

- If updating an existing PR, use the PR base:

```bash
gh pr view --json baseRefName,headRefName
git fetch origin --prune
git diff --stat "origin/<baseRefName>...HEAD"
```

- If creating a new PR (no PR exists yet), use the repo default branch:

```bash
git fetch origin --prune
git symbolic-ref refs/remotes/origin/HEAD
git diff --stat "origin/<defaultBranch>...HEAD"
```

## Step 2: Fetch the GitHub Issue

Parse the issue number from the branch name. Branch names follow the pattern `{issue_number}-{description}` (e.g., `54-return-to-after-login` -> issue #54).

```bash
# Get current branch name
git branch --show-current

# Fetch issue details
gh issue view <issue_number> --json title,body,url
```

Extract the issue title and body for context.

## Step 3: Analyze Git Changes

Run these commands to understand the scope of changes:

```bash
# Check for uncommitted changes (warn user if any)
git status

# Get all commits on this branch (against the *actual* base branch)
git log <base>...HEAD --oneline

# Get the full diff summary against the *actual* base branch
git diff --stat <base>...HEAD

# REQUIRED: Inspect the actual line changes (not just stats)
git diff <base>...HEAD
```

**Required rule:** Before writing/updating the PR body, you must read the diff and extract
**1-3 concrete line-level changes** (e.g., a rename `OldName` -> `NewName`, a new env var key, an endpoint path change).
No speculative bullets.

## Step 3b: Check the size against the ceiling

```bash
git diff --shortstat <base>...HEAD
```

Discount lockfiles, snapshots and generated files, then compare what's left against the changed-line
ceiling in **Decomposing work before Ready for Agent** in the `issue-tracker` skill. That skill
holds the number; don't restate it here, or there are two copies to keep in step.

If the diff is over, say so and propose, in this order:

1. **Simplify.** Drop surplus documentation first, then anything else the change doesn't need. Most
   oversized diffs shrink here.
2. **Split.** Only once it's as small as it's going to get: cut sub-issues, set the leaves to
   `Ready for Agent`, and open this pull request for the slice that's finished — stacking it on the
   previous slice where they depend on each other.

**This never blocks.** It's a prompt to reconsider at the last moment before a reviewer sees the
diff, not a gate. If the author says ship it, ship it.

## Step 4: Generate PR Content

**Title format:** `{issue_number}: {issue_title}`

Example: `54: Return to after login`

**Body template:**

```markdown
## Description

- 1-2 bullets: what/why (not a changelog)

## Changes

- 2-5 bullets max: user-visible or behavior changes
- Avoid file-by-file lists unless it materially helps reviewers

## How to Test

1. 2-4 steps max
2. Include the single most important edge case

## Deployment Notes

1. OPTIONAL: Only include if there are special deployment steps or risks to call out
2. Flag any tasks that must be done before or after merging (e.g., "Add env var X before merging", "Run data migration Y after merging", "Add a variable to 1Password")

## Related Issues

Closes #<issue_number>
```

## Step 5a: Create New PR

Push the branch and open the PR ready for review:

```bash
# Push branch to remote
git push -u origin HEAD

# Create the PR
gh pr create --title "<issue_number>: <issue_title>" --body "$(cat <<'EOF'
## Description
- <1-2 bullets: what/why>

## Changes
- <2-5 bullets max>

## How to Test
1. <2-4 steps max>

## Related Issues
Closes #<issue_number>
EOF
)"

# Ask a human to look at it
gh pr edit --add-reviewer <maintainer>
```

## Step 5b: Update Existing PR

Push latest changes and update the PR description:

```bash
# Push latest changes
git push

# Update PR body
gh pr edit --body "$(cat <<'EOF'
## Description
- <1-2 bullets: what/why>

## Changes
- <2-5 bullets max>

## How to Test
1. <2-4 steps max>

## Deployment Notes
1. OPTIONAL: Only include if there are special deployment steps or risks to call out. If there are none, delete this section entirely.
2. Flag any tasks that must be done before or after merging (e.g., "Add env var X before merging", "Run data migration Y after merging", "Add a variable to 1Password")

## Related Issues
Closes #<issue_number>
EOF
)"
```

Optionally update the title if needed:

```bash
gh pr edit --title "<issue_number>: <issue_title>"
```

## Notes

- **Open ready for review, and request one.** Draft is not the default — it means the run stopped
  short. Reserve it for the **Blocked** and **Error** endings in **How a run ends** in the repo's
  `AGENTS.md`, where the PR carries a comment explaining what is needed. A **Complete** run opens
  the PR ready and asks a human to look at it
- The Description should be scannable and based on the issue contents
- The Changes section should be capped at 2-5 bullets and focus on behavior
- Use `Closes #N` syntax to auto-close the linked issue when merged
- When updating, keep it short; delete stale bullets rather than adding more
