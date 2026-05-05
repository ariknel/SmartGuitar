# 🎸 SmartGuitar Module

**Open-source drop-in PCB that transforms any passive electric guitar into a wireless smart instrument — with a built-in preamp, 24-bit recorder, WiFi file manager, guitar tuner, and a full DSP effects chain. No guitar amp required.**

---

## The Problem It Solves

A raw electric guitar pickup produces a weak, high-impedance signal (~10–100mV). Normally you need a guitar amplifier to make it usable — extra gear, extra cost, extra bulk.

SmartGuitar fixes this with a built-in **NE5532 preamp on a proper ±9V split supply**, which boosts the pickup signal to **line level** (~500mV–1V RMS) and lowers output impedance. The standard 1/4" output jack then drives any of the following directly — **no guitar amp needed:**

| Destination | Connection |
|-------------|-----------|
| Powered speaker / hi-fi | 1/4" or adapter to 3.5mm / RCA |
| PC / Mac line-in or audio interface | 1/4" to 1/4" or 3.5mm |
| Mixer / PA system | 1/4" line input |
| Phone or tablet | 1/4" to USB-C audio adapter |
| DAW (direct recording) | Line-in, no interface needed |
| Studio monitors | 1/4" TRS to XLR with adapter |

A bypass switch restores the raw passive pickup signal for use with a traditional guitar amp.

---

## Recording Without a Computer — Managing Files From Your Phone

SmartGuitar records 24-bit WAV directly to microSD. When you're done playing, connect any phone, tablet, or laptop to the guitar's own built-in WiFi and open a browser. A full React web app is served directly from the ESP32-S3 — no internet, no app install, no cables.

```
1. Play  →  guitar records WAV to SD silently
2. Done  →  connect phone to "SmartGuitar-XXXX" WiFi (scan QR on OLED)
3. Open browser → http://192.168.4.1
4. Full React UI loads instantly from ESP32 RAM
5. Browse takes, stream-play in browser, download, delete
6. Disconnect — guitar ready to record again
```

Works anywhere — on stage, rehearsal room, field, tour bus. No router, no internet, no cables ever required.

---

## How WiFi Works

The ESP32-S3 runs **dual-mode WiFi (AP + STA simultaneously)**:

**When a known router is nearby (home, studio):**
- Connects to your router as a station, announces via mDNS as `smartguitar.local`
- Phone stays connected to the router — internet uninterrupted
- Open `http://smartguitar.local` in any browser
- Android/iOS app discovers the guitar automatically via mDNS — no IP typing
- OTA firmware updates happen silently in the background

**When no router is available (on stage, anywhere):**
- Falls back to pure Access Point mode automatically
- Fixed address `http://192.168.4.1` — works on every OS, no DNS required
- Android app detects `SmartGuitar-XXXX` SSID and connects automatically
- Phone loses internet during this session — acceptable, expected

**Joining is one tap:** the OLED display shows a WiFi QR code. Android and iOS camera apps scan it and join instantly — no typing.

---

## Web UI — Full React App, Served From ESP32 RAM

The entire web interface is a proper React single-page application. On boot, the ESP32-S3 loads all UI assets from LittleFS (internal 16MB flash) into PSRAM once, then serves every browser request directly from RAM. Response times are limited only by WiFi throughput — no SD card reads during browsing.

**Memory allocation for the web UI:**
- React bundle (expanded): ~400KB
- HTML + CSS + assets: ~100KB
- React VDOM + runtime state: ~200KB
- **Total: ~700KB out of 8MB PSRAM** — 3% used for the entire UI

This gives room for a genuinely rich, professional interface — waveform visualizer, real-time meters, DSP controls, settings — without compromise.

### File Manager
- Browse all recordings with name, size, duration, date
- **Stream and play any WAV directly in the browser** — ESP32 serves via HTTP range requests, browser native `<audio>` handles seeking and playback with no plugins
- Download files individually or as a batch ZIP
- Delete takes or entire sessions
- SD card free space indicator

