# SmartGuitar Module — Bill of Materials

**MCU:** ESP32-S3-WROOM-1-N16R8 (16MB flash, 8MB octal PSRAM, native USB)
**Design tool:** KiCad
**Target cavity:** LP/SG-style budget kit guitar (~100 × 65mm, ~30mm depth)

> **Key design goal:** The NE5532 preamp on ±9V brings the pickup to **line level**, allowing direct connection to any speaker, hi-fi, PC, or mixer without a guitar amp. The ESP32-S3-N16R8's 8MB PSRAM enables a full React web UI served from RAM, plus a complete real-time DSP effects chain — all on one PCB.

All packages SMD unless noted.

---

## 1. MCU

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U1 | ESP32-S3-WROOM-1-N16R8 | ESP32-S3-WROOM-1-N16R8 | 18×20mm castellated | 1 | Transplant from dev board. 16MB flash + 8MB octal PSRAM. Native USB on GPIO19/20 — no CP2102 needed. LX7 dual-core 240MHz with PIE vector extension. Antenna end must overhang PCB edge or sit over copper-free cutout |

---

## 2. USB (Native S3 USB — No Bridge Chip)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| J1 | USB-C connector | USB4125-GF-A or similar | SMD | 1 | D+/D− direct to GPIO20/GPIO19. VBUS → TP4056 via polyfuse |
| R1, R2 | USB D+/D− resistors | 27Ω | 0402 | 2 | Signal integrity on GPIO19/20 |
| R3, R4 | USB-C CC resistors | 5.1kΩ | 0402 | 2 | CC1 and CC2 — dumb charger compatibility |
| D1 | USB ESD protection | USBLC6-2SC6 | SOT-23-6 | 1 | Clamps GPIO19/20 and VBUS |
| F1 | Polyfuse | 500mA | 0805 | 1 | VBUS overcurrent before TP4056 |
| SW1 | BOOT button | Tactile SMD | 3×4mm | 1 | GPIO0 pull-down for manual bootloader entry |
| SW2 | EN / Reset button | Tactile SMD | 3×4mm | 1 | ESP32-S3 hard reset |
| R5 | EN pull-up | 10kΩ | 0402 | 1 | EN pin must not float |

> **No CP2102, no auto-reset transistors.** ESP32-S3 native USB handles flashing directly. `idf.py -p /dev/ttyACM0 flash` — no driver needed on Linux/Mac.

---

## 3. Power — Battery & Charging

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| BT1 | LiPo cell | 1S 1000–2000mAh | 103450 flat | 1 | Source separately; measure cavity before ordering |
| J2 | Battery connector | JST-PH 2-pin | SMD | 1 | Polarity on silkscreen |
| U2 | LiPo charger | TP4056 | SOP-8 | 1 | Charges via USB-C VBUS |
| R6 | Charge current | 2kΩ 1% | 0402 | 1 | PROG pin → 500mA |
| U3 | Cell protection | DW01A | SOT-23-6 | 1 | OVP / UVP / OCP |
| Q1 | Protection FET | FS8205A | SOT-23-6 | 1 | Paired with DW01A |
| D2 | Reverse polarity | BAT54S | SOT-23 | 1 | Battery connector mishap protection |
| C1, C2 | TP4056 caps | 10µF × 2 | 0805 | 2 | Input and output bulk |

---

## 4. Power — Boost (+9V for NE5532)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U4 | Boost converter | MT3608 or SX1308 | SOT-23-6 | 1 | VBAT → +9V |
| L1 | Boost inductor | 4.7µH Isat ≥1A | Bourns SRR1260 | 1 | Short traces to switch pin |
| D3 | Boost diode | SS34 | SMA | 1 | 3A 40V Schottky |
| R7, R8 | Feedback divider | 1% for 9V | 0402 | 2 | Vout = 1.25 × (1 + R7/R8) |
| C3–C6 | Boost caps | 22µF + 100nF × 2 sets | 0805/0402 | 4 | Input and output |

---

## 5. Power — Negative Rail (−9V for NE5532)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U5 | Charge pump inverter | TPS60403DBVT | SOT-23-5 | 1 | +9V → −9V, ~60mA |
| C7–C10 | Charge pump caps | 10µF ceramic × 4 | 0805 | 4 | Must be ceramic |

---

## 6. Power — 5V Rail (PCM1808 VCC)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U6 | 5V LDO | MCP1700T-5002E | SOT-23-3 | 1 | Drops from +9V; PCM1808 analog VCC |
| C11, C12 | LDO caps | 1µF × 2 | 0402 | 2 | Input + output |

---

