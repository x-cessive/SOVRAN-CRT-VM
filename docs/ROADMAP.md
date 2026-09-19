# Roadmap

Phases are sequential and gated: a phase is not marked complete until its
evidence exists. Future phases stay "not started" — no optimistic checkmarks.

| Phase | Goal | Entry criteria | Done when | Status |
|---|---|---|---|---|
| 0 | Repository bootstrap | ARCHITECT authorization | This repo exists, public, documented, issues filed | in progress |
| 1 | Legacy GPU authoritative identification | SOVRAN-1 accessible, `lspci` authorized | Vendor/device/subsystem IDs recorded in `evidence/pci/`; `GPU_MODEL` resolved or recorded unidentifiable | not started |
| 2 | SOVRAN-1 virtualization capability audit | Phase 1 done | CPU virt flags, IOMMU firmware state, kernel IOMMU exposure recorded | not started |
| 3 | IOMMU topology capture | Phase 2 done | Full IOMMU group map in `evidence/iommu/`; GPU group membership known | not started |
| 4 | VFIO reservation design | Phase 3 done | Binding mechanism designed on paper; risk review of group co-members | not started |
| 5 | Non-destructive host configuration | Explicit ARCHITECT authorization | `vfio-pci` bound to target functions; host display verified unaffected | not started |
| 6 | QEMU/KVM guest creation | Phase 5 done | Minimal guest boots without GPU attached | not started |
| 7 | Physical GPU passthrough | Phase 6 done | GPU attached via VFIO; guest driver initializes the card | not started |
| 8 | CRT output validation | Phase 7 done | CRT displays guest output over native VGA; mode recorded | not started |
| 9 | Reset/reboot testing | Phase 8 done | Guest reboot, shutdown, and second-launch-without-host-reboot tested; `GPU_RESET_BEHAVIOR` documented | not started |
| 10 | Reproducible automation | Phase 9 PASS or PASS_WITH_WARNINGS | Scripts/configs in `scripts/` and `configs/` reproduce the setup from a clean state | not started |
| 11 | Documentation and acceptance | Phase 10 done | All docs reflect reality; final classification (PASS / PASS_WITH_WARNINGS / BLOCKED / UNSUPPORTED) recorded | not started |

## Notes

- Phase 5 is the first phase that mutates the host. It requires its own
  explicit authorization — the roadmap is not permission.
- The project may terminate at any phase with BLOCKED or UNSUPPORTED. That is
  a complete, honest outcome; record it and stop.
- Guest OS selection happens between Phase 1 and Phase 6, gated on the
  verified GPU model (driver availability decides).
