# Wireless Morse Code Transceiver System

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-Arduino%20Uno-00979D.svg)](https://www.arduino.cc/)
[![Transceiver](https://img.shields.io/badge/RF%20Module-nRF24L01%2B%20(2.4GHz)-orange.svg)](https://www.nordicsemi.com/)
[![Display](https://img.shields.io/badge/Display-16x2%20LCD%20(I2C%20%2F%20Parallel)-green.svg)](https://github.com/johnrickman/LiquidCrystal_I2C)

A dual-node, bidirectional 2.4 GHz wireless Morse code transceiver system built with Arduino Uno and Nordic nRF24L01+ radio frequency modules. 

The system accepts real-time tactile Morse code input (dots, dashes, and word delimiters), decodes the symbols into alphanumeric characters on-chip, visualizes the input and transmitted messages on a 16×2 character LCD, and communicates wirelessly over the 2.4 GHz ISM band.

![Dual Node System Demo](Reference_Images/Demo.jpg)

---

## Table of Contents

- **System Overview**
  - [Key Features](#key-features)
  - [Repository Structure](#repository-structure)
- **Hardware & Electrical**
  - [Hardware Requirements & Bill of Materials](#hardware-requirements--bill-of-materials)
  - [Wiring & Pinout Reference](#wiring--pinout-reference)
    - [I2C LCD Configuration (Recommended)](#1-i2c-lcd-build-recommended---draft-04)
    - [Parallel 4-Bit LCD Configuration](#2-parallel-4-bit-lcd-build-draft-03)
    - [nRF24L01+ Power & Decoupling Guidelines](#3-nrf24l01-power--decoupling-guidelines)
  - [Circuit Schematics](#circuit-schematics)
- **Morse Code Engine**
  - [Input Logic](#input-logic)
  - [Control Sequences & Macros](#control-sequences--macros)
  - [Supported Morse Character Set](#supported-morse-character-set)
- **Firmware Setup & RF Configuration**
  - [Prerequisites & Required Libraries](#1-prerequisites--required-libraries)
  - [I2C Address Identification](#2-identify-i2c-addresses)
  - [Flashing Node A and Node B](#3-flashing-node-a-and-node-b)
  - [RF Pipe Addressing Scheme](#4-rf-pipe-addressing-scheme)
- **Testing & Diagnostics**
  - [Diagnostic Features](#diagnostic-features)
  - [Running the Diagnostic Suite](#running-the-diagnostic)
- **Support & Reference**
  - [Troubleshooting Matrix](#troubleshooting)
  - [Roadmap](#roadmap)
  - [License & Authors](#license--authors)

---

## Key Features

- **Bidirectional Peer-to-Peer RF**: Two-way half-duplex communication between Node A and Node B using independent 40-bit pipe addressing (`00001` and `00002`).
- **Onboard Morse-to-ASCII Decoder**: Translates dot (`.`) and dash (`-`) sequences directly into standard uppercase alphabet (`A`–`Z`) and numerals (`0`–`9`).
- **Real-Time 16×2 Character Display**:
  - **Line 1 (`Txt:`)**: Displays accumulated text message (local and incoming).
  - **Line 2 (`In :`)**: Live visual feedback of current dots and dashes in the input buffer.
- **In-Band Control Sequences**:
  - `......` (6 consecutive dots): Trigger backspace (deletes previous character and transmits control code `47`).
  - `------` (6 consecutive dashes): Resets and clears the full message buffer.
- **Dedicated Hardware Controls**: Separate pushbuttons for Dot, Dash, Space/Send, and Backspace.
- **Visual Status Feedback**: Dedicated activity LED indicating transmission and reception events.
- **Dual Circuit Implementations**:
  - **I2C Architecture (PCF8574)**: Uses only 2 analog pins (`A4`/`A5`), freeing GPIOs and drastically simplifying breadboard wiring.
  - **Direct 4-Bit Parallel Architecture (HD44780)**: Native connection using GPIOs without requiring an auxiliary I2C backpack.
- **Hardware Diagnostic Suite**: Built-in test transmitter and receiver sketches utilizing channel 125, 250 kbps data rate, Auto-ACK with payload, carrier detect (RPD/RSSI), and AVR hardware watchdog timer (`wdt.h`).

---

## Repository Structure

```
Wireless-Morse/
├── Demo.mp4                                # Video demonstration of prototype operation
├── LICENSE                                 # MIT License
├── README.md                               # Project documentation
│
├── Morse_Transceiveres_I2C_LCD/            # Main I2C firmware implementation (Recommended)
│   ├── Diagram_I2C_LCD.jpg                 # Breadboard layout diagram
│   ├── Schematic_I2C_LCD.pdf               # Schematic diagram (PDF)
│   ├── Node_A_Draft_04/
│   │   └── Node_A_Draft_04.ino             # Firmware for Node A (I2C addr: 0x27, TX: 00002, RX: 00001)
│   └── Node_B_Draft_04/
│       └── Node_B_Draft_04.ino             # Firmware for Node B (I2C addr: 0x3F, TX: 00001, RX: 00002)
│
├── Morse_Transceiveres_LCD/                # Parallel 4-bit LCD firmware implementation
│   ├── Diagram_LCD.jpg                     # Breadboard layout diagram
│   ├── Schematic_LCD.pdf                   # Schematic diagram (PDF)
│   ├── Draft_03_NRF_Node_A/
│   │   └── Draft_03_NRF_Node_A.ino         # Node A parallel sketch
│   └── Draft_03_NRF_Node_B/
│       └── Draft_03_NRF_Node_B.ino         # Node B parallel sketch
│
├── NRF_Modules_Diagnosis/                  # Diagnostic utilities for link health and range testing
│   ├── NRF_Modules_Diagnosis_TX/
│   │   └── NRF_Modules_Diagnosis_TX.ino    # Diagnostic transmitter with retries and power monitoring
│   └── NRF_Modules_Diagnosis_RX/
│       └── NRF_Modules_Diagnosis_RX        # Diagnostic receiver with RPD/RSSI ACK payload response
│
├── Reference_Images/                       # Technical pinouts and bench references
│   ├── AS.jpeg                             # Assembled bench prototype photo
│   ├── ASCII.png                           # Standard ASCII reference table
│   ├── Demo.jpg                            # Dual breadboard prototype setup photo
│   ├── International_Morse_Code.png        # International Morse Code character chart
│   ├── LM1117_Pinout.png                   # LM1117-3.3V voltage regulator pinout
│   ├── nRF24L01_Pinout.png                 # nRF24L01 8-pin module pinout
│   └── UNO_Pinout.jpg                      # Arduino Uno pinout reference
│
└── Schematic_Diagrams/                     # Schematics and circuit layouts
    ├── I2C_LCD/
    │   ├── Diagram_I2C_LCD.jpg             # High-resolution I2C breadboard wiring layout
    │   ├── Schematic_I2C_LCD.pdf           # Circuit schematic in vector PDF format
    │   └── Schematic_I2C_LCD.png           # Circuit schematic in PNG format
    └── LCD/
        ├── 16x2_LCD_Pinout.jpg             # Standard 16-pin HD44780 pinout diagram
        ├── Diagram_LCD.jpg                 # High-resolution parallel LCD breadboard wiring layout
        ├── Schematic_LCD.pdf               # Circuit schematic in vector PDF format
        └── Schematic_LCD.png               # Circuit schematic in PNG format
```

---

## Hardware Requirements & Bill of Materials

To build a complete two-node wireless transceiver link, you will need components for **two identical transceiver stations**:

| Component | Quantity (Pair) | Specification / Notes |
|:---|:---:|:---|
| **Microcontroller** | 2 | Arduino Uno R3 (ATmega328P) |
| **RF Transceiver Module** | 2 | nRF24L01+ 2.4 GHz (Standard PCB antenna or PA+LNA with SMA antenna) |
| **Alphanumeric Display** | 2 | 16×2 Character LCD (HD44780 compatible) |
| **I2C LCD Backpack** | 2 | PCF8574 I2C adapter (Default address `0x27` or `0x3F`) |
| **Tactile Pushbuttons** | 8 | Momentary tactile pushbuttons (4 per node: Dot, Dash, Space, Back) |
| **Status Indicator** | 2 | 5mm LED (Green/Red) + 220Ω – 1kΩ current-limiting resistor |
| **Power Regulation (Optional but Recommended)** | 2 | LM1117-3.3V LDO regulator or dedicated nRF24L01 adapter board |
| **Decoupling Capacitors** | 2–4 | 10 µF electrolytic + 0.1 µF (100 nF) ceramic across nRF24 VCC/GND |
| **Potentiometer (Parallel build only)** | 2 | 10 kΩ / 250 kΩ linear potentiometer for LCD contrast |
| **Prototyping** | 2 | Solderless breadboards, jumper wire kit, USB Type-B cables |

---

## Wiring & Pinout Reference

### 1. I2C LCD Build (Recommended - Draft 04)

This configuration requires minimal wiring and is the primary implementation located in `Morse_Transceiveres_I2C_LCD/`.

#### A. nRF24L01+ Transceiver Wiring (Common to both nodes)

| nRF24L01 Pin | Arduino Uno Pin | Description / SPI Function |
|:---|:---|:---|
| **Pin 1 (GND)** | `GND` | Common Ground |
| **Pin 2 (VCC)** | `3.3V` (Regulated) | **3.0V – 3.6V ONLY** (Do NOT connect to 5V) |
| **Pin 3 (CE)**  | `D9` | Chip Enable (Active High) |
| **Pin 4 (CSN)** | `D10` | SPI Chip Select Not (Active Low) |
| **Pin 5 (SCK)** | `D13` | SPI Serial Clock |
| **Pin 6 (MOSI)**| `D11` | SPI Master Out Slave In |
| **Pin 7 (MISO)**| `D12` | SPI Master In Slave Out |
| **Pin 8 (IRQ)** | *Unconnected* | Maskable Interrupt (Optional) |

#### B. User Inputs & Display (Node A and Node B)

| Component | Pin / Terminal | Arduino Uno Pin | Mode / Connection |
|:---|:---|:---|:---|
| **Dash Button** | Switch Contact | `A0` | `pinMode(A0, INPUT_PULLUP)` (Other pin to GND) |
| **Dot Button** | Switch Contact | `A1` | `pinMode(A1, INPUT_PULLUP)` (Other pin to GND) |
| **Space / Send Button** | Switch Contact | `A2` | `pinMode(A2, INPUT_PULLUP)` (Other pin to GND) |
| **Backspace Button** | Switch Contact | `D8` | `pinMode(8, INPUT_PULLUP)` (Other pin to GND) |
| **Status LED** | Anode (+) | `D6` | In series with 220Ω–1kΩ resistor to GND |
| **I2C LCD Module** | `SDA` | `A4` (or SDA pin) | I2C Data line |
| **I2C LCD Module** | `SCL` | `A5` (or SCL pin) | I2C Clock line |
| **I2C LCD Module** | `VCC` | `5V` | Power supply for LCD and backlight |
| **I2C LCD Module** | `GND` | `GND` | System ground |

> **Note on I2C Addresses**:
> - **Node A**: Defaults to address `0x27` (`LiquidCrystal_I2C lcd(0x27, 16, 2);`).
> - **Node B**: Defaults to address `0x3F` (`LiquidCrystal_I2C lcd(0x3f, 16, 2);`).
> If your PCF8574 backpack uses a different address, adjust the constructor in the sketch accordingly.

---

### 2. Parallel 4-Bit LCD Build (Draft 03)

For setups without an I2C adapter (`Morse_Transceiveres_LCD/`):

| LCD Pin (HD44780) | Arduino Uno Pin | Description |
|:---|:---|:---|
| **Pin 1 (VSS)** | `GND` | Ground |
| **Pin 2 (VDD)** | `5V` | Power supply (+5V) |
| **Pin 3 (VO)**  | Potentiometer Center | Contrast adjustment |
| **Pin 4 (RS)**  | `A4` | Register Select |
| **Pin 5 (RW)**  | `GND` | Read/Write (tied to GND for Write) |
| **Pin 6 (E)**   | `A5` | Enable clock |
| **Pins 7–10 (D0–D3)** | *Unconnected* | Lower nibble unused in 4-bit mode |
| **Pin 11 (D4)** | `D5` | Data Bit 4 |
| **Pin 12 (D5)** | `D4` | Data Bit 5 |
| **Pin 13 (D6)** | `D3` | Data Bit 6 |
| **Pin 14 (D7)** | `D2` | Data Bit 7 |
| **Pin 15 (A / LED+)** | `5V` (via 220Ω resistor) | Backlight Anode |
| **Pin 16 (K / LED-)** | `GND` | Backlight Cathode |
| **Buttons (Parallel)** | `A0` (Dash), `A1` (Dot), `A2` (Space), `A3` (Back) | Active LOW (`INPUT_PULLUP`) |

---

### 3. nRF24L01+ Power & Decoupling Guidelines

The nRF24L01+ is sensitive to power supply noise and transient voltage drops:

1. **Supply Voltage**: The radio **must** be supplied with **3.0V – 3.6V**. Connecting VCC directly to 5V will permanently damage the radio chip. (Note: The SPI signal lines CE, CSN, SCK, MOSI are 5V tolerant on genuine Nordic chips).
2. **Current Spikes**: During packet transmission, peak currents can surge to 115 mA or more (especially on PA/LNA power-amplified models). The standard Arduino Uno onboard 3.3V LDO often fails to supply these transients, causing brownouts and packet corruption.
3. **Decoupling Capacitor**: Solder a **10 µF electrolytic capacitor** in parallel with a **0.1 µF (100 nF) ceramic capacitor** directly across the VCC and GND pins of the nRF24L01 module header.
4. **Dedicated Regulator**: For maximum stability and range, power the radio from the 5V rail through an external **LM1117-3.3V** regulator as shown in `Reference_Images/LM1117_Pinout.png`.

---

## Morse Code Engine & Special Commands

### Input Logic

1. **Entering Symbols**:
   - Pressing **Dot (`A1`)** appends `.` to the internal input buffer (`tonesBuffer`).
   - Pressing **Dash (`A0`)** appends `-` to the internal input buffer (`tonesBuffer`).
   - The current sequence is displayed on LCD line 2 (`In : ...-`).
2. **Committing Characters**:
   - Pressing **Space (`A2`)** triggers the translation lookup.
   - If the sequence matches an alphanumeric symbol in `symbolsAlphabet`, the character is appended to `text` and transmitted wirelessly as an ASCII value over the radio pipe.
   - If the sequence is empty when Space is pressed, a whitespace (` `) is appended.

### Control Sequences & Macros

The firmware includes dedicated command sequences entered directly via the Morse buttons:

| Sequence | Action | Firmware Handling |
|:---:|:---|:---|
| `......` (6 Dots) | **Backspace** | Deletes the last character in the buffer and transmits control character ASCII `47` (`/`) to the peer node. |
| `------` (6 Dashes) | **Clear All** | Clears the entire text display and resets the local buffer. |
| **Hardware Back Button** | **Instant Backspace** | Pressing the button on pin `8` (or `A3`) instantly drops the last character. |

### Supported Morse Character Set

```
A: .-     B: -...   C: -.-.   D: -..    E: .      F: ..-.   G: --.    H: ....
I: ..     J: .---   K: -.-    L: .-..   M: --     N: -.     O: ---    P: .--.
Q: --.-   R: .-.    S: ...    T: -      U: ..-    V: ...-   W: .--    X: -..-
Y: -.--   Z: --..
1: .----  2: ..---  3: ...--  4: ....-  5: .....  6: -....  7: --...  8: ---..
9: ----.  0: -----
```

---

## Firmware Setup & Flashing

### 1. Prerequisites & Required Libraries

Install the [Arduino IDE](https://www.arduino.cc/en/software) (version 1.8.x or 2.x). Open the Library Manager (**Sketch > Include Library > Manage Libraries...**) and install:

- **RF24** by TMRh20 (version 1.4.x or later)
- **LiquidCrystal_I2C** by Frank de Brabander / Marco Schwartz
- **LiquidCrystal** (Included with Arduino IDE, for parallel build)
- **SPI** (Standard Arduino Core library)

### 2. Identify I2C Addresses

Run an I2C scanner sketch on both Arduino boards to confirm the LCD I2C addresses. The default sketches use:
- **Node A**: `0x27`
- **Node B**: `0x3F`

If both LCDs share the same address (e.g. `0x27`), either solder the address jumpers (`A0`/`A1`/`A2`) on one PCF8574 board to change it to `0x26`/`0x25`, or update line 6 in `Node_B_Draft_04.ino` to match your hardware.

### 3. Flashing Node A and Node B

1. Connect the **Node A** Arduino board via USB.
2. Open `Morse_Transceiveres_I2C_LCD/Node_A_Draft_04/Node_A_Draft_04.ino`.
3. Select board **Arduino Uno** and the corresponding COM/tty serial port.
4. Click **Upload**.
5. Disconnect Node A and connect the **Node B** Arduino board.
6. Open `Morse_Transceiveres_I2C_LCD/Node_B_Draft_04/Node_B_Draft_04.ino`.
7. Click **Upload**.

### 4. RF Pipe Addressing Scheme

The nodes are preconfigured to establish a half-duplex communication tunnel:

```cpp
const byte addresses[][6] = { "00001", "00002" };

// Node A Configuration:
radio.openWritingPipe(addresses[1]);     // Transmits on "00002"
radio.openReadingPipe(1, addresses[0]);  // Listens on "00001"

// Node B Configuration:
radio.openWritingPipe(addresses[0]);     // Transmits on "00001"
radio.openReadingPipe(1, addresses[1]);  // Listens on "00002"
```

---

## RF Diagnostic Suite

Before operating the full Morse firmware, verify radio functionality and RF link quality using the diagnostic tools in `NRF_Modules_Diagnosis/`.

### Diagnostic Features
- **Module Detection**: Verifies SPI connectivity to nRF24L01 on boot (`radio.begin()`).
- **Low-Interference Channel 125**: Operates at 2.525 GHz (above standard 2.4 GHz Wi-Fi channels 1–13) to avoid ambient interference.
- **250 kbps High-Sensitivity Data Rate**: Optimizes receiver sensitivity (-94 dBm) for maximum transmission distance.
- **Dynamic Payloads & Auto-ACK**: Confirms bidirectional packet receipt using hardware acknowledgment payloads.
- **Carrier / Signal Detection (RPD/RSSI)**: Receiver measures carrier presence using `radio.testRPD()` and returns signal status inside ACK payloads.
- **AVR Watchdog Timer**: Uses `<avr/wdt.h>` (4-second timeout) to recover automatically if an SPI communication lockup occurs.

### Running the Diagnostic

1. Upload `NRF_Modules_Diagnosis/NRF_Modules_Diagnosis_TX/NRF_Modules_Diagnosis_TX.ino` to the first Arduino.
2. Upload `NRF_Modules_Diagnosis/NRF_Modules_Diagnosis_RX/NRF_Modules_Diagnosis_RX` to the second Arduino (ensure the file is named with `.ino` extension inside the Arduino IDE).
3. Open the Serial Monitor on both boards at **9600 baud**.
4. The TX unit will print transmission counters, retry counts, TX power levels, and returned RSSI status. The RX unit will display incoming test payloads and ACK responses.

---

## Circuit Schematics

### I2C LCD Configuration

| Breadboard Wiring Layout | Schematic Diagram |
|:---:|:---:|
| ![I2C Breadboard Diagram](Schematic_Diagrams/I2C_LCD/Diagram_I2C_LCD.jpg) | ![I2C Schematic](Schematic_Diagrams/I2C_LCD/Schematic_I2C_LCD.png) |

*A printable vector schematic is available at [Schematic_I2C_LCD.pdf](Schematic_Diagrams/I2C_LCD/Schematic_I2C_LCD.pdf).*

### Parallel 4-Bit LCD Configuration

| Breadboard Wiring Layout | Schematic Diagram |
|:---:|:---:|
| ![Parallel Breadboard Diagram](Schematic_Diagrams/LCD/Diagram_LCD.jpg) | ![Parallel Schematic](Schematic_Diagrams/LCD/Schematic_LCD.png) |

*A printable vector schematic is available at [Schematic_LCD.pdf](Schematic_Diagrams/LCD/Schematic_LCD.pdf).*

---

## Troubleshooting

| Issue | Likely Root Cause | Solution |
|:---|:---|:---|
| **`NRF24L01 not detected!` on Serial Monitor** | Loose SPI connections or incorrect pin mapping. | Verify SPI wiring: CE to D9, CSN to D10, SCK to D13, MOSI to D11, MISO to D12. Replace jumper wires. |
| **Transmissions fail / high retry rate** | 3.3V power supply instability or voltage sag during TX bursts. | Solder a 10 µF electrolytic capacitor + 0.1 µF ceramic capacitor across nRF24 VCC and GND pins. Use an external 3.3V LDO regulator (LM1117). |
| **LCD displays only black blocks or no text** | Incorrect I2C address or contrast setting. | 1. Run an I2C scanner sketch to determine whether the address is `0x27` or `0x3F`.<br>2. Adjust the blue potentiometer on the back of the I2C backpack using a screwdriver until characters are sharp. |
| **Buttons trigger multiple inputs or erratic dots/dashes** | Contact bounce or floating inputs. | Ensure internal pullups are active (`INPUT_PULLUP`). Check the 15ms–50ms debounce delays in `getInput()`. Verify buttons switch cleanly to GND. |
| **Nodes do not receive each other's messages** | Inverted pipe addresses or channel mismatch. | Confirm Node A writes to `00002` and reads `00001`; Node B writes to `00001` and reads `00002`. Ensure both run matching RF data rates and channels. |

---

## Roadmap

- [ ] **Acoustic Sidetone**: Add passive piezo buzzer PWM output for real-time auditory feedback during keying.
- [ ] **Multi-Node Mesh Networking**: Migrate from point-to-point addressing to `RF24Network` / `RF24Mesh` for group messaging.
- [ ] **Error Detection & Forward Correction**: Implement packet checksums and retransmission request protocols for noisy environments.
- [ ] **Portable Power Management**: Add Li-ion 18650 battery shielding, step-up/step-down regulation, and ATmega328P sleep modes.
- [ ] **Custom PCB & 3D Printed Enclosure**: Design a handheld transceiver enclosure with dedicated mechanical Morse paddles.

---

## License & Authors

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for complete details.

### Author
- Himanshu
- Email: himanshu.chauragade@gmail.com
