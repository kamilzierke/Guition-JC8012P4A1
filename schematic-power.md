# Power Architecture – Guition JC8012P4A1 (ESP32-P4)

This document describes the **complete power architecture** of the Guition JC8012P4A1 device, based on schematic file `1_PWR.png`.

<img width="1220" height="1093" alt="1_PWR" src="https://github.com/user-attachments/assets/a4f3cae5-254b-4930-a2df-d74b1d5f64c9" />

The power system is built around a **power-bank–style PMIC (IP5306)** and two downstream **buck converters (TLV62569)**. The design prioritizes hardware autonomy, robustness, and simplicity over software observability.

---

## 1. High-Level Power Topology

USB 5V ─┐
├─> IP5306 ── VOUT-BAT ──> 3.3V Buck ──> ESP32 / Peripherals
Li-Ion ──┘ └─> 1.2V Buck ──> ESP32-P4 HP Core


Key architectural rule:

> **The ESP32 never sees the battery directly.**  
> All power (USB or battery) flows through the IP5306 PMIC.

---

## 2. Battery Management – IP5306


### 2.1 Role of IP5306

The **IP5306** is a highly integrated PMIC commonly used in power banks. In this design it provides:

- Single-cell Li-Ion charging
- Battery protection
- Boost conversion (battery → VOUT)
- Automatic power-path management (USB vs battery)
- Low-power standby
- User power control via the **KEY** pin

The IP5306 is **not a smart fuel gauge** and exposes no digital telemetry.

---

### 2.2 Power-Path Behavior

| Condition        | System Power Source | Battery |
|------------------|---------------------|---------|
| USB present      | USB via IP5306      | Charging |
| USB absent       | Battery via boost   | Discharging |

Power-path switching is **fully autonomous** and handled internally by the IP5306.

---

## 3. Hardware Power Button (KEY)

### 3.1 Electrical Connection

IP5306 KEY ── R5 (10k) ── SW1 ── GND


- KEY has an internal pull-up inside IP5306
- Pressing SW1 pulls KEY low

---

### 3.2 Functional Behavior

The KEY pin controls the **internal state machine of IP5306**:

- **Short press**: enables / wakes VOUT
- **Long press**: disables VOUT (PMIC-level power-off)

This directly cuts power to all downstream rails.

---

### 3.3 Critical Design Property

- The switch is **not connected to the ESP32**
- The ESP32 **cannot detect** the button
- The ESP32 **cannot prevent shutdown**
- The button works even if firmware is crashed

This provides a **fail-safe hardware power-off**, but prevents graceful shutdown.

---

## 4. Battery Voltage Measurement

### 4.1 Measurement Method

Battery voltage is measured using a **passive resistor divider**:

- R2 = 68 kΩ (BAT+ → ADC)
- R6 = 100 kΩ (ADC → GND)
- ADC pin: **GPIO52**

Divider ratio:

Vadc = Vbat × (100 / (68 + 100)) ≈ 0.595
Multiplier ≈ 1.68


This keeps ADC voltage safely below 3.3 V at full charge (4.2 V).

---

### 4.2 Implications

- Voltage-only measurement
- No current sensing
- No coulomb counting
- No charging/discharging signal from hardware

All battery state logic must be **inferred in firmware**.

---

## 5. Battery State Estimation (Firmware-Level)

Because IP5306 exposes no telemetry:

- State of Charge (SOC) is estimated from voltage
- A piecewise Li-Ion OCV → SOC curve is used
- In this project:
  - **3.50 V is defined as critical (0%)**
  - 4.20 V clamps to 100%

Charging vs discharging is inferred from **SOC trend (%/hour)**, smoothed in software.

This is heuristic, but unavoidable without hardware changes.

---

## 6. Power Rails

### 6.1 VOUT-BAT

- Output of IP5306 boost stage
- Buffered by multiple bulk capacitors
- Feeds all downstream regulators

Voltage varies depending on load and mode.

---

### 6.2 3.3 V Rail (TLV62569 #1)

- Input: VOUT-BAT
- Output: 3.3 V
- Powers:
  - ESP32 I/O
  - Peripherals
  - VDDA

Standard synchronous buck topology.

---

### 6.3 1.2 V Rail (TLV62569 #2)

- Input: 3.3 V
- Output: ~1.2 V (ESP_VDD_HP)
- Powers ESP32-P4 high-performance core domain

Enabled via EN_DCDC.

---

## 7. Design Quirks and Trade-offs

### 7.1 No Fuel Gauge

- No SOC IC
- No current measurement
- Runtime estimation is statistical, not absolute

---

### 7.2 No Charging Status to MCU

- IP5306 LED pins unused
- KEY pin not routed to MCU
- ESP32 cannot detect:
  - USB presence
  - Charging active
  - Charge complete

One additional GPIO would solve this in a future revision.

---

### 7.3 Hard Power Cut

- Power button removes VOUT entirely
- MCU loses power instantly
- No firmware intervention possible

Robust, but software-hostile.

---

## 8. Overall Assessment

**Strengths**
- Electrically sound
- Crash-proof power control
- Simple and cost-effective

**Limitations**
- Software-opaque
- Heavily heuristic battery state
- Limited extensibility without PCB changes

This is a **classic consumer-device power design**: autonomous, robust, and inexpensive.

---

## 9. Possible Improvements (Future Revisions)

- Route IP5306 LED or USB-present signal to MCU
- OR KEY pin into a GPIO for graceful shutdown
- Add a dedicated fuel gauge (e.g. BQ27441)

---

**Schematic source:** `1_PWR.png`