## 7. Power — 3.3V Rail (Digital)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U7 | 3.3V LDO | MCP1700T-3302E | SOT-23-3 | 1 | Fed directly from BATT (3.0–4.2V). Only 178mV dropout so regulates correctly down to ~3.5V battery. 1.6µA quiescent current — critical for battery life. ESP32-S3 WiFi draws 200mA+ peaks; feeding from BATT avoids 1W+ heat dissipation that would occur if stepping down from +9V. Cannot use AMS1117-3.3 here — AMS1117 needs 4.6V minimum input (1.3V dropout + 3.3V) which a LiPo cannot provide at any charge state |
| C14 | LDO input cap | 1µF 0603 | 0603 | 1 | |
| C15 | LDO output cap | 1µF 0603 | 0603 | 1 | |
| C16 | LDO bulk output | 10µF 0805 | 0805 | 1 | Handles ESP32-S3 WiFi current transients |

---

## 8. Power — Soft Switch & Noise Filtering

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| Q2 | Power P-FET | DMG2305UX | SOT-23 | 1 | Main power gate on VBAT |
| SW3 | Power switch | DPDT mini slide | Panel | 1 | Through back cover |
| FB1 | Ferrite — analog | BLM21PG221SN1L | 0805 | 1 | VBAT → analog pi-filter |
| FB2 | Ferrite — digital | BLM21PG221SN1L | 0805 | 1 | 3V3 rail |
| FB3 | Ferrite — star GND | BLM21PG601SN1L | 0805 | 1 | AGND↔DGND star point |
| C15–C18 | Pi-filter caps | 10µF + 100nF × 2 sets | 0805/0402 | 4 | Either side of FB1 and FB2 |

---

## 9. Battery Monitoring

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| R9, R10 | Voltage divider | 100kΩ + 100kΩ | 0402 | 2 | VBAT ÷ 2 → ESP32-S3 ADC |
| C19 | ADC filter | 100nF | 0402 | 1 | Low-pass on divider output |

---

## 10. Audio Preamp — NE5532 (Line-Level Output)

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U8 | Dual op-amp | NE5532 | SOIC-8 | 1 | Stage 1: voltage gain. Stage 2: output buffer |
| C20 | Input coupling | 470nF film | 0805 film | 1 | AC couple guitar input; film preferred |
| R11 | Input load | 1MΩ | 0402 | 1 | High-impedance match for pickup |
| R_load | Input impedance | 1MΩ | 0603 | 1 | On pin 3 (+) non-inverting input to AGND — sets input impedance only, not gain |
| R_in | Gain denominator | 5kΩ 1% | 0603 | 1 | On pin 2 (−) to AGND — sets gain denominator. With 50kΩ pot: gain range 1× to 11× |
| R14 | Output series | 100Ω | 0402 | 1 | Cable drive + anti-pop |
| C21 | Output coupling | 10µF | 0805 | 1 | AC couple to output jack |
| C22, C23 | Supply bypass | 100nF × 2 | 0402 | 2 | V+ and V− pins |
| C24, C25 | Supply bulk | 10µF × 2 | 0805 | 2 | ±9V rails near IC |
| D4 | Input ESD | PRTR5V0U2X | SOT-363 | 1 | Guitar jack transient clamp |

---

## 11. Audio ADC — PCM1808

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U9 | 24-bit ADC | PCM1808PWR | TSSOP-14 | 1 | I²S slave; ESP32-S3 provides MCLK/BCLK/LRCK |
| C26 | VCOM bypass | 1µF | 0402 | 1 | Internal common-mode ref → AGND |
| C27 | VCOM bulk | 10µF | 0805 | 1 | |
| C28, C29 | Input filter | 100nF C0G × 2 | 0402 | 2 | Anti-aliasing; C0G mandatory |
| C30, C31 | VCC bypass | 100nF + 10µF | 0402/0805 | 2 | 5V analog supply |
| C32 | 3.3V bypass | 100nF | 0402 | 1 | Digital supply |
| R15, R16 | MD0, MD1 | 10kΩ pull-down × 2 | 0402 | 2 | GND → slave mode |
| R17 | FMT | 10kΩ pull-down | 0402 | 1 | GND → I²S 24-bit |

---

## 12. MicroSD Card

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| J3 | MicroSD socket | Molex 502570-0893 / GCT MEM2055 | SMD push-push | 1 | SPI mode |
| R18–R21 | SPI damping | 47Ω × 4 | 0402 | 4 | Signal integrity only — no level shifting needed |
| R22 | CS pull-up | 10kΩ | 0402 | 1 | CS idle-high |
| C33, C34 | SD VDD bypass | 100nF + 10µF | 0402/0805 | 2 | |

---