### Live Recording View
- Recording status and elapsed time
- Real-time input level meter (peak hold)
- Remaining SD space
- Battery level and estimated runtime
- Pitch / note display

### DSP Effects Panel
- Per-effect enable/disable toggles
- Parameter controls (knobs/sliders) for each effect
- Signal chain order visualizer
- Preset save/load

### Settings
- WiFi credentials (for OTA updates when available)
- Sample rate (44.1kHz / 48kHz / 96kHz)
- Gain mode (standard / high-output humbucker)
- OLED brightness and screen timeout

---

## OLED Display (128×64 SSD1306)

Built into the PCB, visible through the guitar back cover. Button press cycles between screens:

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   TUNER      │  │   QR CODE    │  │   STATUS     │  │  RECORDING   │
│  E2  82.4Hz  │  │  ████████   │  │ SmartGuitar  │  │ ●REC 02:14  │
│  ◄────┼────► │  │  ██ WiFi ██  │  │ 192.168.4.1  │  │ ♪ E2 82Hz   │
│  flat  sharp │  │  ████████   │  │ Bat: 87%     │  │ SD: 11.9GB  │
│  -8 cents    │  │  Scan to     │  │ SD: 12.4GB   │  │ Bat: 72%    │
│              │  │  join WiFi   │  │ v1.0.4       │  │             │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘
   default idle      hold button        info screen     auto on record
