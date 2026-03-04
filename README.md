# Visible Light Communication (VLC) Using FPGA

> **B.Tech Project (BTP) | LNMIIT Jaipur | Supervisor: Dr. Nishant Gupta & Dr. Harish C. Kumawat**  
> **Team:** Arnab Harsh (22UEC024), Shivam Choudhary (22UEC124), Somil Malik (22UEC134)

---

## Overview

This project implements a **Visible Light Communication (VLC) system on FPGA** using two ZYBO boards — one acting as a transmitter and one as a receiver. Data is transmitted via an LED using **On-Off Keying (OOK) modulation** and received by an LDR (Light Dependent Resistor), with the output displayed on a VGA monitor in real time.

VLC uses visible light as the communication medium instead of radio waves, making it ideal for EMI-sensitive environments like hospitals and aircraft.

---

## Hardware Used

| Component | Quantity | Purpose |
|---|---|---|
| ZYBO FPGA Board | 2 | Transmitter + Receiver |
| LED | 1 | Optical data transmitter |
| LDR (Light Dependent Resistor) | 1 | Optical data receiver |
| VGA Cable + Display | 1 | Output visualization |
| Breadboard + Resistors | - | Circuit connections |
| PC | 2 | Host machines |

---

## System Architecture

```
[PC / .coe file] → [ZYBO TX Board] → [LED] ~~light~~ [LDR] → [ZYBO RX Board] → [VGA Display]
```

### Transmitter (ZYBO Board 1)
- **Single-Port ROM** stores image data from `.coe` file
- **Finite State Machine (FSM)** controls data transmission flow
- **OOK Modulation Logic** drives LED — HIGH = bit '1', LOW = bit '0'
- **Clock Divider** controls transmission timing
- Fully **PL-based** (No PS at runtime)
- GPIO output drives a high-speed LED

### Receiver (ZYBO Board 2)
- **Clock Wizard** generates 25 MHz pixel clock for VGA
- **VGA Timing Generator** syncs `hsync` and `vsync` pulses
- `hcount` / `vcount` track horizontal and vertical pixel positions
- **Shift Register** stores incoming bitstream data
- **LDR Sampling** matches receive frequency to LED transmission rate
- Pixel mapping: bit `'1'` → white pixel (RGB = 111), bit `'0'` → black pixel (RGB = 000)

---

## VGA Timing Specification (640×480 @ ~60 Hz)

### Horizontal (hcount: 0 to 799)
| Part | Pixels | Counter Range |
|---|---|---|
| Visible | 640 | 0 – 639 |
| Front Porch | 16 | 640 – 655 |
| HSYNC Pulse | 96 | 656 – 751 |
| Back Porch | 48 | 752 – 799 |

### Vertical (vcount: 0 to 524)
| Part | Lines | Counter Range |
|---|---|---|
| Visible | 480 | 0 – 479 |
| Front Porch | 10 | 480 – 489 |
| VSYNC Pulse | 2 | 490 – 491 |
| Back Porch | 33 | 492 – 524 |

> Pixel clock = 25 MHz (standard for 640×480 @ ~59.52 Hz — accepted by all monitors)

---

## Key VHDL Modules

| Module | Function |
|---|---|
| `vga_controller.vhd` | Generates hsync, vsync, hcount, vcount |
| `clock_divider.vhd` | Divides clock for OOK transmission timing |
| `rom_single_port.vhd` | Stores image bitstream from .coe file |
| `fsm_transmitter.vhd` | Controls data readout and LED driving |
| `ldr_sampler.vhd` | Samples LDR input at matched frequency |
| `shift_register.vhd` | Stores received bitstream |
| `pixel_mapper.vhd` | Maps received bits to VGA RGB pixels |

---

## Results

- Successfully transmitted a **black & white image** over visible light between two FPGA boards
- System tested at various distances and light intensities (daytime vs. dark conditions)
- Bit error rate (BER) measured at multiple distances — error increased with distance and ambient light interference

---

## Challenges and Solutions

| Challenge | Solution |
|---|---|
| UART receiver unreliable for output | Switched to VGA interface for stable real-time pixel display |
| Clock Wizard configuration for VGA | Generated accurate 25 MHz pixel clock using IP Wizard |
| Incorrect row/column VGA mapping | Corrected pixel addressing synchronized with VGA timing |
| High-speed color image transmission failed | Reduced to B&W image to minimize bitstream size |

---

## Tools Used

- **Xilinx Vivado** (VHDL design, simulation, synthesis, implementation)
- **ZYBO FPGA Board** (Xilinx Zynq-7000)
- **Hardware:** LED, LDR, VGA monitor

---

## Future Scope

- Extend to **Li-Fi** for high-speed internet over LED infrastructure
- Integrate with **smart city** LED streetlights for navigation and data
- Apply in **automotive V2V communication** via headlights/taillights
- Use in **EMI-restricted environments**: hospitals, aircraft, underwater missions
- Scale to **color image transmission** using higher-speed LEDs and photodetectors

---

## References

1. B. N. Ashfaq et al., "FPGA-Based Implementation of a Vehicular VLC System," IEEE VNC, 2021.
2. H. Guo et al., "FPGA Implementation of VLC Communication Technology," IEEE WAINA, 2017.
3. S. Ricci et al., "FPGA-Based VLC Instrument for Ultralow Latency Applications," IEEE TIM, 2023.
