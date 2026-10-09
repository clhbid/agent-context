# The review

One GitHub review in the reviewer's name: a summary, plus one inline comment per finding.

## The draft

Keep the draft in your scratchpad, outside the repository under review. It holds the agreed
findings, and the local-testing checklist under its own heading.

## A finding

Number each finding `F<N>` when you propose it, continuing from the highest number earlier reviews
on this pull request used, and keep that number through to the posted review; a rejected finding's
number goes unused. Each agreed finding is one inline comment:

```
**F<N> <label> (blocking|non-blocking): <title>**

<The problem, because <why it matters>.> <The fix.>
```

Labels come from [Conventional Comments](https://conventionalcomments.org/):

| Label | Use for | Default |
| --- | --- | --- |
| `issue` | Something wrong: a leak, a bug, a broken contract | blocking |
| `todo` | Missing work the pull request needs: a spec, a doc, a feature the reviewer requested | blocking |
| `suggestion` | A clean-up that improves the code | non-blocking |
| `nitpick` | A small style point | non-blocking |
| `note` | Context or a problem that predates the pull request, out of scope | non-blocking |

The reviewer can make any finding blocking or non-blocking; the decoration in the title says which.

Write about the code and what it does. When the fix is a few lines that sit entirely in the diff,
write it as a GitHub `suggestion` block so the author can commit it in one click.

**Anchor every comment to a line in the diff**, on the right-hand side; GitHub rejects the whole
review if one comment lands elsewhere. When the real location isn't in the diff, anchor to the
nearest changed line that calls or uses it, and name the real `file:line` in the text.

**A carried finding** comes from an earlier, unresolved thread and keeps that thread's number. When
the thread already states the fix, list it under **Still open from earlier reviews** with its link
and blocking status, and post no new comment. When the fix is new, or the thread never stated it,
post a new comment that links the thread.

## The summary

In this order:

1. When an agent wrote the pull request, open with an explicit mention, such as
   `@copilot please address the review comments below on this branch.`
2. The verdict in one sentence.
3. Check results, citing CI and anything run locally, and naming any still pending.
4. One or two specific, true things the pull request does well.
5. The requested findings by number, in priority order: fix (`issue`), add (`todo` features),
   missing specs (`todo` specs), clean-ups (`suggestion`, `nitpick`), then notes.
6. **Still open from earlier reviews**: each carried thread, linked.
7. **Not reviewed**: steps skipped or only skimmed, generated files, lockfiles.
8. What the pull request description must say once the changes land.

The concepts the reviewer learned stay in chat; the review carries only findings.

## The event

Recommend one, and let the reviewer choose:

- `REQUEST_CHANGES` when any finding is blocking.
- `COMMENT` when every finding is non-blocking, and always when the reviewer is the pull request's
  author, because GitHub refuses a request for changes on your own pull request.
- `APPROVE` only when the reviewer picks it.

## Posting

1. Check every anchor against the diff's hunks; see [GITHUB.md](GITHUB.md).
2. Show the reviewer the full draft: the summary and every comment.
3. Once the reviewer approves the draft, post the review in one request with the head SHA you
   reviewed, then read it back: its state, and that it carries one comment per finding.

The reviewer may submit it themselves, for example to choose the model Copilot uses. Then post it
**pending**, without an event, and give them the summary in a copyable Markdown block, because
GitHub shows a pending review's body only once it's submitted. A finding agreed after the draft
was shown goes into the pending review; see [GITHUB.md](GITHUB.md#changing-a-pending-review).
