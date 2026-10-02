---
name: email-to-issue
description: Turn flagged Outlook threads, or threads named by subject, into GitHub issues or comments, then reply to yourself with the link and archive the thread.
disable-model-invocation: true
---

# Email to issue

One run is a **sweep**: read the chosen Outlook threads, propose one table covering all of them,
and file only the rows the user approves. Each approved row becomes a new issue or a comment on an
open one, written by the `issue-tracker` conventions. New issues stay in `Backlog` for triage,
which writes any agent brief and routes them; this skill does neither.

Mail is reached through the Microsoft 365 connector. Find its tools by capability — read, search,
reply, move, flag — rather than by name, since names change between connector versions.

## Mail permissions

Every mail action in this skill falls inside this list. Anything outside it is not done, even when
the user asks mid-sweep; say so and leave it to them.

- **Allowed:**
  - Read and search mail.
  - Reply in a thread filed in this sweep, addressed only to the signed-in user, with the issue
    link — see [The reply](#the-reply).
  - Clear the flag on, and move to Archive, a thread filed in this sweep, after the user confirms
    in [step 6](#6-close-the-loop).
- **Never:**
  - Delete mail, drafts included.
  - Send, forward or draft mail to anyone other than the signed-in user.
  - Write to calendar, files or Teams.

## The sweep

Run the steps in order. Each ends on the observable result named in **Done when**.

### 1. Gather

Read the threads to process: every **flagged** thread in the Inbox by default, or the threads the
user names by subject. A subject that matches more than one thread is a question: list the matches
with sender and date, and ask which.

**Skip a processed thread.** A thread is processed when it holds a message from the user, to the
user only, whose first line starts `Filed by /email-to-issue:` — the marker [the reply](#the-reply)
writes.

Attachments are out of scope. Leave them in the mail, and note in the row that the thread has one.

**Done when:** a list of threads to process, each with sender, date and subject, and a count of the
threads skipped as processed.

### 2. Check permissions

List the connector's mail tools this session has, by capability. Writing to mail needs two:

- **Reply** — one that takes its recipient list as an argument, so it can be set to the user alone.
  A reply that can only pre-fill the original sender does not count.
- **Move and flag** — moving a message to Archive and clearing its flag.

Without them the sweep still files issues, and [step 6](#6-close-the-loop) ends in a manual list
instead.

**Done when:** one line tells the user whether this sweep can reply, and whether it can unflag and
archive.

### 3. Match

For each thread, search open issues across the org for one that already covers it:

```bash
gh search issues "<key terms from the thread>" --owner clhbid --state open \
  --json repository,number,title,url --limit 10
```

Try two or three phrasings, and read a candidate before calling it a match. A match becomes a
proposed **comment** that adds only what the thread says and the issue does not — new evidence or a
clarification. No match becomes a proposed **new issue**, in the repo the thread is about; the
aliases in [`issue-tracker` § References](../issue-tracker/SKILL.md#references) name the repos.

**Done when:** every thread has a proposed action and target.

### 4. Propose

Check each target repo's visibility before drafting its row:

```bash
gh repo view clhbid/<repo> --json visibility --jq .visibility
```

Draft each title by [Confidentiality](#confidentiality), then show one table for the whole sweep:

| #   | Source                                 | Action    | Target              | Visibility | Title                         | Label |
| --- | -------------------------------------- | --------- | ------------------- | ---------- | ----------------------------- | ----- |
| 1   | Pat Example (`pat@example.com`), 2 Mar | New issue | `clhbid/<repo>`     | PRIVATE    | Bid export drops the last lot | `bug` |
| 2   | Sam Sample (`sam@example.org`), 3 Mar  | Comment   | `clhbid/<repo>#123` | PUBLIC     | Second report of the timeout  | —     |

The table is shown in the session only, never filed. The user approves, edits or skips each row.
**Nothing is written to GitHub before approval.**

**Done when:** every row is approved, edited and approved, or skipped.

### 5. File

For each approved row, write the issue body or comment by [Confidentiality](#confidentiality), then:

- **New issue** — create it by
  [`issue-tracker` § Creating an issue from the org form](../issue-tracker/SKILL.md#creating-an-issue-from-the-org-form):
  the org form's fields, one category label, and no `Status` beyond `Backlog`. Confirm it landed on
  the board in `Backlog`, as [`issue-tracker` § Status](../issue-tracker/SKILL.md#status) describes.
- **Comment** — `gh issue comment <number> --repo clhbid/<repo> --body "..."`, written as a thread
  comment by `writing-lean`.

Write every reference in full, as [`issue-tracker` § References](../issue-tracker/SKILL.md#references)
requires.

**Done when:** each filed row has a link, and each new issue reads back as `Backlog` on the board.

### 6. Close the loop

**Reply.** For each filed row, send [the reply](#the-reply) in its thread.

**Unflag and archive.** Then list the filed threads and ask whether to clear each one's flag and
move it to Archive. Approval of the table in step 4 is not this confirmation: ask again, and move
only the threads the user confirms now.

**Without write tools**, end with a manual list instead — one line per filed thread with its sender,
date, subject and issue link, and the actions left to do by hand: reply to yourself with the link,
clear the flag, archive. The reply matters: it is what lets the next sweep skip the thread.

**Done when:** a sweep summary lists each thread with its issue link and what happened to it —
replied, unflagged, archived, or left for the user — including every reply that was skipped and why.

## The reply

The reply marks the thread as processed and carries the link back to the mail. It is the only mail
this skill sends.

1. **Know the user's address.** Take it from the connector's signed-in profile. If it is missing,
   or more than one address could be theirs, ask.
2. **Write the body.** The first line is the marker, then the link:

   ```text
   Filed by /email-to-issue: clhbid/<repo>#<number>
   https://github.com/clhbid/<repo>/issues/<number>
   ```

3. **Replace the recipients.** Build the reply with `To` set to the user's address alone, and `Cc`
   and `Bcc` empty — never the thread's own recipients.
4. **Read them back** before sending: every recipient field, from the message as built. Send only if
   the user's address is the one recipient.
5. **Skip on any doubt.** If the recipients cannot be set, cannot be read back, or read back as
   anything else, do not send. Leave the reply out and put the issue link in the sweep summary. If
   a draft was left behind, say where it is so the user can remove it; never delete it.

## Confidentiality

What goes into a title, issue body or comment follows the target repo's visibility from
[step 4](#4-propose):

- **`PRIVATE`** — capture the thread's details in full, confidential ones included.
- **`PUBLIC`** — paraphrase. Remove customer names, contact details, pricing and any other
  proprietary detail, and describe the sender by role ("a seller", "a bidder") rather than by name.
- **Anything else, or unsure** whether a detail is confidential — ask before drafting it.