```

### Guitar Tuner
YIN autocorrelation algorithm — same approach as hardware clip-on tuners. Under 1 cent accuracy at all standard guitar frequencies. Auto-detects the nearest string on pluck — no manual selection needed. Reads from the same I²S audio stream used for recording, zero extra hardware.

---

## DSP Effects Chain

The ESP32-S3-WROOM-1-N16R8 has enough CPU and PSRAM headroom to run a complete real-time DSP effects chain simultaneously with recording and WiFi. All effects are processed on **Core 1** in a dedicated audio task with 5.8ms buffer intervals at 44.1kHz.

### Memory & CPU Analysis

**Internal SRAM (512KB):** Reserved for time-critical only — DMA buffers, audio ring buffer, task stacks, WiFi stack, FreeRTOS. Never used for DSP delay lines.

**PSRAM (8MB):** Used for all large DSP allocations, web UI, and looper buffer.

| Allocation | PSRAM used | Notes |
|------------|-----------|-------|
| Web UI (React, fully loaded) | 700 KB | Served entirely from RAM |
| EQ + compressor + gate (v1) | 6 KB | Coefficient tables only |
| Chorus + flanger + tremolo (v2) | 36 KB | Delay lines in PSRAM |
| Reverb + cabinet IR (v3) | 378 KB | Schroeder combs + convolution |
| Looper (60s mono / 30s stereo) | 5,292 KB | Full loop buffer |
| WAV streaming cache | 512 KB | Optional read-ahead |
| Headroom | 1,268 KB | |
| **Total** | **8,192 KB** | Fits exactly with looper enabled |

**CPU budget per 256-sample buffer (Core 1, 240MHz):**

| Effect | Cycles | Status |
|--------|--------|--------|
| 4-band parametric EQ | ~50K | ✅ Trivial |
| Compressor | ~30K | ✅ Trivial |
| Noise gate | ~10K | ✅ Trivial |
| Chorus (3 voice) | ~150K | ✅ Easy |
| Flanger | ~80K | ✅ Easy |
| Schroeder reverb | ~400K | ✅ Manageable |
| Convolution reverb (512-tap) | ~600K | ✅ OK with LX7 PIE vectors |
| Cabinet IR simulation | ~300K | ✅ OK |
| **All effects combined** | **~1.62M** | ✅ Fits Core 1 (~1.2M budget) |

The ESP32-S3's **LX7 PIE vector extension** accelerates FIR convolution and biquad filters significantly over the original ESP32 — this is what makes convolution reverb feasible without a dedicated DSP chip.

### Effects — v1 (ships with firmware)
- **4-band parametric EQ** — low shelf, two mid peaks, high shelf; biquad IIR
- **Compressor** — RMS detection, adjustable threshold/ratio/attack/release
- **Noise gate** — threshold-based, kills hum between notes

### Effects — v2
- **Chorus** — 3 voices, LFO-modulated delay lines, adjustable depth and rate
- **Flanger** — short delay with feedback and LFO sweep
- **Tremolo** — amplitude modulation, sine/square/triangle LFO

### Effects — v3
- **Reverb (Schroeder)** — 4 comb filters + 2 allpass, room/hall/plate presets
- **Reverb (convolution)** — 512-tap IR, load custom impulse responses via web UI
- **Cabinet IR simulation** — 512-tap FIR, multiple cabinet models

### Looper *(v3 OTA)*
- Loop audio streamed directly to/from SD card — no PSRAM consumed, no RAM constraint
- Loop length limited only by SD card space (1GB SD ≈ 3 hours at 44.1kHz/16-bit mono)
- Overdub, undo, stop controls
- Controlled via **JST-SH expansion header (J6)** — wire a momentary button anywhere on the guitar body (near knobs, upper bout, strap button), or plug a standard footswitch pedal into **3.5mm jack (J7)** for hands-free operation on stage
- All hardware present from v1 — no PCB modification needed when v3 firmware ships

> **All DSP effects and looper hardware are v1 PCB-ready.** Everything is a firmware matter — ship what is stable in v1, unlock more via OTA updates.

---

## Full Feature List

**Audio output:**
- Line-level 1/4" output — plug into any speaker, hi-fi, PC, or mixer without a guitar amp
- Preamp bypass switch — raw passive signal for traditional guitar amps
- Dedicated 3.5mm mic out — always-on line-level secondary output

**Recording:**
- 24-bit WAV to microSD, up to 96kHz sample rate
- One-button record control on guitar back

**Connectivity:**
- Built-in WiFi AP + STA dual mode — works with or without a router
- Fixed AP address `192.168.4.1` — works on every device without DNS

**Web interface:**
- Full React SPA served from ESP32 PSRAM — instant load, no install
- File manager with in-browser WAV streaming and playback
- Real-time recording status, level meter, battery
- DSP effects panel with per-effect controls and presets
- Settings: WiFi, gain, sample rate, display

**Display:**
- 128×64 SSD1306 OLED — tuner, QR code, status, recording screen
- WiFi QR code — one-tap phone joining

**Tuner:**
- YIN autocorrelation, auto string detection, <1 cent accuracy

**DSP effects chain (Core 1, real-time):**
- EQ, compressor, noise gate (v1)
- Chorus, flanger, tremolo (v2 OTA)
- Reverb (Schroeder + convolution), cabinet IR (v3 OTA)
- 60s mono / 30s stereo looper

**Firmware:**
- Automatic OTA updates from GitHub when internet available
- Silent, background, no user action required
- USB-C recovery flashing via native S3 USB (no CP2102 needed)

**Power:**
- Rechargeable LiPo via USB-C, ~8h runtime
- Battery level on OLED and web UI

---

## Android App

**Discovery (fully automatic):**
- Reads current WiFi SSID — if `SmartGuitar-XXXX` detected, uses `192.168.4.1`
- Otherwise runs mDNS discovery for `_smartguitar._tcp` on local network
- User never types an IP address or password into the app

**Permissions:** `ACCESS_FINE_LOCATION` only (required by Android to read SSID)

**Features:**
- File browser with ExoPlayer HTTP streaming (seeking works)
- Background download via Android `DownloadManager`
- Delete recordings
- Live status: recording, battery, SD space
- Tuner display
- DSP effect controls

---

## Hardware

| Block | Component | Notes |
|-------|-----------|-------|
| MCU | ESP32-S3-WROOM-1-N16R8 | 16MB flash, 8MB octal PSRAM, native USB, LX7 PIE |
| Preamp | NE5532 ±9V | Line-level output; drives any line input |
| ADC | PCM1808 24-bit I²S | Up to 96kHz; ESP32-S3 provides all clocks |
| OLED | SSD1306 128×64 | I²C; tuner, QR, status, recording |
| Charger | TP4056 500mA | USB-C |
| Boost | MT3608 → +9V | NE5532 positive rail |
| Neg. rail | TPS60403 → −9V | NE5532 negative rail |
| 5V reg | MCP1700-5002 | PCM1808 VCC |
| 3.3V reg | MCP1700-3302 | Digital rail |
| Storage | MicroSD SPI | WAV recordings only; UI lives in PSRAM |

Full BOM in [`BOM.md`](./BOM.md).

---

## Signal Path

```
Guitar Pickup (~10–100mV, high impedance)
    │
    ├─── [Passive bypass] ─────────────────────────────────────► 1/4" Output Jack
    │                                                             (traditional amp)
    │
    └─── [NE5532 ±9V → ~500mV–1V RMS line level] ─────────────► 1/4" Output Jack
              │                                                    (any speaker, hi-fi,
              │                                                     PC, mixer, DAW)
              ├─────────────────────────────────────────────────► 3.5mm Mic Out
              │
              └──► [PCM1808 24-bit ADC]
                          │
                   [ESP32-S3 Core 1]          [ESP32-S3 Core 0]
                          │                          │
                   I²S DMA capture            WiFi AP+STA
                          │                    HTTP server
                   Audio ring buffer          WebSocket server
                          │                    OLED driver
                   ┌──────┴────────┐           mDNS
                   │               │
              DSP chain         SD writer
              (EQ→comp→gate     (WAV file)
              →chorus→reverb
              →looper)
                   │
              WebSocket stream ──────────────► Browser
              (level, pitch,                  React UI
               battery, status)               (from PSRAM)
