---
name: copilot-session
description: Credentials for a Copilot session. Use in a Copilot session before running a board command or editing a pull request description, or when `gh` reports a permission error.
---

# Copilot session

`COPILOT_PROJECT_TOKEN` is an organisation secret, available to Copilot sessions in every clhbid
repo. A session's own token reaches one repository; this one reads and writes the CLHbid Delivery
board, reads issues, and edits pull requests.

Run these with `GH_TOKEN="$COPILOT_PROJECT_TOKEN"`:

- **Board commands:** **Setting Status** and **Querying the board** in `issue-tracker`.
- **Editing a pull request description:** `gh pr edit --body-file <body-file>`.

**When one fails,** comment on the pull request with what failed, the exact error, and the steps
that would fix it.
