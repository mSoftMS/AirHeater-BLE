# Hcalory / AirHeater BLE – ESPHome + Home Assistant

> Fully working BLE control of diesel air heaters  
> using ESP32 + ESPHome with native Home Assistant integration.

At the time of writing, there is no complete, working solution available online.  
This project is the result of reverse-engineering the BLE protocol used by  
Hcalory / AirHeater mobile applications.

---

## Features

- Read heater status  
- Power ON / OFF (same sequence as the mobile app)  
- Power level control (gear 0–6)  
- Ambient temperature reading  
- Heat exchanger temperature  
- Supply voltage monitoring  
- Error detection (E-01 … E-10)  
- Full control from Home Assistant  

---

## Supported devices

Diesel air heaters with BLE support, including:

- Hcalory  
- AirHeater  
- Compatible clones exposing BLE service `FFF0`  

> Clones may differ slightly. Use the raw debug sensor to verify frames.

---

## Hardware requirements

- ESP32 (tested on `esp32dev`)
- Diesel air heater with BLE
- Home Assistant
- ESPHome (>= 2024.x)
- ESP-IDF framework (required for stable BLE)

---

## Software requirements

- ESPHome
- Home Assistant
- Smartphone (only required to discover the heater BLE MAC address)

---

## Installation

### 1. Find the heater BLE MAC address

Use a BLE scanner application:

- **Android**: nRF Connect  
- **iOS**: LightBlue  

Look for a device named similar to:

- `TY`
- `AirHeater`
- `HCALORY`

Example MAC address:

`EC:B1:D5:01:8B:28`

---

### 2. Configure ESPHome

Edit **only these values** in the YAML file:

```yaml
substitutions:
  wifi_ssid: "YourWiFi"
  wifi_password: "YourPassword"
  heater_mac: "EC:B1:D5:01:8B:28"
```

Do **not** change anything else unless you understand the protocol.  
Frame timing and payload structure are critical.

---

### 3. Flash ESP32

Build and upload firmware:

```bash
esphome run hcalory-ble.yaml
```

After the first flash, OTA updates will work normally.

---

### 4. Add to Home Assistant

- Go to **Settings → Integrations**
- Add **ESPHome**
- The device will be discovered automatically

Entities provided include:

- Heater Power (switch)
- Heater Power Level (select)
- Voltage sensor
- Ambient temperature
- Heat exchanger temperature
- Status and error sensors

---

## How it works

### Power ON / OFF logic

The heater cannot be hard-powered off.

The OFF sequence is **identical to the official mobile application**:

1. Switch to ventilation  
2. Short return to heating  
3. Back to ventilation (cool-down phase)

This prevents:

- overheating
- flame-out errors
- controller faults

---

### Power level (gear) control

- Power is adjusted **step-by-step**
- ESP32 sends incremental plus / minus commands
- After each step, it waits for a BLE status update

This avoids:

- BLE flooding
- controller lock-ups
- undefined heater states

---

## Error handling

If the heater reports an error:

- Status in Home Assistant becomes `E-xx`
- Power level changes are blocked
- No further control commands are sent

Common error codes:

| Code | Meaning |
|------|--------|
| E-01 | General error |
| E-02 | Low / high voltage |
| E-03 | Glow plug |
| E-04 | Fuel pump |
| E-05 | Overheat |
| E-06 | Fan |
| E-07 | Communication |
| E-08 | No fuel / flame-out |
| E-09 | Sensor |
| E-10 | Ignition failure |

---

## Debugging

### Raw BLE frames

Entity in Home Assistant:

**Heater Raw Debug**

This sensor displays the **full HEX frame** received from the heater.

Useful for:

- protocol verification
- comparing different heater firmware versions
- supporting additional heater models
- further reverse-engineering

---

### Common issues

**Frequent connect / disconnect**

- Move the ESP32 closer to the heater
- Avoid metal enclosures
- Ensure stable power supply

**Heater does not turn ON**

- Heater must be in OFF state
- Check supply voltage
- Inspect raw BLE frames

**BLE error `status=14`**

- Fixed by `update_interval: never`
- Already included in this configuration

---

## Important notes

- This is **not an official manufacturer integration**
- Use at your own risk
- Tested on multiple units, but BLE clones may differ

If something does not work:

- Do not randomly change the code
- Inspect raw BLE frames first
- Verify MAC address and signal quality

---

## License

MIT License

You are free to:

- use
- modify
- redistribute

If you improve or extend this project, please share your findings.  
This protocol has been a black hole on the internet for far too long.

---

## Contributing / Next steps

Possible future improvements:

- Support for additional heater models
- Full BLE protocol documentation
- Adaptive polling
- BLE watchdog and auto-reconnect
- Power and safety enhancements

Pull requests and reports are welcome.
