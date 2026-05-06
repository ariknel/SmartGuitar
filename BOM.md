# SmartGuitar Module — Complete Bill of Materials

**MCU:** ESP32-S3-WROOM-1-N16R8 (16MB flash, 8MB octal PSRAM, native USB)
**Design tool:** KiCad
**Target cavity:** LP/SG-style budget kit guitar (~100 × 65mm, ~30mm depth)
**Package standards:** Resistors = 0603, Caps 100nF = 0603, Caps 10µF = 0805, Caps 22µF = 0805, Caps 1µF = 0603

---

## 1. MCU

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U3 | ESP32-S3-WROOM-1-N16R8 | ESP32-S3-WROOM-1-N16R8 | 18×20mm castellated module | 1 | Transplant from dev board. 16MB flash + 8MB octal PSRAM. Native USB GPIO19/20. LX7 dual-core 240MHz with PIE vector extension |

---

## 2. USB-C Connector & Protection

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J2 | USB-C connector | HRO TYPE-C-31-M-12 | SMD 16P + 2× through-hole tabs | 1 | Transplant from donor TP4056 module. VBUS→charging, D+/D−→ESP32-S3 native USB |
| R1 | CC1 resistor | 5.1kΩ | 0603 | 1 | USB-C CC1 pull-down |
| R2 | CC2 resistor | 5.1kΩ | 0603 | 1 | USB-C CC2 pull-down |
| R8 | USB D− series | 27Ω | 0603 | 1 | Signal integrity on GPIO19 |
| R6 | USB D+ series | 27Ω | 0603 | 1 | Signal integrity on GPIO20 |
| R7 | Shield bleed | 1MΩ | 0603 | 1 | USB shield to GND via RC |
| C25 | Shield cap | 100nF | 0603 | 1 | RC filter on USB shield |
| F1 | Polyfuse | 500mA | 1206 | 1 | VBUS overcurrent protection before TP4056 |
| SW1 | BOOT button | Tactile SMD | 3×4mm | 1 | GPIO0 bootloader entry |

---

## 3. Battery Management & Charging

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J1 | Battery connector | JST-PH 2-pin | SMD | 1 | LiPo cell positive and negative |
| U1 | LiPo charger | TP4056 | SOP-8 | 1 | Transplant from donor module. Plain TP4056 — protection handled separately |
| R3 | Charge current | 2kΩ 1% | 0603 | 1 | PROG pin → 500mA charge current |
| R4 | TEMP pull-up | 10kΩ | 0603 | 1 | TEMP pin to VCC — disables thermal sensing |
| C5 | VCC bypass | 100nF | 0603 | 1 | TP4056 VCC pin decoupling |
| C6 | VCC bulk | 10µF | 0805 | 1 | TP4056 VCC bulk cap |
| C3 | BAT bypass | 100nF | 0603 | 1 | TP4056 BAT pin decoupling |
| C4 | BAT bulk | 10µF | 0805 | 1 | TP4056 BAT bulk cap |
| U2 | Cell protection IC | DW01A | SOT-23-6 | 1 | OVP 4.28V, UVLO 2.5V, OCP, short circuit 8µs, 3µA quiescent |
| Q1A, Q1B | Protection dual FET | AO4842 | SO-8 | 1 | Dual N-ch MOSFET, 7.7A, back-to-back on battery negative rail |
| R5 | DW01A VCC resistor | 100Ω | 0603 | 1 | Between VBAT and DW01A VCC pin |
| C1 | DW01A VCC bypass | 100nF | 0603 | 1 | DW01A VCC decoupling |
| C2 | DW01A CS bypass | 100nF | 0603 | 1 | CS pin bypass cap |

---

## 4. Power — Boost Converter (VBAT → +9V)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U4 | Boost converter | MT3608 | SOT-23-6 | 1 | VBAT → +9V regulated |
| L1 | Boost inductor | 4.7µH CD43/CD54 | Coilcraft XAL4040 footprint (4.3×4.3mm) | 1 | AliExpress search "CD43 4R7". Isat ≥1A |
| D1 | Boost diode | SS34 Schottky | SMA | 1 | 3A 40V |
| R11 | Feedback top | 390kΩ 1% | 0603 | 1 | Sets +9V output |
| R13 | Feedback bottom | 27kΩ 1% | 0603 | 1 | Sets +9V output |
| C10 | Boost input bypass | 100nF | 0603 | 1 | |
| C11 | Boost input bulk | 22µF | 0805 | 1 | |
| C12 | Boost output bulk | 22µF | 0805 | 1 | |
| C13 | Boost output bypass | 100nF | 0603 | 1 | |
| FB1 | Ferrite bead +9V | 600Ω@100MHz BLM21PG601SN1L | 0805 | 1 | On +9V rail before NE5532/TPS60403/AMS1117 |

