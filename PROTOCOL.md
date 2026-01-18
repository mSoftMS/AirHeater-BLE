# Hcalory / AirHeater BLE Protocol (Reverse-Engineered)

This document describes the BLE protocol used by Hcalory / AirHeater diesel heaters.
The protocol was reverse-engineered by analyzing BLE traffic between the heater and the official mobile applications.

THIS IS NOT OFFICIAL DOCUMENTATION.

---

## Overview

Transport: Bluetooth Low Energy (BLE)
Primary service: 0xFFF0

Characteristics:
- 0xFFF1 – Notifications (status frames from heater)
- 0xFFF2 – Write (commands sent to heater)

Communication is stateful and timing-sensitive.
Flooding commands may cause the heater to ignore commands or enter an error state.

---

## BLE Services and Characteristics

Service UUID: FFF0

Characteristics:
FFF1 – Notify
FFF2 – Write

---

## Frame Formats

Two frame formats are used.

### Format A – Status / BB Commands

Used for:
- status polling
- power mode commands
- gear increase / decrease

Byte layout:

[0] Header 1 (0xBA)  
[1] Header 2 (0xAB)  
[2] Length  
[3] Command  
[4] Data  
[5] Data  
[6] Data  
[7] Checksum  

Checksum = sum of bytes 0–6 (8-bit)

---

### Format B – Extended Command Frame

[0]  0xAA  
[1]  0x55  
[2]  0x0C  
[3]  0x01  
[4]  Command  
[5]  Value 1  
[6]  Value 2  
[7]  0x00  
[8]  0x00  
[9]  0x00  
[10] 0x00  
[11] 0x00  
[12] 0x00  
[13] 0x00  
[14] 0x00  
[15] Checksum  

Checksum = sum of bytes 0–14 (8-bit)

---

## Status Polling

Command:
BA AB 04 CC 00 00 00 35

Triggers a status notification on FFF1.

---

## Status Notification (FFF1)

Important bytes:

Byte 4 – Status  
Byte 5 – Mode  
Byte 6 – Error or Gear  
Byte 9 – Voltage  
Byte 10 – Temp offset selector  
Byte 11 – Ambient temperature  
Byte 12–13 – Heat exchanger temperature

---

## Status Byte (Byte 4)

0x00 – OFF  
0x01 – Heating  
0x02 – Cooling  
0x04 – Ventilation  

---

## Mode Byte (Byte 5)

0xFF – Error state  
Other – Normal operation

---

## Error Codes (Mode == 0xFF)

1  – E-01 General  
2  – E-02 Voltage  
3  – E-03 Glow plug  
4  – E-04 Fuel pump  
5  – E-05 Overheat  
6  – E-06 Fan  
7  – E-07 Communication  
8  – E-08 No fuel / flame-out  
9  – E-09 Sensor  
10 – E-10 Ignition failure  

---

## Temperatures

Ambient temperature:
ambient = byte[11] - offset

offset:
byte[10] == 0x00 → 30
else → 22

Heat exchanger temperature:
(byte[12] << 8) | byte[13]

---

## Power Control (BB)

Heating ON:
BA AB 04 BB A1 00 00 CS

Ventilation / OFF:
BA AB 04 BB A4 00 00 CS

OFF sequence:
Ventilation → Heating → Ventilation
Delay ~500 ms between steps

---

## Gear Control

Increase power:
BA AB 04 BB A2 00 00 CS

Decrease power:
BA AB 04 BB A3 00 00 CS

Always wait for status after each step.

---

## Timing Rules

- Do not flood BLE
- Delay ≥ 500 ms between commands
- Stop sending commands when error occurs

---

## Safety Notice

This protocol controls a fuel-based heating device.
Use at your own risk.
Not official manufacturer documentation.
