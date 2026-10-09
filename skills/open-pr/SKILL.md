---
name: open-pr
description: Open, push or update a GitHub pull request: base branch, title, closing reference, size check, ready or draft. Use when the user wants to open a PR, push changes to one, or update its description. Write the body with `pr`.
---

# Open PR

Open or update a pull request. The `pr` skill writes the body; this skill does everything around it.

## Step 1: Find the pull request and its base

```bash
git fetch origin --prune
gh pr view --json number,url,baseRefName
```

**Never assume the base.** Diff against the branch the pull request targets:

- **A pull request exists:** its `baseRefName`. Update it in Step 6b.
- **None exists:** the default branch (`main` or `development`), from
  `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name` — or the previous slice's
  branch when this one is stacked on it. Create it in Step 6a.

## Step 2: Fetch the issue

Parse the issue number from the branch name. Branch names follow the pattern `{issue_number}-{description}` (e.g., `54-return-to-after-login` -> issue #54).

**A number alone does not identify an issue.** Issue numbers are per-repository, and the issue is
not always in the repo you are opening the PR from — a change in one repo often implements an issue
tracked in another. Resolve the repo before fetching, defaulting to the current one:

```bash
# Default to this repo; override when the issue lives elsewhere
ISSUE_REPO="$(gh repo view --json nameWithOwner --jq .nameWithOwner)"

gh issue view <issue_number> --repo "$ISSUE_REPO" --json title,body,url
```

If the fetched issue doesn't match the work in the diff, you have resolved the wrong repo — ask
rather than writing a PR body against someone else's issue.

## Step 3: Read the changes

```bash
git status                      # warn the user about uncommitted changes
git log --oneline "origin/<base>"...HEAD
git diff "origin/<base>"...HEAD
```

## Step 4: Check the size

```bash
git diff --shortstat <base>...HEAD
```

Compare it against the ceiling in **Decomposing work before Ready for Agent** in the `issue-tracker`
skill. If it is over, say so and propose that skill's simplify-then-split. **This never blocks**: if
the author says ship it, ship it.

## Step 5: Write the body

Write it with the `pr` skill, then:

- Put any task to do before or after merging ("add env var X before merging", "run migration Y
  after") in Merge Danger's optional description.
- End with the closing line: `Closes #<issue_number>`, or `Part of #<issue_number>` when the pull
  request delivers only part of the issue. **Qualify it when the issue is in another repo** —
  `Closes owner/repo#N`; a bare `#N` silently means this repo's issue N.

**Title format:** `{issue_number}: {issue_title}`, e.g. `54: Return to after login`.

## Step 6a: Create a pull request

Save the body from Step 5 to a file, then:

```bash
git push -u origin HEAD
gh pr create --base "<base>" --title "<issue_number>: <issue_title>" --body-file <body-file>
gh pr edit --add-reviewer <maintainer>
```

**`--base` is not optional.** Without it `gh` targets the default branch, so a slice stacked on a
previous one would show its parent's changes in the diff too.

**Open ready for review, and request one.** Draft means the run stopped short: reserve it for the
**Blocked** and **Error** endings in **How a run ends** in the repo's `AGENTS.md`, where the pull
request carries a comment explaining what is needed.

## Step 6b: Update a pull request

```bash
git push
gh pr edit --body-file <body-file>
```

Rewrite the body against the whole diff rather than appending to it. Update the title with
`gh pr edit --title` if the issue's title has changed.

## Step 7: Check the link

```bash
gh pr view --json baseRefName,closingIssuesReferences
```

A `Closes` line on a pull request into the default branch must show up in `closingIssuesReferences`;
if it doesn't, fix the line. A stacked pull request shows no link until GitHub retargets it after
its parent merges (see **A closing reference fires only on a merge into the default branch** in
`issue-tracker`) — say so in your report. `Part of` never links.