---

## 5. Power — Negative Rail (+9V → −9V)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U5 | Charge pump inverter | TPS60403DBVT | SOT-23-5 | 1 | +9V → −9V for NE5532 V− |
| C15 | Charge pump input | 10µF ceramic | 0805 | 1 | Must be ceramic |
| C16 | Charge pump output | 10µF ceramic | 0805 | 1 | Must be ceramic |
| C14 | Flying capacitor | 10µF ceramic | 0805 | 1 | Between CFLY+ and CFLY− only |

---

## 6. Power — 5V Rail (PCM1808 VCC)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U7 | 5V LDO | AMS1117-5.0 | SOT-223 | 1 | Steps down from +9V → 5V. PCM1808 VCC only. ~40mW waste at 10mA — negligible |
| C20 | LDO input bulk | 10µF | 0805 | 1 | |
| C21 | LDO output bulk | 10µF | 0805 | 1 | |
| C22 | LDO output bypass | 100nF | 0603 | 1 | |

---

## 7. Power — 3.3V Rail (Digital)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U6 | 3.3V LDO | MCP1700T-3302E | SOT-23-3 | 1 | Fed from VBAT. 178mV dropout. 1.6µA quiescent. Feeds ESP32-S3, PCM1808 VDD, SD, OLED, X9C503S |
| C17 | LDO input | 1µF | 0603 | 1 | |
| C18 | LDO output bypass | 1µF | 0603 | 1 | |
| C19 | LDO output bulk | 10µF | 0805 | 1 | Handles ESP32-S3 WiFi current transients |
| C8 | 3.3V rail bypass | 100nF | 0603 | 1 | Near ESP32-S3 3V3 pin |
| C7 | 3.3V rail bulk | 10µF | 0805 | 1 | Near ESP32-S3 3V3 pin |
| C9 | EN pin bypass | 100nF | 0603 | 1 | ESP32-S3 EN pin to GND |

---

## 8. Power — Noise Filtering & Grounding

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| FB2 | Ferrite bead AGND/DGND | 600Ω@100MHz BLM21PG601SN1L | 0805 | 1 | Star ground — AGND+DGND junction to GND |
| FB3 | Ferrite bead PCM1808 VDD | 600Ω@100MHz BLM21PG601SN1L | 0805 | 1 | 3.3V → PCM1808 VDD only |

---

## 9. Battery Monitoring

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| R_bat1 | Voltage divider top | 100kΩ | 0603 | 1 | VBAT ÷ 2 → ESP32-S3 IO42 ADC |
| R_bat2 | Voltage divider bot | 100kΩ | 0603 | 1 | |
| C_bat | ADC filter | 100nF | 0603 | 1 | Low-pass on divider output |

---

## 10. Audio Preamp — NE5532

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U9 | Dual op-amp | NE5532DR | SOIC-8 SMD | 1 | U9A = gain stage (non-inverting), U9B = output buffer, U9C = power pins. Buy from LCSC (NE5532DR, ~€0.05) or Mouser — do NOT buy from AliExpress as cheap modules frequently contain relabeled inferior chips (TL072 etc.) that will degrade audio quality |
| C34in1 | Input coupling | 470nF film | 0805 film | 1 | AC coupling guitar input to pin 3 (+). Film mandatory |
| R18_IN1 | Input load | 1MΩ | 0603 | 1 | Pin 3 (+) to AGND — sets 1MΩ input impedance |
| R20 | Gain denominator | 5kΩ 1% | 0603 | 1 | Pin 2 (−) to AGND — sets gain scale. Gain = (R_fb/5k)+1 |
| R20_OUT1 | Output series | 100Ω | 0603 | 1 | Cable drive + anti-pop current limit on pin 7 output |
| C30 | V+ bypass | 100nF | 0603 | 1 | NE5532 pin 8 V+ to AGND |
| C31 | V+ bulk | 10µF | 0805 | 1 | +9V rail near pin 8 |
| C32 | V− bypass | 100nF | 0603 | 1 | NE5532 pin 4 V− to AGND |
| C33 | V− bulk | 10µF | 0805 | 1 | −9V rail near pin 4 |

