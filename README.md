    # STM32 PCB Design

**Microcontroller Board Based on STM32F103C8T6**

> Designed by **Paul Muyali** | Date: 2026-05-18 | Tool: KiCad EESchema / PCBnew v10.0

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Functional Description](#2-functional-description)
   - [2.1 Microcontroller — U2 (STM32F103C8T6)](#21-microcontroller--u2-stm32f103c8t6)
   - [2.2 Power Supply Subsystem — U1 (AMS1117-3.3)](#22-power-supply-subsystem--u1-ams1117-33)
   - [2.3 Analog Power Domain (+3.3VA)](#23-analog-power-domain-33va)
   - [2.4 USB Interface — J1](#24-usb-interface--j1)
   - [2.5 Crystal Oscillator — Y1](#25-crystal-oscillator--y1)
   - [2.6 BOOT0 Mode Selection — SW1, R2](#26-boot0-mode-selection--sw1-r2)
   - [2.7 SWD Debug Interface](#27-swd-debug-interface)
   - [2.8 Decoupling and Bypass Capacitor Strategy](#28-decoupling-and-bypass-capacitor-strategy)
   - [2.9 Status LED — D1, R1](#29-status-led--d1-r1)
   - [2.10 Expansion Headers — J2, J3, J4](#210-expansion-headers--j2-j3-j4)
3. [Bill of Materials](#3-bill-of-materials)
4. [Net List and Signal Assignments](#4-net-list-and-signal-assignments)
5. [PCB Layout Specifications](#5-pcb-layout-specifications)
   - [5.1 Board Physical Properties](#51-board-physical-properties)
   - [5.2 Routing Rules](#52-routing-rules)
   - [5.3 Component Placement Summary](#53-component-placement-summary)
6. [Design Considerations and Engineering Notes](#6-design-considerations-and-engineering-notes)
7. [Manufacturing Requirements](#7-manufacturing-requirements)
8. [Software and Firmware Development](#8-software-and-firmware-development)
9. [Revision History](#9-revision-history)
10. [References](#10-references)

---

## 1. Project Overview

This repository contains the complete KiCad design files for a compact, self-contained microcontroller board centred on the STMicroelectronics **STM32F103C8T6** ARM Cortex-M3 device. The board is intended as a general-purpose embedded development and evaluation platform, providing:

- USB Micro-B connectivity for power delivery and firmware download via the USB Full-Speed Device peripheral
- On-board 3.3 V linear power supply sourced directly from USB VBUS
- High-speed external (HSE) crystal oscillator for accurate system clocking
- SWD (Serial Wire Debug) interface for in-circuit programming and debugging
- BOOT0 SPDT switch for bootloader / normal boot mode selection
- Three 4-pin expansion headers exposing SWD, USART1, and I2C2 signals

The design is executed as a **two-copper-layer PCB** using predominantly SMD components in 0402 and 0603 package sizes, with through-hole connectors and switches for mechanical robustness. Four M2 mounting holes are provided at the board corners.

---

## 2. Functional Description

### 2.1 Microcontroller — U2 (STM32F103C8T6)

The primary processing element is the STM32F103C8T6, housed in a 48-pin LQFP package (7 × 7 mm, 0.5 mm pitch).

| Parameter | Specification |
|---|---|
| Core Architecture | ARM Cortex-M3, 32-bit RISC |
| Maximum Clock | 72 MHz |
| Flash Memory | 64 KB |
| SRAM | 20 KB |
| Supply Voltage | 2.0 V to 3.6 V |
| Operating Temperature | −40 °C to +85 °C |
| Digital I/O | 37 GPIO pins (select pins 5 V tolerant) |
| USB | Full-Speed USB 2.0 Device (12 Mbit/s) |
| USART | 3 channels (USART1 routed to J3) |
| I2C | 2 channels (I2C2 routed to J4) |
| SPI | 2 channels |
| Timers | 3 general-purpose, 1 advanced-control, 2 basic |
| ADC | 2 × 12-bit, up to 16 external channels |
| Debug Interface | SWD / JTAG |
| Package | LQFP-48, 7 × 7 mm, 0.5 mm pitch |

29 GPIO pins are designated as no-connect stubs in the schematic, indicating that they are available for user expansion via the three 4-pin headers (J2, J3, J4).

---

### 2.2 Power Supply Subsystem — U1 (AMS1117-3.3)

The board derives its operating power exclusively from the USB Micro-B connector (J1). The VBUS line (+5 V nominal) feeds the **AMS1117-3.3** low-dropout linear voltage regulator, which produces the 3.3 V rail consumed by all digital circuitry.

| Parameter | Specification |
|---|---|
| Device | AMS1117-3.3 |
| Package | SOT-223-3 (tab = output, Pin 2) |
| Input Voltage Range | Up to 15 V max. (1.1 V dropout at 800 mA) |
| Output Voltage | 3.3 V (fixed) |
| Maximum Output Current | 800 mA |
| Input Capacitor | C2: 22 µF (0805 SMD) |
| Output Capacitors | C1: 22 µF (0805 SMD), C3: 10 µF (0402 SMD) |

---

### 2.3 Analog Power Domain (+3.3VA)

Ferrite bead **FB1** (120 Ω at 100 MHz, 0603 package) isolates the +3.3V digital rail from the **+3.3VA** analog rail connected to the STM32 VDDA pin, reducing ADC noise. Capacitors **C9** and **C10** (1 µF each, 0402) provide local bypass on the analog domain.

---

### 2.4 USB Interface — J1

The Wuerth Elektronik 629105150521 USB Micro-B receptacle provides the physical USB interface. The D+ and D− differential pair signals are routed directly to **PA12** and **PA11** of the STM32, which natively implement the USB Full-Speed Device peripheral.

- **R3, R4** (1.5 kΩ each): Series resistors on D+ and D− for signal integrity and MCU I/O protection
- **R5** (1.5 kΩ): D+ pull-up resistor required by the USB Full-Speed specification to signal device presence to the host
- The USB ID pin is unconnected, which is the correct configuration for a USB Device-only (non-OTG) implementation

---

### 2.5 Crystal Oscillator — Y1

An external high-speed crystal oscillator (HSE) is provided for accurate clock generation. The crystal is mounted in a **3225 (3.2 × 2.5 mm) 4-pin SMD package** with ground pads on pins 2 and 4. Two 10 pF load capacitors (**C12**, **C13**, 0402) form the Pierce oscillator circuit in conjunction with the STM32 internal inverter amplifier.

> **Note:** The crystal oscillation frequency is not explicitly annotated in the schematic and must be specified at procurement. An **8 MHz** crystal is recommended for designs targeting the maximum 72 MHz system clock via the internal PLL (PLLMUL × 9).

---

### 2.6 BOOT0 Mode Selection — SW1, R2

**SW1** is an E-Switch EG1224 SPDT angled through-hole switch that controls the BOOT0 pin to select the MCU boot source at power-on or after a system reset.

| SW1 Position | BOOT0 Level | Boot Mode |
|---|---|---|
| Position A — GND | Logic 0 (via R2, 10 kΩ pull-down) | Normal Flash boot |
| Position B — +3.3V | Logic 1 | System bootloader (USB DFU) |

Setting SW1 to the +3.3V position and cycling power causes the STM32 to enumerate as a USB DFU device, enabling firmware loading via **STM32CubeProgrammer** or **dfu-util** without an external SWD programmer.

---

### 2.7 SWD Debug Interface

Serial Wire Debug (SWD) signals **SWDIO** (PA13) and **SWCLK** (PA14) are routed to expansion header **J2**, permitting in-circuit programming and real-time debugging with any SWD-capable probe (e.g., ST-LINK/V2, J-Link, Black Magic Probe). Operating in 2-wire SWD mode frees PA15, PB3, and PB4 from debug-only use.

---

### 2.8 Decoupling and Bypass Capacitor Strategy

| Designator(s) | Value | Package | Function |
|---|---|---|---|
| C4, C5, C6, C7, C11 | 100 nF | 0402 | High-frequency bypass on each VDD/VDDA pin group |
| C8 | 10 nF | 0402 | Additional high-frequency bypass |
| C9, C10 | 1 µF | 0402 | Mid-frequency bypass on analog supply |
| C12, C13 | 10 pF | 0402 | Crystal oscillator load capacitors |
| C1, C2 | 22 µF | 0805 | Bulk storage on LDO input and output |
| C3 | 10 µF | 0402 | Secondary bulk, LDO output |

---

### 2.9 Status LED — D1, R1

A 0603 SMD LED (**D1**) is provided as a user-controllable visual indicator. The anode connects to +3.3V and the cathode drives through current-limiting resistor **R1** (1.5 kΩ, 0402) to the MCU GPIO net `/PWR_LED_K`. At 3.3 V supply with a typical LED forward voltage of 2.0 V, the forward current is approximately **0.87 mA**, well within STM32 GPIO drive capability.

---

### 2.10 Expansion Headers — J2, J3, J4

Three 4-pin vertical pin socket headers (2.54 mm pitch) expose key MCU peripheral signals.

| Connector | Pin 1 | Pin 2 | Pin 3 | Pin 4 |
|---|---|---|---|---|
| J2 — SWD Debug | SWDIO (PA13) | SWCLK (PA14) | +3.3V | GND |
| J3 — UART | USART1_TX (PA9) | USART1_RX (PA10) | +3.3V | GND |
| J4 — I2C | I2C2_SCL (PB10) | I2C2_SDA (PB11) | +3.3V | GND |

---

## 3. Bill of Materials

| Designator(s) | Qty | Value / Part | Package / Footprint | Description |
|---|---|---|---|---|
| C1, C2 | 2 | 22 µF | C_0805_2012Metric | Bulk decoupling, LDO output/input |
| C3 | 1 | 10 µF | C_0402_1005Metric | Secondary bulk decoupling |
| C4, C5, C6, C7, C11 | 5 | 100 nF | C_0402_1005Metric | MCU VDD/VDDA high-freq bypass |
| C8 | 1 | 10 nF | C_0402_1005Metric | USB D+ filter / additional bypass |
| C9, C10 | 2 | 1 µF | C_0402_1005Metric | Analog supply bypass |
| C12, C13 | 2 | 10 pF | C_0402_1005Metric | Crystal load capacitors |
| D1 | 1 | LED | LED_0603_1608Metric | Power / status indicator LED |
| FB1 | 1 | 120 Ω @ 100 MHz | L_0603_1608Metric | Ferrite bead, analog supply isolation |
| H1–H4 | 4 | M2 | MountingHole_2.2mm_M2 | PCB mounting holes (non-electrical) |
| J1 | 1 | USB Micro-B | USB_Micro-B_Wuerth_629105150521 | USB 2.0 Micro-B, power and data |
| J2, J3, J4 | 3 | 1×04 Pin Header 2.54 mm | PinSocket_1x04_P2.54mm_Vertical | I/O expansion connectors |
| R1, R3, R4, R5 | 4 | 1.5 kΩ | R_0402_1005Metric | USB D+/D− series / pull-up, LED current limit |
| R2 | 1 | 10 kΩ | R_0402_1005Metric | BOOT0 pull-down resistor |
| SW1 | 1 | SPDT | SW_E-Switch_EG1224_SPDT_Angled | BOOT0 mode selection switch |
| U1 | 1 | AMS1117-3.3 | SOT-223-3_TabPin2 | 3.3 V LDO linear voltage regulator |
| U2 | 1 | STM32F103C8T6 | LQFP-48_7x7mm_P0.5mm | ARM Cortex-M3 microcontroller |
| Y1 | 1 | Crystal (3225, 4-pin, GND24) | Crystal_SMD_3225-4Pin_3.2x2.5mm | HSE crystal oscillator |

---

## 4. Net List and Signal Assignments

| Net Name | Description | Source / Driver | Connected To |
|---|---|---|---|
| `VBUS` | USB VBUS (+5 V from host) | J1 pin 1 | U1 input, SW1 |
| `+3.3V` | Digital 3.3 V supply rail | U1 output | U2 VDD, all digital ICs |
| `+3.3VA` | Analog 3.3 V supply rail | FB1 output | U2 VDDA |
| `GND` | Common ground | J1 GND | All components |
| `/NRST` | MCU active-low reset | U2 NRST | C11 (reset filter cap) |
| `/BOOTO` | BOOT0 logic level | SW1 wiper | U2 BOOT0 |
| `/SW_BOOTO` | BOOT0 switch node | SW1 | R2 pull-down |
| `/HSE_IN` | Crystal oscillator input | Y1 pin 1 | U2 PC14 / OSC_IN |
| `/HSE_OUT` | Crystal oscillator output | Y1 pin 3 | U2 PC15 / OSC_OUT |
| `/SWDIO` | SWD data I/O | U2 PA13 | J2 debug header |
| `/SWCLK` | SWD clock | U2 PA14 | J2 debug header |
| `/USART1_TX` | UART1 transmit | U2 PA9 | J3 header |
| `/USART1_RX` | UART1 receive | U2 PA10 | J3 header |
| `/I2C2_SCL` | I2C2 clock | U2 PB10 | J4 header |
| `/I2C2_SDA` | I2C2 data | U2 PB11 | J4 header |
| `/USB_D+` | USB differential data positive | J1 pin 3 | U2 PA12, R3 series |
| `/USB_D-` | USB differential data negative | J1 pin 2 | U2 PA11, R4 series |
| `/PWR_LED_K` | LED cathode net | D1 cathode | R1 current limit |

---

## 5. PCB Layout Specifications

### 5.1 Board Physical Properties

| Parameter | Value |
|---|---|
| Board Dimensions | ~34.5 × 28 mm (mounting hole span) |
| Board Thickness | 1.6 mm |
| Copper Layers | 2 (F.Cu — signal, B.Cu — power/ground) |
| Copper Thickness | 35 µm (1 oz) per layer |
| Dielectric Material | FR4, εᵣ = 4.5, tan δ = 0.02 |
| Solder Mask | LPI, both sides, tented vias |
| Mounting Holes | 4 × M2 (2.2 mm drill, non-plated), corner-mounted |
| Mounting Hole Coordinates | H1: (82.0, 60.5) mm, H2: (47.5, 60.5) mm, H3: (47.5, 88.5) mm, H4: (82.0, 88.5) mm |

---

### 5.2 Routing Rules

| Rule | Value |
|---|---|
| Signal track width | 0.3 mm |
| Power track width | 0.5 mm |
| Via outer diameter | 0.7 mm |
| Via drill diameter | 0.3 mm |
| Total routed segments | 178 |
| Total vias | 22 |

All signal routing is performed on **F.Cu** (front copper). The **B.Cu** (back copper) layer is designated as the power/ground layer. Via tenting is applied to both sides.

---

### 5.3 Component Placement Summary

- **U2 (STM32F103C8T6):** Board centre at (62.8, 74.3) mm — facilitates short routing to all peripheral signals
- **U1 (AMS1117-3.3):** Adjacent to J1 to minimise VBUS trace length
- **J1 (USB Micro-B):** Board-edge mounted for external cable access
- **Y1 (Crystal):** Placed close to STM32 PC14/PC15 pins to minimise oscillator loop area
- **C12, C13 (Crystal load caps):** Located immediately adjacent to Y1 and relevant MCU pins
- **C1, C2 (22 µF bulk):** Positioned at LDO input and output nodes
- **C4–C11 (decoupling caps):** Distributed around U2 supply pins per MCU datasheet guidelines
- **SW1 (BOOT0 switch):** Board edge, through-hole for mechanical accessibility
- **J2, J3, J4 (expansion headers):** Board edges for external connector access
- **H1–H4 (mounting holes):** Corner-mounted at 34.5 × 28 mm span

---

## 6. Design Considerations and Engineering Notes

### 6.1 USB Signal Integrity

- Series resistors R3 and R4 (1.5 kΩ) damp transmission line reflections and limit over/undershoot at the MCU I/O pins
- D+ and D− traces must be routed as a matched-length differential pair
- Pull-up resistor R5 (1.5 kΩ) on D+ is required by the USB Full-Speed specification to signal device presence to the host
- The USB ID pin is correctly left unconnected for a Device-only (non-OTG) implementation

### 6.2 Crystal Oscillator Load Capacitance

The effective series load capacitance seen by the crystal is:

```
C_L = (C12 × C13) / (C12 + C13) + C_stray
    = (10 pF × 10 pF) / (10 pF + 10 pF) + C_stray
    = 5 pF + C_stray
```

Where `C_stray` is the parasitic capacitance of the PCB traces and MCU pins (typically 2–5 pF for short traces). For a crystal specified at C_L = 10 pF, the values of C12 and C13 may require adjustment based on measured oscillation accuracy.

### 6.3 LDO Stability with Ceramic Output Capacitors

The AMS1117 series requires an output capacitor ESR in the range of 22 mΩ to several hundred milliohms for guaranteed stability. The 22 µF MLCC (C1, 0805) presents a very low ESR (typically < 10 mΩ). Verification against the AMS1117 stability specification is recommended. A tantalum or electrolytic capacitor may be added in parallel with C1 if instability is observed.

### 6.4 BOOT0 and USB DFU Bootloader

To enter DFU bootloader mode:
1. Set SW1 to the **+3.3V** position (BOOT0 = 1)
2. Cycle USB power (disconnect and reconnect)
3. The STM32 will enumerate as a USB DFU device on the host
4. Load firmware using STM32CubeProgrammer or `dfu-util`
5. Return SW1 to the **GND** position before the next reset to boot from Flash

### 6.5 Unused GPIO Pins

Per STM32 application note AN4899, all unused GPIO pins should be configured as **analog inputs** (`GPIO_MODE_ANALOG`) in firmware to minimise power consumption and prevent floating input oscillation.

### 6.6 EMC Considerations

- Ferrite bead FB1 (120 Ω at 100 MHz) isolates digital switching noise from the VDDA analog domain
- Distributed 100 nF bypass capacitors provide low-impedance bypassing at each MCU supply pin group
- Crystal placement close to OSC pins minimises the radiating loop area
- The USB connector shield is unconnected from GND in this revision. For commercial product compliance, a 4.7 nF / 100 V capacitor from shield to GND is recommended for ESD and EMC purposes

---

## 7. Manufacturing Requirements

### 7.1 PCB Fabrication

| Requirement | Specification |
|---|---|
| Board Material | FR4, Tg ≥ 130 °C |
| Board Thickness | 1.6 mm ± 0.1 mm |
| Copper Layers | 2 |
| Minimum Track Width | 0.3 mm |
| Minimum Track Clearance | 0.2 mm |
| Via Drill | 0.3 mm |
| Via Pad | 0.7 mm |
| Minimum Annular Ring | 0.2 mm |
| Solder Mask | LPI, green (both sides) |
| Silkscreen | White (both sides) |
| Surface Finish | HASL (lead-free) or ENIG |
| Workmanship Standard | IPC Class 2 |

### 7.2 Assembly Notes

- All SMD components are placed on the **F.Cu (top) side**
- Through-hole components (J2, J3, J4, SW1, H1–H4) are placed on the top side and soldered from the bottom
- Reflow soldering per IPC/JEDEC J-STD-020 for SMD components; manual or selective soldering for through-hole
- **U2 (LQFP-48, 0.5 mm pitch):** Use a 0.10–0.12 mm stencil with ~15% aperture reduction to prevent solder bridging
- **Y1 (3225, 4-pin):** Ensure GND pads (pins 2 and 4) are soldered to a low-impedance ground connection
- **U1 (SOT-223):** The exposed tab (pin 2) must be soldered to the PCB thermal pad

### 7.3 Test and Inspection

- [ ] Visual inspection per IPC-A-610 Class 2
- [ ] Apply USB power and verify +3.3V rail within 3.3 V ± 2%
- [ ] Verify LED D1 illuminates upon power-on (firmware-dependent)
- [ ] Connect SWD probe to J2 and confirm MCU identification
- [ ] Enumerate board as USB device and confirm host recognition
- [ ] Toggle SW1 and verify BOOT0 pin logic level transitions

---

## 8. Software and Firmware Development

### 8.1 Toolchain

| Tool | Recommendation |
|---|---|
| IDE | STM32CubeIDE |
| HAL / Drivers | STM32CubeF1 (HAL / LL for STM32F1 series) |
| Compiler | GCC ARM Embedded |
| SWD Programming | STM32CubeProgrammer or ST-LINK Utility |
| DFU Programming | STM32CubeProgrammer or `dfu-util` |

### 8.2 Recommended Clock Configuration (8 MHz HSE → 72 MHz SYSCLK)

```c
// RCC configuration for maximum performance
// HSE = 8 MHz crystal (Y1)
// SYSCLK = 72 MHz via PLL

RCC_OscInitStruct.OscillatorType      = RCC_OSCILLATORTYPE_HSE;
RCC_OscInitStruct.HSEState            = RCC_HSE_ON;
RCC_OscInitStruct.HSEPredivValue      = RCC_HSE_PREDIV_DIV1;
RCC_OscInitStruct.PLL.PLLState        = RCC_PLL_ON;
RCC_OscInitStruct.PLL.PLLSource       = RCC_PLLSOURCE_HSE;
RCC_OscInitStruct.PLL.PLLMUL          = RCC_PLL_MUL9;   // 8 MHz × 9 = 72 MHz

RCC_ClkInitStruct.ClockType           = RCC_CLOCKTYPE_SYSCLK |
                                         RCC_CLOCKTYPE_HCLK   |
                                         RCC_CLOCKTYPE_PCLK1  |
                                         RCC_CLOCKTYPE_PCLK2;
RCC_ClkInitStruct.SYSCLKSource        = RCC_SYSCLKSOURCE_PLLCLK;
RCC_ClkInitStruct.AHBCLKDivider       = RCC_SYSCLK_DIV1;   // HCLK  = 72 MHz
RCC_ClkInitStruct.APB1CLKDivider      = RCC_HCLK_DIV2;     // PCLK1 = 36 MHz (max)
RCC_ClkInitStruct.APB2CLKDivider      = RCC_HCLK_DIV1;     // PCLK2 = 72 MHz

// Flash latency: 2 wait states required for SYSCLK > 48 MHz
__HAL_FLASH_SET_LATENCY(FLASH_LATENCY_2);
```

### 8.3 USB Peripheral Notes

- USB pins **PA11** (USB_DM) and **PA12** (USB_DP) must not be remapped
- Pull-up resistor R5 (1.5 kΩ) is permanently connected; the device will enumerate immediately upon VBUS application if the USB peripheral is initialised
- For applications requiring delayed enumeration, defer USB peripheral initialisation until the firmware is ready to respond to host requests

---

## 9. Revision History

| Revision | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-05-18 | Paul Muyali | Initial release. Complete schematic and two-layer PCB layout with all components placed and routed. |

---

## 10. References

1. STMicroelectronics. *STM32F103C8 Datasheet (DS5319)*. Rev. 17. STMicroelectronics, 2023.
2. STMicroelectronics. *STM32F10xxx Reference Manual (RM0008)*. Rev. 21. STMicroelectronics, 2021.
3. STMicroelectronics. *AN4899 — STM32 microcontroller GPIO hardware settings and low-power consumption*. STMicroelectronics, 2021.
4. Advanced Monolithic Systems. *AMS1117 Datasheet*. Revision B. AMS, 2015.
5. USB Implementers Forum. *Universal Serial Bus Specification, Revision 2.0*. USB-IF, 2000.
6. Wuerth Elektronik. *WR-COM USB Micro-B Receptacle 629105150521 Datasheet*. Wuerth Elektronik, 2020.
7. KiCad EDA. *KiCad 10.0 Documentation*. https://docs.kicad.org
8. IPC. *IPC-2221B — Generic Standard on Printed Board Design*. IPC, 2012.
9. IPC. *IPC-A-610G — Acceptability of Electronic Assemblies*. IPC, 2017.

---

*This document was generated from KiCad project files: `Microcontroller_with_STM32.kicad_sch` and `Microcontroller_with_STM32.kicad_pcb`*

    
