# 🎸 SmartGuitar Module

**Open-source drop-in PCB that transforms any passive electric guitar into a wireless recording instrument — and lets you plug directly into any speaker, hi-fi, or computer without a guitar amp.**

---

## The Problem It Solves

A raw electric guitar pickup produces a weak, high-impedance signal. Normally you need a guitar amplifier to make it usable — that's extra gear, extra cost, extra bulk.

SmartGuitar fixes this with a built-in **NE5532 preamp running on a ±9V split supply**, which:

- Boosts the pickup signal to **line level** (~500mV–1V RMS)
- Lowers output impedance so it drives long cables without tone loss
- Lets you plug the standard 1/4" output jack directly into any of the following — **no guitar amp needed:**

| Destination | How |
|-------------|-----|
| Powered speaker / hi-fi system | 1/4" or adapter to 3.5mm / RCA |
| PC / Mac line-in or audio interface | 1/4" to 1/4" or 3.5mm |
| Mixer / PA system | 1/4" line input |
| Phone or tablet | 1/4" to USB-C audio adapter |
| DAW (direct recording) | Line-in, no interface needed |
| Studio monitors | 1/4" TRS to XLR with adapter |

A bypass switch restores the raw passive pickup signal for use with a traditional guitar amp if preferred.

---

## Recording Without a Computer — Managing Files From Your Phone

SmartGuitar records 24-bit WAV directly to a microSD card. When you're done playing, connect any phone, tablet, or laptop to the guitar's built-in WiFi and open a browser. The guitar serves a full file manager — no internet, no app, no cables required.

```
1. Play  →  guitar records WAV to SD card silently
2. Done  →  connect phone/laptop to "SmartGuitar-XXXX" WiFi
3. Open browser → http://192.168.4.1
4. Browse takes, press play to listen in browser,
   download what you want, delete what you don't
5. Disconnect — guitar ready to record again
```

Works anywhere — on stage, in a rehearsal room, in a field, on a tour bus. No router, no internet, no cables.

---

## How WiFi Works

The ESP32 runs in **dual-mode (AP + STA)**:

**When a known router is nearby (home, studio):**
- ESP32 connects to your router as a station (STA) and announces itself via mDNS as `smartguitar.local`
- Your phone stays on the same router network — internet uninterrupted
- Open `http://smartguitar.local` in any browser
- Android app discovers the guitar automatically via mDNS (no IP typing)
- OTA firmware updates happen silently in the background

**When no router is available (on stage, anywhere):**
- ESP32 falls back to pure Access Point mode
- Guitar creates its own WiFi: `SmartGuitar-XXXX`
- Fixed address: `http://192.168.4.1` — no DNS needed, works on every OS
- Android app detects the SSID and connects automatically
- Phone loses internet during this session — acceptable tradeoff

**Connecting is one tap:** the OLED display on the PCB shows a WiFi QR code. Android and iOS camera apps scan it and join the network instantly — no typing.

---

## OLED Display (128×64)

A 128×64 SSD1306 OLED is built into the PCB, visible through the guitar back cover. It cycles between screens with a button press:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   TUNER      │  │   QR CODE    │  │   STATUS     │  │  RECORDING   │
│              │  │              │  │              │  │ (auto-switch)│
│  E2  82.4Hz  │  │  ██ QR ██   │  │ SmartGuitar  │  │ ●REC 02:14  │
│  ◄────┼────► │  │  Scan to     │  │ 192.168.4.1  │  │ ♪ E2 82Hz   │
│  flat  sharp │  │  join WiFi   │  │ Bat: 87%     │  │ SD: 11.9GB  │
│  -8 cents    │  │              │  │ SD: 12.4GB   │  │ Bat: 72%    │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
   default idle      hold button        info screen      auto on record
