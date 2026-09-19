# VFIO setup plan

A staged plan. **Nothing below has been executed.** Each stage gates the next:
a stage that fails stops the plan at BLOCKED or UNSUPPORTED — that is a valid,
documented outcome, not a failure of process.

## Stages

1. **Verify CPU/platform virtualization support.** Check `vmx`/`svm` flags on
   the Z840's Xeons. If absent: UNSUPPORTED, stop.
2. **Verify IOMMU enabled in firmware.** Inspect BIOS settings; record state
   with evidence. If the platform cannot enable IOMMU: BLOCKED/UNSUPPORTED, stop.
3. **Verify Linux exposes IOMMU.** `dmesg` IOMMU lines, non-empty
   `/sys/kernel/iommu_groups/`. If the kernel sees no IOMMU: BLOCKED, diagnose.
4. **Enumerate PCI devices.** Full `lspci -nn` capture into `evidence/pci/`.
5. **Identify legacy GPU vendor/device IDs.** Authoritative `lspci` reading.
   This resolves `GPU_MODEL` from UNKNOWN to a verified value (or records that
   the device is unidentifiable).
6. **Inspect IOMMU grouping.** Map the GPU and all its functions to IOMMU
   group(s); list every co-member device.
7. **Determine whether all required functions can be safely passed together.**
   VGA + audio + secondary functions: if the group contains host-critical
   devices that cannot be detached, the design is BLOCKED pending the ACS
   override decision (which has security tradeoffs — document, do not silently apply).
8. **Confirm the host does not require the legacy GPU.** Host must boot, run
   its desktop, and operate normally with the card bound to `vfio-pci` and
   invisible as a display device.
9. **Reserve the GPU for `vfio-pci`.** Design the binding mechanism (driver
   override / modprobe config / initramfs). Design only in this phase.
10. **Verify driver binding.** After binding, confirm `lspci -nnk` shows
    `vfio-pci` on all target functions and the host display stack is untouched.
11. **Create test VM.** Minimal guest config, no GPU attached yet — prove the
    VM boots and is manageable.
12. **Attach physical GPU.** Add the PCI hostdev device(s) to the guest config.
13. **Attach USB/input devices if authorized.** Keyboard/mouse for the guest —
    only with explicit authorization; document which devices.
14. **Test physical VGA output to CRT.** Boot guest, confirm the CRT shows
    guest output driven by the passed-through card. Photo/record as evidence.
15. **Test guest reboot.** Warm reboot inside the guest; confirm the GPU and
    CRT recover without host intervention.
16. **Test guest shutdown.** Clean shutdown; confirm host stability and that
    the card is released back to `vfio-pci` cleanly.
17. **Test second VM launch without host reboot.** Start the guest again;
    this is where broken reset behavior shows up.
18. **Record reset behavior.** `GPU_RESET_BEHAVIOR` moves from UNKNOWN to a
    documented value (clean / requires host reboot / requires workarounds).
19. **Record failure modes.** Every failure gets symptoms, logs, and what was
    tried. See [TROUBLESHOOTING.md](TROUBLESHOOTING.md).
20. **Only then classify the configuration.** See final states below.

## Possible final states

| State | Meaning |
|---|---|
| PASS | All stages completed with evidence; CRT shows guest output; reset/reboot behavior clean |
| PASS_WITH_WARNINGS | Works, with documented caveats (e.g. needs host reboot between launches) |
| BLOCKED | A solvable obstacle stops progress (e.g. firmware setting, missing package) — recorded with the unblock path |
| UNSUPPORTED | Hardware/firmware cannot do this (no IOMMU, unpassable grouping) — terminal |
| UNKNOWN | Not yet tested — the default state for everything until evidence exists |

## What this plan does NOT authorize

- Changing SOVRAN-1's kernel cmdline, initramfs, modprobe config, or BIOS.
- Installing QEMU/KVM/libvirt on SOVRAN-1.
- Binding or unbinding any driver on the live host.
- Touching the RX 6700 XT's driver or configuration in any way.

Those are Phase 5+ actions requiring a separate explicit authorization from
ARCHITECT. This document is the plan, not the permission.
