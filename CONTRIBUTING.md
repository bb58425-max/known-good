# Contributing

The most valuable contribution is **one field note** about something that cost you real time.

## Submitting a note

1. Copy [`docs/field-note-format.md`](docs/field-note-format.md) and fill it in.
2. Name the file `entries/YYYY-MM-DD-short-slug.md`.
3. Open a PR. In the description, say the one thing reviewers need: **were you there when the fix
   worked?**

## The review bar

A note is merged when it is:

- **Attributable** — it names a recorder, a machine fingerprint and a date.
- **Ordered** — the elimination section is a sequence with a result and a conclusion per step, not a list
  of facts.
- **Honest** — unresolved is stated as unresolved; unverified is labelled unverified; dead ends are
  included.
- **Clean** — no credentials, tokens, serial numbers or personal identifiers.

A note is rejected if it presents a general claim as a verified fix, or a verified fix as a general claim.
That distinction is the whole point of the corpus and it is the one thing worth being pedantic about.

## Also welcome

- **Reproductions.** Re-running someone's elimination tree on your hardware and recording the result is
  as valuable as the original note. Say so in `verified_by`.
- **Contradictions.** If your identical machine behaved differently, file a note and cross-link it in
  `contradicts`. That is a finding, not a dispute.
- **Fingerprint filling.** Adding exact model, board revision and firmware versions to existing notes
  makes them findable.
- **Format and tooling.** Critique the schema in an issue. Phase 1 (an MCP server over the corpus) is
  unstarted and is the largest piece of real work.

## Not welcome

Vendor marketing, unreproducible "it works for me" comments, or anything that requires trusting an
unattributed claim. The corpus is only worth anything if every entry can be traced to an afternoon
somebody actually spent.
