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
