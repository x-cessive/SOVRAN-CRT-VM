# AGENTS.md — SOVRAN-CRT-VM

How to operate in this repository.

## Authority and mode

- **AUTHORITY:** ARCHITECT
- **MODE:** FORGE / SANDBOX unless explicitly elevated by ARCHITECT.

## Rules

1. **Read this repository's docs before acting.** `README.md` first, then the relevant `docs/` file. Do not work from memory of other projects.
2. **Do not mutate host configuration without explicit authorization.** No changes to SOVRAN-1's BIOS, kernel, bootloader, initramfs, VFIO configuration, GPU driver binding, packages, storage, VM configuration, or hardware during bootstrap phases. Observation and documentation are fine; host mutation is not.
3. **Do not promote observations into facts.** A visual guess at the GPU model is not an identification. A worker saying a step completed is not proof — only raw evidence in `evidence/` is.
4. **Preserve UNKNOWN states.** If a value is not proven by evidence, it stays UNKNOWN. Never fill in PCI IDs, IOMMU groups, timings, or statuses from assumptions.
5. **Prefer raw evidence.** Capture command output verbatim with the command and date; put it in `evidence/<category>/`. Sanitize before commit per `docs/SECURITY.md` and document what was redacted.
6. **Never expose secrets.** This repository is public. No tokens, SSH keys, passwords, private network topology, user account IDs, private hostnames beyond approved node IDs, serial numbers, unique hardware identifiers, VM secrets, or auth material.
7. **Do not modify canonical repositories from this project.** This repo is FORGE material, not SOVRAN OS doctrine.
8. **Do not claim successful passthrough without runtime proof.** PASSTHROUGH_STATUS stays UNVERIFIED until a VM demonstrably drives the CRT through the passed-through GPU.
9. **Record significant decisions and test results.** Failures and BLOCKED/UNSUPPORTED outcomes are valid — document them.
10. **Refusal is a valid outcome.** If evidence, authority, or safety is missing, stop and report rather than improvising.

## Evidence layout

- `evidence/pci/` — `lspci` output, device IDs
- `evidence/iommu/` — IOMMU groups, dmesg IOMMU lines
- `evidence/kernel/` — kernel version, cmdline, loaded modules
- `evidence/vfio/` — driver binding state, vfio device nodes
- `evidence/guest/` — VM configs (sanitized), guest PCI view

Each `evidence/` subdirectory carries a README explaining what belongs there. Directories contain no fake evidence — placeholder READMEs only, until real captures land.
