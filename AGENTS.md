# Coding Agent Guide

This repository contains firmware only. Do not add the earlier browser prototype.

## Objective

Maintain the `orca_browser` LVGL application for the 480×320 Free Wili 2 display-side RP2350B.

## Integration

The app is designed to be copied to `wilibsp/apps/orca_browser`. Apply `patches/wilibsp-integration.patch` at the root of a clean `wilibsp` checkout before building.

## Build and flash

```powershell
python tools/fw.py build orca_browser
python tools/fw.py flash orca_browser
```

Use CMSIS-DAP interface 0 for this application. Do not run RTT unless the user explicitly requests it.

## Hardware constraints

- The firmware uses `pico_set_binary_type(... copy_to_ram)` and shares roughly 512 KB SRAM with LVGL. Always verify that the ELF links after adding static assets.
- PSRAM stores expanded topic artwork and decoded playback buffers.
- Runtime QR generation is intentional and uses much less storage than prerendering every URL.
- Audio I2S is full duplex. `audio_capture_start()` must remain active so RX drains and the PIO does not stall.
- Long audio playback uses the patched chained-DMA streaming implementation.
- Silence must call `codec_nau88c10_speaker_low_power()`.
- Avoid creating dozens of LVGL objects simultaneously. Lists are paginated to protect the LVGL heap.

## Content ordering

`story_data.c` is ordered exactly as the home sections display it. Keep `first_story`, `cat_count`, story numbering, and the actual array length synchronized when adding or removing entries.

Every external link must remain accessible as a QR code because the hardware has no web browser.

## Generated assets

- `audio_data.c` contains compressed real recordings.
- `art_data.c` contains compact indexed artwork.
- `logo_data.c` contains the indexed official logo.

Regenerate assets deliberately and check palette ordering, stride, dimensions, RAM usage, and on-device rendering. Indexed-image scaling in LVGL was unreliable; the app expands topic artwork into PSRAM itself.

## Validation

At minimum:

1. Build `orca_browser` successfully.
2. Flash using the BSP tool and verify OpenOCD reports `Verified OK`.
3. Confirm the home list, topic view, QR view, orca/ocean playback, silence, and speaker wake-up behavior on hardware.

