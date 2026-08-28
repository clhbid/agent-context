---
name: cycle-review
description: Run the fortnightly-ish cycle-review meeting round trip on the CLHbid Delivery board — prepare the meeting notes, process the edited notes back into cycles and statuses, or adjust a cycle's membership. User-invoked only.
disable-model-invocation: true
---

# Cycle Review

The cycle review is a **round trip through a document**. This skill drafts Markdown meeting notes,
Mark pastes them into a Google Doc, the meeting edits the doc, Mark pastes the edited Markdown
back, and this skill turns the decisions into tracker writes. The skill never touches the Doc — no
Drive access exists yet.

The Doc is the meeting's working surface, not the record. **The record is two Project Status
Updates** posted straight to the board: a **closing** update carrying the outcome of the cycle just
ended, and an **opening** update carrying the scope committed for the cycle just starting. They are
posted in branch B and they are what anyone reads later to see what was planned versus delivered.

A cycle is an iteration of the `Cycle` field; work state is the `Status` field. Both live on the
CLHbid Delivery project. **Read the field mechanics, the `Status` values and every board query from
the `issue-tracker` skill** — this skill names the recipes it needs and never restates them, so the
two cannot drift. The project spans repositories, so every issue reference in the notes and in the
Status Updates is `<repo>#<number>`; a bare number is ambiguous and resolves wrong.

## Cycle roles

Three roles, resolved from the `Cycle` field's `configuration` by date, never by array order or by
reading a title:

| Role    | Where it comes from                                                             |
| ------- | ------------------------------------------------------------------------------- |
| closing | the most recent entry in `completedIterations` — the cycle the meeting closes    |
| current | the entry in `iterations` whose `startDate + duration` brackets today            |
| next    | the earliest entry in `iterations` starting after current — the parking lot      |

An iteration **ends the day before the meeting that closes it**. **First run: no prior cycle and no
prior Status Update exist.** Say so in the notes rather than inventing a baseline to diff against.

Every membership change is `gh project item-edit` against the `Cycle` field, on **top-level issues
only** — children inherit their parent's cycle. Every state change is a `Status` value from the
`issue-tracker` role map. The milestones API plays no part in this skill.

**Confirm before every mutating step.** Show the proposed cycle and status values, closures, issue
body edits, Status Update text and iteration-configuration diff, get an explicit go-ahead, then
read the write back. A read-back that disagrees straight after a write is unconfirmed, not failed —
re-check after a moment rather than re-issuing it.

## Invocation

- **A. Prepare** — `/cycle-review prepare for the 2026-08-28 meeting, next one is 2026-09-11`
- **B. Process notes** — `/cycle-review notes from the 2026-08-28 meeting:` + pasted Markdown
- **C. Adjust** — `/cycle-review move repo1#120 into the current cycle; park repo1#125`

## Branch A — Prepare the notes

Branch A produces one Markdown document and writes nothing to the tracker except the two automatic
bookkeeping steps below. **They run first**, before a line of the draft is written, because both
change what the tables say. _Automatic_ means no meeting decides them — they still show their list
and take a go-ahead, like every other mutation.

1. **Housekeeping.** Run the `issue-tracker` **stale closed items** recipe. Archive every result
   with `archiveProjectV2Item` and report what you archived to Mark in conversation. **This stays
   out of the notes** — the auto-archive workflow missing a few items is not a decision the meeting
   makes.
2. **Worked-off-cycle backfill.** Run the `issue-tracker` **worked off-cycle** recipe against the
   closing cycle's window. Set `Cycle` immediately on everything it returns: **closed** work is
   credited to the cycle it was completed in, **`In progress`** work with no cycle is credited to
   the cycle now opening. This is bookkeeping, not triage — no meeting decision, no `Decision:`
   line. The backfilled work then shows up in the draft like any other mid-cycle addition:
   completed items as extra `➕ Added mid-cycle, done` rows in **Last Cycle**, in-progress items as
   entries already in **Committed This Cycle**.
