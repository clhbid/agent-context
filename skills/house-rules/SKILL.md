---
name: house-rules
description: The clhbid house rules. Use when writing, testing or reviewing code, issues or business documents in a clhbid repo.
---

# House rules

Write the code, and everything that describes it, with the `writing-lean` skill.

- **Test our code, not the dependency.** Test the behaviour our code adds, and take the package as
  given: import its values rather than copying its tables, and leave its behaviour to its own tests.
- **Single source of truth.** Take each value from where it is defined: the Tailwind scale
  (`max-w-7xl`, not `max-w-[1468px]`), a shared constant, or the package's own types. Reuse
  existing helpers before adding one.
- **Domain terms from the glossary.** `GLOSSARY.md` in clhbid/CLHbid-LiveAuction are the app's
  single source of truth: read it with the `domain-modeling` skill, use its terms, not those marked
  _Avoid_, and propose a missing term there rather than coining one.
- **Spec style.** Set up and tear down in the test framework's hooks. Arrange, act, then assert,
  opening each section with an `Arrange`, `Act` or `Assert` comment, and assert the whole result,
  so an extra item fails the test.
- **Fake the clock** whenever a spec depends on the time, so it passes at any hour and in any time
  zone. Code reads the time only through a clock a spec can fake: `TimeProvider` in .NET, not
  `DateTime.UtcNow`.
- **Write for the reviewer.** Review is our bottleneck; reviewer time is our most precious asset.
  Keep each pull request to one change, separate refactors from the behaviour they enable, and make
  code, tests and snapshots quick to follow. Before finishing, review your diff as a reviewer would
  and fix anything that slows them down.
- **Collect lessons before adding rules.** When a code review or triage turns up a lesson that might
  become a house rule, suggest running `/retro`, and comment what it finds on
  [AC#128](https://github.com/clhbid/agent-context/issues/128) with the finding that prompted it,
  rather than editing this skill. A rule is added once the collected evidence shows the problem
  recurs.
