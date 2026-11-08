# known-good

**A shared, verifiable knowledge base of what actually works on real hardware, in real environments.**

Every hard-won afternoon becomes an entry. Every entry carries provenance. Nothing is recorded that was
not verified by the person who recorded it.

---

## The problem

A brand-new mini-PC arrived with a dead M.2 subsection: three slots, none of them enumerating, the
storage driver never loading. The RAM was fine. The CPU was fine. External storage was fine. The drive
itself was fine. Finding that out took **hours** — a human and a general-purpose LLM working through it
live, one idea at a time, with nobody writing anything down.

That investigation was not novel. Somebody has hit that exact fault, on that exact board family, and
walked that exact elimination path before. **It was never written down anywhere a machine could read.**

That is the gap. Not transport — transport is solved (see below). The gap is that **experience does not
accumulate**. Each technician, each hobbyist, each agent starts from zero on problems that have already
been solved somewhere, and the cost is paid again, in full, every time.

## What already exists

Being precise about this matters more than being first. The remote-hands layer is built and cheap:

| Project | What it does |
|---|---|
| [PiKVM](https://github.com/pikvm/pikvm) | Open-source KVM-over-IP. HDMI capture, full keyboard/mouse emulation, ATX power control, virtual media, and IPMI/Redfish emulation. The mature one. |
| [JetKVM](https://jetkvm.com) | Cheap open-source KVM-over-IP appliance, 1080p60, low latency. Consumer-priced. |
| NanoKVM / GL.iNet Comet / TinyPilot Voyager | Same category, varying price, capture resolution and openness. |
| [kvm-pilot](https://github.com/DustinTrap/kvm-pilot) | An MCP server giving agents write-capable control across **KVM + BMC + SSH**, explicitly *gated, verified, audited*. Drives POST, firmware, bootloader and OS install at the pixel level. |
| [Agentic-KVM](https://github.com/MahdiHedhli/Agentic-KVM) | Gives agents bare-metal out-of-band control through PiKVM, Redfish or iLO. Installs OSes, recovers bricked machines. |
| [PiKVM MCP server](https://github.com/KultivatorConsulting/pikvm_mcp_server) | Lets an agent drive a machine through PiKVM: screenshot, click, type. |
| Redfish MCP servers | Hardware inventory, sensors, power, boot order, firmware updates, via the standard BMC API. |

So: **an agent driving a machine's keyboard, mouse, video and power is a solved problem.** Do not rebuild it.

## What does not exist

None of the above accumulates knowledge **across units, across sessions, across people** in a form an
agent can query before it starts guessing. Each is a session-scoped pair of hands. When the session ends,
everything learned dies with the context window.

Missing pieces, in order of value:

1. **A machine-readable field corpus keyed by hardware fingerprint and environment** — board, firmware
   revision, platform, storage topology, OS — with an ordered **elimination tree** rather than a
   knowledge-base article about the symptom.
2. **Provenance and confidence on every entry.** "This fixed it on this model at this firmware" is a
   different claim from "this is how it generally works." The corpus must never blur them.
3. **Contradiction detection.** Two units of the same model, opposite outcomes — that is *information*,
   not noise, and it should raise a flag rather than sit unnoticed in two files.
4. **Automatic capture.** The elimination tree should be recorded *as it is walked*, so the expensive
   part is captured by default rather than by discipline.
5. **A shared upgrade path** so one operator's afternoon of pain becomes every operator's first-line
   check. Distributed, verified, and attributable.

That is what this project is. The hands exist; **the memory does not.**

## Design

**Phase 0 — the format and the corpus (now).** No code. A plain-text field-note format
([`docs/field-note-format.md`](docs/field-note-format.md)) and a directory of real entries. Usable today
by a person with a KVM and a text editor. See [`entries/`](entries/) — the first entry is the fault that
started this.

**Phase 1 — a corpus server over MCP.** Expose `search_by_fingerprint`, `get_entry`, `propose_entry` and
`get_elimination_tree` so any agent stack (including the projects above) can consult the corpus *before*
improvising. Read-only first; writes go through human review.

**Phase 2 — capture.** Wrap an agent's console session and record every action, its result, and the
branch it caused. The transcript becomes a *draft* entry. The operator's only job is to confirm root
cause and resolution. The expensive part gets recorded as a side effect of doing the work.

**Phase 3 — sync and trust.** Signed entries, per-fingerprint aggregation, confidence scoring based on
independent reproductions, and explicit conflict flags. Provenance is not optional: an unattributed entry
is deleted, not merged.

**Phase 4 — the appliance.** One small box per machine, specified in full in
[`docs/appliance.md`](docs/appliance.md) — hardware BOM, the image library and working space, the local
skill router with a generic fallback, the gating model, and the prior-art/patentability reality. At a
glance:

- HDMI capture in (1080p60 minimum, for BIOS/UEFI compatibility)
- USB HID out — keyboard and mouse emulation
- **Serial console** (USB-UART) — the highest-value channel nobody mentions: many failures show
  themselves on serial when there is no video and no OS
- Power relay, for forced reboot and cold power cycles
- The managed machine's own network, bridged for out-of-band access
- Enough local compute for a small model + local RAG over the corpus, so it works with the WAN down
- Cloud sync of *entries only* — never of credentials, never of live video

**Phase 5 — hands.** Robotics is explicitly out of scope until the corpus, the capture loop and the trust
model are all real. Everything above has value before a single joint moves.

## Non-negotiables

- **Verified or absent.** An entry records what was done and observed, in that order. Untested hunches are
  labelled as hunches. A claim with no provenance is deleted.
- **Record the dead ends.** The expensive part of troubleshooting is not the fix, it is the elimination.
  A tree that omits the branches that failed is worthless.
- **Humans authorize state changes.** The agent may read freely and propose anything; anything that
  changes a machine is gated, logged and reversible. This is the difference between a tool and a hazard —
  and it is why [kvm-pilot](https://github.com/DustinTrap/kvm-pilot) gating its writes is the right
  pattern, not an inconvenience.
- **No credentials, ever, in an entry.** Fingerprints, not secrets.
- **The corpus is for machines and people equally.** Plain text, diff-able, no proprietary format.

## Contributing

Easiest possible contribution: **one field note.** Something cost you two hours; write down the
elimination tree. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the format and the review bar.

## Origin

A dead M.2 subsection in a new mini-PC, and an afternoon that should have taken twenty minutes. The
entry written from it is in [`entries/`](entries/) — including the parts that were never resolved.

## Status

**Specification. No code yet.** This is a call for collaborators: the format, the corpus and the gated
write path are the interesting problems, and they are all still ahead.

## License

MIT — see [LICENSE](LICENSE).
