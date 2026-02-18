# Diesel Heater - Home Assistant Integration

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://github.com/hacs/integration)
[![GitHub release](https://img.shields.io/github/release/MSDATDE/homeassistant-vevor-heater.svg)](https://github.com/MSDATDE/homeassistant-vevor-heater/releases)
[![License](https://img.shields.io/github/license/MSDATDE/homeassistant-vevor-heater.svg)](LICENSE)

Control your diesel heater from Home Assistant via Bluetooth. Supports multiple brands and protocols.

## Features

- **Climate Entity** - Full thermostat control with target temperature and presets (Away/Comfort)
- **Heater Level Control** - Adjust heating power (1-10) via fan entity
- **Running Mode Selection** - Switch between Manual, Level, Temperature, and Ventilation modes
- **Fuel Tracking** - Estimated hourly/daily/total fuel consumption with 30-day history
- **Runtime Tracking** - Daily and total runtime hours with 30-day history
- **Auto Temperature Calibration** - Automatic offset based on external HA temperature sensor
- **Configurable Settings** - Language, temperature unit, altitude unit, pump type, tank volume, backlight
- **Sensors** - Monitor temperature, voltage, altitude, CO (Sunster), and heater status
- **Debug Service** - Send raw BLE commands for troubleshooting
- **Bluetooth LE** - Direct local connection, no cloud required
- **Real-time Updates** - 30-second polling interval with stale data tolerance

## Supported Devices

This integration supports multiple diesel heater brands and protocols:

| Protocol | Brands | Encryption | Packet Size |
|----------|--------|------------|-------------|
| AA55 | Vevor, generic | No | 20 bytes |
| AA55 Encrypted | Vevor, generic | XOR | 48 bytes |
| AA66 | Vevor, BYD | No | 20 bytes |
| AA66 Encrypted | Vevor, BYD | XOR | 48 bytes |
| ABBA | HeaterCC, AirHeaterCC | No | 21 bytes |
| CBFF | Sunster | Double XOR | 47 bytes |
| Hcalory | Hcalory MVP1/MVP2 | No | Variable |

Bluetooth Service UUIDs:
- `0000ffe0-0000-1000-8000-00805f9b34fb` (Vevor/BYD)
- `0000fff0-0000-1000-8000-00805f9b34fb` (HeaterCC/Hcalory MVP1)
- `0000bd39-0000-1000-8000-00805f9b34fb` (Hcalory MVP2)

## Installation

### HACS (Recommended)

1. Open HACS in Home Assistant
2. Go to "Integrations"
3. Click the three dots menu -> "Custom repositories"
4. Add this repository: `https://github.com/MSDATDE/homeassistant-vevor-heater`
5. Category: "Integration"
6. Click "Add"
7. Search for "Diesel Heater" and install
8. Restart Home Assistant

### Manual Installation

1. Download the latest release from [GitHub](https://github.com/MSDATDE/homeassistant-vevor-heater/releases)
2. Copy the `custom_components/diesel_heater` folder to your Home Assistant's `custom_components` directory
3. Restart Home Assistant

> **Note:** If upgrading from v1.x (`vevor_heater` domain), the integration will automatically migrate your configuration and entities to the new `diesel_heater` domain.

## Finding Your Heater's MAC Address

Before adding the integration, you need to find your heater's Bluetooth MAC address:

1. Install Python dependencies:
   ```bash
   pip install bleak
   ```

2. Copy `find_heater.py` to a temporary directory (do NOT run it from the integration folder):
   ```bash
   cp find_heater.py /tmp/
   cd /tmp
   python3 find_heater.py before
   ```

3. **Close the heater app** if it's running

4. Open the heater app and **connect** to your heater

5. Run the finder script again:
   ```bash
   python3 find_heater.py after
   ```

6. The device that **disappeared** is your heater! Note its MAC address (e.g., `69:96:19:04:59:9B`)

## Configuration

### Add Integration

1. Go to **Settings** -> **Devices & Services**
2. Click **"+ Add Integration"**
3. Search for **"Diesel Heater"**
4. Enter your heater's MAC address and PIN (default: `1234`)
5. Click **"Submit"**

### Options

Go to **Settings** -> **Devices & Services** -> **Diesel Heater** -> **Configure** to adjust:

- **PIN Code** - Heater authentication PIN (default: 1234)
- **Temperature Offset** - Manual calibration offset (-20 to +20 C)
- **Preset Temperatures** - Away and Comfort preset targets
- **External Temperature Sensor** - Select an HA sensor for auto-calibration
- **Max Auto Offset** - Limit for automatic temperature offset

### Entities Created

After setup, you'll have these entities:

#### Climate
- `climate.diesel_heater` - Main thermostat control with Away/Comfort presets

#### Fan
- `fan.diesel_heater_heater_level` - Heater power level (1-10)

#### Select
- `select.diesel_heater_running_mode` - Mode selector (Off/Level/Temperature/Ventilation)
- `select.diesel_heater_language` - Display language
- `select.diesel_heater_pump_type` - Oil pump type
- `select.diesel_heater_tank_volume` - Tank volume setting
- `select.diesel_heater_backlight` - Display backlight brightness

#### Sensors
- `sensor.diesel_heater_case_temperature` - Heater case temperature
- `sensor.diesel_heater_interior_temperature` - Room/cabin temperature
- `sensor.diesel_heater_interior_temperature_raw` - Raw sensor value (before offset)
- `sensor.diesel_heater_supply_voltage` - Power supply voltage
- `sensor.diesel_heater_altitude` - Altitude compensation
- `sensor.diesel_heater_running_step` - Current operation step
- `sensor.diesel_heater_error` - Error status
- `sensor.diesel_heater_running_mode` - Current running mode
- `sensor.diesel_heater_set_level` - Current set level
- `sensor.diesel_heater_protocol` - Detected protocol
- `sensor.diesel_heater_temperature_offset` - Current heater offset
- `sensor.diesel_heater_estimated_hourly_fuel_consumption` - Fuel rate (L/h)
- `sensor.diesel_heater_estimated_daily_fuel_consumed` - Daily fuel usage
- `sensor.diesel_heater_estimated_total_fuel_consumed` - Total fuel usage
- `sensor.diesel_heater_estimated_daily_fuel_history` - 30-day fuel history
- `sensor.diesel_heater_daily_runtime` - Daily runtime hours
- `sensor.diesel_heater_total_runtime` - Total runtime hours
- `sensor.diesel_heater_daily_runtime_history` - 30-day runtime history
- `sensor.diesel_heater_estimated_fuel_remaining` - Estimated fuel remaining
- `sensor.diesel_heater_last_refueled` - Last refuel timestamp
- `sensor.diesel_heater_estimated_fuel_since_refuel` - Fuel since last refuel
- `sensor.diesel_heater_carbon_monoxide` - CO in PPM (Sunster only)
- `sensor.diesel_heater_remaining_run_time` - Countdown timer (Sunster only)

#### Binary Sensors
- `binary_sensor.diesel_heater_active` - Heater active status
- `binary_sensor.diesel_heater_problem` - Error indicator
- `binary_sensor.diesel_heater_connected` - BLE connection status
- `binary_sensor.diesel_heater_auto_start_stop` - Auto start/stop enabled

#### Switches
- `switch.diesel_heater_power` - Simple ON/OFF control
- `switch.diesel_heater_auto_start_stop` - Auto temperature-based start/stop
- `switch.diesel_heater_auto_temperature_offset` - Auto calibration from external sensor
- `switch.diesel_heater_fahrenheit_mode` - Toggle Celsius/Fahrenheit
- `switch.diesel_heater_feet_mode` - Toggle Meters/Feet
- `switch.diesel_heater_high_altitude_mode` - High altitude mode (ABBA only)

#### Number
- `number.diesel_heater_level` - Set heater level (1-10)
- `number.diesel_heater_target_temperature` - Set target temperature
- `number.diesel_heater_temperature_offset` - Heater offset (-9 to +9)
- `number.diesel_heater_tank_capacity` - Tank capacity for fuel estimation

#### Buttons
- `button.diesel_heater_sync_time` - Sync heater clock with HA
- `button.diesel_heater_reset_estimated_fuel_remaining` - Reset fuel counter (refuel)

### Services

#### `diesel_heater.send_command`
Send raw BLE commands for debugging:
- **device_id** - MAC address or last 4-5 chars (optional, sends to all if empty)
- **command** - Command number (0-255)
- **argument** - Argument value (-128 to 127)

## Troubleshooting

### Heater Not Found

- Make sure Bluetooth is enabled on your Home Assistant host
- Ensure the heater is powered on
- Heater app must be **disconnected** (only one BLE connection allowed)
- If using ESPHome Bluetooth Proxy, ensure it has available connection slots (max 3)

### No Data / Connection Issues

- Check Home Assistant logs: **Settings** -> **System** -> **Logs**
- Look for entries with `diesel_heater`
- The integration uses exponential backoff and stale data tolerance (3 cycles)

### Commands Not Working

- Verify you're using the correct PIN (default: `1234`)
- Some commands only work in specific modes:
  - Temperature setting: Temperature Mode
  - Level setting: Level Mode

### Debug Logging

Enable debug logging in `configuration.yaml`:

```yaml
logger:
  default: info
  logs:
    custom_components.diesel_heater: debug
    diesel_heater_ble: debug
```

## Protocol Details

This integration uses the [diesel-heater-ble](https://pypi.org/project/diesel-heater-ble/) library for protocol handling.

### Commands
- Command 1: Status query
- Command 2: Set running mode
- Command 3: Turn ON (arg=1) / OFF (arg=0)
- Command 4: Set level or temperature
- Command 10: Sync time
- Command 14: Set language
- Command 15: Set temperature unit
- Command 16: Set tank volume
- Command 17: Set pump type
- Command 18: Set auto start/stop
- Command 19: Set altitude unit
- Command 20: Set temperature offset
- Command 21: Set backlight

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Based on the [vevor-ble-bridge](https://github.com/andyrak/vevor-ble-bridge) protocol documentation
- Protocol library based on work by [Spettacolo83](https://github.com/Spettacolo83/homeassistant-diesel-heater)
- Thanks to the Home Assistant community for support

## Support

If you encounter issues, please:
1. Check the [Issues](https://github.com/MSDATDE/homeassistant-vevor-heater/issues) page
2. Enable debug logging and include logs in your report
3. Provide your heater model and protocol type

---

**Disclaimer**: This is an unofficial integration and is not affiliated with Vevor, BYD, HeaterCC, Sunster, or Hcalory.