3. **Read the board.** Project items, the `Cycle` configuration, and the prior Status Updates.
4. **Draft Last Cycle.** Restate the closing cycle's goal, then table every issue that was
   committed to it, with `Result` and `Reason` columns **left blank for the meeting to fill in**.
   Diff the membership against the **previous opening Status Update's** frozen scope so slippage
   and mid-cycle additions are visible rather than silently absorbed.
5. **Draft Current Cycle**, sections in this order, because each one feeds the next: the proposed
   next-cycle end date (from the next meeting date, marked **proposed** — the iteration's dates are
   not written at this step), **New Issue Triage** (every issue opened since the closing cycle
   started, `Decision` blank), **Stale Issues** (step 6), **Waiting on Input** (every
   `Status = Waiting on input` issue, listed **one at a time** with its own `Decision`, never
   summarised as a count), **Significant Dates** (a blank prompt), then **Committed This Cycle**
   and **Parking Lot** left empty apart from anything the backfill already credited.

   Each **Waiting on Input** line is a one-line summary of the question plus its link; the question
   itself lives on the issue as a Triage Notes comment. **Confirm that comment exists before
   listing the item.** If it doesn't, the question has never actually been put to anyone — say so
   rather than reconstructing one from the title, because that gap means the `Status` is wrong.
6. **Draft Stale Issues.** Run the `issue-tracker` **stale issues** recipe and report two numbers:
   the total stale, and how many went **newly stale** during the cycle just closing — that second
   number is derived from the same `updatedAt`, anchored to the closing cycle's `startDate`, not
   from a baseline anyone stored. Then do the reading:
   - **Close candidates** — a handful, judged on _old_, _underspecified_ or _duplicate_. This means
     reading each candidate's body and cross-checking the board for duplicates. Sorting by age is
     not the job.
   - **Needs a cycle, not a close** — stale work that plainly still needs doing, flagged for
     pulling into this cycle's triage instead.
   - **Newly stale, worth a look** — anything that went stale this cycle and looks important on its
     own evidence: comment volume, a prior assignee, other issues referencing it. A judgement call,
     not a threshold.

   Every item in all three groups carries its own blank `Decision:`, exactly like every other
   review section.
7. **Draft the closing sections.** **Out of Office** and **Significant Dates** are blank prompts
   for the team — they exist for the meeting, not for the tracker. **Next Actions** is present but
   empty, showing the owner-first shape. **Next Cycle Review** is templated with the date, time and
   location of the recurring slot.
8. **Present the draft in the conversation and stop.** It is a draft until Mark returns it edited.
   Write it nowhere.

**Every section is worked one decision at a time.** That is why each triage line, each
`Waiting on input` question and each stale candidate carries its own `Decision:` rather than a
shared verdict at the end of a table — a section with one decision on it gets skimmed, and the
items underneath get silently agreed to.

