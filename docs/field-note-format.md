# Field note format

A field note is a **claim about the world**, made by someone who was there. It is the atomic unit of the
corpus. It is not an article, not a tutorial, and not a conversation — it is evidence.

## Rules

1. **Provenance or deletion.** Every note names who recorded it, on what hardware, when, and whether they
   personally saw the outcome. A note without that is removed, not merged.
2. **Observation before interpretation.** Say what appeared on the screen, in the BIOS, on the serial
   line. Do not say what it "means" until the evidence section is complete.
3. **The elimination tree is the product.** The fix is often a footnote. The value is the ordered list of
   tests and what each one proved — including the branches that were wasted.
4. **Unresolved is a valid outcome.** "Root cause unknown, RMA'd" is an entry. It stops the next person
   from assuming the question was answered.
5. **Label everything you did not personally verify** with `unverified:` or `unknown:`. Never guess in the
   voice of certainty.
6. **No credentials, tokens, serial numbers or account identifiers.** Fingerprint the machine, not the
   owner.

## Schema

```yaml
id:            kebab-case-slug
date:          YYYY-MM-DD            # when the work happened
fingerprint:                         # how a machine finds its own relevant notes
  vendor:
  model:
  board_revision:
  firmware:                          # BIOS/UEFI version, exact
  platform:                          # CPU/SoC family
  storage_topology:                  # slots, lanes, bifurcation, what is populated
  os:
symptom:       one line, as observed
evidence:                            # what was actually seen; quote log/BIOS text verbatim
  - ...
elimination:                         # THE core section: ordered, each with a result
  - step: what was tried
    result: what happened
    proved: what that rules in or out
root_cause:    what it actually was, or "unresolved"
resolution:    what fixed it, or "RMA'd" / "workaround"
known_good:    facts this session established (parts/configs proven working)
time_cost:     how long it took
time_expected: what it should have taken, and why
confidence:    high | medium | low  — plus a one-line reason
applies_to:    [models / firmware ranges / revisions]
contradicts:   [ids of notes with conflicting outcomes]
recorded_by:   who
verified_by:   who reproduced it, or "unreproduced"
```

## Why `elimination` is not optional

A symptom can have twenty causes. An ordered elimination tree with results is the only artifact that
turns a two-hour investigation into a two-minute check — because the *second* person does not need the
answer, they need the **order**. They need to know which test to run first, and what each result means.

## Why `time_expected` is required

It is the only honest measure of the corpus's value. If a note says "took 4 hours, should have taken 20
minutes", it is telling the next reader exactly how much this document is worth. Collect those numbers
and the argument for the whole project makes itself.