```

### Built-in Guitar Tuner
The tuner uses the **YIN autocorrelation algorithm** — the same approach used in hardware clip-on tuners. Accuracy is under 1 cent at all standard guitar frequencies. Pluck any string and it auto-detects the nearest note — no manual string selection.

Standard tuning reference displayed:
```
E2 82.4Hz  A2 110Hz  D3 146.8Hz  G3 196Hz  B3 246.9Hz  E4 329.6Hz
```

The tuner reads from the same PCM1808 I²S audio stream used for recording — no extra hardware. It runs on a dedicated FreeRTOS task so it never interferes with recording or WiFi.

---

## Full Feature List

- **Line-level 1/4" output** — plug into any speaker, hi-fi, PC, or mixer; no guitar amp needed
- **Preamp bypass switch** — raw passive signal for traditional guitar amps
- **Dedicated 3.5mm mic out** — always-on line-level secondary output for parallel recording
- **24-bit WAV recording** to microSD, up to 96kHz sample rate
- **Built-in WiFi AP + STA dual mode** — works with or without a router
- **Browser file manager** — served from ESP32, works offline on any device
- **In-browser WAV playback** — listen to takes without downloading; HTTP range request streaming
- **128×64 OLED display** — tuner, QR code, recording status, battery, SD space
- **WiFi QR code on OLED** — one-tap phone joining, no typing
- **Guitar tuner** — YIN algorithm, auto string detection, <1 cent accuracy
- **Automatic OTA firmware updates** — silent, on boot, when internet is available
- **Real-time pitch detection** to web UI and OLED
- **Battery monitoring** on OLED and web UI
- **Rechargeable LiPo** via USB-C, ~8h runtime
- **USB-C flashing** via CP2102 for firmware recovery

---

## Android App

A companion Android app provides a native experience when connected to the guitar on the same network:

**Discovery (automatic, no credentials):**
- Reads current WiFi SSID — if `SmartGuitar-XXXX`, uses `192.168.4.1` directly
- Otherwise runs mDNS discovery for `_smartguitar._tcp` on the local network
- User never types an IP address or WiFi password into the app

**Permissions required:**
- `ACCESS_FINE_LOCATION` — required by Android to read current WiFi SSID (system policy, not optional)
- That's it

**Features:**
- File browser with playback via ExoPlayer (HTTP streaming, seeking works)
- Download via Android `DownloadManager` (background, notification tray)
- Delete recordings
- Live recording status and battery level
- Tuner display

---

## Hardware

| Block | Key Component | Notes |
|-------|--------------|-------|
| MCU | ESP32-WROOM-32 | WiFi AP+STA, I²S master, SPI, HTTP+WebSocket server |
| Preamp | NE5532 (±9V split supply) | Line-level output; drives any line input directly |
| ADC | PCM1808 (24-bit I²S) | 24-bit, up to 96kHz; ESP32 provides all clocks |
| OLED | SSD1306 128×64 | I²C; tuner, QR code, status, recording screen |
| Charger | TP4056 (500mA) | USB-C charging |
| Boost | MT3608 → +9V | NE5532 positive rail |
| Neg. rail | TPS60403 → −9V | NE5532 negative rail; proper bipolar supply |
| 5V reg | MCP1700-5002 | PCM1808 analog VCC |
| 3.3V reg | MCP1700-3302 | Digital rail |
| USB bridge | CP2102 | Firmware flashing and serial debug via USB-C |
| Storage | MicroSD SPI | WAV recordings + `/www/` web UI assets |

Full BOM in [`BOM.md`](./BOM.md).

---

## ESP32 RAM Budget (v1)

Understanding RAM constraints is critical for this project. The ESP32-WROOM-32 has **520KB SRAM total**, of which roughly **370KB is available** after FreeRTOS and WiFi stack overhead.

### System Overhead (fixed, non-negotiable)

| Subsystem | RAM used |
|-----------|----------|
| FreeRTOS kernel | ~10 KB |
| WiFi stack (AP+STA dual mode) | ~80 KB |
| TCP/IP stack (lwIP) | ~30 KB |
| WiFi dynamic buffers | ~20 KB |
| System / boot reserves | ~10 KB |
| **Total system** | **~150 KB** |

**Available for application: ~370 KB**

### Application RAM Budget (v1)

| Allocation | Size | Notes |
|------------|------|-------|
| I²S DMA buffers | ~8 KB | 4× 256-sample buffers, 32-bit stereo |
| Audio ring buffer | ~44 KB | 256ms rolling window at 44.1kHz/32-bit — see note |
| WAV write double-buffer | ~32 KB | 2× 16KB; large enough to absorb SD write stalls |
| YIN pitch detection | ~16 KB | 2048-sample window + working buffer |
| HTTP server | ~12 KB | Stack + chunked file send buffer |
| WebSocket server | ~6 KB | Frame buffers for up to 3 clients |
| FatFS + SD buffers | ~8 KB | FatFS work buffer + file write buffer |
| FreeRTOS task stacks | ~32 KB | 7 tasks: audio, SD write, pitch, HTTP, WS, OTA, OLED |
| mDNS | ~8 KB | |
| NVS settings cache | ~4 KB | WiFi credentials, gain mode, sample rate |
| Miscellaneous + heap frag | ~16 KB | |
| **Total application** | **~186 KB** |

**Remaining headroom: ~184 KB**

> **Audio ring buffer note:** A 500ms window would use 88KB — too large. The ring buffer is sized to 256ms (~44KB), which is sufficient for pitch detection (YIN needs ~46ms) and WebSocket streaming. This is the single most important RAM decision in the firmware.

### v1 Verdict

**WROOM-32 works for v1** with careful buffer sizing. The 256ms ring buffer is the key constraint that makes it fit. Headroom is comfortable enough for stable operation.

### DSP / FX — Future v2 Considerations

DSP effects were intentionally excluded from v1. Here is why, and what v2 would require:

| Effect | Extra RAM needed | Feasibility on WROOM-32 |
|--------|-----------------|------------------------|
| Biquad EQ (per band) | ~1 KB | ✅ Fine |
| Compressor / noise gate | ~2 KB | ✅ Fine |
| Chorus / flanger | ~10–20 KB | ⚠️ Marginal — eats headroom |
| Reverb (Schroeder) | ~64–256 KB | ❌ Impossible without PSRAM |
| Reverb (convolution) | ~512 KB+ | ❌ Impossible without PSRAM |

**For v2 DSP:** swap to **ESP32-WROVER-32** — identical footprint to WROOM-32, same PCB, adds 4MB PSRAM. Reverb delay lines live in PSRAM, SRAM is freed up, all effects become feasible. This is a firmware + module swap only — no PCB redesign required if the footprint is reserved correctly.

The PCB footprint for the WROOM-32 and WROVER-32 is identical — designing for WROOM-32 now means a WROVER-32 can be dropped in for v2 without any hardware changes.

---

## Signal Path

```
Guitar Pickup (~10–100mV, high impedance)
    │
    ├─── [Passive bypass] ───────────────────────────────────────► 1/4" Output Jack
    │                                                               (traditional guitar amp)
    │
    └─── [NE5532 Preamp ±9V → ~500mV–1V RMS line level] ────────► 1/4" Output Jack
              │                                                      (any speaker, hi-fi,
              │                                                       PC, mixer, DAW —
              │                                                       no amp needed)
              │
              ├──────────────────────────────────────────────────► 3.5mm Mic Out
              │                                                     (always-on line-level)
              │
              └──► [PCM1808 24-bit ADC] ──► [ESP32-WROOM-32]
                                                   │
                                     ┌─────────────┼─────────────┐
                                     │             │             │
                                SD Card       WiFi AP+STA     OLED
                              (WAV files,   192.168.4.1 /   Tuner /
                              /www/ UI)    smartguitar.local  Status