See [The notes](#the-notes) for the shape, with every `Decision`, `Result`, `Reason` and
**Next Actions** line blank in the draft and filled in on the way back.

## Branch B — Process the returned notes

The notes arrive as Markdown pasted out of a Google Doc, so **parse for meaning, not for exact
syntax**. The round trip reliably mangles them: `#` comes back escaped as `\#`, headers collapse
into bold paragraphs, bare domains arrive auto-linked. Read the sections and the decisions through
the damage rather than failing on it.

1. **Last Cycle** — its table and decision become the **closing** Status Update's content, the
   backfilled `➕ Added mid-cycle, done` rows included. Their `Cycle` was written in branch A;
   there is nothing left to write here.
2. **Sweep, then post.** Before the closing Status Update goes out, **no open item may still carry
   the closing cycle** (`cycle:@previous is:open` on the board). Reassign every straggler with
   `gh project item-edit` — usually to current, or clear the field if the work was dropped. This
   step is dull and sits in front of the interesting work, which is exactly the shape that invites
   skipping it; an orphan left behind is invisible to every cycle-scoped view from then on.
3. **New Issue Triage** — apply each line: set `Cycle` and `Status` with `gh project item-edit`, or
   close the issue with `--reason "not planned"`.
4. **Stale Issues** — apply each line: close candidates are closed `--reason "not planned"`; work
   pulled into a cycle gets its `Cycle` set and joins the Committed / Parking Lot lists like
   anything else triaged at this meeting; items merely flagged as worth watching get **no write at
   all** — they are informational, and they resurface in next cycle's stale report on their own.
5. **Waiting on Input** — for each answered question, record it on the issue in two places: reply
   to the Triage Notes comment that asked it, prefixed
   `> *Recorded from the <date> planning meeting.*`, and **fold the answer into the issue body** so
   someone picking the work up cold has the whole spec. The comment is the audit trail; the body is
   the spec. Never delete what was there. An item the meeting could not resolve is **left
   untouched**, `Waiting on input` and all — it reappears in next cycle's notes by itself.

   Then route it off `Waiting on input`, and only when **every** question on the issue is answered:
   `Ready for Agent` **only if the updated body now reads as a complete brief an agent could work
   from cold** — this is the gate that keeps the AFK queue trustworthy; `Ready for Human` if it
   needs judgement, external access, a design decision or manual testing; `Backlog` if it is
   answered but still underspecified, said out loud rather than guessed at. An answer that raises a
   **new** question has not unblocked anything: post the new question as a Triage Notes comment in
   the same shape and leave the issue on `Waiting on input`.
6. **Significant Dates** — folded into the **opening** Status Update's narrative. No per-issue
   action.
7. **Out of Office** — informational only. It is context for what the team could commit to; it
   produces no tracker write.
8. **Committed This Cycle** and **Parking Lot** — set `Cycle` on each top-level issue, skipping
   anything the branch A backfill already credited.
9. **Next Actions** — decide by **what the line describes, not who owns it**. `MARK:` covers both
   tracker work he delegates to you and follow-ups he does himself, so the owner tells you nothing.
   Execute the tracker actions — cycle and status changes, closures, the new-issue draft, the
   Status Update posts, the cycle end-date change. Leave person-to-person follow-ups alone.
10. **Post both Status Updates** with `createProjectV2StatusUpdate` — closing (Last Cycle's
    outcome) and opening (the committed list, with Significant Dates in the narrative) — and read
    each back.
11. **Apply the agreed next-cycle end date** to the `Cycle` field's `iterationConfiguration` via
    `gh api graphql` on `updateProjectV2Field`. **That input replaces the entire iteration list in
    one call** — there is no per-iteration patch. So: read the full list, change only the target
    entry, show the diff, get confirmation, then write it back. Ad-hoc `gh` like every other write
    in this skill; no script gets built for it.
12. **Draft an issue** for work in the notes that matches nothing on the board — category label
    only, no state, body drawn from the notes. It is new input, so it lands in `Backlog` like
    anything filed from a template.
13. **Re-check for issues closed since the closing update posted.** Meeting-morning merges land
    after the cut and would otherwise be credited to the wrong cycle.

## Branch C — Adjust

Move membership with `gh project item-edit` on `Cycle`, or clear it when work is dropped, then note
the change in the **next opening Status Update's** narrative. There is no second scope record to
keep in step.

## The notes

Below is a **completed** set of notes — the input branch B receives, every `Decision` and
**Next Actions** line filled in. It is also the target branch A drafts toward: the same headings,
the same order, the same prompts, with the `Result` / `Reason` / `Decision` fields and the
**Next Actions** list blank instead. Reproduce this shape.

Notice the **Next Actions** register: outcomes in business language, owner first. Field names and
`gh` syntax never appear there — the Doc is read by the team, and branch B recognises a tracker
action by what it describes.

````markdown
# Cycle Review — 2026-08-28 meeting

_Illustrative example. Issue numbers, repos, and titles below are generic placeholders chosen to
cover the range of states the skill needs to handle — not real work, not real bugs._

## Last Cycle: Cycle 12 (14 Aug → 28 Aug 2026)

**Goal:** Ship Initiative A and clear Maintenance Task B.

| Issue                             | Result                     | Reason                                                              |
| --------------------------------- | -------------------------- | ------------------------------------------------------------------- |
| repo1#101 Initiative A, part 1    | ✅ Done                    | —                                                                   |
| repo1#102 Initiative A, part 2    | ✅ Done                    | —                                                                   |
| repo2#210 Maintenance Task B      | ❌ Slipped → current cycle | Blocked on an external dependency; fix landed 26 Aug, retrying now  |
| repo3#55 Internal tooling cleanup | ❌ Slipped → current cycle | Overcommitted — Initiative A ran two days over                      |
| repo2#215 Small fix               | ➕ Added mid-cycle, done   | Flagged as urgent partway through the cycle; nothing else displaced |
| repo3#61 Small fix                | ➕ Added mid-cycle, done   | Closed 22 Aug, no `Cycle` set — credited automatically while preparing this cycle's notes |

**Decision:** repo2#210 and repo3#55 both carry into the current cycle as-is, no rework needed.

## Current Cycle: Cycle 13 (28 Aug → 11 Sep 2026)

**Next cycle dates:** Agreed 11 Sep 2026 — 14 days, same length as the cycle just closed.

### New Issue Triage

_List every issue opened since the last cycle and record whether it's committed, parked, or
closed._

| Issue                                  | Decision                                                                         |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| repo1#120 Small internal request       | ➡️ Committed — small, unblocks another team                                      |
| repo1#121 Minor copy fix               | ➡️ Committed — trivial, bundled with #122                                        |
| repo2#221 Inconsistent behavior report | ➡️ Committed — reported by more than one person this week                        |
| repo3#60 Exploratory idea              | 🚫 Closed, not planned — no one has asked for this; revisit if it comes up again |
| repo1#125 Nice-to-have UI polish       | ⏸ Parking lot — no urgency                                                       |

### Stale Issues

_Report the stale count and how many went newly stale this cycle. Propose close candidates that
are old, underspecified, or duplicates. Flag any that still need doing and pull them into a
cycle instead. Call out anything newly stale that looks important._

**38 stale issues** across `Backlog` and `Ready for *` (untouched 3+ weeks); **6 newly stale**
since the last cycle closed.

**Close candidates**

- **repo1#40** — "Investigate an older idea," open 6 months, no requirements beyond the title.
  **Decision:** Closed, not planned — too vague to act on; revisit if someone writes it up
  properly.

- **repo2#45** — "Consider a future enhancement," open 4 months, duplicates repo2#48.
  **Decision:** Closed, not planned — tracked under repo2#48 instead.

- **repo3#12** — "Old internal cleanup idea," open 5 months, no activity, nothing references it.
  **Decision:** Closed, not planned — no evidence anyone still wants this.

**Needs a cycle, not a close**

- **repo1#20** — Still genuinely needed (blocks another team's reporting), but has sat in
  `Backlog` for 5 weeks without ever being triaged.
  **Decision:** Pull into this cycle — see Committed This Cycle below.

**Newly stale, worth a look**

- **repo2#88** — Went stale this cycle. Real design discussion in the comments and looked close
  to done before it went quiet — worth checking in on before it becomes a close candidate.

### Waiting on Input

_Go through each item one at a time and record a decision, even the ones nobody can resolve._

- **repo1#90** — Product question A: option 1 or option 2?
  **Decision:** Option 1 — matches how a related flow already works.

- **repo2#95** — Which segment gets Feature X?
  **Decision:** All of them, not just one subset.

- **repo1#98** — A pricing-display question.
  **Decision:** Not resolved — no one on the call had the definitive answer. Follow up with the
  right team before next cycle; leave as `Waiting on input`.

### Significant Dates

_Flag any upcoming sales, trade shows, marketing campaigns, or other significant dates that might impact planning._

- **04 Sep 2026:** The Humble River sale will be our first large sale using the new dynamic bidding system. Make sure development and QA teams are prepared for any issues that might arise.
- **15 Sep 2026:** Quarterly marketing review meeting. Ensure all relevant teams have prepared their updates and reports.

### Committed This Cycle

_List every issue committed to this cycle and note why it made the cut._

- **repo2#210** — Maintenance Task B. Carried over from last cycle; the blocking dependency is
  now resolved.
- **repo3#55** — Internal tooling cleanup. Carried over from last cycle; straightforward once it
  wasn't competing with Initiative A for time.
- **repo1#120** — Small internal request. Small enough to fit alongside the carry-over work, and
  unblocks another team.
- **repo1#121** — Minor copy fix. Trivial; bundled in alongside repo2#221 since they touch the
  same area.
- **repo2#221** — Inconsistent behavior report. Reported by more than one person this week — that
  volume is what pushed it in ahead of repo1#125.
- **repo1#20** — Pulled out of Stale Issues review; blocks another team's reporting and should
  have been triaged weeks ago.
- **repo2#230** — Found already `In progress` with no `Cycle` during prep; credited to this cycle
  automatically, no action needed.

### Parking Lot (Next)

_List every issue deferred to the next cycle and note why it didn't make the cut._

- **repo1#118** — A follow-up enhancement request, carried from last cycle's parking lot. Still
  waiting on input from another team, so not committed again until that lands.
- **repo1#125** — Nice-to-have UI polish. No urgency, and this cycle's capacity went to
  repo2#221's higher report volume instead.

## Out of Office

_Flag any planned time off during the current or upcoming cycle so it's factored into future
commitments._

- **ALEX:** Out 2–4 Sep.
- No other absences reported this cycle.

## Next Actions

_List each follow-up as **OWNER:** action, stated as the outcome — not the mechanics of how it
gets done. Tracker writes default to Mark, delegated to the agent as needed; name the person when
it's someone else's job._

- **MARK:** Set cycles for the Committed / Parking lot items listed above.
- **MARK:** Clear `Waiting on input` on repo1#90 and repo2#95, recording the answers, then move
  them into the appropriate cycles.
- **MARK:** Close repo3#60, repo1#40, repo2#45, and repo3#12 as not planned.
- **MARK:** Draft the new issue mentioned above.
- **MARK:** Post the closing and opening Status Updates per the summaries above.
- **MARK:** Update the current cycle's end date (11 Sep 2026).
- **MARK:** Follow up on repo1#98 pricing question with Jordan.
- **ALEX:** Confirm the outstanding input on repo1#118 with the other team before next cycle.
  Update the issue with your findings.

## Next Cycle Review

_The confirmed date, time, and location for the next cycle-review meeting._

**Date:** 11 Sep 2026
**Time:** 10:00 AM MT
**Location:** Video call — link in the calendar invite
````

## Known limitations

- **Rescheduling has no mechanism.** Correcting a cycle date that has already been agreed and
  already gone out in a Status Update is a different problem from setting the next cycle's date for
  the first time, which branch B does directly. No convention exists for it. Work one out and
  document it here the next time a cycle actually needs moving.
- **The Doc round trip is manual.** Copy out, copy back. Giving the agent Drive access is a future
  decision.

## Iterating on the skill

This skill is committed, so it versions and reviews like code. Three habits keep it improving
rather than calcifying:

- **Fix the skill, not the output.** Whenever a draft needs hand-editing, that edit is a bug
  report. Change the skill and re-run rather than patching the notes by hand.
- **File skill bugs as issues** (`enhancement`) so they flow through the same triage and cycle
  machinery as everything else — the workflow tests itself.
- **Edit it with `/writing-for-agents`.** Its no-op test (delete whole sentences that don't
  change behaviour) is the antidote to a skill that accretes caveats after every awkward run.
