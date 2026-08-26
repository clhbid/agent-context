---
name: cycle-review
description: Run the fortnightly-ish business-update cycle on GitHub milestones — prepare a meeting, process its notes, adjust cycle membership, or reschedule. User-invoked only.
disable-model-invocation: true
---

# Cycle Review

> ⚠️ **Known stale.** This skill still assumes GitHub milestones and the
> `needs-info` / `ready-for-agent` label vocabulary. Both have been retired: cycles are now
> iterations of the `Cycle` field and state is the `Status` field, both on the CLHbid Delivery
> project. The `issue-tracker` skill is authoritative where the two disagree. Retargeting this one
> is tracked in clhbid.com#2312 — do not treat the milestone mechanics below as current.

The recurring business update _is_ the milestone — there is no separate report artefact.
Three milestones are live at any time:

| Milestone | State                     | Role                                                  |
| --------- | ------------------------- | ----------------------------------------------------- |
| closing   | Completed, about to close | the cycle just ending                                 |
| current   | Planning → In progress    | about to be agreed / just agreed                      |
| next      | Parking lot               | where deferred and newly-triaged work lands mid-cycle |

**Naming.** Title is the meeting date that closes the cycle, ISO (`2026-08-14`), with `due_on`
set to match — sorts chronologically and handles variable cadence (2, 3, or 4 weeks over
holidays) for free. The human phrasing goes in the description's first line.

**Role resolution is by date, never by description shape:**

- **closing** = open milestone with `due_on` ≤ the meeting date being processed
- **current** = open milestone with `due_on` == the next meeting date
- **next** = anything beyond

This is deterministic and survives a rescheduled meeting (branch D). **First run: none of these
exist.** Skip straight to creating them (branch A, step 5) rather than inventing a fictional
prior cycle.

**The core rule: description holds narrative, membership holds the work.** Never list issues in
the description — that copy drifts. Progress is computed from membership. The one exception is
the meeting agenda, which exists only in the planning state, because it records what was true
_at the meeting_ — a frozen fact, not live state.

**Questions live on the issue, not in the milestone.** The full question is a Triage Notes
comment on the issue (`needs-info` template, see **Triage roles** in the `issue-tracker` skill);
the milestone carries a one-line summary plus the link. This skill's job is to _guarantee the
issue-side comment exists first_, then summarise it — never invent a question from a title.

**Unanswered questions roll over for free.** Planning queries `label:needs-info` fresh each
cycle (see the `issue-tracker` skill for the query), so anything still outstanding
reappears automatically. There is no carry-over bookkeeping.

**Verified trap: closing a milestone does NOT clear its open issues.** Orphaned issues are
genuinely invisible afterward — absent from `is:open milestone:<current>`, and not caught by
`no:milestone` either, since they still have one. **Sweeping open issues into the next milestone
before closing is mandatory**, not tidiness.

Read the standard queries from the `issue-tracker` skill rather than duplicating them here — that
skill is the single source of truth.

## The states

A milestone starts as a **parking lot** — `Goal` and `Scope` only, no agenda:

```markdown
Cycle 14 Aug → 28 Aug 2026. Not yet planned.

## Goal

TBC at the 14 Aug meeting.

## Scope

Parked here so far: #2219 (Sold badge, deferred 31 Jul), #2225 (GTM event, needs a
spec from the requester).

## Out of scope

Salesforce work — not scheduled this cycle.
```

`Goal`, `Scope` and `Out of scope` appear in **all three** states below, so the milestone reads
as one document maturing and cycles compare at a glance. `Next cycle dates`, `Needs input` and
`To triage` are the **meeting agenda** and exist **only in the planning state**.

Three rules for the agenda sections:

- **`Next cycle dates` comes first, and is decided first.** The cycle's length sets how much
  work can be committed to, so agreeing the closing date is a precondition for agreeing scope,
  not a tidy-up at the end. Cadence drifts — holidays and summer availability make cycles
  shorter or longer — so it is agreed every meeting rather than assumed. State the proposed
  closing date, the resulting length in days, and the length of the cycle just ending for
  comparison. Never present scope as settled against a date nobody has confirmed.
- **Every `Needs input` item is a one-line summary plus the issue link.** The full question
  lives on the issue as Triage Notes. Confirm that comment exists before listing the item — if
  it doesn't, the question hasn't actually been asked of anyone.
