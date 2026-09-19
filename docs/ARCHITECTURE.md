# Architecture

## Topology (target)

```
SOVRAN-1 (HP Z840, Linux/Omarchy)
    |
    +-- AMD RX 6700 XT
    |       +-- Host desktop session
    |       +-- Modern displays (DP/HDMI)
    |       +-- Gaming, GPU compute
    |       +-- NEVER handed to a guest
    |
    +-- Legacy MSI PCIe GPU (UNKNOWN model)
            +-- Bound to vfio-pci on the host (reserved)
            +-- Passed directly to the QEMU/KVM guest
                    |
                    +-- Guest sees it as physical hardware
                    |
                    +-- Native VGA output
                            |
                            +-- Physical CRT monitor
```

## Components

| Component | Role | Status |
|---|---|---|
| Linux host (SOVRAN-1, Omarchy direction) | KVM hypervisor, VFIO driver binding | OS install in progress (separate effort); VFIO not configured |
| AMD RX 6700 XT | Host primary GPU | Owned by host; never passed through |
| Legacy MSI PCIe GPU | Passthrough candidate | Model UNKNOWN; visually: passive heatsink, VGA + DVI + S-Video-style TV out, no aux power |
| QEMU/KVM + libvirt | Virtualization stack | Not yet installed/configured for this project |
| CRT monitor | Physical analog display | Model UNKNOWN |
| Guest OS | Retro workload | Not selected; candidates below |

## Why this topology

The host keeps the modern GPU and everything it already does. The legacy card is
electrically separate: no shared display duty, no host dependency. If passthrough
fails, the host is unaffected. If the guest dies, the host is unaffected. The only
coupling is the PCI bus and the IOMMU.

## Guest options (none selected)

| Candidate | Why it fits | Why it may not |
|---|---|---|
| Windows XP | Period-correct for legacy GPU drivers; native VGA-era software | Licensing; no security updates; driver availability for specific card UNKNOWN |
| Windows 7 | Wider driver support; still retro-friendly | Licensing; heavier than XP |
| Linux | Free; `lspci`/driver behavior easy to inspect inside guest | Legacy GPU driver situation on modern kernels UNKNOWN |
| Other retro OS | Niche authenticity | Driver/support UNKNOWN |

Guest selection waits on authoritative GPU identification (Phase 1) — the exact
model determines which drivers exist, which decides which guests are viable.
Do not select a permanent guest OS until then.

## Data flow (once built)

1. Host boots; `vfio-pci` claims the legacy GPU at bind time. The host never
   initializes it as a display device.
2. libvirt/QEMU starts the guest with the GPU's PCI address(es) attached via
   VFIO (`hostdev`).
3. Guest driver initializes the card as if it were bare metal.
4. Card's VGA DAC drives the CRT over the analog cable.

## Non-goals

- Replacing the RX 6700 XT as host GPU.
- Using the legacy GPU for host compute or host display, ever.
- Looking-glass / virtual display tricks — the point is a real CRT on a real cable.
- Audio passthrough design (out of scope until video path is proven).
