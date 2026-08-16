# Real-Time Co-Processor — `RP2350_COPROCESSOR.kicad_sch`

Raspberry Pi RP2350A and its complete support circuit. Handles every timing-critical task in
the product.

| | |
|---|---|
| **Sheet** | 4 of 10 |
| **Components** | 31 |
| **MCU** | Raspberry Pi RP2350A, QFN-60 |
| **Program flash** | 16 MB QSPI (W25Q128) |
| **Reference** | 12 MHz crystal |

---

## Why there is a second processor

The ESP32-S3 runs the application: display, menus, file system, and the shared SPI and I²C
buses. Its timing is good, but not exact.

Sub-GHz capture and replay is a timing problem rather than a data problem. A key fob transmits
an on/off keyed waveform in which the **width of each pulse** carries the information — replay
it with a few microseconds of jitter and the receiver rejects it. Interrupt latency on a
general-purpose MCU is enough to ruin that.

The RP2350's **PIO blocks** are deterministic state machines that run independently of the
CPU, one instruction per clock, with no interrupt latency. Each radio's demodulated bitstream
goes to a PIO state machine that timestamps every edge; another plays edges back out with the
same precision.

In short: the ESP32 owns the buses and the user interface, the RP2350 owns the microseconds.

---

## What it handles

| Function | Why it lives here |
|---|---|
| Radio edge capture (3 channels) | PIO timestamping — the reason for the chip |
| Replay output | PIO edge playback |
| CAN bus | software CAN implemented in PIO |
| NFC | dedicated SPI port, keeps NFC off the shared bus |
| LF 125 kHz coils | PWM coil drive plus demodulator capture |
| USB-A host | native USB controller |
| Link to ESP32 | UART plus a hardware sync line |

---

## Support circuit

- **Program flash** — external 16 MB QSPI device; the RP2350 has no internal flash.
- **Reference** — 12 MHz crystal with load capacitors and a series drive-limiting resistor,
  per Raspberry Pi's hardware design guidance.
- **Core supply** — the RP2350's on-chip regulator generates the 1.1 V core rail from 3.3 V
  through an external inductor, with its analog reference separately filtered.
- **Reset and boot** — RUN (reset) and BOOTSEL push-buttons, arranged so that entering boot
  mode does not disturb the flash chip-select.
- **Debug** — 5-pin header carrying SWCLK, SWDIO, RUN, 3.3 V and ground.
- **USB** — series-terminated data pair to the USB-A host receptacle.

---

## Interfaces

| Signal | Direction | Peer sheet |
|---|---|---|
| `LINK_ESP_TX` | in | MAIN_CONTROLLER |
| `LINK_ESP_RX` | out | MAIN_CONTROLLER |
| `LINK_SYNC` | bidirectional | MAIN_CONTROLLER |
| `CC1101_A/B/C_GDO0` | in | SUB_GHZ_RF |
| `REPLAY_OUT` | out | SUB_GHZ_RF |
| `USB_HOST_VBUS_EN` | out | POWER |
| `USB_HOST_FAULT_N` | in | POWER |
| `USB_HOST_D_P` / `USB_HOST_D_N` | bidirectional | WIRED_IO |
| `CAN_TX` / `CAN_RX` | out / in | WIRED_IO |
| `NFC_SCK` / `NFC_MOSI` / `nCS_NFC` | out | NEAR_FIELD |
| `NFC_MISO` / `NFC_IRQ` | in | NEAR_FIELD |
| `LF_COIL_A` / `LF_COIL_B` | out | NEAR_FIELD |
| `LF_DEMOD` | in | NEAR_FIELD |

> The link to the ESP32 is a **crossover**: signal names are from the ESP32's point of view,
> so `LINK_ESP_TX` lands on this device's UART receive pin and `LINK_ESP_RX` on its transmit
> pin. Link speed is 921600 baud.

---

## Spare capacity

**11 GPIO remain free**, four of them ADC-capable. These are the main reserve for the
subsystems still to be captured, since the application MCU's pins are fully allocated.

---

## Status

Schematic capture complete. Interfaces to the wired-I/O and near-field sheets are defined and
waiting on those sheets.