```

---

## Firmware Architecture

```
Boot
 ├── Init I²S + PCM1808 (audio capture ready)
 ├── Mount SD card → load /www/ assets check
 ├── Start WiFi
 │    ├── Saved credentials exist?
 │    │    ├── YES → connect STA (5s timeout)
 │    │    │    ├── Connected → AP+STA dual mode + mDNS announce
 │    │    │    │    └── Check GitHub version.json → OTA if newer
 │    │    │    └── Timeout → fall back to AP only
 │    │    └── NO → AP only mode
 │    └── AP always running: "SmartGuitar-XXXX" @ 192.168.4.1
 ├── Start HTTP server (serve /www/ from SD, fallback to LittleFS)
 ├── Start WebSocket server
 └── Main FreeRTOS tasks
      ├── [Task] Audio capture: I²S DMA → ring buffer (Core 1)
      ├── [Task] SD writer: ring buffer → WAV file (Core 1)
      ├── [Task] Pitch/YIN: ring buffer → note + cents (Core 0)
      ├── [Task] HTTP server: file manager + streaming (Core 0)
      ├── [Task] WebSocket: push level, battery, pitch, status (Core 0)
      ├── [Task] OLED: tuner / QR / status / recording screens (Core 0)
      └── [ISR]  Record button → start/stop recording
```

---

## SD Card Layout

```
SD card (FAT32, up to 32GB)
├── /www/                   ← Web UI assets (HTML, JS, CSS)
│   ├── index.html
│   ├── app.js
│   └── style.css
├── /recordings/            ← WAV files
│   ├── 20250504_143022.wav
│   └── 20250504_151100.wav
└── /system/
    └── config.json         ← Device config backup