```

---

## Memory Architecture

```
Internal SRAM (512KB)          PSRAM (8MB)
─────────────────────          ──────────────────────────────────
FreeRTOS + WiFi: 150KB         Web UI (React SPA):      700KB
I²S DMA buffers:   8KB         DSP v1 (EQ/comp/gate):     6KB
Audio ring buf:   44KB         DSP v2 (chorus/flanger):   36KB
WAV write buf:    32KB         DSP v3 (reverb/cab IR):   378KB
YIN pitch buf:    16KB         Looper (60s mono):       5292KB
Task stacks:      32KB         WAV stream cache:         512KB
HTTP/WS/mDNS:     26KB         Headroom:                1268KB
DSP coefficients:  4KB         ──────────────────────────────────
Misc:             16KB         Total:                   8192KB
─────────────────────
Used:            328KB
Free:            184KB
```

---

## Firmware Architecture

```
Boot
 ├── Init I²S + PCM1808
 ├── Mount SD card
 ├── Load web UI assets: LittleFS → PSRAM
 ├── Allocate DSP buffers in PSRAM
 ├── Start WiFi
 │    ├── Saved credentials? → attempt STA (5s timeout)
 │    │    ├── Connected → AP+STA + mDNS + OTA check
 │    │    └── Timeout → AP only
 │    └── AP always on: "SmartGuitar-XXXX" @ 192.168.4.1
 ├── Start HTTP server (UI from PSRAM, WAV from SD)
 ├── Start WebSocket server
 └── FreeRTOS tasks
      Core 1 (audio, realtime):
        ├── Audio capture: I²S DMA → ring buffer
        ├── DSP chain: EQ → comp → gate → chorus → reverb → looper
        ├── SD writer: ring buffer → WAV file
        └── YIN pitch detection
      Core 0 (networking, UI):
        ├── HTTP server: UI from PSRAM + WAV streaming from SD
        ├── WebSocket: push level, battery, pitch, status
        ├── OLED: tuner / QR / status / recording
        └── Record button ISR handler
```

---

## Partition Table (16MB flash)

```
# Name      Type   SubType   Offset     Size
nvs         data   nvs       0x9000     0x5000     #   20KB — WiFi creds, settings
otadata     data   ota       0xe000     0x2000     #    8KB — OTA slot state
ota_0       app    ota_0     0x10000    0x300000   # 3.0MB — firmware slot A
ota_1       app    ota_1     0x310000   0x300000   # 3.0MB — firmware slot B
littlefs    data   littlefs  0x610000   0x9F0000   #  ~10MB — React UI + IR files
```

LittleFS holds the full React app build output plus convolution reverb IR files. Updating the UI or loading new IR files happens automatically via OTA.

---

## SD Card Layout

```
SD card (FAT32, up to 32GB)
└── /recordings/
    ├── 20250504_143022.wav
    └── 20250504_151100.wav
