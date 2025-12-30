# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an ESPHome configuration repository for managing IoT smart home devices that integrate with Home Assistant. All devices run on ESP8266/ESP32 microcontrollers and communicate via WiFi with the Home Assistant API.

## Device Inventory

The repository contains 7 device configurations:

- **Smart Outlets**: `garden-outlet.yaml`, `shed-outlet.yaml` - GPIO-based relay controls with LED indicators and physical buttons
- **Roller Shutters**: `roller-shutter-back.yaml`, `roller-shutter-front.yaml`, `roller-shutter-upstairs.yaml` - Time-based window blind/shutter controllers
- **Utility Meters**:
  - `watermeter.yaml` - Water consumption sensor with pulse meter, temperature/humidity monitoring (HDC1080 sensor via I2C)
  - `slimmelezer.yaml` - Dutch smart meter P1 reader using DSMR protocol for electricity and gas data

## Development Workflow

### Prerequisites
- ESPHome installed (via pip, Home Assistant add-on, or ESPHome dashboard)
- Access to secrets file containing: `wifi_ssid`, `wifi_password`, `hotspot_password`

### Common Commands

**Validate configuration:**
```bash
esphome config <device-name>.yaml
```

**Compile firmware (without uploading):**
```bash
esphome compile <device-name>.yaml
```

**Upload to device (OTA - device must already have ESPHome):**
```bash
esphome upload <device-name>.yaml
```

**View device logs (real-time):**
```bash
esphome logs <device-name>.yaml
```

**Initial flash via USB (first time setup):**
```bash
esphome run <device-name>.yaml
```

**Clean build files:**
```bash
esphome clean <device-name>.yaml
```

### Secrets Management

Create a `secrets.yaml` file in the repository root (git-ignored) with:
```yaml
wifi_ssid: "your_wifi_network"
wifi_password: "your_wifi_password"
hotspot_password: "fallback_ap_password"
```

All device configurations reference secrets using `!secret <key_name>` syntax.

## Configuration Architecture

### Standard Device Structure

Every device configuration follows this pattern:

1. **Substitutions** - Device-specific variables (name, friendly_name, etc.)
2. **ESPHome Core** - Device name and optional project metadata
3. **Platform** - ESP8266/ESP32 board type (esp01_1m, d1_mini, etc.)
4. **WiFi** - Network credentials with fallback AP/captive portal
5. **Core Services** - Logger, Home Assistant API, OTA updates, web server
6. **Components** - Device-specific sensors, switches, binary_sensors, etc.

### Hardware Pin Mapping Conventions

**Smart Outlets (ESP-01 1MB):**
- GPIO12: Relay control
- GPIO4: LED indicator (inverted)
- GPIO14: Physical button (INPUT_PULLUP, inverted)

**Water Meter (Wemos D1 Mini):**
- D3: Status LED
- D4: Green LED (pulse indicator)
- D5: Pulse meter input
- GPIO4/GPIO5: I2C (SDA/SCL) for HDC1080 sensor

**SlimmeLezer (Wemos D1 Mini):**
- D7: UART RX (P1 protocol)
- UART config: 9600 baud, 7 data bits, EVEN parity, 1 stop bit

**Roller Shutters:**
- Time-based control (no position feedback)
- Interlock mechanism prevents simultaneous open/close
- Device-specific open/close durations in configuration

### Key ESPHome Patterns Used

**Template switches** - Combine multiple outputs (relay + LED) into single logical switch
**Pulse meters** - Count pulses for water/energy measurement with flash feedback
**DSMR component** - Parse Dutch smart meter P1 protocol data
**Uptime timestamp pattern** - Send device boot time once instead of continuous uptime updates (reduces database bloat)
**Captive portal fallback** - All devices support setup via fallback WiFi AP if network unavailable

## Project Conventions

### Naming Standards
- Device names: Use snake_case for internal IDs (water_meter, garden_outlet)
- File names: Use kebab-case matching device name (water-meter.yaml, garden-outlet.yaml)
- Friendly names: Human-readable titles for Home Assistant ("Water meter", "Garden Outlet")
- Hotspot SSIDs: Format as "{friendly_name} Hotspot" or descriptive installation name

### Configuration Organization
- All device YAML files at repository root
- Flat structure (no subdirectories)
- Each file is self-contained and independent
- Shared configuration via ESPHome substitutions, not includes

### Device Metadata
Projects like watermeter and slimmelezer include:
- `project.name` - Author/project identifier
- `project.version` - Configuration version
- Optional custom version/model substitutions and text sensors

## Home Assistant Integration

All devices automatically:
- Discovered via Home Assistant API integration
- Expose entities matching configured sensors/switches/buttons
- Support OTA updates via ESPHome dashboard or Home Assistant
- Provide diagnostic entities (WiFi signal, IP address, uptime, ESPHome version)

No manual MQTT configuration needed - uses native Home Assistant API protocol.

## Documentation

ESPHome component documentation: https://esphome.io/components/

## Troubleshooting

**Device offline/unreachable:**
- Check WiFi credentials in secrets.yaml
- Device may have activated fallback hotspot - connect to "{device} Hotspot" WiFi
- Access captive portal at http://192.168.4.1 to reconfigure WiFi

**OTA update fails:**
- Verify device is on same network
- Try `esphome logs` first to confirm connectivity
- Worst case: USB flash via `esphome run` with `--device /dev/ttyUSB0`

**Compilation errors:**
- Validate YAML syntax: `esphome config <file>.yaml`
- Check ESPHome version compatibility
- Verify all referenced secrets exist in secrets.yaml

**UART/sensor issues (slimmelezer, watermeter):**
- Check physical pin connections match GPIO definitions
- Verify baud rate and UART parameters for P1 readers
- For I2C sensors: confirm SDA/SCL pins and sensor address
