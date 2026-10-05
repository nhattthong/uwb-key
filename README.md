# UWB Smart Key

A portfolio-focused overview of the UWB smart-key subsystem developed as part of [Smart-CBOX](https://github.com/nhattthong/Smart-CBOX). This repository documents the system; it intentionally does not redistribute the firmware.

## Project overview

The prototype uses ultra-wideband (UWB) two-way ranging to estimate the distance between a vehicle-mounted anchor and a portable key tag. The anchor uses that distance to control a relay output with separate activation and release thresholds, while reporting key state and distance to an optional ESP32-S3 monitor.

## Main components

- **Key tag:** ESP32-C3 and DW3000 UWB module; uses interrupt wake and returns to deep sleep between ranging events.
- **Vehicle anchor:** STM32F103C8T6 (Blue Pill) and DW3000 UWB module; initiates ranging and calculates distance.
- **Output and monitoring:** Relay output for the key circuit; optional UART connection to an ESP32-S3 monitor.

## System flow

1. The anchor sends a UWB ranging request.
2. The tag wakes on the DW3000 interrupt, checks the received frame, and sends a timed response.
3. The anchor uses the exchange timestamps to estimate distance.
4. Hysteresis controls the relay: the prototype turns it on below 1.5 m and off above 3.0 m. If ranging is lost for 5 seconds, it turns the relay off.
5. The anchor can send distance and key state to the ESP32-S3 monitor over UART.

## Pinout

Signal names and GPIOs below are taken from the Smart-CBOX prototype firmware. Confirm the exact board variant and module wiring before connecting hardware.

### Key tag — ESP32-C3

| DW3000 signal | ESP32-C3 pin |
| --- | --- |
| SCK | GPIO4 |
| MISO | GPIO5 |
| MOSI | GPIO6 |
| CS | GPIO7 |
| IRQ | GPIO2 |
| RST | GPIO3 |

### Vehicle anchor — STM32F103C8T6

| Signal | STM32 pin |
| --- | --- |
| DW3000 SPI1 SCK / MISO / MOSI / CS | PA5 / PA6 / PA7 / PA4 |
| DW3000 IRQ / RST | PB0 / PA1 |
| Relay control | PB1 |
| UART1 TX / RX to ESP32-S3 | PA9 / PA10 |

The ESP32-S3 monitor uses UART2 RX=GPIO17 and TX=GPIO18.

## Security and limitations

The prototype uses UWB frame checks, short-address filtering, a watchdog on the anchor, and a fail-off relay timeout. The current ranging configuration does **not** enable secure ranging (STS), cryptographic peer authentication, or payload encryption. Address filtering and frame checks are not proof of key ownership, so this design should not be treated as production-grade vehicle security without a security redesign and validation.

## Source

System details are summarized from the SmartKey tag and anchor materials in [Smart-CBOX](https://github.com/nhattthong/Smart-CBOX). This overview focuses on the UWB key subsystem; the wider CBOX telemetry and vehicle-control features are outside this repository's scope.
