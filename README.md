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
npx skills update          # pull the latest
npx skills list            # what's installed, and from where
```

It also works as a Claude Code plugin marketplace:

```
/plugin marketplace add clhbid/agent-context
```

## What's in it

| Skill | What it covers |
| --- | --- |
| `issue-tracker` | Issues via `gh`, the `Status` field, the board query recipes, triage roles, cycles, labels, the commit convention, and how to decompose work |
| `afk-loop` | Which work to hand to Copilot versus Claude Code, what makes a complete agent brief, reviewing what comes back, and what to do when a run goes wrong |
| `cycle-review` | The recurring business-update cycle. **Known stale** — still written against GitHub milestones; see the banner in the skill |
| `open-pr` | Opening and updating a pull request, with the size backstop |

`skills/issue-tracker/board.graphql` sits beside the skill that uses it: one query against the
delivery board, filtered per use with `--jq`. The recipes resolve it from wherever the skill was
installed.

`labels.yml` is the shared label vocabulary — a category and the wayfinder ticket types. Labels
never carry state; state is the `Status` field on the board.

## What does *not* belong here

**Anything that has to be copied into a repo to be useful.** That is the whole point: if a file
must live in the repo, it belongs in that repo's `AGENTS.md` instead, where it is reviewed in a
pull request alongside the code it constrains.

Concretely, each repo keeps its own:

- **`AGENTS.md`** — build, test and lint commands, and the **How a run ends** contract. Copilot
  reads this and nothing else, so it has to be self-sufficient.
- **`CONTEXT.md` and `docs/adr/`** — domain vocabulary and decisions.

Matt Pocock's skills are **installed alongside** these rather than vendored, so upstream fixes keep
flowing and we maintain only our own.

## Changing a skill

Edit it here, open a pull request, and merge. Consumers pick it up on the next `npx skills update`
or the next container build — there is no sync step and no CI check, because nothing is copied.

Skill bugs are filed as `enhancement` issues and flow through the same board as everything else.
Edit skills with `/writing-for-agents`.