## 13. OLED Display — SSD1306 128×64

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| U10 | OLED | SSD1306 | 0.96" module or bare IC | 1 | I²C 0x3C; 3.3V; tuner, QR, status, recording |
| R23, R24 | I²C pull-ups | 4.7kΩ × 2 | 0402 | 2 | SDA + SCL; omit if module has built-in |
| C35 | OLED bypass | 100nF | 0402 | 1 | |

---

## 14. Audio Output, Bypass & Jacks

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| J4 | 1/4" main jack | Cliff FC68131 / Neutrik NYS229 | Panel | 1 | Switched stereo jack |
| J5 | 3.5mm mic out | CUI SJ-3523-SMT | SMD | 1 | Always-on line-level secondary output |
| SW4 | Bypass switch | DPDT mini slide | Panel | 1 | Pole 1: passive vs preamp routing. Pole 2: preamp input disconnect when passive |

---

## 15. Anti-Pop Mute

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| Q3 | N-MOSFET mute | 2N7002 | SOT-23 | 1 | Mutes output on power up/down |
| R25 | Gate resistor | 10kΩ | 0402 | 1 | |
| C36 | Mute delay | 10µF | 0805 | 1 | ~100ms mute release after power stable |

---

## 16. Controls & Indicators

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| SW5 | Record button | Tactile SMD | 3×4mm | 1 | Back cover; WAV record start/stop |
| LED1 | Red LED | 0402 | 0402 | 1 | Recording / low battery / error |
| LED2 | Green LED | 0402 | 0402 | 1 | Power / WiFi / OTA |
| R26, R27 | LED resistors | 330Ω | 0402 | 2 | |

---

## 17. Looper Expansion Header & Footswitch Jack *(v3 feature — populated on PCB, unused until v3 firmware)*

> The looper requires fast, hands-free button access during performance — the back cover button is not suitable. The JST header exposes GPIO lines so a button can be wired anywhere on the guitar body: near the volume knob, on the upper bout, at the strap button, or to an external footswitch. The 3.5mm footswitch jack allows a standard momentary pedal to control the looper completely hands-free.
>
> **All pads are on the PCB from v1.** No hardware modification needed when v3 firmware ships — just wire a button to J6 or plug a footswitch into J7.

| Ref | Component | Part | Package | Qty | Notes |
|-----|-----------|------|---------|-----|-------|
| J6 | Looper expansion header | JST-SH 6-pin 1mm pitch | SMD | 1 | Pin 1: 3V3 / Pin 2: GND / Pin 3: LOOP_BTN_1 / Pin 4: LOOP_BTN_2 / Pin 5: LOOP_LED / Pin 6: GND. Wire to button mounted anywhere on guitar body |
| J7 | Footswitch jack | 3.5mm mono SMD | SMD | 1 | Tip = LOOP_BTN_1, sleeve = GND. Standard TS momentary footswitch pedal plugs in directly |
| R28, R29 | Button pull-ups | 10kΩ × 2 | 0402 | 2 | LOOP_BTN_1 and LOOP_BTN_2 GPIO lines |
| R30 | LED current | 330Ω | 0402 | 1 | External looper status LED via J6 pin 5 |

**Looper button logic (v3 firmware):**

| Action | Result |
|--------|--------|
| Tap (idle) | Start recording loop to SD |
| Tap (recording) | Stop recording → start playback + overdub |
| Tap (overdubbing) | Stop overdub, keep playing |
| Hold 2s | Clear loop, return to idle |
| LOOP_BTN_2 tap | Stop playback immediately |
| LOOP_BTN_2 double tap | Undo last overdub layer |

**Mounting options for the external button:**
- Near volume/tone knobs — most natural for picking hand thumb
- Upper bout — accessible without moving fretting hand position
- Strap button replacement — foot or hand accessible
- External footswitch via J7 — fully hands-free during performance

**Loop storage:** streamed directly to/from SD card (`/loop/current.raw`) — no PSRAM consumed. Loop length limited only by SD space, not RAM. At 44.1kHz/16-bit mono, 1GB SD holds ~3 hours of loop.

---

## 18. General Decoupling

| Ref | Component | Value | Package | Qty | Notes |
|-----|-----------|-------|---------|-----|-------|
| C37–C56 | Bypass caps | 100nF X7R | 0402 | 20 | One per IC power pin minimum |
| C57–C62 | Bulk caps | 10µF | 0603 | 6 | Supply entry points |
| R31–R34 | Pull-up/down | 10kΩ | 0402 | 4 | GPIO bootstrap + button lines |
| R35 | GPIO45 tie | 0Ω or DNP | 0402 | 1 | GPIO45 → GND (VDD_SPI select) |
| R36 | GPIO46 pull | 10kΩ | 0402 | 1 | GPIO46 → GND (suppress ROM boot log) |

---

## 19. Test Points

