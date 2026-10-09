# agent-context

The agent context every clhbid repo shares — tracker conventions, the AFK loop, the business-update
cycle, and the pull request template — as installable [Agent Skills](https://agentskills.io).

Nothing here is copied into a repo. A container installs it, so there is one place to change
something and every repo picks it up.

## Install

```bash
npx -y skills add clhbid/agent-context --global --yes
npx -y skills add mattpocock/skills --global --yes
```

`--global` puts the skills in the user directory so they survive across repos in a container;
`--yes` is required in a non-interactive `postCreateCommand`. The repo is public, so neither line
needs a token.

Later:

```bash
npx skills update --global --yes   # pull the latest
npx skills list                    # what's installed, and from where
```

**`--global` is not optional on `update`.** The skills were installed with `--global`, and a bare
`npx skills update` only considers *project* skills — from a directory with none it prints
"No project skills to update" and exits successfully, having changed nothing. `--yes` skips the
scope prompt, which a container has no way to answer.

## What's in it

| Skill | What it covers |
| --- | --- |
| `issue-tracker` | The delivery language, issues via `gh`, the `Status` field, the board query recipes, triage roles, cycles, epics, issue types, labels, the commit convention, and how to decompose work |
| `afk-loop` | Which work to hand to Copilot versus Claude Code, what makes a complete agent brief, reviewing what comes back, and what to do when a run goes wrong |
| `cycle-review` | The recurring cycle-review meeting: publishing the business-first notes as a GitHub Discussion and email agenda, and processing the returned decisions back into `Cycle` and `Status` |
| `email-to-issue` | Run by typing `/email-to-issue`. Turns flagged Outlook threads into issues or comments through the Microsoft 365 connector. A `/morning` section such as "Flagged Outlook email not yet in GitHub" lists them, and its button starts the sweep |
| `open-pr` | Opening, pushing and updating a pull request: base branch, title, closing reference, size check, ready or draft. Upstream `pr` writes the body |
| `house-rules` | The rules every agent loads before changing code: testing, single source of truth and spec style, deferring to `writing-lean` |
| `writing-lean` | How to write fluent code, and its comments, docs, pull requests, issues, reviews and commits — every other skill points here |

`skills/issue-tracker/board.graphql` sits beside the skill that uses it: one query against the
delivery board, filtered per use with `--jq`. `snapshot.graphql` beside it reads every item's
`Cycle`, archived items included, for a cycle configuration write. The recipes resolve both from
wherever the skill was installed.

`labels.yml` is the shared label vocabulary — the wayfinder ticket types and `security`. Labels
carry neither state nor category; state is the `Status` field on the board, and category is the
issue type.

## What does *not* belong here

**Anything that has to be copied into a repo to be useful.** That is the whole point: if a file
must live in the repo, it belongs in that repo's `AGENTS.md` instead, where it is reviewed in a
pull request alongside the code it constrains.

Concretely, each repo keeps its own:

- **`AGENTS.md`** — build, test and lint commands, and the **How a run ends** contract. Copilot
  reads this and nothing else, so it has to be self-sufficient.
- **`GLOSSARY.md` and `docs/adr/`** — domain vocabulary and decisions.

Matt Pocock's skills are **installed alongside** these rather than vendored, so upstream fixes keep
flowing and we maintain only our own. We don't use `implement-spec` for now: it lands a whole spec
as one pull request.

## Changing a skill

Edit it here, open a pull request, and merge. Consumers pick it up on the next
`npx skills update --global --yes` or the next container build — there is no sync step and no CI
check, because nothing is copied.

Skill bugs are filed as `enhancement` issues and flow through the same board as everything else.
Edit skills with `/writing-for-agents`.