```

SD card contains recordings only. The web UI lives entirely in ESP32 flash/PSRAM — the UI works even with no SD card inserted (empty file list).

---

## Getting Started

### Initial Firmware Flash
```bash
git clone https://github.com/yourname/smartguitar
cd smartguitar/firmware
idf.py build
idf.py -p /dev/ttyACM0 flash    # Linux (native USB, no driver needed)
idf.py -p COM3 flash            # Windows
```

No CP2102, no driver install on Linux/Mac — the ESP32-S3's native USB handles it.

### Recording
1. Insert microSD (FAT32, up to 32GB), power on — OLED shows tuner
2. Press record button — red LED solid, OLED shows recording screen
3. Play
4. Press record again — file saved as `/recordings/YYYYMMDD_HHMMSS.wav`

### Managing Files
1. Scan QR code on OLED — phone joins `SmartGuitar-XXXX` WiFi instantly
2. Browser opens → `http://192.168.4.1`
3. Browse, play, download, delete

### Plugging In
- **Guitar amp:** bypass switch to passive
- **Speaker / hi-fi / PC / mixer:** bypass switch to preamp — line level, plug straight in
- **Both at once:** 1/4" main to amp, 3.5mm mic out to recording device

### OTA Updates
Open `http://192.168.4.1` → Settings → enter home WiFi credentials. Updates happen automatically on next boot near that network. New DSP effects unlock as OTA updates — no hardware change ever needed.

---

## Repository Structure

