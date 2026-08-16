# Sub-GHz RF — `SUB_GHZ_RF.kicad_sch`

Multi-band sub-GHz radio subsystem: four radios, four SMA ports, switched front-ends covering
315 / 433.92 / 868 MHz.

| | |
|---|---|
| **Sheet** | 5 of 10 |
| **Components** | 231 |
| **Radios** | 3 × TI CC1101 · 1 × Silicon Labs Si4463 |
| **Bands** | 315 · 433.92 · 868 MHz |
| **RF ports** | SMA1 – SMA4 |

---

## Channels

| Port | Radio | Role | Transmit | Receive |
|---|---|---|---|---|
| SMA1 | CC1101-A | capture / replay | ✔ ~+24 dBm | ✔ |
| SMA2 | CC1101-B | jammer / primary TX | ✔ +33 dBm | — |
| SMA3 | CC1101-C | sweep receive | — | ✔ |
| SMA4 | Si4463 | secondary TX/RX | ✔ +20 dBm | ✔ |

Three CC1101s rather than one retuning radio, because the product's core use case is
simultaneous operation — capture on A while jamming on B while sweeping on C. The Si4463 adds
+20 dBm without an external amplifier and a second, independent modem family.

---

## How a channel works

Channel A, the fully-featured one:

```
                    ┌── 315 MHz SAW ──┐
CC1101-A ── balun ──┼── 433 MHz SAW ──┼── T/R switch ──┬── PA ──┬── T/R switch ── SMA1
            + match └── 868 MHz SAW ──┘                └── LNA ─┘
                        (band select)
```

Channel B replaces the receive branch with a 2 W amplifier followed by a switched three-way
low-pass filter bank. Channel C is receive-only. The Si4463 has its own matching network and
antenna switch, with no shared filtering.

The three bands span 2.75:1, which no single filter can cover — hence a switched SAW bank per
channel rather than one fixed filter.

---

## Band selection

One 2-bit code per channel selects the SAW filter. On Channel B the **same** code also selects
the matching post-amplifier low-pass filter, so the filter can never be mismatched to the band
in firmware.

| SEL0 | SEL1 | Selected |
|---|---|---|
| 0 | 0 | isolated (all paths off) |
| 1 | 0 | 315 MHz |
| 0 | 1 | 433.92 MHz |
| 1 | 1 | 868 MHz |

`(0, 0)` is also the power-on state, so the RF path comes up muted.

---

## Control

Slow, set-once lines — band select, amplifier enables, radio shutdown — run through a
**TCA9535 I²C expander at 0x20**. The transmit/receive switch line is a native MCU GPIO,
because it must flip between packets.

All amplifiers and both LNAs are held **off** at power-on, and every radio chip-select idles
high.

---

## Interfaces

| Signal | Direction | Peer sheet |
|---|---|---|
| `SPI_SCK` / `SPI_MOSI` | in | MAIN_CONTROLLER |
| `SPI_MISO` | out | MAIN_CONTROLLER |
| `nCS_CC1101_A/B/C`, `nCS_SI4463` | in | MAIN_CONTROLLER |
| `RF_SW_A` | in | MAIN_CONTROLLER |
| `I2C_SDA` / `I2C_SCL` | bidir / in | MAIN_CONTROLLER |
| `CC1101_A/B/C_GDO0` | out | RP2350_COPROCESSOR |
| `REPLAY_OUT` | in | RP2350_COPROCESSOR |
| `CHG_CE_N`, `BOOST_5V_EN` | out | POWER |

---

## Key part choices

- **SAW filters** — EPCOS/TDK, 315 / 433.92 / 868 MHz. All three share one 3.0 × 3.0 mm
  package, so a single footprint serves all nine positions.
- **Balun** — Anaren B0310J50100AHF, 300 MHz – 1 GHz. One wideband part covers all three bands
  per channel instead of three tuned baluns.
- **RF switches** — Infineon, selected per role: SP3T for band select, a fast SPDT for
  transmit/receive switching, and a 37 dBm-rated SPDT after the 2 W amplifier.
- **LNAs** — Qorvo TQP3M9036, internally matched to 50 Ω so no per-band matching is needed.
- **Amplifiers** — Mini-Circuits TSS-13LN+ on Channel A, Qorvo TQP7M9106 (2 W) on Channel B.
- **References** — 26 MHz ±10 ppm per CC1101; dedicated crystal for the Si4463.

Each radio is supplied through its own ferrite bead from the 3.3 V rail, keeping the radios
isolated from each other and from digital switching noise.

---

## Status

Schematic capture complete. Component values for the Channel B post-amplifier filter bank and
amplifier matching networks are being finalised, and will be set from RF calculation and
first-article measurement.
