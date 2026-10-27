---
name: cycle-review
description: Run the fortnightly-ish cycle-review meeting round trip on the CLHbid Delivery board — publish the meeting notes discussion, process the returned notes back into cycles and statuses, or adjust a cycle's membership.
---

# Cycle Review

The cycle review is a **round trip through a GitHub Discussion**. Branch A publishes the meeting
notes as a Discussion and drafts them as the email agenda, the meeting works through them, the
project manager posts their decisions as a comment, and branch B turns those decisions into tracker
writes. **Before branch A or B, load `house-rules` and `writing-lean`.**

**The Discussion is the record**, and the baseline the next cycle's notes are diffed against. See
[Editing Discussions](#editing-discussions) for where they live and how to read and write one.

**The notes are not edited during the meeting.** The project manager posts a comment using the same
headers, and branch B integrates it afterwards.

A cycle is an iteration of the `Cycle` field; work state is the `Status` field. Both live on the
CLHbid Delivery project. **Read the field mechanics, the `Status` values, the reference convention
and every board query from the `issue-tracker` skill** — this skill names the recipes it needs and
never restates them, so the two cannot drift.

## Writing for the business

**The notes are one document, written for the business.** The business has no GitHub access, so
every item makes sense on its own in plain language and ends with a reference to its issue, for the
project manager. No agent commentary — tooling state, recipe caveats and process notes go to the
project manager in conversation or into your own TODO list.

**In the project manager's voice.** The notes and the email are theirs, drafted on their behalf.
While they are the only developer, development work is "I" ("I'm building the fixes"), never "we";

**The notes are also the email agenda**, sent **two business days before the meeting**, weekends
excluded — a Tuesday review means Friday — so decisions can be settled by reply and the meeting
takes only what email couldn't. Create it as an **Outlook draft the project manager sends; never
send it**. Address it to the meeting invite's invitees, and give it the notes' text, Decisions
first, GitHub references removed — the business can't open them — and the meeting link last.

**Email first.** Business decisions and triage, bugs included, go to the business by email as they
arrive — see **Ask the business by email** in `issue-tracker`. Developer-only triage never reaches
the business, by email or in the notes.

**Business work only, except the counts.** Every item in the notes is business work, and developer
work shows up only in the whole-board counts in Last Cycle. The `business` filter — see **Business
and delivery** in `issue-tracker`: top-level issues and the children of epics — is the starting
point, not the answer: **propose which items are business work** and let the project manager
decide item by item, either way.
The one thing that must not be lost to the filter is a **question**: when the issue blocking on
business input is a sub-issue of an ordinary parent, describe the parent's work and reference the
child, so branch B knows which issue the answer lands on.

**Aim for about 60 lines and tables of up to five rows.** These are goals, not hard limits. Size
each section against a 30-minute meeting with this budget; the notes show only the order.

| Section               | Minutes |
| --------------------- | ------- |
| Last Cycle            | 3       |
| Upcoming Dates        | 4       |
| Next Review           | 1       |
| Decided by Email      | 1       |
| Decisions             | 12      |
| This Cycle            | 4       |
| Cycle After & Backlog | 4       |
| Next Actions          | 1       |

**Number in reading order.** Decided by Email items are `E1`, `E2`…; Decisions are `D1`, `D2`….
Until the agenda is emailed, renumber whenever an item moves so the numbers still run in reading
order. **Once it is emailed, numbers are fixed**: an item added later takes the next unused number,
sits in the section it belongs to, and is marked _(added after the agenda)_.

**Use emoji sparingly, where they make the notes quicker to scan**: ❓ marks each item awaiting an
answer, and branch B turns it into ✅ once the meeting answers it. One per item, at its start, and
none as decoration.

## Cycle roles

Roles are resolved from the `Cycle` field's `configuration` against the **meeting date**, never
today, by array order or by reading a title:

| Role    | Where it comes from                                                                          |
| ------- | -------------------------------------------------------------------------------------------- |
| closing | the iteration ending the day before the meeting, in `iterations` or `completedIterations`    |
| current | the iteration starting on the meeting date                                                  |
| future  | the two iterations starting after current                                                    |

An iteration **ends the day before the meeting that closes it**, so notes prepared ahead of the
meeting find the closing cycle still running. **Branch A needs the meeting date and stops without
one**; branch B takes it from the notes' title; branch C, which has no meeting, resolves the running
cycle against today as `issue-tracker` describes. **If the roles cannot be resolved, stop and say
so** — never invent an assignment.

Every membership change is `gh project item-edit` against the `Cycle` field, on **committed units
only** — a top-level issue or a child of an epic; descendants inherit, and an epic's own `Cycle` is
never set. Every state change is a `Status` value from the `issue-tracker` role map. The
milestones API plays no part in this skill.

**Confirm before every mutating step.** Show the proposed cycle and status values, closures, issue
body edits, notes body and iteration-configuration diff, get an explicit go-ahead, then read the
write back. A read-back that disagrees straight after a write is unconfirmed, not failed — re-check
after a moment rather than re-issuing it.

## Invocation

- **A. Prepare** — `/cycle-review prepare for the 2026-08-28 meeting, next one is 2026-09-11`
- **B. Process notes** — `/cycle-review process the notes from discussion 2323`
- **C. Adjust** — `/cycle-review move clhbid/clhbid.com#2325 into the current cycle; defer
  clhbid/CLHbid-LiveAuction#922 to the next one`

## Branch A — Prepare the notes

Branch A publishes the notes discussion and writes to the tracker only in bookkeeping steps 2–5.
**They run first**, before a line of the draft is written, because they change what the notes say.
No meeting decides them: **their findings, and every write they propose, go to the project manager
as the first `grill-me` round** — numbered questions, each with a recommendation — and the answers
are the go-ahead.

1. **Check Microsoft 365.** Run a mail search, and a calendar search for the meeting, through the
   Microsoft 365 connector, finding its tools by capability. **If either fails, stop** and tell the
   project manager to fix the connector before continuing. Outlook is then a source for decisions
   settled by email, questions still waiting on a reply, the meeting's invitees and join link, and
   the status of issues discussed only by email.
2. **Open the next cycle.** Current and both future cycles must exist. If either is missing, add
   it; if the meeting has moved off the planned date, the roles will not resolve until the new
   dates are written. **This is a destructive configuration write** — follow the procedure in
   `issue-tracker`, and fold every other pending change into the same write, because a second one
   costs a second restore of the whole board.
3. **Housekeeping.** Run the `issue-tracker` **stale closed items** recipe and flag what it
   returns, along with any issue that should be on the board and is not. **Both are healthy empty**,
   so a result is a signal that the sweep behind them is failing. Archive by hand with
   `archiveProjectV2Item` only if the meeting cannot wait. Then run the **Epic candidates** recipe,
   read each result, and raise the ones that slipped or whose open children plainly exceed a cycle
   — work committed "this cycle and the next few" is the signature. **Converting is the project
   manager's call, made before the meeting**, never a Decision: for each agreed one, PATCH its
   type, clear its `Cycle`, set `Cycle` on the children being committed and move its `Status` along
   the ladder — see **Epics** in `issue-tracker`. Raise what the **Epics ready to close** recipe
   returns too; an epic reaches Decisions only if the business must decide. **All of this stays
   out of the notes.**
4. **Merged, still open.** Run the `issue-tracker` **merged pull request, open issue** recipe.
   GitHub closed nothing for these — the recipe says why — so each is finished work that no cycle
   report has ever credited. Every row ends one of two ways: closed as `completed` naming the pull
   request, or judged still live because the pull request was a deliberate partial. **This stays
   out of the notes** for the same reason step 3 does.
5. **Worked-off-cycle backfill.** Run the `issue-tracker` **worked off-cycle** recipe against the
   closing cycle's window and keep the business results — top-level issues and epic children. A
   sub-issue of an ordinary parent needs no write — it inherits its parent's cycle. This is
   bookkeeping, not triage: no meeting decision.
   - **Closed** work is credited to the closing cycle — set `Cycle` — and is a candidate for a
     Last Cycle highlight.
   - **`In progress`** work touched during the closing cycle was picked up without ever being
     committed to it. Credit it to the current cycle — unfinished work belongs to the cycle that
     will finish it — and list it under This Cycle.
   - **Dormant** work — the **dormant** variant of the same recipe, untouched since before the
     closing cycle began — gets **no write**. Raise it with its status flagged **suspect**. Never
     auto-credit it, and never reset it to `Backlog`.
6. **Read the board.** Project items, the `Cycle` configuration, the previous cycle's notes
   discussion, and Outlook since the last review.
7. **Open the draft.** Create a Claude Doc through the docs connector and draft the notes straight
   into it; the project manager comments on and edits it there. **Read their edits before every
   write, and keep them.** With no docs connector, draft in a local Markdown file and say so.
8. **Draft Last Cycle**, a TL;DR under the closing cycle's theme and dates:
   - **A one-word verdict** on its goal: met, partly met or missed.
   - **Whole-board counts** — opened, closed and open, with the trend since the last review — and
     how many of the opened issues were developer work, so the business knows it exists without
     seeing it listed.
   - **Up to five highlights**, including progress on epics the business cares about and any epic
     closed during the window.
   - **Slipped items**, each with its reason, filled **only from direct evidence**: a closing pull
     request, a comment, sub-issue state. Leave the reason out rather than infer one. Diff the
     membership against the previous notes so slippage is visible rather than silently absorbed.
9. **Draft Upcoming Dates**, a `Date | What` table, one row per date, running to the end of the
   first future cycle: sales, business dates such as conventions, proposed releases each on its own
   row, and out-of-office. Dates are bold so they stand out, and each row opens with its kind
   (`Sale:`, `Release (proposed):`). Keep only what affects planning. **Carry forward every date
   from the previous notes that still falls in the window.** Sale dates and release-date rules
   defer to the `RUNBOOK` in `clhbid/CLHbid-LiveAuction` — **Checking the Sale Schedule** and
   **Choosing a Release Date** — never the homepage, which lists only the next few sales. Close
   the section asking attendees for anything missing, and: _Any high-risk sales we should avoid
   releasing around?_
10. **Draft Next Review** — date, time and location, from the invocation or proposed. It comes
    **before** Decisions because it sets when the current cycle ends, and therefore how much fits
    in it. **Never on a sale day.**
11. **Draft Decided by Email** — a brief `ID | Description | Decision` table of decisions settled by
    email since the last review, kept for reference. Find them in Outlook and in the answers
    recorded on issues since then, and confirm the list with the project manager.
12. **Draft Decisions** — an `ID | Description | Recommendation` table. It holds every issue still
    `Waiting on input`, every Outlook question still waiting on a reply, and anything else that
    needs real discussion, such as stale work the project manager has triaged and brought for
    input, **one row each**, never summarised as a count. The description opens with a bold
    plain-language title and carries the rationale; the recommendation is something the team can
    accept as written.

    **The question must already have been asked** — a plain comment on the issue, or the email, is
    enough. If neither asks it, that is an **error**: the question has never been put to
    anyone, so say so rather than reconstructing one from the title.
13. **Draft This Cycle** — the current cycle's theme and dates, why in one sentence, and bullets of
    the main work, carrying anything the backfill already credited. **We finish what we start**:
    work already `In progress` takes precedence, and new work should not be accepted into the cycle
    while it is outstanding — tell the project manager how much there is.
14. **Draft Cycle After & Backlog** — the first future cycle's theme and main work, then business
    items deliberately left without a cycle and work the team has asked about that has none. Work
    committed to the second future cycle goes here too, under its name. **Before anything stays in
    the backlog, ask whether it's worth doing at all**; if not, recommend closing it as not planned,
    with a reminder if it should come back — see **Deferring work** in `issue-tracker`.
15. **Draft Next Actions**, last — present but empty, showing the owner-first shape.
16. **Check before publishing**, and fix the draft rather than note the gap. Settle gaps and
    decisions the draft raises with `grill-me`, one round at a time:
    - epics and items closed since the last draft are credited to a cycle and reflected in the
      highlights;
    - work the team has asked about has a cycle, or is listed under Backlog;
    - sale dates come from the `RUNBOOK`'s sources;
    - every item reads on its own, without GitHub;
    - D and E numbers run in reading order, apart from items added after the agenda went out.
17. **Reconcile against the live board.** Every cycle section must agree with what the board
    actually says, item for item.
18. **Publish.** When the project manager calls the notes final, create the Discussion from the
    doc's text, or update it if this cycle's notes already exist, then create the email draft.
    Delete the doc when the project manager asks. **Branch A is re-runnable**: run it again
    whenever the board changes and it revises the same discussion. **Read the live body before
    every update**, and build on it: the project manager edits the notes on GitHub too.

See [The notes](#the-notes) for the shape, drafted without the **Decision** column and with **Next
Actions** blank; both are filled in on the way back.

## Branch B — Process the returned notes

Read the published body and the project manager's comment from the API by discussion number.

**Resolve references** per **References** in `issue-tracker`: the comment may use bare numbers and
aliases, and a bare `#<number>` resolves against `clhbid/clhbid.com`, the repo hosting the discussion.

1. **Last Cycle** — the discussion is the record and the backfill already set every `Cycle` in
   branch A. Nothing to write.
2. **Sweep.** **No open item may still carry the closing cycle** (`cycle:@previous is:open` on the
   board). Reassign every straggler with `gh project item-edit` — usually to current, or clear the
   field if the work was dropped. An orphan left behind is invisible to every cycle-scoped view
   from then on.
3. **Upcoming Dates** — informational; no tracker write. The next notes carry them forward.
4. **Next Review** — the agreed date sets the current cycle's end. Apply it to the `Cycle` field's
   configuration. **The write is destructive** — follow the procedure in `issue-tracker`. Branch A
   has usually already made this cycle's one write, so an agreed date should have gone in with it.
   Then create a private one-hour **Cycle review prep** calendar event, with no attendees, on the
   business day two before that review.
5. **Decided by Email** — the writes were made when each answer came in. Check each issue carries
   its answer, and record any that doesn't from Outlook as a Decision.
6. **Decisions** — for each answered question, record it on the issue in two places: a comment
   quoting the question, prefixed `> *Recorded from the <date> planning meeting.*` — `email` for an
   email decision — and **the answer folded into the issue body** so someone picking the work up
   cold has the whole spec. The comment is the audit trail; the body is the spec. Never delete what
   was there. An item the meeting could not resolve is **left untouched**, `Waiting on input` and
   all.

   Then route it off `Waiting on input`, and only when **every** question on the issue is answered:
   `Ready for Agent` **only if the updated body now reads as a complete brief an agent could work
   from cold**; `Ready for Human` if it needs judgement, external access, a design decision or
   manual testing; `Backlog` if it is answered but still underspecified. An answer that raises a
   **new** question has not unblocked anything: post the new question as a comment in the same
   shape and leave the issue on `Waiting on input`.

   A decision that commits or defers work sets `Cycle` or follows **Deferring work** in
   `issue-tracker`. An agreed epic close is `gh issue close --reason completed`; an epic kept open
   has what remains filed as a new child, so its progress stops reading complete.
7. **This Cycle** and **Cycle After & Backlog** — set `Cycle` on each committed unit, skipping
   anything the branch A backfill already credited. The first child of an epic committed to any
   cycle moves the epic to `In progress` if it is not there already. Backlog work agreed not worth
   doing is closed `--reason "not planned"`, with a reminder comment if it should come back; the
   rest stays in `Backlog` with no `Cycle`.
8. **Next Actions** — decide by **what the line describes, not who owns it**: the owner tells you
   nothing, since the project manager's own lines cover both delegated tracker work and follow-ups
   they handle themselves. Execute the tracker actions — cycle and status changes, closures, the
   new-issue draft. Leave person-to-person follow-ups alone.
9. **Draft an issue** for work in the notes that matches nothing on the board — category type
   only, no state, body drawn from the notes. It is new input, so it lands in `Backlog` like
   anything filed from a template.
10. **Update the discussion** with the decisions integrated — a **Decision** column on the
    Decisions table, each answered ❓ turned to ✅, and Next Actions filled in — then **re-check
    for issues closed since the notes were published**: meeting-morning merges land after the cut and would otherwise be credited to
    the wrong cycle.

## Branch C — Adjust

Move membership with `gh project item-edit` on `Cycle`, or clear it when work is dropped, then note
the change in the next notes discussion.

## The notes

Below is a **completed** set of notes — the input branch B receives, with the **Decision** column
and **Next Actions** filled in. It is also the target branch A drafts toward: the same headings,
the same order, the same prompts, without those. Reproduce this shape.

Notice the register: outcomes in business language, owner first on **Next Actions**, a reference
closing every item. Field names and `gh` syntax never appear — the team reads this, and branch B
recognises a tracker action by what it describes.

_Illustrative example. The organisation, people, repositories, issue numbers and titles are
invented. Nothing here is real work._

````markdown
# Cycle Review — 18 Mar 2026

## Last Cycle: Faster checkout (4 Mar → 17 Mar)

**Met.** 41 issues opened and 47 closed, leaving 212 open, down from 218 at the last review. 29 of
the new issues were behind-the-scenes developer work.

- Checkout finishes in under two seconds, even on sale days. (acme/shop#412)
- Customers can save a card for next time. (acme/shop#418)
- Gift cards are 5 of 8 steps done; balance lookup went live on 12 Mar. (acme/shop#400)
- Refund rounding is fixed, after finance flagged it mid-cycle. (acme/shop#430)

**Slipped:** moving saved cards to the new payment provider. The provider limits how fast cards can
be imported, so it runs in batches this cycle. (acme/shop#424)

## Upcoming Dates

| Date                | What                                |
| ------------------- | ----------------------------------- |
| **Sat 21 Mar**      | Sale: Spring sale                   |
| **Tue 24 Mar**      | Release (proposed): better search   |
| **Tue 24 – 26 Mar** | Jordan out                          |
| **Sat 28 Mar**      | Sale: Clearance sale                |
| **Tue 7 Apr**       | Trade show; Priya and Sam attending |
| **Thu 9 Apr**       | Release (proposed): gift cards      |
| **Sat 11 Apr**      | Sale: Easter sale                   |

Anything missing? Any high-risk sales we should avoid releasing around?

## Next Review

**Wed 1 Apr 2026, 10:00 AM MT**, by video call — link in the calendar invite.

## Decided by Email

| ID  | Description                                                              | Decision         |
| --- | ------------------------------------------------------------------------ | ---------------- |
| E1  | **Delivery estimates on product pages.** (acme/shop#441)                 | Yes, this cycle  |
| E2  | **Coupons that expire mid-checkout show an error.** (acme/shop#447)      | Fix after search |

## Decisions

| ID  | Description | Recommendation | Decision |
| --- | ----------- | -------------- | -------- |
| ✅ D1 | **Who sees the new prices first?** Switching everyone at once risks confusing regulars mid-order; a staged rollout lets us watch support calls. (acme/shop#433) | Returning customers first, everyone two weeks later. | Agreed. |
| ✅ D2 | **Pay for a hosted search service?** Our search struggles on sale days, but the service costs about $400 a month and this cycle's work may fix it. (acme/api#96) | Decide after this cycle, with real sale-day numbers. | Agreed; back as a Decision on 1 Apr. |
| ✅ D3 | **Close the old admin project?** Everything is done except one old web address still pointing at the retired server. (acme/shop#380) | Close it once the address is removed this cycle. | Agreed. |

## This Cycle: Search that finds things (18 Mar → 31 Mar)

Empty search results are our biggest source of complaints, and customers give up on sale days.

- Finish moving saved cards to the new provider. (acme/shop#424)
- Search copes with hyphens and misspellings. (acme/shop#437)
- Delivery estimates on product pages. (acme/shop#441)

## Cycle After & Backlog

**Gift cards (1 Apr → 14 Apr):** finish gift cards and launch them before the Easter sale.
(acme/shop#400)

**Backlog:**

- Loyalty points, asked about by Priya — waits until gift cards have launched. (acme/shop#450)
- Dark mode for account pages — nobody has asked since it was filed; close it, and reopen if that
  changes. (acme/shop#444)

## Next Actions

- **SAM:** Remove the old web address, then close the old admin project.
- **SAM:** Close dark mode as not planned.
- **JORDAN:** Collect sale-day search numbers for the hosted-search decision on 1 Apr.
````

## Editing Discussions

Notes go in the
[**Meetings** category](https://github.com/orgs/clhbid/discussions/categories/meetings).

**Every org-wide discussion is hosted by the `clhbid/clhbid.com` repository**, whatever
`github.com/orgs/clhbid/discussions` implies. That is the repo the API addresses them through, and
it is why a bare `#<number>` in the notes resolves against `clhbid.com`.

`gh` has no discussion command, so use `gh api graphql`: `createDiscussion` and `updateDiscussion`
to write, `repository { discussion(number:) }` to read a body and its comments back.

## Iterating on the skill

**Fix the skill, not the output.** Whenever a draft needs hand-editing, in the doc or on GitHub,
that edit is a bug report. Change the skill and re-run rather than patching the notes by hand.
