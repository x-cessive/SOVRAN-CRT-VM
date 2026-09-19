# CRT output — safety first

## Current state

| Field | Value |
|---|---|
| CRT_MODEL | UNKNOWN |
| MAX_HORIZONTAL_SCAN | UNKNOWN |
| MAX_VERTICAL_REFRESH | UNKNOWN |
| SUPPORTED_RESOLUTIONS | UNKNOWN |

Until the exact monitor is identified, **no timings are assumed and no
modelines are recommended**. An incorrect modeline can drive a CRT outside its
rated scan range. Modern monitors usually protect themselves; vintage ones do
not always.

## Philosophy

- Use **conservative, detected, or EDID-provided modes** only.
- Never invent a modeline. Never recommend overclocked scan rates or
  unsupported refresh rates without monitor-specific evidence.
- If the CRT's specifications cannot be found, stay at safe defaults
  (e.g. 640x480@60Hz, 800x600@60Hz — VESA standard timings the overwhelming
  majority of VGA CRTs support) and document that the ceiling is unknown.
- Record the monitor's identity (make/model from the rear label, serial
  redacted) as soon as it is known, then look up its real specifications.

## Signal paths, in order of preference

1. **Native VGA output (preferred).** The legacy GPU's VGA port drives its own
   RAMDAC directly to the CRT over an analog cable. Shortest path, no
   conversion, timings controlled by the GPU driver. This is the design target.
2. **Passive DVI-I to VGA.** Only valid if the card's DVI port is DVI-I
   (carries analog pins). A passive adapter just re-routes the existing analog
   signals — electrically equivalent to native VGA. Verify the port is DVI-I,
   not DVI-D, before assuming this works.
3. **Active HDMI/DisplayPort to VGA conversion.** A last resort. An active
   adapter re-clocks the signal with its own DAC; quality and timing behavior
   vary by adapter. Document the exact adapter if this path is ever used.

Do not substitute path 2 or 3 silently and call it native VGA. They are
different signal paths with different failure modes.

## Evidence to collect (once authorized)

- CRT make/model (rear label; serial redacted).
- EDID block if exposed over VGA (`edid-decode`), with a note that many CRTs
  expose minimal or no EDID.
- Whatever mode the guest successfully displays, recorded with resolution and
  refresh actually observed — not assumed.
