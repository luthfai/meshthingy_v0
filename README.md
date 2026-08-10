# meshthingy v0

A Meshtastic-compatible LoRa mesh node built around an ESP32-S3 and an Ai-Thinker
HT-RA62 (RA-01SH) LoRa module, with GPS, OLED, environmental and motion sensing,
and USB-C Li-Ion charging.

KiCad project name: `meshtastic-v0` — open [meshtastic-v0.kicad_pro](meshtastic-v0.kicad_pro).

## Status

Schematic and layout are complete and routed. **Not yet released for fabrication** —
see [Known issues](#known-issues) below.

| Check | Result |
| --- | --- |
| DRC (`--severity-all`) | 0 violations, 0 unconnected pads |
| Schematic parity | 3 footprint mismatches |
| ERC (`--severity-all`) | 8 errors, 15 warnings |

## Board

| | |
| --- | --- |
| Size | 71.0 × 46.1 mm |
| Layers | 2 (F.Cu / B.Cu), 1.6 mm |
| Components | 87 footprints, 341 pads, 370 vias |
| Copper pours | GND on both layers, plus 3 antenna keepout regions |
| Netclasses | `Default` only — 0.175 mm clearance, 0.2 mm track, 0.6/0.3 mm via |

## Design blocks

Hierarchical schematic, root sheet [meshtastic-v0.kicad_sch](meshtastic-v0.kicad_sch):

| Sheet | Contents |
| --- | --- |
| [Power Management](Power%20Management.kicad_sch) | BQ24072 Li-Ion charger, TPS62046 buck, load switching |
| [Microcontroller](Microcontroller.kicad_sch) | ESP32-S3-WROOM-1 |
| [LORA](LORA.kicad_sch) | HT-RA62 (RA-01SH) module, U.FL + SMA edge-mount antenna |
| [GPS](GPS.kicad_sch) | ATGM336H receiver, XH414H backup cell |
| [Display](Display.kicad_sch) | DM-OLED096-636 0.96" OLED |
| [Air Sensors](Air%20Sensors.kicad_sch) | BME280 temperature / humidity / pressure |
| [Accelerometer](Accelerometer.kicad_sch) | LIS3DSHTR |
| [Peripherals](Peripherals.kicad_sch) | AT24C32A EEPROM, buzzer, LED, buttons |
| [Connectors](Connectors.kicad_sch) | USB-C receptacle, JST-PH battery |

Project-local symbols and footprints live in [libs/](libs/) and are registered in
[sym-lib-table](sym-lib-table) / [fp-lib-table](fp-lib-table) via `${KIPRJMOD}`,
so the project is self-contained.

## Known issues

Run the checks yourself with the commands below before trusting this list.

**Blocking fabrication**

- `R22` is **1k** in the schematic but **10k** on the PCB — the board was not
  fully re-synced from the schematic. Resolve before ordering.
- 8 ERC errors, mostly power-input pins with no driving source (missing
  `PWR_FLAG` on `#PWR02`, `#PWR020`, `U6` pin 13, `U1` pins 1–2) plus two
  dangling hierarchical sheet pins (`VSYS`, `3V3`) on the root sheet and an
  unconnected `U7` pin 24 (IO47).

**Worth reviewing**

- Several DRC checks are set to *ignore* in the project settings, including
  silkscreen clearance/overlap, missing courtyard, and text height/thickness.
  The clean DRC result does not cover these.
- Per-rail labels (`GPS_VDD`, `LORA_VDD`, `ACCEL_VDD`, `BME_VDD`, `3V3`) are all
  electrically merged into `+3V3`. Fine if the names are decorative; a bug if
  separate filtered rails were intended.
- `libs/RA-01SH/` is not registered in either library table — the `U4` symbol
  works only from the schematic's embedded cache.
- BOM rows carry no MPN or manufacturer, so the BOM is not yet orderable.

## Checks and outputs

```bash
kicad-cli sch erc  --severity-all --output erc.rpt meshtastic-v0.kicad_sch
kicad-cli pcb drc  --severity-all --schematic-parity --output drc.rpt meshtastic-v0.kicad_pcb

kicad-cli pcb export gerbers --output fab/ meshtastic-v0.kicad_pcb
kicad-cli pcb export drill   --output fab/ meshtastic-v0.kicad_pcb
kicad-cli sch export bom     --output bom.csv meshtastic-v0.kicad_sch
```

Generated outputs are gitignored — regenerate them, or attach them to a release tag.

## Repository notes

`.history/` is KiCad 10's Local History: a **separate nested git repository**
(~77 MB, thousands of autosave commits). It is gitignored and must stay that way —
committing it would create a broken gitlink. Delete it freely if you don't need
the local undo history.
