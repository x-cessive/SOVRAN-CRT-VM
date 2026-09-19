# evidence/

Raw captured evidence, organized by category. Placeholder READMEs only —
no fake evidence files. Directories stay empty until real captures land.

- `pci/` — `lspci` output: vendor/device/subsystem IDs, bound drivers, all GPU functions
- `iommu/` — IOMMU group maps, `dmesg` IOMMU lines
- `kernel/` — kernel version, cmdline, loaded modules
- `vfio/` — `vfio-pci` binding state, vfio device nodes
- `guest/` — sanitized VM configs, guest-side PCI view

Rules: capture raw output verbatim with the exact command and date; sanitize
before commit per `../docs/SECURITY.md`; document every redaction in the file
header. Never fabricate a capture.
