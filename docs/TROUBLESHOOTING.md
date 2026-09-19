# Troubleshooting

Failure states are documentation, not embarrassment. Record symptoms, logs,
what was tried, and the outcome. A dead end with evidence is worth more than
a workaround nobody wrote down.

## Failure-state log

| Date | Stage | Symptom | Evidence | Tried | Outcome |
|---|---|---|---|---|---|
| — | — | (no failures recorded yet) | — | — | — |

## Known pitfalls (anticipated, not yet observed)

- **GPU shares an IOMMU group with host-critical devices.** The group must be
  passed as a unit. If a needed device is in the group, options are: pass it
  too (only if safe), move the card to a different slot, or enable the ACS
  override patch (security tradeoff — document explicitly, never silently).
- **No FLR / broken reset.** Legacy cards often lack function-level reset.
  Symptom: first guest boot works, second boot hangs or shows garbage without
  a host reboot. Record as `GPU_RESET_BEHAVIOR`; workarounds (vendor-reset
  module, host reboot between launches) are documented, not assumed.
- **Guest driver refuses the card.** Some drivers detect virtualization.
  Record exact driver version and error; do not claim a fix without testing it.
- **No VGA output on CRT.** Distinguish: card not initialized (guest-side)
  vs. signal path wrong (cable/adapter) vs. CRT out of range (timings). Check
  in that order.
- **Host loses display.** Should never happen — the RX 6700 XT is untouched by
  design. If it does, the vfio-pci binding targeted the wrong PCI address.
  Stop immediately and re-verify `lspci` before continuing.
- **EDID missing on CRT.** Many CRTs expose no EDID over VGA. Fall back to
  conservative standard modes; never invent timings (see [CRT-OUTPUT.md](CRT-OUTPUT.md)).

## Rules

- Never report a fix as working unless it was re-tested from the failing state.
- Keep raw logs in `evidence/`; summarize here.
- If a stage is BLOCKED, say so in the log and in the relevant GitHub issue.
  BLOCKED is a status, not a shame.
