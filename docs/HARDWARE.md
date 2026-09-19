# Hardware evidence plan

Nothing in this document is asserted about the installed hardware. It defines
**what evidence must be collected** (on SOVRAN-1, once authorized) before any
VFIO design decisions are made.

## Required future evidence

### PCI enumeration (legacy GPU)

- [ ] Full `lspci -nn` line(s) for the card — **vendor ID : device ID** (e.g. `1002:5b60` — example format only, not a claim)
- [ ] `lspci -nnk` — kernel driver currently bound, kernel modules available
- [ ] `lspci -vv` — PCIe link width/speed, capabilities
- [ ] Subsystem vendor/device IDs (`lspci -nn` shows them as `SVendor:SDevice`)
- [ ] **All functions of the device**, not just function 0 — VGA, audio, and any secondary functions must be enumerated separately (VFIO passthrough must include every function in the device's IOMMU-relevant set, or explicitly justify excluding one)
- [ ] PCIe slot / physical location (`lspci -t`, `lspci -vv` "Physical Slot"), and which slot was used if moved

### IOMMU

- [ ] IOMMU group number for the GPU and every function (`/sys/kernel/iommu_groups/`)
- [ ] Complete membership of that IOMMU group — every device in the group must be accounted for
- [ ] `dmesg` IOMMU initialization lines (Intel VT-d or AMD-Vi)
- [ ] Whether the GPU shares its group with devices the host needs (deal-breaker analysis input)

### Platform / firmware

- [ ] CPU virtualization capability (`lscpu`, `/proc/cpuinfo` flags: `vmx`/`svm`)
- [ ] IOMMU enabled in firmware (BIOS setting state, recorded from the setup screen — photo or written note)
- [ ] Linux exposes IOMMU (`dmesg | grep -i iommu`, `/sys/kernel/iommu_groups` non-empty)
- [ ] Kernel version, kernel cmdline (`/proc/cmdline` — redact anything sensitive)

### GPU behavior

- [ ] ROM / option-ROM behavior if relevant (does the card expose a ROM BAR; is shadowing needed)
- [ ] Reset behavior after guest shutdown: does the card reinitialize cleanly for a second guest launch without host reboot? (tested in Phase 9, recorded as PASS/FAIL with symptoms)
- [ ] Any function-level reset (FLR) support reported by `lspci -vv`

### CRT

- [ ] CRT model (rear label photo/transcription — serial number redacted)
- [ ] EDID data if the monitor exposes any over VGA (`edid-decode` output)
- [ ] Supported timings **only** where safely discoverable — never invent modelines (see [CRT-OUTPUT.md](CRT-OUTPUT.md))

## Current state (visual observations only — not authoritative)

| Observation | Status |
|---|---|
| MSI branding visible | unconfirmed (visual) |
| PCIe interface | unconfirmed (visual) |
| Passive heatsink, no fan | unconfirmed (visual) |
| Native VGA + DVI + TV/S-Video-style output | unconfirmed (visual) |
| No auxiliary PCIe power connector | unconfirmed (visual) |
| Resembles Radeon X300 SE HyperMemory family | **hypothesis, not identification** |

## Rules

- Capture raw command output verbatim into `evidence/` with the exact command and date.
- Do not convert the X300 SE resemblance into a fact. `GPU_MODEL` stays UNKNOWN until `lspci` says otherwise.
- Sanitize before commit (see [SECURITY.md](SECURITY.md)); document any redaction in the evidence file header.
