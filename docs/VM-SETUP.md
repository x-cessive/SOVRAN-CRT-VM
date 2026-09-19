# VM setup planning

No guest OS has been selected. This document records the candidates and the
criteria that will decide — selection waits on authoritative GPU identification
(Phase 1), because the exact card determines which drivers exist.

## Candidates

| Candidate | Case for | Case against |
|---|---|---|
| Windows XP | Period-correct for a mid-2000s legacy GPU; native VGA-era games/apps | Licensing; no security updates; driver availability for the specific card UNKNOWN |
| Windows 7 | Broader driver support; still retro-friendly | Licensing; heavier than XP |
| Linux | Free; easy to inspect (`lspci`, driver state) inside the guest | Legacy GPU support on modern kernels UNKNOWN — may need an older distro or proprietary legacy driver |
| Other retro OS | Niche authenticity | Driver/support UNKNOWN |

## Selection criteria

1. A real driver exists for the **verified** GPU model on that guest OS.
2. The driver initializes the card under VFIO passthrough (not just bare metal).
3. The guest can drive the VGA output at a safe, known-good mode.
4. Licensing is legitimate and documented.

## VM design notes (for later phases)

- Start minimal: prove the VM boots and is manageable **before** attaching the GPU.
- Attach the GPU only after `vfio-pci` binding is verified on the host.
- USB keyboard/mouse passthrough for the guest requires explicit authorization —
  document exactly which devices are attached.
- Disk images (`*.qcow2`) are never committed to this repository (see `.gitignore`).
- VM XML/configs may live in `configs/` only after sanitization per [SECURITY.md](SECURITY.md).
