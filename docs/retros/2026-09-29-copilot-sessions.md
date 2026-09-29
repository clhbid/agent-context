# Copilot sessions retro: Timezones, Secrets & Release

Cycle 2026-09-15 to 2026-09-29. Written 2026-09-29 for clhbid/agent-context#39. This is a dated
record and is not updated afterwards. The per-session baseline is in
[`2026-09-29-copilot-sessions.csv`](2026-09-29-copilot-sessions.csv).

**The cycle ran 126 Copilot sessions, not 38.** `gh agent-task list` returns only the latest
session for each pull request. The 38 in the brief is the number of pull requests. The session API
lists every session: 36 started from an issue assignment and 90 were follow-ups on an existing pull
request. Four ended `failed` and two `cancelled`. The CSV has all 126 rows, with
`in_gh_agent_task_list` marking the 38 the CLI shows.

## Recommendations

My top pick is **1**. It targets the largest block of reviewer time, and one text reaches every
repo.

### 1. Write the house rules down once, where Copilot and its reviewer both read them

- **Change:** add `copilot/organization-instructions.md` to this repo and paste it into the clhbid
  organization's Copilot custom instructions. That setting reaches the cloud agent and Copilot code
  review in every repo [[1]](#sources). It holds only the four rules reviewers kept writing by hand,
  each as one line:
  - **Lean writing.** Comments and docs say why, in a line. The code and specs are the source of
    truth.
  - **Test our code.** Take a dependency's data from its exports or a mock. Never copy its tables,
    and don't test its behaviour.
  - **One source of truth.** Use shared constants, the Tailwind scale and the dependency's own
    types.
  - **Spec style.** Use `beforeEach`/`afterEach`, arrange–act–assert, and assert the exact set.
- **Lands in:** this repo (the text, reviewed as a PR), then org settings.
- **Evidence:** Mark wrote 126 inline review comments on the cycle's Copilot pull requests. **55
  (44%) restate one of those four rules:** 29 on lean writing, 11 on testing the dependency, 8 on
  single source of truth and 7 on spec style. Twelve of the 18 sampled sessions produced or fixed
  this kind of rework. Examples are
  [clhbid/canadian-time-zone-hotpatch#42](https://github.com/clhbid/canadian-time-zone-hotpatch/pull/42)
  (`f4844586`: 20 inline comments from the reviewer, 6 sessions) and
  [clhbid/CLHbid-LiveAuction#1024](https://github.com/clhbid/CLHbid-LiveAuction/pull/1024) (`d9f75f9b`:
  five comments asking that tests stop copying or exercising the package, 8 sessions). On
  clhbid/canadian-time-zone-hotpatch#41 the reviewer pasted the whole lean-writing rule into a comment.
- **Human time saved:** about 2 h per cycle, with a range of 1.5 to 3 h. This assumes the text
  prevents half of the 55 comments (about 27 × 2 min to write, roughly 55 min). It also assumes
  about 8 fewer change-request rounds out of 43 (× 10 min to wait for and re-review a follow-up,
  roughly 80 min).
- **AI credits saved:** sessions triggered by a human review or comment used 32.6% of the cycle's
  `ai_credits` (65 sessions). If house-rule comments drive 44% of that and half is prevented, the
  saving is about 7% of the cycle total, roughly 1.2 × 10¹² raw. `ai_credits` is an undocumented
  field. GitHub prices an AI credit at $0.01 [[2]](#sources) but doesn't document this field's
  scale, so these are raw figures and shares.
- **Effort:** half a day, including an org owner pasting the text.
- **Measure:** in next cycle's CSV, sessions per pull request whose `trigger` is
  `pull_request_review` or `pull_request_comment`. The baseline is 65 (44 + 21) across 38, or 1.71. Also re-run the comment classification against the 44%
  baseline.
- **`writing-for-agents` [[10]](#sources):** this relies on leading words ("lean writing", "one
  source of truth") and keeps context load low by adding four lines that load everywhere. It states the rules
  positively to avoid the negation failure, and it is enforced twice because the reviewer checks
  the same text. The rules are visible in a diff, so the change is likely to shift behaviour
  reliably. Keep the text out of `AGENTS.md` to avoid duplication.
- **Risks:** organization instructions have the lowest priority, below repository and personal
  instructions [[1]](#sources). A repo rule that conflicts with them wins silently.

### 2. Make "How a run ends" something Copilot can do, and let a workflow do the rest

- **Change:** every repo's `AGENTS.md` makes three things the definition of done: assign the
  issue, set `Status`, and request review. Copilot can do none of them. Its token is scoped to the
  repository it works in, it can push only to its pull request branch, and it reads only secrets in
  the `copilot` environment [[3]](#sources) [[4]](#sources). Add a scheduled workflow to this repo.
  It uses a token with org project write access and finds open Copilot pull requests. It sets the
  linked issue's `Status` from the PR: ready → `Ready for Human`, draft with a `Blocked:` comment →
  `Waiting on input`, other drafts → `Ready for Human`. Then it requests review from whoever
  assigned Copilot. The Copilot half of the contract already maps onto draft and ready, so no
  agent-facing text has to change. Trimming each `AGENTS.md` afterwards is a follow-up and not part
  of this recommendation.
- **Lands in:** GitHub Actions in this repo, as one PR.
- **Evidence:**
  - None of the 18 sampled sessions set `Status`, assigned the issue or requested review. Only one
    said it couldn't: `d297852b` on
    [clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423) wrote "I have no tool to set
    the issue's Status, assign it, or request a reviewer". The other 17 stopped without saying
    anything.
  - All four `failed` sessions ended that way after finishing and posting their summary (see
    Problems found). A reviewer has no reliable signal about the state a run ended in.
- **Human time saved:** about 1.5 h per cycle. That is 38 pull requests × about 1 min to move
  `Status` by hand, plus about 1 min each to classify a PR that has no state comment, plus 4
  sessions reported as failed that finished their work × about 10 min to find that out.
- **AI credits saved:** none measurable from `ai_credits` (an undocumented field). The workflow
  runs in Actions, not in agent sessions.
- **Effort:** 1–2 days, including creating the token and storing it as a secret.
- **Measure:** add a `status_after_run` column to next cycle's CSV. The baseline is 0 of 18
  sampled runs leaving the issue in the state the contract asks for.
- **Risks:** the workflow token can write to the org board. Scope it to project write and the
  review-request endpoint only.

### 3. Give clhbid.com a `copilot-setup-steps.yml`

- **Change:** add `.github/workflows/copilot-setup-steps.yml` to clhbid.com. It should use
  `actions/setup-node` with the repo's pinned Node version and run a frozen-lockfile `yarn install`
  with `SENTRYCLI_SKIP_DOWNLOAD=1`. A full install also produces the generated types that
  typecheck needs.
  GitHub recommends setup steps to "deterministically install tools or dependencies", and they
  run before the firewall applies [[5]](#sources) [[6]](#sources). Drop "Ensure dependencies are up
  to date (`yarn install`)" from `AGENTS.md`.
- **Lands in:** repo setup in clhbid/clhbid.com. None of the four target repos has this file
  today.
- **Evidence:** 8 of the 9 sampled clhbid.com sessions spent calls on the environment:
  - The runner's Node version doesn't match the version the repo pins, so `yarn` refused to run
    until the agent added `--ignore-engines`. This happened three times in `2929c632`.
  - `node_modules` was missing (`44ae9cd2`, `40122156`).
  - The Sentry CLI download is blocked by the firewall. `d297852b` needed two retries to get past
    it.
  - Several sessions re-proved that a typecheck error from missing generated types already exists
    on main (`ca431ee7`, `3a5173c1`).
  - `6bb19ee3` spent 55 minutes trying to build the site without the credentials the build needs
    before it was cancelled.
- **Human time saved:** about 45 min per cycle. About 20 min went on `6bb19ee3`: cancelling it,
  steering two follow-up sessions from the phone, and writing up the build it couldn't run. Add
  about 3 pull requests × 8 min to re-check validation a session ran under workarounds
  (`--ignore-engines`, skipped installs), roughly 25 min.
- **AI credits saved:** about 3–4% of the cycle's `ai_credits` (an undocumented field; raw
  shares). `6bb19ee3` alone was 2.8%. Environment
  calls make up roughly a fifth of the other eight sessions, which used 5.1% of the total between
  them.
- **Effort:** half a day.
- **Measure:** the median `duration_min` of clhbid.com `issues_agent_assignment` sessions, against
  a baseline of 15.8 min (hotpatch's is 6.1). Also count cancelled sessions, against a baseline
  of 2.
- **`writing-for-agents` [[10]](#sources):** this removes a line that is sediment. The environment will already
  be set up, so the line becomes a no-op.

## Problems found

The categories come from the 18 sampled sessions. A session can fall into several. Credit shares
are of the whole cycle's `ai_credits`.

| Category | Sampled sessions | Credit share | Examples |
| --- | --- | --- | --- |
| Run end skipped: no `Status`, assignment or review request | 18 | 27.8% | `d297852b` [clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423) |
| Review rework on house rules | 12 | 17.9% | `f4844586` [clhbid/canadian-time-zone-hotpatch#42](https://github.com/clhbid/canadian-time-zone-hotpatch/pull/42), `d9f75f9b` [clhbid/CLHbid-LiveAuction#1024](https://github.com/clhbid/CLHbid-LiveAuction/pull/1024) |
| Environment not prepared | 9 | 7.9% | `2929c632` [clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423), `6bb19ee3` [clhbid/clhbid.com#2405](https://github.com/clhbid/clhbid.com/pull/2405) |
| Design or scope decided in review | 5 | 4.8% | `44ae9cd2` [clhbid/clhbid.com#2384](https://github.com/clhbid/clhbid.com/pull/2384) |
| Reported `failed` after finishing | 4 | 2.1% | `ca431ee7` [clhbid/clhbid.com#2397](https://github.com/clhbid/clhbid.com/pull/2397) |
| Dispatched before it could be verified | 3 | 3.6% | `0d9304fe` [clhbid/clhbid.com#2386](https://github.com/clhbid/clhbid.com/pull/2386) |
| Automated reviewer's feedback forwarded without triage | 3 | 1.8% | `2929c632`, `a69db851` [clhbid/clhbid.com#2412](https://github.com/clhbid/clhbid.com/pull/2412) |

**Run end skipped.** Root cause: agent instructions and platform. The contract asks for steps
Copilot's token can't take, and the skill it points to isn't installed in a Copilot container.
Only `d297852b` flagged it.

**Review rework on house rules.** Root cause: agent instructions. The rules exist only in
reviewers' heads and in skills Copilot never loads. Once, the brief itself was the source:
clhbid/clhbid.com#2411 asked for tests that repeated the package's NWT offsets, and the review of
`d297852b`'s pull request had to reverse that.

**Environment not prepared.** Root cause: repo setup. Every agent-assigned session installs
dependencies from scratch, and clhbid.com's installs fail in several different ways. LiveAuction
has the same missing `node_modules` (`6fbb5ef4`) but installs cleanly. Hotpatch is small enough
that it didn't matter.

**Design or scope decided in review.** Root cause: which work goes to Copilot.
[clhbid/clhbid.com#2384](https://github.com/clhbid/clhbid.com/pull/2384) took 12 sessions over two days
while the team iterated on copy, spacing and a sketch. Each round was one or two review lines and
a five-minute session. The work itself was right, but it was design iteration run as AFK work.

**Reported `failed` after finishing.** Root cause: platform. All four `failed` sessions are on
clhbid.com and changed `package.json` or `yarn.lock`. Each finished its work and posted its summary.
Then Copilot's post-run dependency-graph comparison aborted with exit 134 while printing the graph,
after the agent had stopped. The session API records `error: null` for all four. Both pull
requests merged.

**Dispatched before it could be verified.** Root cause: the brief and dispatch.
- `0d9304fe` was assigned before the devcontainer feature it depended on was published. It was
  cancelled after two minutes and the PR was closed.
- `6bb19ee3` was asked for work whose verification needed a full site build with credentials
  the agent doesn't have.
- `8a3f1a5f` couldn't publish to GHCR and left that to a human, and the PR took 10 human
  commits.

**Automated reviewer's feedback forwarded without triage.** Root cause: review. On
[clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423), the reviewer's overview asked to
"force and verify the `rule_outdated` correction path" and was forwarded to Copilot as written.
The next session wrote that test, and the reviewer's following round then objected to fixtures
built around the package's rule table. There were 16 "Fix with Copilot" sessions this cycle, using
9.2% of credits.

## Not this round

- Route UI design iteration to Claude Code with a local preview instead of review rounds on a
  Copilot PR.
- Report the dependency-graph abort (exit 134, `error: null`) to GitHub Support with the four
  session IDs.
- Batch each review round into one review, instead of a comment and a review that each start a
  session.

## Method

**Population.** I paged through every session from the session API
(`GET /agents/sessions`, the endpoint `gh agent-task` uses) and kept those created from
2026-09-15 to 2026-09-29. That gave 126 sessions across 38 pull requests in five repos, all started
by one user.

**Baseline columns.** Each column is an API read, not taken from a transcript:
- `pr_human_commits` counts pull request commits whose first author isn't Copilot. Copilot lists
  the assigner as a co-author on its own commits, so the first author is the only reliable signal.
  It includes Claude Code commits made from Mark's account.
- `sessions_on_issue` and `session_seq_on_issue` group sessions by the linked issue, falling back
  to the pull request when there is none.
- `ai_credits_raw` is the undocumented field as returned.
- `problem_categories` uses one slug per category in Problems found, in table order:
  `run-end-skipped`, `house-rules`, `environment`, `design-in-review`, `false-failed`,
  `dispatched-unverifiable` and `bot-review-forwarded`.

**Sample.** I started with all four `failed` and both `cancelled` sessions (all on clhbid.com).
Next I added an initial session from each other repo, then the pull requests with the
most rework: the most sessions, the most human commits or the most change requests. The last four
added (`a69db851`, `6fbb5ef4`, `e1424928`, `52a195cd`) turned up no new category, so I stopped at
18. That covers 27.8% of the cycle's credits.

| Session | Pull request | Why sampled |
| --- | --- | --- |
| `ca431ee7-3485-436a-a3fc-e10cf61dc6d1` | [clhbid/clhbid.com#2397](https://github.com/clhbid/clhbid.com/pull/2397) | failed |
| `3a5173c1-cd35-41db-8a8d-f7ff308bcc01` | [clhbid/clhbid.com#2397](https://github.com/clhbid/clhbid.com/pull/2397) | failed |
| `d297852b-4891-4c54-bba1-43cbb5392b23` | [clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423) | failed |
| `2929c632-8555-4c1e-bcfb-bb5608f39f97` | [clhbid/clhbid.com#2423](https://github.com/clhbid/clhbid.com/pull/2423) | failed |
| `0d9304fe-49b5-4f48-ad91-44b934fed40c` | [clhbid/clhbid.com#2386](https://github.com/clhbid/clhbid.com/pull/2386) | cancelled |
| `6bb19ee3-7d6a-4d17-8632-6d699c3e808a` | [clhbid/clhbid.com#2405](https://github.com/clhbid/clhbid.com/pull/2405) | cancelled |
| `8a3f1a5f-0fcb-4967-98f9-bed3e86b9fe7` | [clhbid/devcontainer-features#3](https://github.com/clhbid/devcontainer-features/pull/3) | repo; 10 human commits |
| `93f88062-345f-49bb-9d44-e1473dd144f8` | [clhbid/agent-context#27](https://github.com/clhbid/agent-context/pull/27) | repo |
| `12da5aaa-45bb-4550-b9a2-f29f3b122171` | [clhbid/CLHbid-LiveAuction#1006](https://github.com/clhbid/CLHbid-LiveAuction/pull/1006) | repo; 6 human commits |
| `eb71f8c5-d4e7-4f32-aa0e-33b595c2495f` | [clhbid/canadian-time-zone-hotpatch#14](https://github.com/clhbid/canadian-time-zone-hotpatch/pull/14) | repo; most credits of any session |
| `44ae9cd2-7fea-405c-ae12-23ac8acf32dc` | [clhbid/clhbid.com#2384](https://github.com/clhbid/clhbid.com/pull/2384) | 12 sessions, 9 change requests |
| `d9f75f9b-43fa-4d5b-a86d-584f338a054d` | [clhbid/CLHbid-LiveAuction#1024](https://github.com/clhbid/CLHbid-LiveAuction/pull/1024) | 8 sessions |
| `f4844586-3fc1-43ff-bcb3-2776ca7f6e7b` | [clhbid/canadian-time-zone-hotpatch#42](https://github.com/clhbid/canadian-time-zone-hotpatch/pull/42) | 6 sessions, 5 human commits |
| `40122156-af44-4d20-88d6-f60359979574` | [clhbid/clhbid.com#2390](https://github.com/clhbid/clhbid.com/pull/2390) | 7 sessions |
| `a69db851-3f43-4426-b2ff-a981b546a922` | [clhbid/clhbid.com#2412](https://github.com/clhbid/clhbid.com/pull/2412) | Fix with Copilot; 5 human commits |
| `6fbb5ef4-b36d-4ea9-b6cd-9fa1b94a3fbf` | [clhbid/CLHbid-LiveAuction#1024](https://github.com/clhbid/CLHbid-LiveAuction/pull/1024) | PR comment follow-up |
| `e1424928-8b49-4898-82ff-c5deef692492` | [clhbid/CLHbid-LiveAuction#1027](https://github.com/clhbid/CLHbid-LiveAuction/pull/1027) | second-highest credits |
| `52a195cd-fe51-40b6-b892-54a1e0ffe90b` | [clhbid/canadian-time-zone-hotpatch#41](https://github.com/clhbid/canadian-time-zone-hotpatch/pull/41) | Fix with Copilot |

For each session I outlined the transcript for tool calls,
non-zero exits and progress updates, then read the failures and the final message. I read the
pull request's reviews and commits alongside it, and checked the Actions run behind each `failed`
and `cancelled` session.

**Review comments.** I classified all 160 inline review comments posted from Mark's account. I
dropped 34 that Claude Code or the agent had posted there as replies ("Fixed in …"), which left
126.

**Limits.**
- One reviewer wrote nearly all the reviews, so "house rules" are one person's rules.
- Human-time figures are estimates from counts, not measured time.
- Sampling leans towards clhbid.com (9 of 18) because every failure was there.
- `ai_credits` is undocumented and may change, and sessions report no token counts.
- The dependency-graph explanation for the `failed` state rests on four runs plus one completed
  run that skipped the step. It is a strong pattern but not a confirmed cause.

**Where sources disagree with what we do.**
- This repo's README says Copilot reads `AGENTS.md` "and nothing else". GitHub documents
  `.github/copilot-instructions.md`, path-specific instructions and organization instructions as
  well [[1]](#sources) [[7]](#sources).
- Our `AGENTS.md` files ask the agent to run `yarn install` itself. GitHub recommends installing
  dependencies deterministically in setup steps [[5]](#sources).
- Our run-end contract assumes org-board access that the agent's token doesn't have
  [[3]](#sources) [[4]](#sources).
- The AGENTS.md convention [[8]](#sources) and Anthropic's guidance to find "the smallest set of
  high-signal tokens" [[9]](#sources) both support keeping the rules short. That is why
  recommendation 1 is four lines.

## Sources

1. <https://docs.github.com/en/copilot/customizing-copilot/adding-organization-custom-instructions-for-github-copilot>
2. <https://docs.github.com/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises>
3. <https://docs.github.com/en/copilot/responsible-use/copilot-cloud-agent>
4. <https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent>
5. <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment>
6. <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-firewall>
7. <https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions>
8. <https://agents.md/>
9. <https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>
10. <https://github.com/mattpocock/skills/blob/main/skills/productivity/writing-for-agents/SKILL.md>
