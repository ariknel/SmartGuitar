# 🎸 SmartGuitar Module

**Open-source drop-in PCB that transforms any passive electric guitar into a wireless recording instrument — without modifying the guitar.**

---

## What It Does

SmartGuitar is a compact PCB designed to fit inside the standard control cavity of a budget electric guitar (LP/SG style). It adds high-quality audio recording, WiFi streaming, and smart features while keeping the guitar fully playable as a normal instrument.

Plug in, play, and your performance is recorded locally to SD card and streamed wirelessly to a browser-based DAW interface — no audio interface, no cables, no laptop required.

---

## Features

- **24-bit audio capture** via PCM1808 ADC — professional quality from passive pickups
- **Local SD card recording** — WAV files written directly, no PC needed
- **WiFi WebSocket audio streaming** — real-time stream to web UI on any device on the same network
- **Browser-based recording UI** — capture, monitor, and save sessions from a phone or laptop
- **Automatic OTA firmware updates** — fetches latest release from GitHub on every boot
- **Real-time pitch detection** — streamed via WebSocket alongside audio
- **Battery monitoring** — level reported to web UI, charged via USB-C
- **Preamp bypass toggle** — switch between clean passive signal and boosted preamp output
- **Dedicated mic out** — always-preamped secondary output (3.5mm) for parallel recording
- **Rechargeable LiPo** — TP4056 charger, full cell protection, ~8h estimated runtime

---

## Hardware

| Block | Key Component |
|-------|--------------|
| MCU | ESP32-WROOM-32 |
| Preamp | NE5532 (±9V split supply) |
| ADC | PCM1808 (24-bit I²S) |
| Charger | TP4056 (500mA) |
| Boost | MT3608 → 9V |
| Neg. rail | TPS60403 → −9V |
| 5V reg | MCP1700-5002 |
| 3.3V reg | MCP1700-3302 |
| USB bridge | CP2102 |
| Storage | MicroSD (SPI) |

Full BOM in [`BOM.md`](./BOM.md).

---

## Signal Path

```
Guitar Pickup
    │
    ├─── [Passive bypass] ──────────────────────────────► 1/4" Output Jack
    │
    └─── [Preamp: NE5532 ±9V] ──┬──── [Preamp bypass] ──► 1/4" Output Jack
                                 │
                                 ├──────────────────────► 3.5mm Mic Out
                                 │
                                 └──► [PCM1808 ADC] ──► [ESP32-WROOM-32]
                                                              │
                                                    ┌─────────┴─────────┐
                                                    │                   │
                                               SD Card            WiFi Stream
                                             (WAV file)        (WebSocket → Browser)
```

---

## Firmware Architecture

```
Boot
 ├── Connect WiFi (captive portal on first boot)
 ├── Check GitHub for firmware update
 │    ├── Newer version found → OTA flash → reboot
 │    └── Up to date → continue
 ├── Start WebSocket server
 ├── Start HTTP web UI server
 └── Main loop
      ├── I²S DMA → audio buffer
      ├── Buffer → SD WAV writer
      ├── Buffer → WebSocket broadcast
      └── Pitch detection → WebSocket broadcast
```

---

## OTA Updates

Firmware updates are automatic. On every boot the device fetches `version.json` from this repository. If a newer version is available it downloads and flashes the new firmware silently, then reboots. No USB, no opening the guitar.

Manual flashing via USB-C (CP2102 bridge) is always available as a fallback.

---

## PCB

- Target cavity: LP/SG-style budget kit guitar (~100 × 65mm footprint, ~30mm depth)
- Designed in KiCad (files in `/hardware`)
- Star grounding — separate AGND / DGND planes joined at single star point
- Pi-filter on battery rail before analog section
- Ferrite beads on all supply crossings
- No copper under ESP32 antenna (cutout or board edge)

---

## Getting Started

### Hardware
1. Order PCB from `/hardware/gerbers`
2. Source components from `BOM.md`
3. Solder following assembly notes in `/hardware/ASSEMBLY.md`
4. Flash initial firmware via USB-C (see below)

### Initial Firmware Flash
```bash
git clone https://github.com/yourname/smartguitar
cd smartguitar/firmware
idf.py build
idf.py -p /dev/ttyUSB0 flash
```

After initial flash, all future updates are OTA.

### First Boot
1. Guitar creates WiFi AP: `SmartGuitar-Setup`
2. Connect and open `192.168.4.1`
3. Enter your WiFi credentials
4. Device reboots and connects — find its IP from your router or mDNS: `smartguitar.local`
5. Open `http://smartguitar.local` for the recording interface

---

## Repository Structure

```
smartguitar/
├── firmware/          # ESP-IDF firmware source
│   ├── main/
│   ├── components/
│   └── partitions.csv
├── hardware/          # KiCad project
│   ├── smartguitar.kicad_pro
│   ├── schematic/
│   ├── pcb/
│   └── gerbers/
├── web/               # Web UI (served from SPIFFS)
├── BOM.md             # Complete bill of materials
├── version.json       # Current firmware version (read by OTA)
└── README.md
```

---

## License

Hardware: CERN-OHL-S v2  
Firmware: MIT  
Documentation: CC BY 4.0

---

## Contributing

PRs welcome. Please open an issue before starting large changes. See `CONTRIBUTING.md`.

---

*Built for the open-source guitar community. Make it yours.*
