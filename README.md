# Task 02: STM32 Battery Management System (BMS) Front-End

#### Simulation Note:
Since STM32CubeIDE does **not provide ADC simulation** in the IDE:
- ADC values are simulated in firmware using incrementing/dummy arrays  
- This allows testing of filtering and protection logic  

## 1. Project Overview

This project implements a lightweight Battery Management System (BMS) firmware for monitoring a 4-cell Lithium-ion battery pack. The system continuously tracks individual cell voltages and temperature, and enforces safety limits using interrupt-driven protection logic.

### Core Features

- **Multi-Channel Sensing:** ADC + DMA used to monitor 4 battery cells efficiently  
- **Signal Conditioning:** 8-sample moving average filter for stable readings  
- **Safety-Critical Design:** <10 ms fault response using interrupt-based logic  
- **Telemetry:** UART output at 1 Hz with command-based interaction  

---

## 2. Hardware Design (Schematic)

The EasyEDA schematic (`BMS_Schematic.png`) represents a simple interface between the STM32 and external battery sensing components.

### Key Sections

#### Voltage Sensing
- 4 battery cells connected via resistor divider networks  
- Ensures voltages are scaled safely within the 0–3.3 V ADC input range  
- Example:
  - Lower cells: 10k / 10k divider (~1:2 scaling)
  - Higher cells: 47k / 10k divider (higher attenuation)

#### Temperature Sensing
- LM75A temperature sensor connected via I2C  
- Pins:
  - SDA → PB9  
  - SCL → PB8  
- Includes 4.7 kΩ pull-up resistors  

#### Protection Output
- GPIO pin (PA5) used as a fault indicator  
- Acts as:
  - LED indicator  
  - Or MOSFET gate control (active HIGH = system OK)

#### Communication Interface
- UART1 used for telemetry and commands  
- Pins:
  - TX → PA9  
  - RX → PA10  

---

## 3. Firmware Implementation Walkthrough

The firmware is structured around a **safety-first architecture**, where critical protection logic is handled using interrupts instead of relying on the main loop.

---

### A. Data Acquisition (The "Eyes")

- ADC1 configured in **scan mode** to read 4 channels  
- **DMA** transfers ADC data directly into memory without CPU overhead  

#### Processing Steps:
1. Raw ADC values are collected continuously  
2. Values are scaled to millivolts  
3. An 8-sample moving average filter is applied to smooth noise  

#### Simulation Note:
Since no hardware is used:
- ADC values are simulated using incrementing/dummy data arrays  
- This allows testing of filtering and protection logic  

---

### B. Protection Logic (The "Brain")

To achieve **fast response (<10 ms)**, protection checks are performed inside:

This runs every ~5 ms and evaluates:

- **Over-Voltage (OV):** Any cell > 4200 mV  
- **Under-Voltage (UV):** Any cell < 3000 mV  
- **Over-Temperature (OT):** Temperature > 45°C  

#### Fault Behavior:
- GPIO (PA5) is immediately turned OFF  
- A `fault_flag` is set (latched state)  
- System remains in FAULT until reset conditions are safe  

---

### C. Communication (The "Voice")

#### UART Telemetry

- Runs at 1 Hz  
- Outputs formatted system state:

#### Supported Commands

| Command | Action |
|--------|--------|
| `R` | Reset fault (only if safe) |
| `S` | Print current system status |
| `F` | Inject simulated fault |

- UART RX is interrupt-driven  
- No blocking calls used  

---

## 4. Runtime Walkthrough (Simulation)

Since no hardware is used, system behavior is described through expected runtime scenarios:

| Scenario | Trigger | System Response | UART Output |
|---------|--------|----------------|------------|
| Normal Start | Cells ~3.8V, Temp 25°C | LED ON | `STATE: OK` |
| Over-Voltage | Cell > 4.2V | LED OFF (<10ms), FAULT latched | `STATE: FAULT (OV)` |
| Over-Temp | Temp > 45°C | LED OFF, FAULT latched | `STATE: FAULT (OT)` |
| Reset (Safe) | Values normal + `R` | LED ON, FAULT cleared | `STATE: OK` |
| Reset (Unsafe) | Fault still present + `R` | No change | `STATE: FAULT` |

---

## 5. Pin Assignment Summary

| Pin | Function | Type |
|-----|--------|------|
| PA0–PA3 | ADC Inputs | Analog |
| PB8/PB9 | I2C1 (SCL/SDA) | Digital |
| PA9/PA10 | UART1 TX/RX | Communication |
| PA5 | Fault Output | GPIO |

---

## 6. Simulation Approach

Since physical hardware is not used:

- ADC values are simulated in firmware  
- I2C temperature readings are stubbed with fixed/incrementing values  
- System behavior is verified through:
  - UART logs  
  - Code walkthrough  
  - Logical validation of fault handling  

---

## 7. Build Status

- Project builds successfully in STM32CubeIDE  
- No errors or warnings  
- HAL-based implementation only (no Arduino)

---

## 8. Use of AI

AI tools were used during this task to assist with understanding and implementing STM32-based firmware, as this was a new platform and development environment for me. They were particularly helpful in exploring concepts such as ADC with DMA, interrupt-driven protection logic, and HAL-based peripheral configuration.

While many of the underlying concepts were familiar in theory, they had not been fully explored in practice. I used this project as an opportunity to work through the system architecture, follow the data flow, and understand how each subsystem—data acquisition, filtering, protection logic, and communication—interacts within the overall design.

This project ultimately served as a hands-on introduction to STM32 development and embedded safety-critical system design, and I made a deliberate effort to validate and understand the implementation rather than rely on generated code alone.

**Attachments**

 circuit diagram showing MCU pin connections  
 screenshot of STM32CubeIDE successful build  
