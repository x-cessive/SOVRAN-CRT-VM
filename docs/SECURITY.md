# Security and publication policy

This repository is **public**. Everything committed here is visible to the
world. The rules below are hard boundaries, not suggestions.

## Never commit

- Access tokens, API keys, or any credential material
- SSH private keys (`*.pem`, `*.key`, `id_rsa*`, `id_ed25519*`)
- Passwords or password hashes
- Private IP topology (internal addresses, subnets, routing) unless explicitly approved
- User account identifiers (usernames, UIDs, email addresses)
- Private hostnames beyond approved role/node IDs (e.g. `SOVRAN-1` is fine; internal FQDNs are not)
- Hardware serial numbers, unless needed for identification **and** explicitly approved
- Unique hardware identifiers that create unnecessary privacy exposure (MAC addresses, UUIDs — redact or omit)
- VM secrets (guest passwords, product keys, unattended-install answer files with credentials)
- Authentication material of any kind
- Unrelated system logs (full `dmesg`, `journalctl` dumps — extract only the relevant lines)

## Evidence sanitization

Raw captures from SOVRAN-1 will contain identifiers. Before anything lands in
`evidence/`:

1. Capture raw to a local file **outside** the repo (it is gitignored via
   `evidence/**/raw-unsanitized/` if you must stage it temporarily — prefer
   keeping it out of the tree entirely).
2. Redact: serials, MACs, UUIDs, usernames, hostnames beyond node IDs, private IPs.
3. Commit the sanitized version with a header noting:
   - the exact command run and the date,
   - what was redacted and why.

Do not sanitize so aggressively that the evidence loses technical value —
PCI IDs, IOMMU group numbers, driver names, and kernel messages about the
target devices are the point and stay.

## Redaction log

| Date | File | Redacted | Reason |
|---|---|---|---|
| — | — | (nothing redacted yet) | — |

## Reporting

If you discover committed secrets or identifiers: remove them, rewrite the
affected history if the exposure is recent and the repo is young (it is —
prefer a clean history over preserving a mistake), and record the incident
here. Do not commit a "fix" that leaves the secret in history.
