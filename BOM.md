# SmartGuitar Module — Bill of Materials

**PCB Target:** LP/SG-style budget guitar control cavity  
**MCU:** ESP32-WROOM-32 (transplanted from dev board)  
**Design tools:** KiCad  

All packages SMD unless noted. Quantities are per board.

---

## 1. MCU & Wireless

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U1 | ESP32-WROOM-32 | 4MB or 8MB flash variant | Module (18×20mm) | 1 | Transplant from dev board or buy fresh; 8MB preferred for OTA headroom |

---

## 2. USB & Flashing

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U2 | USB-UART bridge | CP2102 | QFN-28 | 1 | Transplant from dev board; or use CH340C (SOIC-16) as easier-to-solder replacement |
| Q1 | Auto-reset NPN | MMBT3904 | SOT-23 | 2 | Transplant from dev board; DTR→EN, RTS→GPIO0 auto-reset circuit |
| J1 | USB-C connector | USB4125-GF-A or similar | SMD | 1 | Dual-purpose: charging (VBUS→TP4056) + flashing (D+/D−→CP2102) |
| R1, R2 | USB D+ / D− resistors | 22Ω | 0402 | 2 | Signal integrity on USB data lines |
| R3, R4 | USB-C CC resistors | 5.1kΩ | 0402 | 2 | CC1 and CC2 for dumb charger compatibility |
| D1 | USB ESD protection | USBLC6-2SC6 | SOT-23-6 | 1 | VBUS + data line transient protection |
| F1 | Polyfuse | 500mA | 0805 | 1 | VBUS overcurrent protection before TP4056 |
| C1, C2 | CP2102 decoupling | 100nF + 10µF | 0402 / 0805 | 2 | VDD pin decoupling |
| SW1 | BOOT button | Tactile SMD | 3×4mm | 1 | GPIO0 pull-down for manual flash mode entry |
| SW2 | EN / Reset button | Tactile SMD | 3×4mm | 1 | ESP32 enable / reset |
| R5, R6 | Auto-reset resistors | 10kΩ | 0402 | 2 | Base resistors for auto-reset transistors |

---

## 3. Power — Battery & Charging

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| BT1 | LiPo cell | 1S 1000–2000mAh | 103450 flat cell | 1 | Source separately; verify fits cavity; JST-PH 2-pin connector |
| J2 | Battery connector | JST-PH 2-pin | Through-hole or SMD | 1 | Polarity marked on PCB |
| U3 | LiPo charger | TP4056 | SOP-8 | 1 | 500mA charge current |
| R7 | Charge current set | 2kΩ 1% | 0402 | 1 | PROG pin resistor — sets 500mA |
| U4 | Cell protection | DW01A | SOT-23-6 | 1 | OVP / UVP / OCP protection IC |
| Q3 | Protection FET | FS8205A dual MOSFET | SOT-23-6 | 1 | Paired with DW01A |
| D2 | Reverse polarity | BAT54S dual Schottky | SOT-23 | 1 | Battery wire reverse polarity protection |
| C3 | TP4056 input cap | 10µF | 0805 | 1 | |
| C4 | TP4056 output cap | 10µF | 0805 | 1 | |

---

## 4. Power — Boost Converter (VBAT → +9V)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U5 | Boost converter | MT3608 or SX1308 | SOT-23-6 | 1 | Up to 28V out; set feedback for 9V |
| L1 | Boost inductor | 4.7µH, Isat ≥1A | Bourns SRR1260 or similar | 1 | |
| D3 | Boost diode | SS34 Schottky | SMA | 1 | 3A 40V |
| R8, R9 | Boost feedback divider | Calculate for 9V out | 0402 1% | 2 | Use 1% tolerance |
| C5 | Boost input cap | 22µF | 0805 | 1 | |
| C6 | Boost input bypass | 100nF | 0402 | 1 | |
| C7 | Boost output cap | 22µF | 0805 | 1 | |
| C8 | Boost output bypass | 100nF | 0402 | 1 | |

---

## 5. Power — Negative Rail (−9V for NE5532)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U6 | Charge pump inverter | TPS60403DBVT | SOT-23-5 | 1 | +9V → −9V, ~60mA; ICL7660SCBAZ (SOIC-8) is easier to solder alternative |
| C9–C12 | Charge pump caps | 10µF / 6.3V ceramic | 0805 | 4 | Two input, two output; ceramic mandatory for charge pump |

