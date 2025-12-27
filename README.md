# Guition JC8012P4A1 – Hardware Schematic Documentation

This repository contains **reverse-engineered documentation** of the Guition JC8012P4A1 hardware, based on the original schematic image set.

The goal of this documentation is to provide:
- Clear subsystem-level explanations
- Engineering-grade analysis
- Identification of design trade-offs and quirks
- A foundation for firmware work, debugging, and future hardware revisions

---

## 📁 Schematic Files

Original schematic images:

| File | Description |
|----|----|
| `1_PWR.png` | Power supply and battery management |
| `2_LCD.png` | LCD panel and display interface |
| `3_ESP32-P4.png` | ESP32-P4 core connections |
| `4_CONN.png` | External connectors |
| `5_ESP32-C6.png` | ESP32-C6 coprocessor (WiFi/BLE) |
| `6_USB.png` | USB interface |
| `7_CODEC.png` | Audio codec and audio path |

---

## 📄 Documentation Index

### Power Subsystem
- [`schematic-power.md`](schematic-power.md)  
  Complete analysis of the power architecture, battery management, PMIC behavior, and all known quirks.

### (Planned)
- `schematic-lcd.md` – LCD, MIPI-DSI, backlight, touch
- `schematic-esp32-p4.md` – ESP32-P4 core rails, clocks, reset
- `schematic-esp32-c6.md` – ESP32-C6 coprocessor and SDIO
- `schematic-usb.md` – USB power and data path
- `schematic-audio.md` – Codec, I2S, speaker, microphone
- `schematic-connectors.md` – External IO and headers

---

## ⚠️ Important Notes

- This documentation is **not vendor-provided**
- Some behaviors are inferred from schematics and real-world testing
- Battery-related metrics are **heuristic**, not fuel-gauge accurate

---

## 🎯 Intended Audience

- Firmware developers (ESPHome / ESP-IDF)
- Hardware hackers and reverse engineers
- Anyone planning a modified or improved hardware revision

---

## 📌 Status

- Power subsystem: **documented**
- Remaining subsystems: **pending**

Contributions, corrections, and extensions are welcome.
