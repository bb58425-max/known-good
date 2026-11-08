---
id: minisforum-no-nvme-detected
date: 2026-11-07
fingerprint:
  vendor: Minisforum
  model: AI X1 Pro (Ryzen AI 9 HX 370)
  board_revision: unknown
  firmware: unknown — not recorded before RMA
  platform: AMD Ryzen AI 9 HX 370 (12C/24T) / Radeon 890M; dual DDR5-5600 SO-DIMM, board max 128 GB
  memory: 96 GB (2×48 GB DDR5-5600, both channels populated)
  storage_topology: 3× M.2 2280 PCIe 4.0 — SSD0 and SSD1 are PCIe 4.0 x4; SSD2 is PCIe 4.0 x1 (vendor spec)
  drive_intended: Samsung 970 EVO 2 TB (PCIe 3.0 x4 — backward compatible in a Gen4 slot)
  os: none installed — fault found during first bring-up
symptom: No NVMe device visible in BIOS on any of three M.2 slots; storage drivers never load; the machine otherwise powers up normally.
evidence:
  - BIOS storage/NVMe device list is empty with a known-good drive installed.
  - No storage-controller device appears, so the driver never loads.
  - RAM checks out; CPU functions; USB/external storage works normally.
  - The NVMe drive itself was verified good by isolating it (tested outside the machine).
elimination:
  - step: Power-on and check BIOS device enumeration
    result: empty device list, no NVMe entry
    proved: fault is below the OS — this is firmware/board level, not a driver-in-OS problem
  - step: Reseat the drive; try a second and then third M.2 slot
    result: identical on all three slots
    proved: not a single-slot fault, not a seating fault
  - step: Isolate the drive (test it outside this machine)
    result: drive works elsewhere
    proved: DRIVE IS KNOWN-GOOD — the fault is the host, not the media
  - step: Verify the rest of the platform (memory, CPU, external storage)
    result: all functional
    proved: KNOWN-GOOD — memory, CPU, USB/external storage
  - step: BIOS exploration (deep settings) and hard reset
    result: no change
    proved: not a stale-configuration problem reachable by reset
  - step: BIOS storage mode — legacy/CSM boot mode, and any AMD RAID mode toggle
    result: not recorded
    proved: nothing; flagged as a first-line check for this symptom
    note: Intel VMD/RST is an Intel-platform setting and does NOT apply to this AMD box; the analogous
      suspects here are CSM/legacy mode and AMD RAID (RaidX) mode
  - step: Vendor storage/lane matrix consulted
    result: recorded *after* the fault, from the vendor product page — see "Vendor spec" below
    proved: the three slots do not share one lane configuration (SSD0/SSD1 are x4, SSD2 is x1)
root_cause: unresolved — suspected dead NVMe/PCIe subsection or lane/root-port failure. Platform-level, since firmware never enumerates any device on three slots while the rest of the machine works. The lane matrix makes a lane-allocation explanation *less* likely: three dead slots spanning two different lane configurations point at something common upstream — the SoC PCIe complex, the M.2 power rail, or the connector plane.
resolution: RMA'd. No software fix was attempted beyond BIOS reset and drive isolation.
known_good:
  - external USB storage (worked throughout)
  - memory (passed)
  - CPU (functional)
  - the NVMe drive itself (works in another machine)
time_cost: several hours, including an extended live debugging session with a general-purpose LLM
time_expected: 20-30 minutes, if an ordered elimination tree had been available
confidence: medium — external fault strongly indicated and the drive proven good, but root cause was never confirmed because the unit was returned. Recorded so the next person does not assume the question was answered.
applies_to: [that unit; possibly this board family if the fault recurs]
contradicts: []
recorded_by: Bill (operator), with JJ Hermes (recording)
verified_by: unreproduced — unit RMA'd before confirmation
---

## Vendor spec (recorded after the fault, 2026-11-07)

From the vendor's product page for the AI X1 Pro:

| Item | Value |
|---|---|
| Platform | AMD Ryzen AI 9 HX 370 (12C/24T), Radeon 890M |
| Memory | dual DDR5-5600 SO-DIMM, **max 128 GB** (this unit: 96 GB = 2×48 GB) |
| SSD0 / SSD1 | M.2 2280 **PCIe 4.0 x4** — 6,700–7,300 MB/s |
| SSD2 | M.2 2280 **PCIe 4.0 x1** — 1,500–1,900 MB/s |
| Capacity | up to 12 TB across the three slots |
| Other | dual USB4, OCuLink, 2.5 GbE, Wi-Fi 7, SD slot |

Three consequences worth having on the record:

1. **The slots are not equivalent.** SSD2 is x1, not x4 — roughly a quarter of the bandwidth. The boot
   drive belongs in SSD0. Worth knowing *before* buying three drives.
2. **The fault spanned two different lane configurations.** SSD0/SSD1 (x4) and SSD2 (x1) failed
   identically. That is evidence *against* a lane-allocation or bifurcation cause, and *for* something
   common and upstream of the slots.