- **Every `To triage` item is listed individually** with its title, never as a count — this list
  gets hand-edited before the meeting.

**Planning** — created when the cycle is set up, `state: open`, `due_on` set:

```markdown
Cycle 31 Jul → 14 Aug 2026. Review at the 31 Jul meeting.

## Goal

Ship the podcast section and clear the axios CVE.

## Scope

9 issues proposed — see the list below. Carried over from last cycle: #2141, #1926.

## Next cycle dates

Proposed: 14 Aug → 28 Aug 2026, closing at the 28 Aug meeting — 14 days. The cycle
just ending ran 14 days. Agree this before the scope below.

## Needs input

- #1848 — YouTube videos: on every sale strategy page, or only where the seller
  has recorded one?
- #1974 — Team bios: whose have changed since February, and do we have photos?

## To triage

- #2218 — Duplicate auction cards on mobile Safari
- #2219 — Add a "Sold" badge to past sales
- #2220 — Broken link on the financing page
- #2225 — GTM event for brochure downloads

## Out of scope

Salesforce work — not scheduled this cycle.
```

**In progress** — updated straight after the meeting from the pasted notes. The agenda sections
are gone; the answers now live on the issues:

```markdown
Cycle 31 Jul → 14 Aug 2026. Agreed at the 31 Jul meeting.

## Goal

Ship the podcast section and clear the axios CVE.

## Scope

9 committed. Deferred #1993 (podcast menu) — needs the header work to land first.
Added 5 Aug: #2214 (secret rotation), urgent; displaced #2141.

Answers from the meeting are recorded on the issues: #1848, #2218, #2220. #1974
went unanswered and rolls into the next cycle's Needs input.

## Out of scope

Salesforce work — not scheduled this cycle.
```

**Completed** — swept, outcome recorded, then closed:

```markdown
Cycle 31 Jul → 14 Aug 2026. Closed at the 14 Aug meeting.

## Goal

Ship the podcast section and clear the axios CVE. Both delivered.

## Scope

7 of 9 delivered. Slipped to 2026-08-28: #2141 (GTM events — still waiting on
input), #1926 (blocked on the unanswered bios question).

## Out of scope

Salesforce work — unchanged, still not scheduled.
```

## Invocation

A team member invokes `/cycle-review` with a natural-language description of what they want.
Route to the branch that matches:

- **A. Prepare** — `/cycle-review prepare for the 2026-07-31 meeting, next one is 2026-08-14`
- **B. Process notes** — `/cycle-review notes from the 2026-07-31 meeting:` + pasted markdown
- **C. Adjust** — `/cycle-review add #2230 to the current cycle — it displaces #2141`
- **D. Reschedule** — `/cycle-review the 14 Aug meeting moved to 21 Aug`

## Branch A — Prepare

1. Resolve roles by date (see above). **First run: none of these exist** — skip to step 5 and
   create them; state this explicitly rather than improvising a fictional prior cycle.
2. Summarise the closing milestone: delivered (`is:closed milestone:"X" reason:completed`) vs
   still open.
3. **Sweep** every still-open issue in the closing milestone into the current milestone, or
   clear its milestone if the work was dropped. **Completion criterion:**
   `gh api repos/<owner>/<repo>/milestones/<n> --jq .open_issues` returns **0**. Do not
   proceed to step 4 until it does. This step is dull and sits in front of the interesting
   drafting — exactly the shape that invites skipping it, and skipping it orphans issues
   invisibly (see the verified trap above).
4. Write the **completed** description onto the closing milestone, then close it.
5. Ensure the current and next milestones exist, creating them as **parking lots** if not.
6. Regenerate the current milestone's **planning** agenda _now_ — never reuse what was written
   at creation. Open it with `Next cycle dates`: propose the next meeting date, state the
   resulting length in days alongside the closing cycle's length, and mark it as proposed
   rather than agreed. For each `needs-info` issue, confirm a Triage Notes comment carrying the
   question exists on the issue, then write a one-line summary plus link. For `To triage`, list
   the `Needs Triage` view (from the `issue-tracker` skill) individually with titles.
7. **Present both drafts for editing before writing.** The `To triage` list gets hand-edited —
   items needing business input to even start stay, the rest are cut.

