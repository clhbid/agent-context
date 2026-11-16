---
name: domain-docs
description: Read local and LiveAuction domain documentation before exploring or proposing domain changes; use its glossary terms, surface glossary and ADR conflicts, and record shared knowledge in LiveAuction.
---

# Domain docs

Read domain documentation before exploring a codebase or proposing a domain change. Adapted from
Matt Pocock's [`domain.md`](https://github.com/mattpocock/skills/blob/main/skills/engineering/setup-matt-pocock-skills/domain.md).

## Read before exploring

1. Read this repo's `GLOSSARY.md` and `docs/adr/`, when present.
2. Read the corresponding files in
   [clhbid/CLHbid-LiveAuction](https://github.com/clhbid/CLHbid-LiveAuction). Use `gh api` to fetch
   `repos/clhbid/CLHbid-LiveAuction/contents/GLOSSARY.md` and list
   `repos/clhbid/CLHbid-LiveAuction/contents/docs/adr/`, then fetch each ADR with the same contents
   endpoint and decode its `.content` from base64. In LiveAuction itself, the local files are these
   same files, so read them once.

If a glossary or ADR directory is absent locally or in LiveAuction, continue silently.

## Apply the domain model

- LiveAuction's glossary and ADRs are the org's source of truth. Flag a local term or ADR that
  conflicts with them; surface the conflict rather than choosing one silently.
- Use glossary terms in issue titles, proposals and test names. A missing term is either invented
  language to reconsider or a gap to note for `/domain-modeling`.
- Flag output that contradicts an ADR rather than overriding it silently.
- Write recorded domain knowledge to LiveAuction unless it applies only to this repo.