```

The web UI is served from `/www/` on the SD card. This means the UI can be updated simply by replacing files on the SD card — no firmware flash required for UI changes. A minimal fallback UI is baked into LittleFS (internal flash) in case no SD card is present.

---

## Partition Table (Internal Flash, 4MB minimum — 8MB recommended)

```
# Name      Type   SubType   Offset     Size
nvs         data   nvs       0x9000     0x5000    # WiFi creds, settings
otadata     data   ota       0xe000     0x2000    # OTA state
ota_0       app    ota_0     0x10000    0x1C0000  # Firmware slot A (~1.75MB)
ota_1       app    ota_1     0x1D0000   0x1C0000  # Firmware slot B (~1.75MB)
littlefs    data   littlefs  0x390000   0x70000   # Fallback web UI (~448KB)
```

> **8MB flash strongly recommended.** On 4MB flash the two OTA partitions plus LittleFS leave very little margin. The 8MB WROOM-32 variant (`ESP32-WROOM-32E-N8`) uses the identical module footprint — same PCB, just order the N8 suffix.

---

## PCB

- Target cavity: LP/SG-style budget kit guitar (~100 × 65mm footprint, ~30mm depth)
- Designed in KiCad (files in `/hardware`)
- Star grounding — AGND and DGND joined at single star point via 600Ω@100MHz ferrite
- ±9V split supply for NE5532 — proper bipolar, no virtual ground
- Pi-filter on battery rail before analog section
- Ferrite beads on all supply rail crossings
- No copper under ESP32 antenna (cutout or board edge overhang)
- OLED mounted flush to back cover cutout
- All SMD, 0402/0603/0805

---

## Getting Started

### Initial Firmware Flash
```bash
git clone https://github.com/yourname/smartguitar
cd smartguitar/firmware
idf.py build
idf.py -p /dev/ttyUSB0 flash monitor
```

### SD Card Setup
Format SD card as FAT32. Copy the `/www/` folder from this repo onto it. Insert into guitar.

### Recording
1. Power on — OLED shows tuner, green LED steady
2. Press record button — OLED switches to recording screen, red LED solid
3. Play
4. Press record button again — file saved, OLED returns to tuner

### Managing Files
1. Scan QR code on OLED with phone camera → joins `SmartGuitar-XXXX` WiFi
2. Browser opens automatically → `http://192.168.4.1`
3. Browse, play, download, or delete recordings

### Plugging In
- **Guitar amp:** bypass switch to passive
- **Speaker / hi-fi / PC / mixer:** bypass switch to preamp — line level, plug straight in
- **Both simultaneously:** 1/4" main out to amp, 3.5mm mic out to recording device

### OTA Updates (optional)
Open `http://192.168.4.1` → Settings → enter home WiFi credentials. On next boot near that network, firmware updates automatically if a newer version exists on GitHub.

---

## Repository Structure

```
smartguitar/
├── firmware/               # ESP-IDF source
│   ├── main/
│   ├── components/
│   │   ├── yin/            # YIN pitch detection
│   │   ├── qrcodegen/      # QR code generation (Nayuki)
│   │   └── ssd1306/        # OLED driver
│   ├── web/                # Web UI (built into LittleFS + copied to /www/)
│   └── partitions.csv
├── hardware/               # KiCad project
│   ├── schematic/
│   ├── pcb/
│   └── gerbers/
├── android/                # Android companion app
├── BOM.md
├── version.json
└── README.md
```

---

## Roadmap

### v1 (this release)
- [x] Hardware design + BOM
- [ ] PCB schematic + layout
- [ ] Firmware: audio capture, SD recording, WiFi AP+STA
- [ ] Firmware: HTTP file manager, WebSocket status
- [ ] Firmware: YIN tuner, OLED UI, QR code
- [ ] Firmware: OTA update check
- [ ] Web UI: file manager, playback, settings
- [ ] Android app: discovery, playback, download

### v2 (planned)
- [ ] Swap to ESP32-WROVER-32 (PSRAM) — same PCB footprint
- [ ] DSP effects: EQ, compressor, noise gate (SRAM-only)
- [ ] DSP effects: reverb, chorus, flanger (PSRAM required)
- [ ] Looper (PSRAM required)
- [ ] MIDI output

---

## Who This Is For

- Beginners who bought a cheap guitar kit and don't want to buy an amp
- Players who want to record directly into a DAW without an audio interface
- Performers who want to capture rehearsals or gigs without extra gear
- Tinkerers who want a smart guitar without paying premium prices

---

## License

Hardware: CERN-OHL-S v2
Firmware: MIT
Documentation: CC BY 4.0

---

## Contributing

PRs welcome. Open an issue before starting large changes.

---

*Built for the open-source guitar community. Plug into anything. Record everything. Manage it from your phone.*