3. **The drive was never a compatibility question.** PCIe 3.0 x4 in a PCIe 4.0 slot is fully backward
   compatible — it simply runs at its own ~3.5 GB/s ceiling instead of the slot's ~7 GB/s. It was also
   separately proven working in another machine.
4. **No local redundancy, by design.** RAID was considered and rejected for this node: resilience comes
   from graceful failover to the cloud and to a second machine, not from a mirror inside one box. So a
   storage-subsystem failure is a **restore** event, not a **rebuild** event — which makes the
   snapshot/restore path the load-bearing component, not the array. Worth stating in a note like this
   because it decides what the next person should do when a slot dies: restore, don't rebuild.

## What this note is for

Not the answer — the **order**. Anyone hitting "no NVMe in BIOS" on a machine like this should be able to
run steps 1-4 above before spending an hour in deep BIOS, and should know that "drive works elsewhere" is
the test that splits host fault from media fault.

Two checks were never run at the time, and both are first-line for this symptom: **legacy/CSM boot mode**
and any **AMD RAID (RaidX) mode** toggle — note that Intel's VMD/RST is an Intel-platform setting and does
not apply to this box — plus the vendor **lane matrix** recorded above, which only surfaced afterwards.

## The meta-lesson

The hours were not lost to the fault. They were lost to the absence of:

1. an ordered elimination tree for a common symptom, and
2. per-model knowledge (lane allocation, storage mode quirks, known firmware issues), and
3. out-of-band access, so the machine could be inspected without a person sitting in front of it.

The first is a document. The second is a corpus. The third is a $60 appliance. This note is the beginning
of the first two.

## Follow-up — same day, later (2026-11-07)

New evidence arrived after the RMA was raised, and it **partly contradicts the root cause recorded above.**
Appended rather than edited, per this repository's rule: the front matter stands as what was known at the
time, because the sequence of belief is itself the useful part.

- The NVMe drive **is functional**: it is visible from **GRUB**, and **writes to it succeed**.
- The **firmware still does not enumerate it** — the drive is absent from the firmware's own storage view.
- The **RMA remains open and unshipped**, so this is the same unit with new information rather than a
  replacement.

**Revised assessment: not dead silicon — a firmware-level enumeration or visibility fault.** When software
drivers (Linux, GRUB) enumerate and drive a device correctly while the firmware does not list it at all, the
fault sits in the firmware's NVMe driver or its reporting, not in the drive, and not necessarily in the slot.

**The test that should have come first — recorded here so it does next time:** determine whether the drive
is visible to *software* before condemning hardware. A Linux live environment, or GRUB's own `ls`, answers
it in a minute. Writes succeeding is the strongest form of the answer.

**And the decisive firmware-side test:** from a **UEFI Shell**, `map -r` lists what the firmware *itself* can
read. If the NVMe's ESP appears there, the firmware can boot from it and only the boot entry needs creating
(`efibootmgr -c -d /dev/nvme0n1 -p 1 -L Fedora -l '\EFI\fedora\grubx64.efi'`). If it does not appear, the
firmware's NVMe driver genuinely will not bind, and the fallback is to keep the **ESP on a removable device
with the root filesystem on the NVMe**.

Also in the "cheap and first" class, and skipped on the day: **check for a firmware update.** Vendor BIOS
releases routinely carry NVMe enumeration fixes, and a firmware that fails where Linux succeeds is precisely
the signature of a firmware bug rather than a hardware one.

```yaml
follow_up:
  date: 2026-11-07
  status: drive proven functional (GRUB-visible, writes succeed); firmware does not enumerate it;
          RMA open and unshipped
  revised_root_cause: firmware NVMe enumeration/visibility fault — not dead silicon
  confidence: medium
  confidence_reason: the drive is proven good by write test; the firmware-visibility conclusion is inferred
                     from software seeing what firmware cannot, and is not yet confirmed by the UEFI shell
                     test or a firmware update
  contradicts: this note's own earlier root_cause
  next_tests: [UEFI shell `map -r`, vendor firmware update, create boot entry with efibootmgr]
```

## Correction to the follow-up above (same day)

The follow-up claims the drive is *"proven functional"* and that *"writes succeed."* That is **stronger than
the evidence supports** and is corrected here rather than quietly edited out — the recorder's own
overstatement is exactly the kind of thing this format exists to catch.

- The drive has been seen in **exactly one place: GRUB.** No installer, no installed OS, and no firmware
  storage page has enumerated it.
- The Ubuntu installer **hangs at various points.** It *appears* to be writing; **no write has been
  verified.**
- So "visible to software" is **not** established — only "visible to GRUB". Confidence drops from medium to
  **low**.

Two precision notes that make GRUB-visibility weaker evidence than it appears:

1. **The boot mode decides what it proves.** In **UEFI** mode GRUB reaches disks through the firmware's own
   block I/O, so seeing the drive would mean *the firmware* can see it — contradicting the reported symptom.
   In **legacy/CSM** mode GRUB uses its own drivers and says nothing about firmware visibility. Which mode
   was in use must be recorded before this observation carries any weight.