```
smartguitar/
├── firmware/
│   ├── main/
│   ├── components/
│   │   ├── yin/            # YIN pitch detection
│   │   ├── qrcodegen/      # QR code (Nayuki, single file)
│   │   ├── ssd1306/        # OLED driver
│   │   └── dsp/            # Effects chain (EQ, comp, reverb, looper)
│   ├── web/                # React app source → built into LittleFS
│   │   ├── src/
│   │   ├── package.json
│   │   └── vite.config.js
│   └── partitions.csv
├── hardware/
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

### v1 — Core (this release)

#### Schematic Progress (KiCad)
- [x] USB-C connector (HRO TYPE-C-31-M-12, 16P) — VBUS, D+/D−, CC1/CC2
- [x] TP4056 LiPo charger — PROG 2kΩ (500mA), TEMP pull-up, polyfuse F1
- [x] DW01A battery protection IC — OVP, UVLO, OCP, short circuit
- [x] AO4842 dual N-MOSFET protection FETs — back-to-back on B− rail
- [x] MT3608 boost converter — BATT → +9V, feedback 390kΩ/27kΩ 1%, SS34 diode, CD43 4.7µH inductor
- [x] TPS60403 charge pump — +9V → −9V, 4× 10µF ceramic caps
- [x] ESP32-S3-WROOM-1-N16R8 — USB_D+/D− (GPIO19/20), EN pull-up, BOOT button, IO45/IO46 strapping, PSRAM NC flags, UART_TX/RX labels
- [ ] MCP1700-3302 LDO — BATT → 3.3V digital rail
- [ ] MCP1700-5002 LDO — +9V → 5V for PCM1808
- [ ] PCM1808 24-bit ADC — I²S slave, MCLK/BCLK/LRCK from ESP32-S3
- [ ] NE5532 preamp — ±9V, DIP-8, gain stage + output buffer
- [ ] MicroSD card socket — SPI, 47Ω damping resistors
- [ ] SSD1306 OLED 128×64 — I²C
- [ ] Audio jacks — 1/4" main (bypass switch), 3.5mm mic out
- [ ] JST-SH expansion header (J6) + 3.5mm footswitch jack (J7) — looper v3
- [ ] Record button, LEDs, power switch
- [ ] Star ground — AGND/DGND planes, ferrite beads, pi-filters
- [ ] Test points — VBAT, +9V, −9V, +5V, 3V3, AGND, DGND, AUDIO_IN, AUDIO_OUT
- [ ] PCB layout
- [ ] Firmware: audio capture, SD recording, WiFi AP+STA
- [ ] Firmware: HTTP server, React UI served from PSRAM
- [ ] Firmware: WebSocket live status
- [ ] Firmware: YIN tuner, OLED UI, WiFi QR code
- [ ] Firmware: OTA update check
- [ ] DSP: 4-band EQ, compressor, noise gate
- [ ] React UI: file manager, in-browser playback, DSP controls
- [ ] Android app: auto-discovery, ExoPlayer, download

#### Key Design Decisions Made
- **TP4056 plain + separate DW01A/FS8205A** chosen over integrated ESDP8 — better threshold accuracy (±25mV), faster short circuit response (8µs), lower quiescent current (3µA vs 20µA), independent protection from charger
- **AO4842 SO-8 dual N-MOSFET** used as FS8205A replacement — 7.7A rated, 30V Vds, pin compatible
- **ESP32-S3 native USB** (GPIO19/20) eliminates CP2102 entirely — no bridge chip, no auto-reset transistors, simpler BOM
- **PSRAM (8MB octal)** inside N16R8 module enables full React SPA in RAM, complete DSP effects chain, SD-backed looper
- **MT3608 + TPS60403** for ±9V split supply — boost then charge pump inversion; pi-filter on +9V output before NE5532
- **CD43/CD54 SMD power inductor** (4.7µH, 4.3×4.3mm) for boost converter — Coilcraft XAL4040 footprint in KiCad
- **0603 resistors and capacitors** throughout for hand soldering; 0805 for bulk caps (10µF+); 1206 for polyfuse
- **USB-C HRO TYPE-C-31-M-12 connector** transplanted from donor TP4056 module — verified footprint match
- **Battery negative rail** routed through AO4842 back-to-back MOSFETs controlled by DW01A OD/OC pins
- **ADC charge state inferred** from VBAT voltage divider — no CHRG/STDBY LED pins needed

### v2 — Effects OTA
- [ ] DSP: chorus, flanger, tremolo
- [ ] DSP: Schroeder reverb with room/hall/plate presets
- [ ] DSP: convolution reverb + IR loader via web UI
- [ ] DSP: cabinet IR simulation

### v3 — Looper OTA
> The PCB includes all hardware for the looper from v1 — JST-SH expansion header (J6) and 3.5mm footswitch jack (J7) are populated on the board. No hardware change needed when v3 firmware ships.

- [ ] Looper firmware: SD-backed loop recording (no PSRAM consumed; loop length limited only by SD space)
- [ ] Looper controls: tap record → tap overdub → tap stop, hold to clear
- [ ] Looper secondary button: stop playback, undo overdub layer
- [ ] External button wiring: JST header routes to anywhere on guitar body
- [ ] Footswitch support: 3.5mm mono jack for hands-free pedal control
- [ ] Looper UI in React web app: waveform display, layer management
- [ ] MIDI output (3.5mm TRS-A)

**Looper button placement options (user choice, wired to J6):**
- Near volume/tone knobs — natural for picking hand thumb
- Upper bout — accessible without moving fretting position
- Strap button replacement — foot or hand accessible
- External footswitch via J7 — fully hands-free on stage

---

## Who This Is For

- Beginners who bought a cheap guitar kit and don't want to buy an amp
- Players who want to record directly into a DAW without an audio interface
- Performers who want to capture gigs without extra gear
- Tinkerers who want a smart guitar without paying premium prices
- Developers who want to build on an open platform

---

## License

Hardware: CERN-OHL-S v2
Firmware: MIT
Documentation: CC BY 4.0

---

## Contributing

PRs welcome. Open an issue before starting large changes.

---

*Built for the open-source guitar community. Plug into anything. Record everything. Shape your tone.*
