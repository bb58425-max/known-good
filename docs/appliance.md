# The appliance — a reverse-KVM box that can bring bare iron back to life

The corpus in this repository is the brain. This document is the **body** it belongs in: a small box that
can drive any machine that has no OS, no network and no working storage — because it interfaces the machine
at the level below all of those.

> **Origin of this spec.** It came from a bad afternoon: a new mini-PC with a dead M.2 subsection, three
> slots enumerating nothing, and hours spent looking at a box that could not be asked any questions. The
> conclusion was not "try harder" but "there should be a device plugged into this that can see what the
> machine sees."

## What it must do

Drive a machine that is **dead as far as software is concerned**:

- see what it displays, at BIOS/UEFI level, before any OS exists (**HDMI capture in**)
- type and click as though sitting in front of it (**USB HID out** — keyboard and mouse emulation)
- read the channel that survives when there is no video at all (**serial/UART console**)
- power-cycle it without a person present (**relay / ATX control**)
- hand it media it can boot from (**virtual media** — an image library it can mount as if it were a USB stick)
- reason about what it sees, using **skills it already knows** and a generic tree when it doesn't
- remember everything it learned, in a form other units can use

## Bill of materials

| Part | Requirement | Notes |
|---|---|---|
| Compute | small SBC or N100-class box | must be able to host a small language model + local retrieval |
| HDMI capture | 1080p60 minimum | 1080p is the BIOS-compatibility floor; some firmware outputs only at low modes |
| USB HID | full keyboard + absolute mouse | relative-only mouse makes BIOS navigation painful |
| Serial | USB-UART, 3.3 V tolerant | **the highest-value channel nobody mentions** — many failures are visible here and nowhere else |
| Power control | relay, plus optional ATX header | brief press = power, long press = force off; must fail to *no change* if the controller dies |
| Network | 2.5 GbE, bridged to the managed machine's own segment | out-of-band must not depend on the in-band network |
| Secondary uplink | optional LTE/5G | for the case where the whole site is down |
| Storage | enough for an image library + working space | see below |
| Enclosure | cheap. This is a $60–150 class device, not a premium one | the value is in the software |

## The library and the working space

Two distinct stores:

- **Library** — bootable OS images, rescue ISOs, vendor firmware, driver sets, and offline package sets.
  Mounted to the target via virtual media, on demand. (Prior art: this is what KVM-over-IP products call
  *virtual media*, and what BMCs have offered for years. Do not reinvent the mechanism — reinvent what it
  *knows*.)
- **Working space** — captured screenshots, serial transcripts, POST/boot logs, the action log of
  everything the agent did, and draft field notes awaiting review.

The library is what makes "restore from nothing" possible with **no network at all**: image, package set and
firmware all local, as prepared by the offline-cache pattern in `../minisforum-bringup/`.

## The brain: a skill router, not a chatbot

The requirement, precisely stated, is: *a small local model that knows which skill to apply, or falls back
to a generic approach when it does not recognise the situation.*

1. **Identify.** Read the machine's fingerprint — vendor, model, firmware, platform, storage topology, what
   it reports about itself. This is the key that selects prior knowledge.
2. **Match.** Query the corpus for entries whose fingerprint overlaps and whose symptom plausibly matches.
3. **Route.**
   - **Known skill** → execute the recorded elimination tree, in order, stopping where the evidence diverges
     from what the entry predicted. A diverging result is *information*: it either invalidates the entry or
     identifies a variant.
   - **Unknown situation** → run the generic tree (power/posts? video? storage enumerated? bootable media?
     serial silent?) and record the whole path.
4. **Record.** Either path produces a draft field note automatically. Root cause and resolution get
   confirmed by a human; everything else is captured as it happens.
5. **Publish.** Entries sync outward. Nothing else does — never credentials, never live video, never the
   target's data.

A small model is sufficient and preferable: the *knowledge* lives in the corpus, and the model only has to
route and recognise evidence. That is a categorisation task, not a reasoning marathon — which is exactly
why this can run offline on cheap hardware.

## One unit per machine, one channel per machine

Each appliance owns exactly one managed machine and one conversation channel. Fleet-scale correlation is a
separate, higher layer:

- per-machine: skills, fingerprints and local history
- fleet: "three other units with this board revision hit the same symptom, and the lane matrix explains it"
- the fleet layer **never** needs write access to any machine — it aggregates entries, flags contradictions,
  and raises confidence

That split is what lets the system be both useful and safe: judgment stays local, learning is shared.

## Safety, gating, and the human

- **Read freely, write under gate.** Anything that changes a machine's state is authorised, logged and
  reversible. This is the pattern [`kvm-pilot`](https://github.com/DustinTrap/kvm-pilot) already takes, and
  it is the right one.
- **Auditability is what preserves human authority.** If the agent cannot state what it is about to do in a
  form a person can accept or reject in seconds, "human in the loop" is decorative.
- **Fail to no-change.** A dead controller must never hold a power line or an input line in a stuck state.
- **No credentials, ever.** Fingerprints, not secrets — in the corpus *or* in the appliance's logs.
- **Assume the managed machine's network is hostile**, because that is precisely when out-of-band is needed.

## Prior art and patentability — read this before filing anything

Every *component* has been shipping for years, and much of it is open source:

- KVM-over-IP with BIOS-level access and no OS: Intel vPro/AMT, ATEN and other KVM-over-IP switches, IPMI
  BMCs, iDRAC/iLO, and open hardware (PiKVM, JetKVM, NanoKVM, TinyPilot).
- Virtual media / ISO mounting as a boot source: standard in the above for years.
- Power control: ATX/relay control is a checkbox feature on every product in this category.
- Agents driving KVM/BMC directly: `kvm-pilot` and `Agentic-KVM` (2026), plus MCP servers for PiKVM and
  Redfish.
- Vendors are already marketing AI-assisted server management.

So a broad claim over "AI + KVM + relay + image library" would meet published prior art for the
combination, and AI/method claims additionally face the §101 eligibility bar under *Alice*, where recent
Federal Circuit decisions have been unkind to claims of the form "apply a known technique to a known
problem with a computer."

The part for which no prior art was found in this survey — and therefore the only part worth a lawyer's
time — is **the accumulating, provenance-carrying experience layer: per-environment skills, ordered
elimination trees, cross-unit correlation and confidence**. Note carefully that this repository *publishes*
that mechanism, which makes this document a **defensive publication**: it forecloses patenting it, and
guarantees nobody else can either. That is a deliberate trade — freedom to operate, and a standard anyone
can build on, in exchange for exclusivity. If exclusivity is ever wanted, that decision has to be made
*before* publishing any further detail, and with counsel.

Not legal advice. For an actual opinion, the person to ask is a patent attorney.

## What to build first

1. Buy the cheapest existing KVM-over-IP device that meets the BOM, and one image library.
2. Write field notes by hand while using it. **Phase 0 of the corpus is a text editor and discipline.**
3. Add the capture loop so notes draft themselves as the tree is walked.
4. Only then build the appliance — because by that point it will be obvious which parts are worth
   integrating and which are worth making.
