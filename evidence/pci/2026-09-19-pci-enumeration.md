# PCI enumeration — 2026-09-19

Commands run on SOVRAN-1, verbatim (no redactions needed — PCI IDs and driver
names are not sensitive per SECURITY.md).

## `lspci -nn` — VGA/3D/Display class devices only

```
06:00.0 VGA compatible controller [0300]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 22 [Radeon RX 6700/6700 XT/6750 XT / 6800M/6850M XT] [1002:73df] (rev c1)
06:00.1 Audio device [0403]: Advanced Micro Devices, Inc. [AMD/ATI] Navi 21/23 HDMI/DP Audio Controller [1002:ab28]
```

Full `lspci -nn` output (all ~110 host devices, Intel C610/X99 chipset + dual
Xeon E5/E7 v3 uncore) was reviewed. These are the **only** two functions
belonging to any VGA/3D/Display-class device on the bus.

## Finding: no second (legacy) GPU is currently enumerated

The "Legacy MSI PCIe GPU" described in `docs/HARDWARE.md` / `README.md`
(visually resembling an ATI/AMD Radeon X300 SE HyperMemory family card) does
**not** appear anywhere in `lspci -nn` output on this boot. Only the RX 6700
XT (`1002:73df`) and its companion audio function are present.

This does not confirm the card is absent from the chassis — only that it is
not currently enumerated on the PCI bus. Possible explanations (not
distinguished by this evidence): not physically installed yet, not fully
seated in its slot, dead/faulty card, or installed in a slot that isn't
receiving power/link training. Physical inspection is required to
distinguish these.

`GPU_MODEL`, `GPU_PCI_ID`, `GPU_VENDOR_ID`/`GPU_DEVICE_ID` remain UNKNOWN —
this evidence does not resolve them, it rules out "already enumerated and we
just haven't looked."

## `lspci -t` (tree, relevant branch)

```
+-04.0-[04-06]----00.0-[05-06]--+-00.0-[06-06]--+-00.0  AMD/ATI Navi 22 [1002:73df]
                                                 \-00.1  AMD/ATI Navi 21/23 Audio [1002:ab28]
```

Single PCIe switch (Navi 10 XL upstream/downstream ports at 04:00.0/05:00.0)
leads to one endpoint slot (06:00.x) occupied by the RX 6700 XT. No second
populated downstream port was observed under this switch.