---

## 6. Power — 5V Rail (for PCM1808 VCC)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U7 | 5V LDO | MCP1700T-5002E | SOT-23-3 | 1 | From +9V rail; low quiescent current; PCM1808 VCC only |
| C13 | LDO input cap | 1µF | 0402 | 1 | |
| C14 | LDO output cap | 1µF | 0402 | 1 | |

---

## 7. Power — 3.3V Rail (Digital)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U8 | 3.3V LDO | MCP1700T-3302E | SOT-23-3 | 1 | Feeds ESP32, PCM1808 digital, SD card, CP2102 |
| C15 | LDO input cap | 1µF | 0402 | 1 | |
| C16 | LDO output cap | 1µF | 0402 | 1 | |

---

## 8. Power — Soft Power Switch

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| Q4 | P-channel MOSFET | DMG2305UX | SOT-23 | 1 | Main power gate on VBAT rail |
| SW3 | Power slide switch | DPDT mini slide | Through-hole panel | 1 | Or use record button as soft power (hold = power, tap = record) |

---

## 9. Power — Noise Filtering

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| FB1 | Ferrite bead (analog) | BLM21PG221SN1L 220Ω@100MHz | 0805 | 1 | On VBAT rail feeding analog section |
| FB2 | Ferrite bead (digital) | BLM21PG221SN1L 220Ω@100MHz | 0805 | 1 | On 3V3 digital supply |
| FB3 | AGND–DGND bridge | BLM21PG601SN1L 600Ω@100MHz | 0805 | 1 | Single star point connection between ground planes |
| C17, C18 | Pi-filter caps (analog) | 10µF + 100nF | 0805 / 0402 | 2 | Either side of FB1 |
| C19, C20 | Pi-filter caps (digital) | 10µF + 100nF | 0805 / 0402 | 2 | Either side of FB2 |

---

## 10. Battery Monitoring

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| R10, R11 | Voltage divider | 100kΩ + 100kΩ | 0402 | 2 | VBAT ÷ 2 → ESP32 ADC pin |
| C21 | ADC filter cap | 100nF | 0402 | 1 | Low-pass on voltage divider output |

---

## 11. Audio Preamp (NE5532)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U9 | Dual op-amp | NE5532 | SOIC-8 | 1 | Stage 1: gain; Stage 2: output buffer |
| C22 | Input coupling cap | 470nF film | 0805 film | 1 | Guitar signal AC coupling; film preferred over ceramic for audio |
| R12 | Input load resistor | 1MΩ | 0402 | 1 | Pickup loading at preamp input |
| R13, R14 | Gain setting resistors | 10kΩ + 100kΩ | 0402 | 2 | Set stage 1 gain (~10×); make configurable with solder jumper |
| R15 | Output series resistor | 100Ω | 0402 | 1 | Anti-pop / short circuit protection on preamp output |
| C23 | Output coupling cap | 10µF | 0805 | 1 | AC couple preamp output to jack |
| C24, C25 | NE5532 supply bypass | 100nF × 2 | 0402 | 2 | On V+ and V− supply pins |
| C26, C27 | NE5532 bulk bypass | 10µF × 2 | 0805 | 2 | Bulk on ±9V rails near IC |
| D4 | Input ESD clamp | PRTR5V0U2X | SOT-363 | 1 | Guitar jack transient protection before preamp input |

---

## 12. Audio ADC (PCM1808)

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| U10 | 24-bit stereo ADC | PCM1808PWR | TSSOP-14 | 1 | I²S output; ESP32 is I²S master, PCM1808 is slave |
| C28 | VCOM bypass | 1µF | 0402 | 1 | Internal common-mode voltage; bypass to AGND |
| C29 | VCOM bulk | 10µF | 0805 | 1 | |
| C30, C31 | Input filter caps | 100nF C0G/NP0 × 2 | 0402 | 2 | Anti-aliasing before ADC inputs |
| C32 | PCM1808 VCC bypass | 100nF | 0402 | 1 | 5V analog supply pin |
| C33 | PCM1808 VCC bulk | 10µF | 0805 | 1 | |
| C34 | PCM1808 3.3V bypass | 100nF | 0402 | 1 | Digital supply pin |
| R16, R17 | Mode pins MD0, MD1 | 10kΩ pull-down | 0402 | 2 | Both to GND → PCM1808 slave mode |
| R18 | Format pin FMT | 10kΩ pull-down | 0402 | 1 | GND → I²S 24-bit format |