## Branch B — Process notes

8. Parse the pasted notes for decisions: committed, deferred and why, questions answered,
   triage accepted or rejected.
9. **Answer on the issue, in two places**, for each answered question:
   - **Reply to the question comment** — the Triage Notes comment that asked it — so the
     exchange reads as a thread. Prefix with a provenance marker mirroring the triage
     convention: `> *Recorded from the <date> planning meeting.*`
   - **Integrate the answer into the issue description**, so someone picking the work up cold
     has complete context without reconstructing it from the comment thread. The comment is the
     audit trail; the description is the spec. Never delete what was there — fold the answer in.
10. **If an answer raises new questions**, post them as a comment in the same Triage Notes shape
    ("What we still need from you") and keep `needs-info`. An answer that opens a new question
    has not unblocked the issue.
11. **Advance the state only when every question is answered:**
    - Remove `needs-info`.
    - Apply `ready-for-agent` **only if the updated description now reads as a complete brief an
      agent could work from cold.** This is the gate that keeps the AFK queue trustworthy.
    - Apply `ready-for-human` if it needs human judgement, external access, a design decision, or
      manual testing.
    - Apply `needs-triage` if it's answered but still underspecified — say so explicitly rather
      than guessing at a ready state.

    Deviation from `triage`'s default: it returns `needs-info` to `needs-triage` on reply and
    re-forks there. Advancing straight to `ready-for-*` here is a maintainer override — the
    maintainer can override at any time — so treat it as a deliberate choice, not the default.

12. Rewrite the current milestone to the **in-progress** shape, dropping `Needs input` and
    `To triage`, and adjust membership. Deferred work moves to the next milestone's parking lot.
13. Leave any still-unanswered `needs-info` issue labelled as-is — it resurfaces in the next
    cycle's agenda by itself. Never carry it forward by hand.
14. For work in the notes with no matching issue, **draft an issue** — category label only, no
    state label, body drawn from the notes — and create it on confirmation. It's new input, so
    it belongs in the never-triaged bucket like anything filed from a template.
15. Re-check for issues closed _since_ the closing milestone was closed — meeting-morning merges
    land after the close and would otherwise be misattributed to the next cycle.

## Branch C — Adjust

Move membership (`gh issue edit <n> --milestone "<title>"`, or clear it), and note the change in
the current milestone's `Scope` (e.g. "Added 5 Aug: #2230; displaced #2141").

## Branch D — Reschedule

Update the milestone's title and `due_on` to the new meeting date. Never force-close a cycle
just because its date passed — a skipped or moved meeting extends the cycle; it isn't a reason
to invent an artificial close.

## Requirements

- Include the worked examples above verbatim in any output — they are the target to imitate.
  `Goal`, `Scope` and `Out of scope` in every state; `Needs input` and `To triage` in planning
  only; parking lots get neither.
- Read query definitions from the `issue-tracker` skill rather than duplicating them — single
  source of truth, so the two can't drift.
- **Confirm before mutating.** This skill closes milestones, moves issues, comments on them,
  edits issue descriptions, changes labels, and creates issues. Show the proposed changes and
  get an explicit go-ahead. Editing a description rewrites something a human wrote — show the
  diff.
- **Never invent a question.** If a `needs-info` issue has no Triage Notes comment, say so and
  ask rather than summarising from the title — an unrecorded question hasn't been put to
  anyone, and that gap means the label is wrong.
- Write prose the business can read and paste into a Google Doc. Issue titles alone are not a
  status update.

**Known limitation:** each state overwrites the last, so there's no record of what was
originally planned versus delivered beyond what `Scope` states in the completed description.
Accepted for now.

## Iterating on the skill

This skill is committed, so it versions and reviews like code. Three habits keep it improving
rather than calcifying:

- **Fix the skill, not the output.** Whenever a draft needs hand-editing, that edit is a bug
  report. Change the skill and re-run rather than patching the milestone by hand.
- **File skill bugs as issues** (`enhancement`) so they flow through the same triage and
  milestone machinery as everything else — the workflow tests itself.
- **Edit it with `/writing-for-agents`.** Its no-op test (delete whole sentences that don't
  change behaviour) is the antidote to a skill that accretes caveats after every awkward run.
