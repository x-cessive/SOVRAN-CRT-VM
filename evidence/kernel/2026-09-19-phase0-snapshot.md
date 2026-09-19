# Phase 0 pre-mutation snapshot — 2026-09-19

Read-only capture. No host configuration was changed to produce this file.

## Distribution

```
NAME="Omarchy"
PRETTY_NAME="Omarchy"
ID=omarchy
ID_LIKE=arch
BUILD_ID="4.0.4"
```

Package manager: `pacman` (Arch-family).

## Kernel

```
7.2.3-arch1-3
```

## Bootloader

`bootctl status` (as non-root; full loader-entry read needs root and wasn't
escalated for this snapshot):

```
Firmware: UEFI 2.31 (Hewlett-Packard BIOS ID: 2.61)
Firmware Arch: x64
Secure Boot: disabled
TPM2 Support: yes
Measured UKI: yes
Measured OS: yes
Current Stub: systemd-stub 261.2-1-arch
```

This is **systemd-boot with Unified Kernel Images (UKI)**, not GRUB and not
a directly hand-edited Limine config. Kernel cmdline is sourced from
`/etc/kernel/cmdline` and baked into the UKI by `mkinitcpio` at build time —
confirmed present:

```
cryptdevice=PARTUUID=<redacted>:root root=/dev/mapper/root zswap.enabled=0 rootflags=subvol=@ rw rootfstype=btrfs
```

(PARTUUID redacted per SECURITY.md.) This is the exact file that would need
`intel_iommu=on` appended, followed by `mkinitcpio -P` and a reboot, to
change the running kernel cmdline (do not edit `/proc/cmdline` directly —
it's read-only/derived).

## CPU

```
Model name: Intel(R) Xeon(R) CPU E5-2690 v3 @ 2.60GHz
Sockets: 2
CPU(s): 48 (24 cores x2 sockets, HT on)
```

Confirms dual-Xeon HP Z840. `vmx` and `ept` flags present (VT-x + Extended
Page Tables). E5-2690 v3 supports VT-d per Intel's published spec, but that
says nothing about firmware enablement — see the existing
`evidence/iommu/2026-09-19-iommu-and-virt-capability.md` finding that
`/sys/kernel/iommu_groups/` is currently empty.

## Loaded GPU-relevant modules

```
kvm_intel, kvm, irqbypass   <- KVM stack loaded and active
amdgpu (+ its usual DRM/TTM dependency chain)  <- bound to the RX 6700 XT
```

No `radeon`, `nouveau`, or `vfio*` modules are loaded. Consistent with: no
second GPU present to claim, and no VFIO reservation attempted yet.

## Initramfs tooling

`mkinitcpio` (not `dracut`). Config at `/etc/mkinitcpio.conf`.
`/etc/mkinitcpio.d/` preset listing was empty in this snapshot (needs a
root-level check to enumerate installed kernel presets accurately — not
escalated for this read-only pass).

## Already-installed virtualization stack

Contrary to the assumption that Phase 4 (install virtualization stack) is
unstarted, these packages are **already installed**:

```
qemu-base   11.1.1-1
libvirt     1:12.7.0-1
virt-manager 5.1.0-4
edk2-ovmf   202608-1
dnsmasq     2.93-1
swtpm       0.10.1-2
```

`libvirtd.service` is `enabled` and `active`. User `architect` is already in
the `libvirt` (and `docker`) groups. `qemu-system-x86_64` reports version
`11.1.1`. No dedicated `vfio` pacman package exists on Arch — `vfio-pci`,
`vfio_iommu_type1` etc. are kernel modules built into the stock `linux`
package, loaded via `modprobe`/module config, not a separate install.

**Revised Phase 4 status: effectively satisfied already.** The remaining
gap to reach a usable passthrough-capable host is IOMMU enablement (Phase
5), not package installation.

## Known pacman database inconsistency (pre-existing, unrelated)

`pacman -Q`/`-S` operations print:
```
error: could not open file /var/lib/pacman/local/qemu-system-mips-11.1.1-1/desc: No such file or directory
```
on every invocation. This is a stale local-db entry unrelated to this
project (predates this session). It does not block package queries or
installs, only adds a harmless warning line. Not fixed here since it's out
of scope and not a host mutation this project requested.

## Repository state

`SOVRAN-CRT-VM` is cloned at `~/Projects/SOVRAN-CRT-VM`, `origin` =
`https://github.com/x-cessive/SOVRAN-CRT-VM.git`, `main` branch, evidence
commit `21f6a44` pushed.
