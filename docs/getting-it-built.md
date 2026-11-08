# Getting it built — where to find a builder, and how to structure it

The short version, because two of these points decide everything else:

1. **The hardware is mostly integration, and the parts are commodity.** A tiny SBC, an off-the-shelf HDMI
   capture path, USB HID, a relay, a UART — all modules. A competent maker is assembling, not inventing.
   That is what makes "on the cheap" realistic.
2. **"Cheap" and "exclusive rights" pull against each other.** Free labour wants no strings; exclusivity is
   something you pay for. The workable structures exist, and one of them (an **option agreement**) maps
   exactly onto *"licensing rights if it's marketable."*
3. **And the honest constraint:** this repository is a **defensive publication**. The corpus mechanism —
   skills, elimination trees, cross-unit correlation — is now free to everyone and cannot be exclusively
   licensed by anyone. What can still be owned: a specific implementation, the name and brand, a hosted
   service, the hardware design files, and support. Plan around that rather than pretending otherwise.

## Where to find people, ranked by what they cost

### 1. Upstream: the projects that already built the hard part — cost ≈ $0
[PiKVM](https://github.com/pikvm/pikvm), [JetKVM](https://jetkvm.com) and the MCP projects
([kvm-pilot](https://github.com/DustinTrap/kvm-pilot), [Agentic-KVM](https://github.com/MahdiHedhli/Agentic-KVM))
have maintainers who are precisely the people who would want this. Open an issue or discussion with the
appliance spec attached and ask whether the corpus layer belongs upstream. This is the cheapest and most
durable path, and it produces no exclusivity — which may suit fine, given the defense-publication point
above. It also happens to be the honest reading of *"write it up for some nut to work on."*

### 2. A local university capstone team — cost: parts, plus patience
Senior design / capstone courses in EE and CS exist to build exactly this kind of defined, self-contained
system, with a semester of supervised student time. You supply the spec, the acceptance test, and parts.
Expect to negotiate IP in writing: student work is frequently owned by the student, sometimes shared with
the university. Slow, genuinely capable, and near-free.

### 3. Hackaday.io and the open-hardware community — cost: parts, plus a bounty
[Hackaday.io](https://hackaday.io/) is the largest collaborative hardware development community and is
built for exactly this: post the project, state what you want built, and recruit collaborators. A small
cash bounty ("first working reference build") with a **non-exclusive license back** is a well-trodden
pattern and keeps the work moving without you paying a design house.

### 4. A freelancer or design house — cost: hundreds to a few thousand
Realistic ranges for this scope, because it is integration rather than invention:

| Scope | Typical cost |
|---|---|
| Breadboard reference build from existing modules, firmware + agent wiring, working demo | ~$300–1,500 |
| Custom PCB (schematic → layout → prototype run), enclosure designed and printed | ~$1,500–5,000 |
| Enclosure tooling for injection moulding | Add an order of magnitude — don't, until there is real demand |
| Turnkey fabrication (PCBWay, Seeed Fusion, JLCPCB, MacroFab) | Per-unit, low hundreds at qty 10 |

Places: [PCBWay design services](https://www.pcbway.com/design-services.html),
[Upwork hardware engineers](https://www.upwork.com/hire/hardware-engineers/),
[Cad Crowd](https://www.cadcrowd.com/hire/pcb-design) (vetted), Fiverr (cheap, variable).
Local **maker spaces and fab labs** are underrated: you hand someone the box and parts, and they are right
there.

## Write the ask before you shop for people

The price is set by how precisely you define done. This project has an unusually clean acceptance test:

> **Reference build is complete when** the device is plugged into a machine with no OS and no working
> storage, and — unattended — it captures the BIOS screen, mounts a bootable image from its library,
> installs, force-reboots via the relay, then walks a recorded elimination tree and produces a **draft
> field note** whose root cause a human confirms.

Publish that, the BOM from [`appliance.md`](appliance.md), and the field-note format, and the project
becomes biddable in an afternoon. Hand over a vague idea and you will pay for the vagueness.

## Deal structures — pick one on purpose

**The rule that decides whether you own anything:** in the US, absent a **written assignment**, the
author owns the copyright and the inventor owns the patent rights. "I paid him" is not the same as "I own
it." Whatever structure you choose, it is a document, not a handshake. *(Not legal advice — a short
contract reviewed once by a lawyer is the cheap insurance here.)*

| Structure | You get | They get | Use when |
|---|---|---|---|
| **Work-for-hire + written IP assignment** | Full ownership of design files and code | Cash (modest is fine, if the assignment is in writing) | You intend to sell or license it |
| **Option agreement** | 60–90 days of exclusivity to evaluate the market, on defined terms | A small fee, and time to decide | Exactly "licensing rights **if it's** marketable" — this is the professional structure for that sentence |
| **Royalty license (exclusive, field-limited)** | Sole right to manufacture/sell in a defined market | Royalty per unit (commonly single-digit %) with a minimum | The builder wants upside, not a fee |
| **Bounty + non-exclusive license back** | Free right to use and sell a specific build | Small prize, credit, and a portfolio piece | Community route; cheapest, least control |
| **Open-core split** | Brand, hosted service, support, enclosure design | Open firmware and hardware files under a real licence | You want adoption more than exclusivity |

**Check the licences of whatever you build on** before choosing any exclusive structure. Copyleft
software (GPL-family) and open-hardware licences constrain what you can close, and that is a real
constraint on "exclusive" — worth reading before the structure is chosen rather than after.

## The recommended sequence

1. **Publish the brief and the acceptance test** (this document, `appliance.md`, the format).
2. **Post it to Hackaday.io and open an issue upstream.** Cost: an afternoon. You find out within weeks
   whether somebody builds it for the love of it.
3. **In parallel, price a local freelancer** for the breadboard reference build, with a written assignment
   clause. A few hundred dollars answers "does this box actually work."
4. **Only then** consider exclusivity — and if you do, use an option agreement, not a handshake.
5. **Do not tool up for manufacturing** until a reference build exists and somebody who is not you wants
   to buy one. At $60–150 with open-source competition, the box is unlikely to be the product; the support
   and the corpus are.
