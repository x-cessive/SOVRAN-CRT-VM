# SOVRAN-CRT-VM

**PROJECT_STATUS=FORGE — PASSTHROUGH_STATUS=UNVERIFIED**

A reproducible Linux KVM/QEMU/VFIO lab for passing a **legacy PCIe GPU** directly to a virtual machine on **SOVRAN-1** (HP Z840 workstation, Linux/Omarchy direction), so the VM drives a **physical CRT monitor** through the legacy GPU's **native VGA output**.

Nothing here claims working passthrough. This repository exists to document the plan, stage the evidence, and — only after runtime proof — provide a reproducible setup.

## The idea

```
Linux Host (SOVRAN-1)
    |
    +-- AMD RX 6700 XT ---------> Host desktop, modern displays, gaming, GPU workloads
    |
    +-- Legacy MSI PCIe GPU ----> VFIO PCI passthrough ----> VM
                                                            |
                                                      native VGA
                                                            |
                                                         CRT monitor
```

- **Why a second GPU:** modern GPUs (like the RX 6700 XT) dropped analog VGA output long ago. A legacy card with a native VGA port is the only honest way to drive a real CRT.
- **Why the RX 6700 XT stays with the host:** it is the primary GPU. The legacy card will not replace it, will not be used for host display, and the host must never depend on it. The legacy card's only job is passthrough.
- **Why native VGA matters for CRT hardware:** a CRT is an analog display. The GPU's native VGA DAC produces a true analog signal. Active digital-to-analog converters (HDMI/DP → VGA) can work, but they add a conversion stage with its own timing and quality quirks. Native VGA is the shortest, most faithful signal path. See [docs/CRT-OUTPUT.md](docs/CRT-OUTPUT.md).
- **What VFIO PCI passthrough is:** the Linux kernel binds the legacy GPU to the `vfio-pci` driver instead of a normal display driver, which hands the physical PCI device to a QEMU/KVM guest. The VM sees the card as real hardware — including its VGA output.

## Current status

| Field | Value |
|---|---|
| PROJECT_STATUS | FORGE (experimental, not doctrine) |
| PASSTHROUGH_STATUS | UNVERIFIED (no passthrough attempted or proven) |
| GPU_MODEL | UNKNOWN (visually resembles ATI/AMD Radeon X300 SE HyperMemory family — **not confirmed**) |
| GPU_PCI_ID | UNKNOWN |
| GPU_VENDOR_ID / GPU_DEVICE_ID | UNKNOWN |
| GPU_RESET_BEHAVIOR | UNKNOWN |
| IOMMU_GROUP | UNKNOWN |
| PASSTHROUGH_COMPATIBILITY | UNKNOWN |
| CRT_MODEL | UNKNOWN |
| CRT supported timings | UNKNOWN |

### Known facts (evidence-backed)

- Host node: SOVRAN-1, HP Z840 workstation, dual Xeon, AMD RX 6700 XT, 64 GB-class memory.
- Host OS direction: Linux / Omarchy. Windows preservation is **not** required on this node.
- Legacy GPU observations (visual only, **not** authoritative): MSI branding; PCIe interface; passive heatsink (no fan); native VGA + DVI + round TV/S-Video-style output; no auxiliary PCIe power connector → low-power card.
- **2026-09-19, `lspci -nn`: the legacy GPU is not currently enumerated on the PCI bus.** Only the RX 6700 XT (`1002:73df`) and its audio function are present. See [evidence/pci/2026-09-19-pci-enumeration.md](evidence/pci/2026-09-19-pci-enumeration.md). Does not confirm the card's physical absence — only that it isn't link-trained/visible to the OS right now.
- **2026-09-19: CPU virtualization (Intel VT-x) present; `/sys/kernel/iommu_groups/` is empty and the kernel cmdline lacks `intel_iommu=on`.** IOMMU/VFIO is not usable in the current boot state. See [evidence/iommu/2026-09-19-iommu-and-virt-capability.md](evidence/iommu/2026-09-19-iommu-and-virt-capability.md).
- The RX 6700 XT remains the host's primary GPU and is never handed to a guest.
- LICENSE_STATUS=UNKNOWN — no license selected yet; do not assume one.

### Explicit UNKNOWNs (not facts, not guesses)

- Exact GPU model, vendor/device/subsystem PCI IDs, bound kernel driver, available kernel modules.
- PCIe slot placement, IOMMU group membership, associated GPU functions (VGA/audio/secondary).
- Whether IOMMU/VT-d is enabled in firmware; whether the Linux kernel exposes IOMMU.
- GPU reset behavior after guest shutdown; whether a second guest launch works without host reboot.
- CRT model, maximum horizontal scan rate, maximum vertical refresh, supported resolutions.
- Which guest OS will be used (candidates: Windows XP, Windows 7, Linux, other retro OSes — none selected).

## Safety and governance boundaries

- **This is FORGE material, not SOVRAN OS doctrine.** It does not modify canonical SOVRAN OS artifacts.
- **Do not change the SOVRAN-1 host** (BIOS, kernel, bootloader, initramfs, VFIO config, GPU binding, packages, storage, VM config, hardware) during the bootstrap phases. Host work needs a separate, explicit authorization.
- **Never fabricate detection results.** A worker saying something is complete does not make it true — only raw evidence does.
- **No secrets in this repository** (it is public). See [docs/SECURITY.md](docs/SECURITY.md).
- Failure, refusal, and "not supported" are valid outcomes. This project may end at BLOCKED or UNSUPPORTED, and that is a legitimate result.

## Phased roadmap

| Phase | Goal | Status |
|---|---|---|
| 0 | Repository bootstrap (this repo) | in progress |
| 1 | Legacy GPU authoritative identification (`lspci`) | not started |
| 2 | SOVRAN-1 virtualization capability audit | not started |
| 3 | IOMMU topology capture | not started |
| 4 | VFIO reservation design | not started |
| 5 | Non-destructive host configuration | not started |
| 6 | QEMU/KVM guest creation | not started |
| 7 | Physical GPU passthrough | not started |
| 8 | CRT output validation | not started |
| 9 | Reset/reboot testing | not started |
| 10 | Reproducible automation | not started |
| 11 | Documentation and acceptance | not started |

Future phases are **not** marked complete until evidence proves them. Full roadmap: [docs/ROADMAP.md](docs/ROADMAP.md).

## Documentation

| Document | Contents |
|---|---|
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Topology, components, guest options |
| [docs/HARDWARE.md](docs/HARDWARE.md) | Evidence plan for authoritative GPU/CRT identification |
| [docs/VFIO-SETUP.md](docs/VFIO-SETUP.md) | Staged 20-step passthrough plan (nothing executed yet) |
| [docs/CRT-OUTPUT.md](docs/CRT-OUTPUT.md) | CRT safety, native VGA vs conversion paths |
| [docs/VM-SETUP.md](docs/VM-SETUP.md) | Guest planning and selection criteria |
| [docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) | Failure states, known pitfalls |
| [docs/SECURITY.md](docs/SECURITY.md) | Public-repo publication boundaries |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Full phased roadmap |
| [evidence/](evidence/) | Raw captured evidence (empty until collected) |
| [scripts/](scripts/) | Helper scripts (empty until validated) |
| [configs/](configs/) | Config examples (empty until validated) |
| [diagrams/](diagrams/) | Diagrams (empty until produced) |

## Who this is for

Humans and AI agents doing retro-computing, Linux users, homelab builders. You need a host with IOMMU, a spare GPU with native VGA, and a CRT — plus patience. Procedures here use exact commands only where validated; unvalidated steps are plans, not instructions.