---

## 11. Gain Control — X9C503S Digital Potentiometer

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U10 | Digital potentiometer | X9C503S (50kΩ) | SOIC-8 | 1 | Feedback resistor for NE5532 gain. VH→pin1 OUT, VW→pin2(−), VL→AGND. Non-volatile. Gain range 1× to 11× with R20=5kΩ. Transplant from existing module |
| R16 | X9C U/D series | 100Ω | 0603 | 1 | Series dampening on U/D control line |
| R19 | X9C CS series | 100Ω | 0603 | 1 | Series dampening on CS control line |
| R21 | X9C INC series | 100Ω | 0603 | 1 | Series dampening on INC control line |
| C34 | Wiper filter | 470pF C0G | 0603 | 1 | VH/RH to AGND — filters 850kHz charge pump noise |
| C35 | X9C VCC bypass | 100nF | 0603 | 1 | X9C503S VCC pin to AGND |

---

## 12. Signal Switching & Mute — 2N7002 MOSFETs

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| Q2 | Anti-pop mute | 2N7002 | SOT-23 | 1 | Shorts GUITAR_OUT to AGND for ~100ms on power-up. Gate RC: R24 10kΩ to 3.3V + C38 10µF to AGND |
| Q3 | Passive bypass | 2N7002 | SOT-23 | 1 | Routes raw pickup to JACK_TIP in passive mode. Gate: R23 100kΩ + C (10nF) + PASSIVE_EN |
| Q4 | Active switch | 2N7002 | SOT-23 | 1 | Routes preamp output to JACK_TIP in active mode. Gate: R22 100kΩ + C36 10nF + PREAMP_EN |
| Q5 | Software mute | 2N7002 | SOT-23 | 1 | Shorts NE5532 pin3(+) to AGND = complete silence. Kill switch / AGC. Gate: R17 10kΩ + C37 100nF + MUTE_EN |
| R22 | Q4 gate resistor | 100kΩ | 0603 | 1 | Anti-pop RC for active switch |
| R23 | Q3 gate resistor | 100kΩ | 0603 | 1 | Anti-pop RC for passive switch |
| R17 | Q5 gate resistor | 10kΩ | 0603 | 1 | Q5 mute gate |
| R24 | Q2 gate resistor | 10kΩ | 0603 | 1 | Q2 anti-pop gate pull-up |
| C36 | Q4 gate cap | 10nF | 0603 | 1 | Anti-pop smoothing |
| C_q3 | Q3 gate cap | 10nF | 0603 | 1 | Anti-pop smoothing |
| C37 | Q5 gate cap | 100nF | 0603 | 1 | Mute smoothing |
| C38 | Q2 gate cap | 10µF | 0805 | 1 | ~100ms power-up mute delay |

---

## 13. Audio ADC — PCM1808

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U8 | 24-bit stereo ADC | PCM1808PWR | TSSOP-14 | 1 | I²S slave. ESP32-S3 provides MCLK/BCLK/LRCK. VCC=5V, VDD=3.3V |
| C28 | VCC bypass | 100nF | 0603 | 1 | PCM1808 5V analog supply pin |
| C29 | VCC bulk | 10µF | 0805 | 1 | |
| C24 | VDD bypass | 100nF | 0603 | 1 | PCM1808 3.3V digital supply pin |
| C25 | VDD bulk | 10µF | 0805 | 1 | |
| C26 | VREF bypass | 1µF | 0603 | 1 | Internal reference to AGND |
| C27 | VREF bulk | 100nF | 0603 | 1 | |
| R15 | FMT pull-down | 10kΩ | 0603 | 1 | FMT→DGND: I²S 24-bit format |
| R16_pcm | MD0 pull-down | 10kΩ | 0603 | 1 | MD0→DGND: slave mode |
| R_md1 | MD1 pull-down | 10kΩ | 0603 | 1 | MD1→DGND: slave mode |

