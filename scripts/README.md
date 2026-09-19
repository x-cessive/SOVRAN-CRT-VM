# scripts/

Helper scripts for the VFIO/CRT lab. Empty until scripts are written **and
validated** — no untested scripts, no placeholders that pretend to work.

Intended future contents: PCI/IOMMU capture helpers, `vfio-pci` bind/unbind
wrappers, guest launch helpers. Every script documents what it changes and
requires explicit invocation — nothing here auto-mutates the host.
