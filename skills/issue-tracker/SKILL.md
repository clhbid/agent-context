---
name: issue-tracker
description: Read and set work state on the CLHbid Delivery board — issues via gh, the Status field, the board query recipes, cycles, labels, the commit convention and the decomposition rules. Use for any GitHub issue or project-board operation in a clhbid repo.
---

# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. **Work state lives on the
[CLHbid Delivery](https://github.com/orgs/clhbid/projects/4) org project**, which spans every
clhbid repo. Use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line
  bodies. This does not apply an issue form — see
  [Templates apply a category, never a state](#templates-apply-a-category-never-a-state) for what
  you then owe.
- **Read an issue**: `gh issue view <number> --comments` for human-readable output. To filter with
  `jq`, drop `--comments` and ask for the fields instead — `--jq` requires `--json` and errors out
  otherwise: `gh issue view <number> --json number,title,body,labels --jq '{number, title, labels: [.labels[].name]}'`
- **List issues**: `gh issue list --state open --json number,title,body,labels --jq '[.[] | {number, title, labels: [.labels[].name]}]'`
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Set state**: a project field, not a label — see [Status](#status).
- **Close**: an issue closes as `completed`, `not planned` or `duplicate`. The reason is the
  record, so pick the one that matches and say why in a closing comment.

  ```bash
  # Completed — the work is done
  gh issue close <number> --reason completed --comment "Shipped in #<pr>."

  # Not planned — we are not delivering it, abandoned and deferred alike
  gh issue close <number> --reason "not planned" --comment "Deferred to <date>; <owner> holds it."

  # Duplicate — another issue carries the work
  gh issue close <number> --duplicate-of <other-number> --comment "Tracked under #<other-number>."
  ```

  - `completed` is the default when `--reason` is omitted.
  - `not planned` is two words — `not-planned` is rejected. It is the whole record for a deferral
    or an abandonment; nothing else is needed.
  - `--duplicate-of` sets the reason to `duplicate` and links the two issues, so the survivor's
    thread becomes the history. Take the number of the issue that stays open.
- **Fix a close reason**: `gh issue edit` cannot set one, and `gh issue close` no-ops on an
  already-closed issue. For `completed` and `not planned`, PATCH it:
  `gh api --method PATCH repos/{owner}/{repo}/issues/{n} -f state=closed -f state_reason=not_planned`
  To record a duplicate after the fact, reopen and re-close:
  `gh issue reopen <number> && gh issue close <number> --duplicate-of <other-number>`.

Infer the repo from `git remote -v` — `gh` does this automatically when run inside a clone.

## Status

`Status` on the CLHbid Delivery project is the **single source of truth** for what state an issue
is in.

| Status           | Meaning                                                                                        |
| ---------------- | ---------------------------------------------------------------------------------------------- |
| Backlog          | The inbox — everything not yet routed. New issues land here                                    |
| Waiting on input | We asked the reporter or the business something and cannot proceed until they answer           |
| Ready for Agent  | Fully specified, sliced to one pull request, and carrying an agent brief                       |
| Ready for Human  | Needs a person — judgement, external systems, manual verification, or a pull request to review |
| In progress      | Claimed and being worked                                                                       |
| Done             | Closed, for any reason (`completed`, `not planned` and `duplicate`)                            |

Everything else stays native and is never duplicated onto the board: **assignee** (who holds it),
**linked pull requests** (what is in review), **sub-issues** (decomposition), **issue
dependencies** (blocking), **closed** (finished).

**Triage empties `Backlog`.** Every issue arrives there and leaves by being routed to
`Ready for Agent`, `Ready for Human` or `Waiting on input`, or by being closed. Work considered and
deliberately deferred goes back to `Backlog` and is reconsidered next time — so `Backlog` never
means "already dealt with", and a full inbox is the normal state rather than a backlog of triage
debt.

**An empty `Waiting on input` is a healthy state**, not a query to debug: nothing is blocked on the
business.

**Prefer `Ready for Agent`.** It applies only when both halves of its row hold — fully specified
_and_ already sliced. When choices must be made before the work can be specified at all, that is
`Waiting on input`: a question to answer, not work to schedule. Only set `Ready for Human` when no
further work can be done without human assistance or judgement. Refer to the `/afk-loop` skill
for details on dispatching work to an agent.

**Claiming is an assignee write.** `gh issue edit <n> --add-assignee @me` is the atomic first
write that stops two agents taking the same issue; setting `Status` to `In progress` follows it.

**An issue that is not on the board has no state** and is invisible to every query below. Auto-add
workflows put newly-opened issues on the board; after creating one, give the workflow a moment and
then confirm it landed, adding anything that was missed.

**Adding one issue adds its whole tree.** Adding a parent pulls in every descendant, across
repository boundaries and including repos nobody had in scope. So
[planning being top-level only](#business-and-delivery) does not make the board top-level only —
most of it is children, and they arrive with `Cycle` unset. Any group-by-`Cycle` view therefore
carries a large **No Cycle** bucket of children whose parents are planned.

### Setting Status

`gh project item-edit` needs three IDs. Look them up once per session:

```bash
# project id, Status field id, and the option id for each status
gh api graphql -f query='
  query { organization(login: "clhbid") { projectV2(number: 4) {
    id field(name: "Status") { ... on ProjectV2SingleSelectField { id options { id name } } } } } }'

# the item id for an issue (an issue may sit on several projects — take project 4)
gh api graphql -f query='
  query($owner: String!, $repo: String!, $num: Int!) {
    repository(owner: $owner, name: $repo) { issue(number: $num) {
      projectItems(first: 10) { nodes { id project { number } } } } } }' \
  -f owner="$(gh repo view --json owner --jq .owner.login)" \
  -f repo="$(gh repo view --json name --jq .name)" \
  -F num=<number> \
  --jq '.data.repository.issue.projectItems.nodes[] | select(.project.number == 4) | .id'
```

**Take the repo from the checkout, never a literal.** The board spans every clhbid repo and issue
numbers are per-repo, so a pinned `repo=` resolves the wrong issue somewhere else — usually a
`NOT_FOUND`, but a number that exists in both repos returns a real item id for the wrong issue, and
`item-edit` accepts it.

Then:

```bash
gh project item-edit --project-id <project-id> --id <item-id> \
  --field-id <status-field-id> --single-select-option-id <option-id>
```

Read the value back afterwards — a wrong option id is accepted silently.

## Triage roles

The skills speak in terms of five canonical triage roles. In this org a role is a **`Status` value
on the CLHbid Delivery project**, not a label — see [Status](#status) above for how to read and set
it.

| Role in mattpocock/skills | Status here      | Meaning                                                                                 |
| ------------------------- | ---------------- | --------------------------------------------------------------------------------------- |
| `needs-triage`            | Backlog          | Not yet triaged — the inbox                                                             |
| `needs-info`              | Waiting on input | We asked a question and cannot proceed until it's answered                              |
| `ready-for-agent`         | Ready for Agent  | Fully specified and sliced; An agent brief has been added, and an agent can complete it |
| `ready-for-human`         | Ready for Human  | Needs a person                                                                          |
| `wontfix`                 | —                | Close with `--reason "not planned"`; the reason is the record                           |

When a skill says "apply the AFK-ready triage label", set `Status` to the value in this table.

`In progress` and `Done` have no counterpart in the skills' vocabulary — they carry the rest of the
lifecycle and are in the full table under [Status](#status).

## Business and delivery

**Decomposition is a delivery concern.** Work is broken into sub-issues so it can be tracked,
sliced and reviewed one pull request at a time — see
[Decomposing work before Ready for Agent](#decomposing-work-before-ready-for-agent). The business
does not track work at that grain: a **top-level issue is the unit of business work**, and its
children are how that work gets done.

The board carries both audiences as views — **📋 Delivery board** is everything, **💼 Business** is
`no:parent-issue`. The same split governs anything read outside the board:

- **Reporting to the business** — cycle notes, Status Updates, anything the business reads — is
  top-level only. Add `and .content.parent == null` to any recipe below to get its business view,
  as the **Planning view** recipe does.
- **Dispatching and doing the work** reads the leaves, because that is where a branch and a pull
  request attach. The **Agent frontier** recipe is the example: it excludes anything with children.

This is also why [`Cycle`](#cycles) is set on top-level issues only.

## Querying the board

Issue search cannot read project fields: `status:"In progress"` is not a qualifier and matches
nothing. Every state query therefore runs against the board with
[`board.graphql`](board.graphql) — one query, filtered per use with `--jq`.

**Find `board.graphql` first.** It sits beside this file, but where that is depends on how the
skill was installed. Resolve it once per session and reuse it:

```bash
for p in ~/.claude/skills/issue-tracker/board.graphql \
         ~/.agents/skills/issue-tracker/board.graphql \
         .claude/skills/issue-tracker/board.graphql; do
  [ -f "$p" ] && BOARD_QUERY="$p" && break
done
```

If none of them exists, say so rather than guessing — every recipe below needs it.

```bash
# Agent frontier — ready, leaf, unblocked, unclaimed
gh api graphql --paginate -F query=@"$BOARD_QUERY" --jq '
  .data.organization.projectV2.items.nodes[]
  | select(.content.state == "OPEN" and .status.name == "Ready for Agent"
           and .content.assignees.totalCount == 0
           and .content.subIssues.totalCount == 0
           and .content.issueDependenciesSummary.blockedBy == 0)
  | "\(.content.repository.name)#\(.content.number)  \(.content.title)"'

# Needs triage — the inbox
... | select(.content.state == "OPEN" and .status.name == "Backlog")

# Needs business input
... | select(.content.state == "OPEN" and .status.name == "Waiting on input")

# In flight
... | select(.content.state == "OPEN" and .content.assignees.totalCount > 0)

# Planning view — top-level work only
... | select(.content.state == "OPEN" and .content.parent == null)

# Status/state mismatch — the Item closed workflow missing one. Healthy result is empty
... | select((.content.state == "CLOSED" and .status.name != "Done")
          or (.content.state == "OPEN"   and .status.name == "Done"))

# Worked off-cycle — work that never got credited to the cycle it happened in
... | select((.content.state == "CLOSED"
              and .content.closedAt >= env.CYCLE_START and .content.closedAt < env.CYCLE_END
              and (.cycle.title // "") != env.CYCLE_TITLE)
          or (.content.state == "OPEN" and .status.name == "In progress" and .cycle == null))

# Stale closed items — auto-archive missed these. Healthy result is empty
... | select(.content.state == "CLOSED" and .isArchived == false
             and (.content.closedAt | fromdateiso8601) < (now - 1814400))

# Stale issues — actionable work untouched for three weeks
... | select(.content.state == "OPEN"
             and (.status.name | IN("Backlog", "Ready for Agent", "Ready for Human"))
             and (.content.updatedAt | fromdateiso8601) < (now - 1814400))

# ...of which newly stale — add this clause to keep the ones not yet stale when the cycle began
             and (.content.updatedAt | fromdateiso8601)
                 >= ((env.CYCLE_START | strptime("%Y-%m-%d") | mktime) - 1814400)
```

`...` stands in for the full command above. `--paginate` applies `--jq` per page, so filtering and
listing work as written; counting needs a pipe (`| wc -l`).

**Three weeks is 1814400 seconds**, and it is the definition of _stale_ — an item is stale on the
board, not in someone's judgement. The cycle recipes read `CYCLE_TITLE`, `CYCLE_START` and
`CYCLE_END` from the environment; set them from the iteration's `title`, `startDate` and
`startDate + duration`. `CYCLE_END` is **exclusive** — an iteration ends the day before the meeting
that closes it, so a cycle titled `14 Aug → 28 Aug` has `CYCLE_START=2026-08-14` and
`CYCLE_END=2026-08-28`.

**Newly stale is derived, never stored.** It compares one `updatedAt` against two thresholds: stale
against today, and not-yet-stale against `CYCLE_START`. There is no baseline to snapshot at the end
of a cycle. Anchoring it to `CYCLE_END` instead would ask which items went stale between today and
today, and return nothing.

**These recipes are canonical**, and no API returns a view's contents — so a view can never be
queried directly, only mirrored. `yarn board:sync` will assert each view's filter against this
table — it is not built yet:

| #   | View              | Filter                                             | Purpose                                   |
| --- | ----------------- | -------------------------------------------------- | ----------------------------------------- |
| 1   | 📋 Delivery board | _(none)_                                           | everything, grouped by `Status`           |
| 2   | 🎯 This cycle     | `cycle:@current`                                   | the running cycle                         |
| 3   | ⏭️ Next cycle     | `cycle:@next`                                      | what is planned next                      |
| 4   | 💼 Business       | `no:parent-issue`                                  | top-level work, delivered and outstanding |
| 5   | 🤖 Frontier       | `status:"Ready for Agent" no:assignee -is:blocked` | mirrors the frontier recipe               |
| 6   | ⏳ Waiting        | `status:"Waiting on input"`                        | mirrors the needs-business-input recipe   |
| 8   | 📥 Triage         | `status:Backlog no:parent-issue`                   | the inbox, top-level only                 |

`board:sync` checks **filters only**, and identifies each view by its **number** — the column above.
Numbers are stable and are not reused after a delete, which is why 7 is missing. Matching on
anything else breaks the audit: the filter is the value being compared, so it cannot also be the
key, and a view's name, columns and tab position are ergonomic choices belonging to whoever uses
it — treat a change to those as intent rather than drift.

## Cycles

A cycle is an iteration of the `Cycle` field — the title carries the theme, the dates carry the
schedule. An iteration **ends the day before the meeting that closes it**.

Read them from the field's `configuration`. `completedIterations` holds the closed ones; the
running cycle is the entry in `iterations` whose `startDate` is on or before today and whose
`startDate + duration` is after it, and the next is the earliest starting after that. Select by
date rather than by index — nothing guarantees the array's order.

`Cycle` is set on top-level issues only; children inherit their parent's by definition, so a
sub-issue with no `Cycle` is planned, not missed.

**`iterations` holds exactly two** — the running cycle and next — and next is the **parking lot**,
where deferred work is put. Keep it at two: a third is a plan someone has to maintain by hand,
which is what the parking lot exists to avoid.

## Deferring work

Deferring is a decision to record. Its form follows when the work comes back:

- **Next cycle** — set `Cycle` to next, the parking lot. The issue stays open and the board carries
  it.
- **A later date** — comment with the decision, the owner, and when it returns, then close with
  `--reason "not planned"`. The owner sets the calendar reminder; nothing in GitHub will raise it
  for them.
- **Unknown** — leave it in [`Backlog`](#status) for triage to reconsider.

**Reopen rather than refile** when it comes back, so the thread stays the history.

## Labels

Labels never carry state. Two matter:

- **`bug`** — something is broken.
- **`enhancement`** — everything else: new features, and the tooling, config and refactor work we
  used to call chores.

Issue templates apply exactly one of them. `/wayfinder` additionally uses `wayfinder:map` for a map
issue and `wayfinder:research` / `prototype` / `grilling` / `task` for its tickets.

Any other label is decoration — read it if you like, but nothing keys off it.

## Templates apply a category, never a state

Issue forms come from the org defaults in `clhbid/.github` under `.github/ISSUE_TEMPLATE/`. **A
person filing an issue should use one** — they collect fields a triager otherwise has to ask for.

A form applies exactly one **category** label (`bug` or `enhancement`) and **no state**. Category
says what kind of thing an issue is; `Status` says where it has got to. They are independent, and
nothing about the category implies a state.

**`gh issue create` does not apply a form**, and it is the normal path here — an agent has no
browser to fill one in. Creating directly is fine; it means you owe what the form would have done:

- Apply exactly one category label, and no state.
- Write the body the form would have collected, not a bare title and a sentence.
- Confirm the auto-add put it on the board.

A new issue is auto-added to the board and lands on `Backlog`, the triage inbox. That bucket is
supposed to fill up; see **Triage empties `Backlog`** under [Status](#status) for what moves an
issue out of it and what comes back.

## Commit convention

Commit subjects carry the issue number as a bare prefix — `2217: <subject>`. A trailing `(#NNNN)`
is the **pull request**, added by squash-merge, and is never the issue. When resolving the spec for
a diff, read the leading `NNNN:` and ignore the trailing `(#NNNN)`.

## Decomposing work before Ready for Agent

Keep `1 issue = 1 branch = 1 PR`. If work is too large, split the **issue**, not the pull request.
Size is a precondition of `Ready for Agent`.

**Smaller is better.** ~1000 changed lines is the ceiling — excluding lockfiles, snapshots and
generated files — but it is a limit, not a target: a changeset that splits cleanly should be split
well below it, because a reviewer reads a small diff and skims a large one. If a plan crosses the
ceiling, simplify first, dropping surplus documentation before anything else, then split. There is
no CI gate for this.

- A valid slice is **independently mergeable and green**. Behaviour-neutral slices (rename,
  extraction, refactor) count when they stand alone.
- **Stack the pull requests when slices depend on each other.** A pull request can target a branch
  other than `main`, and GitHub retargets it automatically when its base merges — so a dependent
  slice can be opened and reviewed straight away instead of waiting for its parent to land.
- **If an agent finds mid-flight that work is too large**, it simplifies, creates sub-issues, links
  them as children, sets the leaves to `Ready for Agent`, opens a pull request for the work it has
  finished — stacked on the previous slice where they depend on each other — and comments on the
  original issue explaining the cut.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / close**: `gh pr comment`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either — resolve with
`gh pr view 42` and fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies**. Add an edge with `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/<owner>/<repo>/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only — the live gate). A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children, drop any with an open blocker or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --add-assignee @me` — the session's first write.
- **Resolve**: `gh issue comment <n> --body "<answer>"`, then `gh issue close <n>`, then append a context pointer (gist + link) to the map's Decisions-so-far.

## Traps

**A wrong filter looks correct.** GitHub ignores an unknown qualifier rather than erroring. It can
fail in either direction — a typo may return everything, while `status:"In progress"` in issue
search returns nothing — so a result count proves nothing on its own. Check a new filter against a
query whose answer you already know.

**REST lies about parentage.** Nothing over REST will tell you an issue's parent:
`gh issue view --json parent` errors outright, and
`gh api repos/{owner}/{repo}/issues/{n} --jq .parent.number` returns `null` for a parented issue
exactly as it does for a top-level one. GraphQL's `Issue.parent` answers truthfully — it is what
[`board.graphql`](board.graphql) selects — and the only other direction is downwards, listing a
candidate parent's `sub_issues`. Checking issue by issue costs a request per candidate and will be
skipped under pressure, so **put `no:parent-issue` (or `.content.parent == null`) in the query
itself**. Anything relying on someone remembering to check drifts back into reporting sub-issues to
the business.

**A read-back can be stale.** The project API is eventually consistent. Verify every write by
read-back, but treat a mismatch straight after a write as unconfirmed rather than failed — re-check
after a delay, and never re-issue the write on the strength of one disagreeing read.
`issueDependenciesSummary.blockedBy` lags the same way — it read `0` immediately after a blocker was
linked and `1` shortly after — so the **Agent frontier** recipe run straight after a write can hand
out an issue that is already blocked.