---

## 14. MicroSD Card

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J3 | MicroSD socket | Hirose DM3AT-SF-PEJM5 or equivalent | SMD push-push 9-pin | 1 | Transplant socket from donor blue SD module |
| R26 | SD CS damping | 47Ω | 0603 | 1 | Series on DAT3/CS line |
| R27 | SD MOSI damping | 47Ω | 0603 | 1 | Series on CMD/MOSI line |
| R28 | SD CLK damping | 47Ω | 0603 | 1 | Series on CLK line |
| R29 | SD MISO damping | 47Ω | 0603 | 1 | Series on DAT0/MISO line |
| R25 | SD CS pull-up | 10kΩ | 0603 | 1 | DAT3/CS idle-high pull-up to 3.3V |
| C39 | SD VDD bulk | 10µF | 0805 | 1 | SD card VDD decoupling |
| C40 | SD VDD bypass | 100nF | 0603 | 1 | |

---

## 15. OLED Display — SSD1306 128×64

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U11 | OLED controller | SSD1306 128×64 | 0.96" module or bare IC | 1 | I²C address 0x3C. 3.3V. Tuner, QR code, status, recording screen |
| R_sda | I²C SDA pull-up | 4.7kΩ | 0603 | 1 | Omit if module has built-in pull-ups |
| R_scl | I²C SCL pull-up | 4.7kΩ | 0603 | 1 | Omit if module has built-in pull-ups |
| C_oled | OLED VCC bypass | 100nF | 0603 | 1 | |

---

## 16. Audio Connectors & Jacks

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J4 | Guitar input jack | 1/4" mono jack | Panel mount | 1 | Guitar pickup input → GUITAR_IN net |
| J5 | Guitar output jack | 1/4" switched jack | Panel mount | 1 | JACK_TIP net. Switched contact detects cable insertion |
| J6 | AUX out jack | 3.5mm stereo vertical | Through-hole vertical | 1 | Always-on line-level from NE5532. Mounts vertically through back cover hole |

---

## 17. Controls & Indicators

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| SW5 | Record button | Tactile SMD | 3×4mm | 1 | Through back cover hole. Pull-up active low. REC_BTN → IO3 |
| LED1 | Red status LED | 0603 red | 0603 | 1 | Recording / low battery / error → IO40 |
| LED2 | Green status LED | 0603 green | 0603 | 1 | Power / WiFi / OTA → IO41 |
| R_led1 | LED1 resistor | 330Ω | 0603 | 1 | |
| R_led2 | LED2 resistor | 330Ω | 0603 | 1 | |

---

## 18. Looper Expansion Header *(v3 feature — populate now, use in v3)*

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J7 | Looper header | JST-SH 6-pin 1mm pitch | SMD | 1 | 3V3 / GND / LOOP_BTN1 / LOOP_BTN2 / LOOP_LED / GND |
| J8 | Footswitch jack | 3.5mm mono SMD | SMD | 1 | Tip=LOOP_BTN1, sleeve=GND |
| R_loop1 | BTN1 pull-up | 10kΩ | 0603 | 1 | |
| R_loop2 | BTN2 pull-up | 10kΩ | 0603 | 1 | |
| R_loop3 | LED current | 330Ω | 0603 | 1 | External looper LED |

---

## 19. ESP32-S3 Support Components

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| R9 | EN pull-up | 10kΩ | 0603 | 1 | EN pin to 3.3V |
| R10 | IO0 pull-up | 10kΩ | 0603 | 1 | BOOT button pull-up |
| R12 | IO46 pull-down | 10kΩ | 0603 | 1 | ROM log suppress |

---

## 20. Test Points

| Ref | Net | Purpose |
|-----|-----|---------|
| TP1 | VBAT | Battery voltage |
| TP2 | +9V | Boost output |
| TP3 | −9V | Charge pump output |
| TP4 | +5V | PCM1808 VCC |
| TP5 | 3V3 | Digital rail |
| TP6 | AGND | Analog ground star |
| TP7 | DGND | Digital ground |
| TP8 | GUITAR_IN | Pre-gain audio signal |
| TP9 | GUITAR_OUT | Post-gain preamp output |