| Ref | Net | Purpose |
|-----|-----|---------|
| TP1 | VBAT | Battery voltage after protection FET |
| TP2 | +9V | Boost output |
| TP3 | −9V | Charge pump output |
| TP4 | +5V | PCM1808 VCC |
| TP5 | 3V3 | Digital rail |
| TP6 | AGND | Analog ground star point |
| TP7 | DGND | Digital ground |
| TP8 | AUDIO_IN | Guitar signal post input coupling cap |
| TP9 | AUDIO_OUT | Preamp output post gain, pre jack switch |

---

## GPIO Allocation

| GPIO | Function | Notes |
|------|----------|-------|
| GPIO0 | BOOT button | Bootloader entry |
| GPIO1 | LOOP_BTN_1 | Looper main (v3); pull-up on PCB |
| GPIO2 | LOOP_BTN_2 | Looper secondary (v3); pull-up on PCB |
| GPIO3 | Record button | WAV record; check boot-strap state |
| GPIO4 | LED red | |
| GPIO5 | LED green | |
| GPIO6 | VBAT ADC | Voltage divider input |
| GPIO7 | Anti-pop mute | |
| GPIO8 | Bypass switch sense | |
| GPIO9 | SPI CS | SD card |
| GPIO10 | SPI MOSI | SD card |
| GPIO11 | SPI CLK | SD card |
| GPIO12 | I²S DIN | PCM1808 DOUT |
| GPIO13 | SPI MISO | SD card |
| GPIO15 | I²S LRCK (WS) | PCM1808 |
| GPIO16 | I²S BCLK | PCM1808 |
| GPIO17 | I²S MCLK (SCKI) | PCM1808 system clock |
| GPIO19 | USB D− | Native USB → USB-C |
| GPIO20 | USB D+ | Native USB → USB-C |
| GPIO21 | I²C SDA | SSD1306 OLED |
| GPIO22 | I²C SCL | SSD1306 OLED |
| GPIO48 | LOOP_LED | External looper status LED via J6 (v3) |
| GPIO45 | — | Tie to GND (VDD_SPI) |
| GPIO46 | — | Pull to GND (ROM log suppress) |

---

## Component Count Summary

| Category | Qty |
|----------|-----|
| ICs & modules | 11 |
| MOSFETs / transistors | 4 |
| Passives (R, C, L) | ~105 |
| Connectors & switches | 11 |
| Ferrite beads | 3 |
| LEDs | 2 |
| Test points | 9 |
| **Total** | **~145** |

---

## DSP & Memory Summary

| Resource | Total | Used | Free |
|----------|-------|------|------|
| Internal SRAM | 512KB | ~328KB | ~184KB |
| PSRAM | 8192KB | ~1120KB (v1 DSP + UI) | ~7072KB |
| Flash | 16MB | ~6MB firmware + UI | ~10MB LittleFS |

| Effect | PSRAM | CPU Core 1 | Ships in |
|--------|-------|-----------|---------|
| 4-band parametric EQ | 2KB | Trivial | v1 |
| Compressor | 2KB | Trivial | v1 |
| Noise gate | 1KB | Trivial | v1 |
| Chorus (3 voice) | 27KB | Easy | v2 OTA |
| Flanger | 8KB | Easy | v2 OTA |
| Tremolo | 1KB | Trivial | v2 OTA |
| Reverb (Schroeder) | 200KB | Manageable | v3 OTA |
| Reverb (convolution) | 176KB | OK with PIE | v3 OTA |
| Cabinet IR sim | 2KB | OK | v3 OTA |
| Looper (SD-backed) | 0KB | Minimal | v3 OTA |

All effects are hardware-ready from v1. They unlock via OTA — no PCB changes ever needed.

---

## Transplant List (from donor ESP32-S3 dev board)

| Component | Notes |
|-----------|-------|
| **ESP32-S3-WROOM-1-N16R8 module** | Only component needed. Hot air 270°C, generous flux. Verify N16R8 markings before desoldering |
| Everything else | Leave it. Native USB on S3 replaces CP2102 and all supporting circuitry |

---

## Sourcing

- **LCSC** — passives, ferrites, TP4056, MT3608, TPS60403, DW01A, FS8205A, MCP1700, JST-SH connectors
- **Mouser / DigiKey** — PCM1808, NE5532, USBLC6-2SC6, PRTR5V0U2X, ESP32-S3-WROOM-1-N16R8 (if buying fresh)
- **AliExpress** — LiPo cells (103450), JST-PH battery connector, panel jacks, slide switches, SSD1306 modules
- Most 0402 passives: LCSC in strips of 100 at <€0.01/unit

*Estimated BOM cost excluding module, cell, and PCB: ~€18–28*