2. **An installer that hangs mid-write is a reliability signal, not only a visibility one.** Until a full
   write-and-verify pass completes (`f3write`/`f3read`, or `badblocks -w`), the drive's health is unproven.

**Revised status — honest as of this correction:** drive healthy? unknown. Slot working? unknown. Firmware
visibility? unknown. What *is* established: the drive is not entirely invisible, because GRUB listed it.

## Reassessment — the hardware reading is probably right (same day, later still)

The recorder's firmware-visibility hypothesis rested on *"the drive is visible to software and writes
succeed."* That claim has since been withdrawn by the person at the machine, and the hypothesis weakens with
it. The evidence now reads as **bus-level presence with a storage pathway that has never actually worked**:

- **Firmware storage page: absent.** The operator's objection is correct and important — a firmware storage
  page normally lists NVMe devices *it can initialize*. Absence there is not "not listed yet"; it is a
  failure to initialise.
- **GRUB: a device on a bus.** PCIe-level enumeration is not a working NVMe namespace. Nothing mountable has
  ever been observed. No partition, anywhere.
- **Installer: hangs at varying points, under write load.** This is the strongest single signal. Software and
  configuration faults fail *deterministically*; failures whose location varies, and which occur under I/O
  load, are the signature of **marginal hardware**.

Taken together — PCIe visible, storage unreliable — this is consistent with the original dead/marginal
subsection reading. **The operator's suspicion is better supported than the recorder's hypothesis was.**

Two discriminators, in order of decisiveness:

1. **Is there a namespace?** `nvme list`, `nvme id-ns`, or `cat /sys/block/nvme0n1/size`. A namespace with a
   sensible capacity means the controller is alive and this is a *stability* problem. No namespace means
   controller-level failure — hardware — and the RMA is the answer.
2. **What did the link negotiate?** `lspci -vv` on the device, comparing **LnkCap** against **LnkSta**. A link
   trained at x1, or at a lower PCIe generation than the slot supports, indicates a marginal slot, trace, or
   drive.

Also worth reading verbatim: `dmesg | grep -i nvme` — controller resets and command timeouts are explicit
evidence of a link or device failing, as opposed to merely being slow.

**Operational caution recorded with this reassessment:** do not build a node on storage that is only
*probably* good. An installation that appears to succeed on a marginal device is worse than one that fails
cleanly — it yields a node that corrupts silently later. Verify the pathway, or take the RMA.

## Resolution (provisional) — the drive is fine, the firmware just doesn't *show* it

`echo $grub_platform` returned **`efi`**, and that single value decides how to read everything else: in EFI
mode GRUB enumerates disks through the **firmware's** block I/O, so a device GRUB lists is a device *the
firmware already has a working block driver for*.

With that established:

- `ls` → **hd0** = the external 1 TB, two msdos partitions, formatted and readable
- `ls` → **hd1** = the 2 TB drive, listed as a block device, **unformatted** — expected, since no install
  ever completed

**Conclusion: the NVMe is enumerated by the firmware and addressable at block level.** Neither of the two
earlier readings survives intact:

- the original **"dead NVMe subsection"** is **not supported** — the device answers
- the **"firmware cannot enumerate it"** hypothesis is **also not supported** — the firmware produced the
  block device

What remains true is the observation that started the disagreement: the drive is **absent from the
firmware's setup screen**. That is now best explained as a **display/reporting gap in the setup utility** —
cosmetic, and not a barrier to booting, because a UEFI boot entry is created by an installed bootloader and
the firmware's block access is already proven.

**The unexplained residue is reliability, not visibility.** Installers hanging at varying points under
write load is not explained by anything in this entry. But that question is now narrow and testable: a full
write-and-verify pass (`f3write`/`f3read`, or `badblocks -w` on the empty device). Pass → the drive is good
and the hangs were media or platform-driver problems. Fail or wedge → the drive is marginal, and the RMA
instinct was right after all.

**Lessons recorded, because they would have saved the afternoon:**

1. **Establish the boot mode first.** `echo $grub_platform`: in `efi`, GRUB's disk list *is* the firmware's
   view; in `pc` it is GRUB's own, and therefore independent evidence. The same observation means opposite
   things depending on it.
2. **A device missing from a firmware's setup UI is not evidence the firmware cannot use it.** The setup
   utility and the runtime driver stack are different things, and they disagree more often than anyone
   expects.
3. **Both earlier theories in this entry were wrong, and the entry is more useful for showing it.** The
   order that resolved it — software-visible → firmware-visible → UI-visible — is the reusable part.

```yaml
resolution_provisional:
  date: 2026-11-07
  finding: NVMe enumerated as hd1 by the firmware's own block layer (GRUB in EFI mode); unformatted, as expected
  verdict: drive functional; "invisible in BIOS" is a setup-UI reporting gap, not an enumeration failure
  open: stability under sustained write load (installer hangs) — needs a write-and-verify pass
  confidence: high
  contradicts: both earlier root-cause readings in this note
```