---

## GPIO Allocation Summary

| GPIO | Net label | Function |
|------|-----------|---------|
| IO0 | BOOT | Bootloader entry button |
| IO3 | REC_BTN | Record button |
| IO4 | X9C_CS | Digipot chip select |
| IO5 | X9C_INC | Digipot increment |
| IO6 | X9C_UD | Digipot up/down |
| IO7 | MUTE_EN | Q5 software mute / kill switch |
| IO8 | PREAMP_EN | Q4 active mode switch |
| IO9 | PASSIVE_EN | Q3 passive mode switch |
| IO10 | SD_MOSI | SD card |
| IO11 | SD_CLK | SD card |
| IO12 | I2S_DIN | PCM1808 DOUT |
| IO13 | SD_MISO | SD card |
| IO15 | I2S_LRCK | PCM1808 LRCK |
| IO16 | I2S_BCLK | PCM1808 BCK |
| IO17 | I2S_MCLK | PCM1808 SCKI |
| IO19 | USB_D− | Native USB |
| IO20 | USB_D+ | Native USB |
| IO21 | I2C_SDA | OLED SDA |
| IO22 | I2C_SCL | OLED SCL |
| IO38 | SD_CS | SD card CS |
| IO40 | LED_RED | Red status LED |
| IO41 | LED_GREEN | Green status LED |
| IO42 | VBAT_ADC | Battery voltage monitor |
| IO45 | — | GND strap (VDD_SPI) |
| IO46 | — | Pull-down (ROM log) |
| TXD0 | UART_TX | Serial debug |
| RXD0 | UART_RX | Serial debug |

---

## Component Count Summary

| Category | Unique parts | Total qty |
|----------|-------------|-----------|
| ICs & modules | 11 | 11 |
| MOSFETs / transistors | 3 types | 5 |
| Resistors | ~25 values | ~55 |
| Capacitors | ~10 values | ~45 |
| Inductors | 1 | 1 |
| Connectors & switches | 9 | 9 |
| Ferrite beads | 1 value | 3 |
| LEDs | 2 | 2 |
| Test points | — | 9 |
| **Total** | | **~140** |

---

## Package Reference

| Component type | Package used |
|---------------|-------------|
| All resistors | **0603** |
| Caps 100nF | **0603** |
| Caps 470pF | **0603** |
| Caps 1µF | **0603** |
| Caps 10µF | **0805** |
| Caps 22µF | **0805** |
| Caps 10µF ceramic (charge pump) | **0805** |
| Polyfuse | **1206** |
| Ferrite beads | **0805** |
| LEDs | **0603** |
| NE5532DR | **SOIC-8 SMD** (3.9×4.9mm P1.27mm) |
| Boost inductor | **CD43/XAL4040 4.3×4.3mm SMD** |
| SS34 diode | **SMA** |
| AMS1117 | **SOT-223** |
| MCP1700 | **SOT-23-3** |
| TPS60403 | **SOT-23-5** |
| MT3608 | **SOT-23-6** |
| 2N7002 | **SOT-23** |
| AO4842 | **SO-8** |
| DW01A | **SOT-23-6** |
| X9C503S | **SOIC-8** |

---

## Transplant List (from donor boards)

| Component | Source | Notes |
|-----------|--------|-------|
| ESP32-S3-WROOM-1-N16R8 | ESP32-S3 dev board | Hot air 270°C, generous flux |
| TP4056 IC | TP4056 blue module | SOP-8, hot air |
| USB-C connector | TP4056 blue module | Through-hole tabs need care |
| MicroSD socket | Blue SD module | Hot air, check pad alignment |
| X9C503S | X9C503 module | SOIC-8, hot air |

---

## Sourcing

- **LCSC** — passives, ferrites, MT3608, TPS60403, DW01A, AO4842, MCP1700, AMS1117, PCM1808
- **Mouser / DigiKey** — NE5532, PCM1808 (alternative)
- **AliExpress** — LiPo cell, CD43 inductor, JST connectors, panel jacks, SSD1306 module, 2N7002
- **Donor modules** — ESP32-S3 dev board, TP4056 module, SD module, X9C503 module

*Estimated BOM cost excluding donor modules, LiPo cell, and PCB: ~€20–30*
