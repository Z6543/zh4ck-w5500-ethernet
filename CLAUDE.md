# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ESP-IDF component that brings up W5500 Ethernet over SPI on the Tanmatsu hardware's J4 PMOD connector. Meant to be dropped into `components/` of an ESP-IDF project.

## Build

This is a pure ESP-IDF component — there is no standalone build. Build it from the parent project:

```bash
idf.py build
```

No tests, no linter config, no CI exist in this repo.

## Architecture

Two files total:

- `include/zh4ck_w5500_ethernet.h` — public API: `ethernet_init()` and `ethernet_connected()`
- `zh4ck_w5500_ethernet.c` — implementation

**Initialization flow (`ethernet_init`):**
1. Initialize SPI2 bus on the J4 PMOD pins
2. Probe the W5500 by reading its version register (address `0x0039`); bail with `ESP_ERR_NOT_FOUND` if chip not detected — this is intentionally non-fatal so boards without W5500 connected still boot
3. Create W5500 MAC + PHY drivers, install the ESP-IDF Ethernet driver
4. Derive a locally-administered MAC address from the chip's eFuse base MAC
5. Create an `esp_netif` and attach it via glue layer
6. Register `ETH_EVENT` and `IP_EVENT` handlers; start the driver

**Connection state** is tracked by the module-level `s_connected` flag, set `true` in `on_got_ip` and `false` on disconnect/stop events.

**Pin mapping (fixed, Tanmatsu J4 PMOD):**

| Signal | GPIO | Pad name |
|--------|------|----------|
| SCK    | 2    | MTCK     |
| MOSI   | 3    | MTDI     |
| MISO   | 5    | MTDO     |
| CS     | 4    | MTMS     |
| INT    | 15   | SAO_IO1  |
| RST    | 34   | SAO_IO2  |

**Caller requirements:** `esp_netif_init()` and `esp_event_loop_create_default()` must be called before `ethernet_init()`.