---

## 13. MicroSD Card

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J3 | MicroSD socket | Molex 502570-0893 or GCT MEM2055 | SMD push-push | 1 | SPI mode; push-push for retention |
| R19–R22 | SPI line damping | 47Ω × 4 | 0402 | 4 | MOSI, MISO, CLK, CS — signal integrity, NOT level shifting |
| R23 | CS pull-up | 10kΩ | 0402 | 1 | CS line idle-high pull-up |
| C35 | SD VDD bypass | 100nF | 0402 | 1 | |
| C36 | SD VDD bulk | 10µF | 0805 | 1 | |
| — | 74VHCT125A | **NOT USED** | — | 0 | ESP32-S3 is 3.3V native; level shifter from donor module not needed |

---

## 14. Audio Output & Bypass

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| J4 | 1/4" main output jack | Cliff FC68131 or Neutrik NYS229 | Panel mount | 1 | Switched stereo jack; switch contact used for passive/preamp bypass detection |
| J5 | 3.5mm mic out jack | CUI SJ-3523-SMT | SMD | 1 | Always-preamped secondary output |
| SW4 | Bypass switch | DPDT mini slide | Panel mount | 1 | Pole 1: switches signal source to jack; Pole 2: disconnects preamp input when passive |

---

## 15. Anti-Pop Mute Circuit

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| Q5 | Mute N-MOSFET | 2N7002 | SOT-23 | 1 | Mutes preamp output on power-up/down to prevent pop |
| R24 | Gate resistor | 10kΩ | 0402 | 1 | |
| C37 | Mute delay cap | 10µF | 0805 | 1 | RC delay on gate for timed mute release |

---

## 16. Controls & Indicators

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| SW5 | Record button | Tactile SMD | 3×4mm | 1 | Through back cover hole; pull-up, active low |
| LED1 | Status LED red | 0402 red | 0402 | 1 | Record active / error |
| LED2 | Status LED green | 0402 green | 0402 | 1 | Power on / WiFi connected / up to date |
| R25, R26 | LED resistors | 330Ω | 0402 | 2 | |

---

## 17. General Decoupling & Passives

| Ref | Component | Value / Part | Package | Qty | Notes |
|-----|-----------|-------------|---------|-----|-------|
| C38–C57 | Bypass caps | 100nF X7R | 0402 | 20 | One per IC power pin minimum |
| C58–C63 | Bulk caps | 10µF | 0603 | 6 | Strategic placement on supply rails |
| R27–R30 | Pull-up / pull-down | 10kΩ | 0402 | 4 | GPIO bootstrap and button lines |

---

## 18. Test Points

| Ref | Component | Notes |
|-----|-----------|-------|
| TP1 | VBAT | Battery voltage |
| TP2 | +9V | Boost output |
| TP3 | −9V | Charge pump output |
| TP4 | +5V | PCM1808 VCC rail |
| TP5 | 3V3 | Digital rail |
| TP6 | AGND | Analog ground star point |
| TP7 | DGND | Digital ground |
| TP8 | AUDIO_IN | Guitar signal after input coupling cap |

---

## Component Count Summary

| Category | Unique parts | Total qty |
|----------|-------------|-----------|
| ICs & modules | 12 | 12 |
| MOSFETs / transistors | 5 types | 7 |
| Passives (R, C, L) | ~30 values | ~90 |
| Connectors & switches | 8 | 8 |
| Ferrite beads | 2 values | 3 |
| LEDs | 2 | 2 |
| Test points | — | 8 |
| **Total** | | **~130** |

---

## Transplant List (from donor ESP32 dev board)

These components are desoldered from your existing dev board and reused:

| Component | Notes |
|-----------|-------|
| ESP32-WROOM-32 module | Hot air; 270°C; careful of antenna |
| CP2102 USB-UART | QFN-28; use flux generously |
| 2× MMBT3904 auto-reset transistors | SOT-23; easy |
| Associated resistors (auto-reset) | Check values with multimeter before reuse |
| EN + BOOT tactile buttons | Optional; buy fresh SMD ones instead (cheap) |
| AMS1117-3.3V LDO | **Skip** — replaced by your MCP1700 |

---

*Prices not included — source from LCSC, Mouser, or DigiKey. Most passives available in strips from LCSC at <€0.01/unit.*
