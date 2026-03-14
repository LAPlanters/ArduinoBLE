# ArduinoBLE — BLE 5.0 PHY Fork

> **Fork of [arduino-libraries/ArduinoBLE](https://github.com/arduino-libraries/ArduinoBLE)** with BLE 5.0 PHY support (2M PHY, Coded PHY).

## What's Added

The stock ArduinoBLE library doesn't expose BLE 5.0 PHY control, even though the underlying hardware (nRF52840, etc.) and BLE stack (Cordio) fully support it. This fork adds ~70 lines to expose PHY selection through the existing HCI layer.

### New API

```cpp
#include <ArduinoBLE.h>

void setup() {
  BLE.begin();
  
  // Set preferred PHY for all new connections
  // Bit 0 = LE 1M, Bit 1 = LE 2M, Bit 2 = LE Coded
  BLE.setPreferredPhy(0x03, 0x03);  // prefer 2M, accept 1M fallback
  
  // ... rest of your setup
}
```

| Method | Description |
|--------|-------------|
| `BLE.setPreferredPhy(txPhys, rxPhys)` | Set default PHY preference for all new connections |
| `BLE.requestPhy(handle, txPhys, rxPhys, options)` | Request PHY change on an active connection |
| `BLE.readPhy(handle, txPhy, rxPhy)` | Read current PHY of a connection |

### PHY Bit Values

| Bit | PHY | Benefit |
|-----|-----|---------|
| `0x01` | LE 1M | Default, always supported |
| `0x02` | LE 2M | ~2x throughput |
| `0x04` | LE Coded | ~4x range (long range) |

OR bits together for preferences: `0x03` = "prefer 2M, accept 1M".

### Backward Compatible

- Falls back to 1M if the remote device doesn't support 2M/Coded
- No changes to existing API — all your current code works unchanged
- PHY negotiation happens at the controller level; your application code doesn't need to handle it

### Tested On

- Arduino Nano 33 BLE (nRF52840 + mbed/Cordio)
- Should work on any board with a BLE 5.0-capable controller

## Installation

### Arduino IDE
1. Download this repo as a ZIP
2. Sketch → Include Library → Add .ZIP Library
3. Select the downloaded ZIP

### Manual
```bash
cd ~/Arduino/libraries
git clone https://github.com/LAPlanters/ArduinoBLE.git
```
This replaces the stock ArduinoBLE. To revert, delete the folder and reinstall via Library Manager.

## Changes from Upstream

All changes are in 4 files:

- `src/utility/HCI.h` — Added `READ_PHY`, `SET_DEFAULT_PHY`, `SET_PHY` to `LE_COMMAND` enum; `PHY_UPDATE_COMPLETE` to `LE_META_EVENT` enum; method declarations
- `src/utility/HCI.cpp` — Implemented `leSetDefaultPhy()`, `leSetPhy()`, `leReadPhy()`; added `PHY_UPDATE_COMPLETE` event handler
- `src/local/BLELocalDevice.h` — Declared `setPreferredPhy()`, `requestPhy()`, `readPhy()`
- `src/local/BLELocalDevice.cpp` — Implemented the public API methods

## Original README

Enables Bluetooth® Low Energy connectivity on the Arduino MKR WiFi 1010, Arduino UNO WiFi Rev2, Arduino Nano 33 IoT, Arduino Nano 33 BLE, Arduino Portenta H7, Arduino Giga R1 and Arduino UNO R4 WiFi.

This library supports creating a Bluetooth® Low Energy peripheral & central mode.

For the Arduino MKR WiFi 1010, Arduino UNO WiFi Rev2, and Arduino Nano 33 IoT boards, it requires the NINA module to be running [Arduino NINA-W102 firmware](https://github.com/arduino/nina-fw) v1.2.0 or later.

For the Arduino UNO R4 WiFi, it requires the ESP32-S3 module to be running [firmware](https://github.com/arduino/uno-r4-wifi-usb-bridge) v0.2.0 or later.


For more information about this library please visit us at:
https://www.arduino.cc/en/Reference/ArduinoBLE

## License

```
Copyright (c) 2019 Arduino SA. All rights reserved.

This library is free software; you can redistribute it and/or
modify it under the terms of the GNU Lesser General Public
License as published by the Free Software Foundation; either
version 2.1 of the License, or (at your option) any later version.

This library is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
Lesser General Public License for more details.

You should have received a copy of the GNU Lesser General Public
License along with this library; if not, write to the Free Software
Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
```
