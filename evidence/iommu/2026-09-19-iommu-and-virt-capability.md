# IOMMU and virtualization capability — 2026-09-19

## CPU virtualization extensions

`grep -oE "vmx|svm" /proc/cpuinfo | sort -u`

```
vmx
```

Intel VT-x is present (expected: dual Xeon E5/E7 v3). No `svm` (expected —
not an AMD CPU). This confirms CPU-level hardware virtualization support
only; it says nothing about VT-d/IOMMU.

## IOMMU groups

`ls /sys/kernel/iommu_groups/ | wc -l`

```
0
```

Zero IOMMU groups are currently exposed by the kernel. This means VFIO PCI
passthrough is **not possible in the current boot state** — not a device
problem, a platform/kernel-configuration one.

## Kernel cmdline

`cat /proc/cmdline` (PARTUUID redacted per SECURITY.md — unique hardware
identifier):

```
cryptdevice=PARTUUID=<redacted>:root root=/dev/mapper/root zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs resume=/dev/mapper/root resume_offset=1944760 initramfs_async=0 quiet splash loglevel=0 systemd.show_status=false rd.udev.log_level=0 vt.global_cursor_default=0
```

No `intel_iommu=on` (or `iommu=pt`) is present on the kernel command line.
This is the most likely proximate cause of the empty `iommu_groups`
directory, independent of whether VT-d is enabled in firmware.

## dmesg IOMMU lines (sudo, filtered to iommu only per SECURITY.md — no full dmesg dump)

```
[    0.821053] iommu: Default domain type: Translated
[    0.821053] iommu: DMA domain TLB invalidation policy: lazy mode
```

These two lines are the generic IOMMU subsystem banner and appear regardless
of whether a hardware IOMMU (VT-d) was actually detected/enabled. No DMAR
table / VT-d detection line was present in this filtered output.

## Kernel version

`uname -r`

```
7.2.3-arch1-3
```

## Loaded vfio modules

`lsmod | grep -i vfio` — no output (no vfio modules loaded, expected at this
stage).

## Conclusion for Phase 3 (IOMMU topology)

Not yet capturable. Two independent gaps must be closed first, neither of
which is a host mutation to attempt casually:

1. **BIOS/firmware**: VT-d must be confirmed enabled in the Z840's firmware
   setup screen (not yet checked — requires physical access at boot, not a
   Linux command).
2. **Kernel cmdline**: `intel_iommu=on` needs to be added regardless of (1).

Both are host-mutating or require physical reboot access, and per
`AGENTS.md` rule 2, are out of scope until ARCHITECT explicitly authorizes
moving past Phase 1/bootstrap.
