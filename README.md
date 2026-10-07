# Marlin for BTT Octopus Pro v1.0 – custom CNC/laser machine

This is a fork of [MarlinFirmware/Marlin](https://github.com/MarlinFirmware/Marlin) with the configuration for one specific machine: a CNC router/laser built on a **BIGTREETECH Octopus Pro v1.0** (STM32F446ZE) with an **MKS TS35-R V2.0** touch display.

The working branch is **`laser-config`** (default branch). `bugfix-2.1.x` tracks upstream Marlin unchanged.

The same machine now also runs grblHAL, see [Steppge/grblHAL-OctopusPro-CNC-Laser](https://github.com/Steppge/grblHAL-OctopusPro-CNC-Laser). The grblHAL pin map was derived from this Marlin configuration.

## What is configured

### Machine
- `MOTHERBOARD BOARD_BTT_OCTOPUS_PRO_V1_0`, TMC2209 drivers via UART (X 1800 mA, Y/Y2 1300 mA, Z 1400 mA), no extruders, no heaters.
- Work area 290 × 235 × 214 mm, steps/mm 640 / 400 / 1600. X and Y home to min, Z homes to max first (`HOME_Z_FIRST`).
- Dual Y with its own endstop per side (`Y_DUAL_ENDSTOPS`) for squaring.
- S-curve acceleration, adaptive step smoothing, babystepping in 0.01 mm units, `G38` probing (probe on Z2-STOP).
- USB serial plus a second serial port, emergency parser, realtime reporting (`FULL_REPORT_TO_HOST_FEATURE`, Grbl-like status for CNC senders).
- SD card on the board, settings in EEPROM.

### Motor slots
The pin file [`pins_BTT_OCTOPUS_V1_common.h`](Marlin/src/pins/stm32f4/pins_BTT_OCTOPUS_V1_common.h) is modified so the axes use these driver slots:

| Axis | Driver slot |
|---|---|
| X | MOTOR 0 |
| Y | MOTOR 6 |
| Y2 | MOTOR 7 |
| Z | MOTOR 1 |
| – | MOTOR 2 – spare, no driver fitted |

`DIAG_JUMPERS_REMOVED` is set in [`pins_BTT_OCTOPUS_PRO_V1_0.h`](Marlin/src/pins/stm32f4/pins_BTT_OCTOPUS_PRO_V1_0.h).

### Laser
- `LASER_FEATURE`, enable on the bed output (`HEATER_BED_PIN`), PWM power control in `PWM255` units.
- Air assist on `HEATER_0_PIN` (M8/M9).
- Controller fan on FAN5.

### Display
MKS TS35-R V2.0 (ST7796, 480×320, SPI) with `TFT_COLOR_UI`, touch screen with calibration, rotated 180°.

## Fixes in this fork
**USB not recognized when the cable is plugged in at power-on, wrong SD card size over USB** – `Marlin/src/HAL/STM32/sd/msc_sd.cpp`, `Marlin/src/HAL/STM32/sdio.cpp`. Submitted upstream as [MarlinFirmware/Marlin#28638](https://github.com/MarlinFirmware/Marlin/pull/28638):
- `MSC_SD_init` no longer restarts USB while the host is still enumerating.
- The USB drive reports "no medium" until Marlin has finished mounting the SD card.
- `SDIO_GetCardSize` returns 512-byte blocks instead of bytes (also overflowed for cards > 4 GB).

## Troubleshooting: Marlin on the BTT Octopus Pro

**Board not recognized over USB when the cable is plugged in before power-on (STM32, SD card as USB drive)**
- Marlin restarted the USB stack while the PC was still enumerating the board. Fixed by the change above (PR #28638).

**SD card shows the wrong size or is not readable over USB (SDIO, cards > 4 GB)**
- The SDIO driver reported the card size in bytes instead of 512-byte blocks. Fixed by the change above.

**Config from an older bugfix-2.1.x does not compile on current Marlin**
- Some options were renamed or moved when this configuration was ported from bugfix-2.1.x of 2024-04-03: `DISABLE_ENCODER` is now `NO_BACK_MENU_ITEM`, `SDSUPPORT` moved, TMC UART RX pins now default to the TX pin, `LCD_INFO_SCREEN_STYLE` is only valid for character LCDs.

**Octopus Pro v1.0 vs. v1.1**
- Select `BOARD_BTT_OCTOPUS_PRO_V1_0` for the v1.0 board, some pins differ from v1.1.

## Build
1. Install VS Code with the PlatformIO extension (Auto Build Marlin recommended).
2. Clone this repository and check out `laser-config`:
   ```
   git clone -b laser-config https://github.com/Steppge/Marlin-OctopusPro-CNC-Laser.git
   ```
3. Build the PlatformIO environment `STM32F446ZE_btt_usb_flash_drive` (SD card also available as USB drive) or `STM32F446ZE_btt`.
4. Copy the resulting `firmware.bin` to the SD card and power up the board.

## Credits & license
Based on [Marlin](https://github.com/MarlinFirmware/Marlin) by the Marlin Firmware team and contributors, documentation at [marlinfw.org](https://marlinfw.org).

Licensed under the GNU General Public License v3, see [LICENSE](LICENSE). The copyright notices in the source files apply.
