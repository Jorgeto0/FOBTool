# FOBtool Hardware

FOBtool Hardware is the KiCad source repository for the FOBtool handheld electronic platform. It contains the project-level schematic and PCB files, custom symbols and footprints, and supporting documentation used to develop and maintain the hardware design.

## Project contents

| Path | Purpose |
| --- | --- |
| `FOBtool_HW.kicad_pro` | Main KiCad project file. Open this file to work with the design. |
| `FOBtool_HW.kicad_sch` | Top-level hierarchical schematic. |
| `POWER.kicad_sch` | Battery, charging, power-path, monitoring, and regulated-supply circuitry. |
| `MAIN_CONTROLLER_sch.kicad_sch` | ESP32-S3 main-controller circuitry and system control interfaces. |
| `FOBtool_HW.kicad_pcb` | PCB layout file. |
| `Custom_Components/` | Project-specific symbol and footprint resources. |
| `docs/` | Short descriptions of the project and schematic sheets. |

## Opening the design

1. Install [KiCad](https://www.kicad.org/) 10.x or a compatible later version.
2. Clone or download the complete repository so the project files and custom libraries remain together.
3. Open `FOBtool_HW.kicad_pro` from KiCad's Project Manager.
4. Confirm that the project symbol and footprint libraries load without errors before editing the design.

Open the project file rather than an individual child sheet so KiCad retains the full hierarchy and project library configuration.

## Schematic documentation

See the [documentation index](docs/README.md) for the purpose of each schematic sheet currently included in the project.

## Repository use

This repository contains active engineering source files. Design files may change as schematic capture, verification, PCB layout, and manufacturing preparation progress. Manufacturing outputs should be issued separately as a controlled release.

## Confidentiality

This project contains proprietary client hardware information. Keep the repository private and share its contents only with authorized project participants.
