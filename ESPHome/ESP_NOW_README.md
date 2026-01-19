# ESP-NOW Communication for TX Ultimate Easy

This feature enables direct communication between multiple TX Ultimate Easy switches using ESP-NOW, allowing switches to control relays on other switches without going through Home Assistant.

## Overview

- **ESP-NOW Communication**: Switches communicate directly via ESP-NOW (peer-to-peer)
- **Home Assistant Configuration**: Home Assistant is only used for configuration, not for runtime control
- **Flexible Mapping**: Each button can be configured to toggle any relay on any of the 3 switches
- **Fallback Behavior**: If ESP-NOW is not configured, buttons fall back to local relay control

## Setup Instructions

### 1. Include ESP-NOW Package

Add the ESP-NOW package to your device configuration:

```yaml
packages:
  remote_package:
    url: https://github.com/edwardtfn/TX-Ultimate-Easy
    ref: stable
    refresh: 5min
    files:
      - ESPHome/TX-Ultimate-Easy-ESPHome_core.yaml
      - ESPHome/TX-Ultimate-Easy-ESPHome_standard.yaml
      - ESPHome/TX-Ultimate-Easy-ESPHome_espnow.yaml  # Add this line
```

### 2. Flash Configuration to All Switches

Flash the updated configuration to all 3 switches via Home Assistant.

### 3. Find MAC Addresses

For each switch, note its MAC address:
- In Home Assistant, check the **"ESP-NOW MAC Address"** text sensor
- Or check the device information in Home Assistant

### 4. Configure Each Switch

For each switch, configure which relay each button should toggle:

1. **Enable ESP-NOW**: Turn on the **"ESP-NOW Enabled"** switch

2. **For each button** (Button 1-4):
   - **"Button X ESP-NOW Target MAC Address"**: Enter the MAC address of the target device (format: `AA:BB:CC:DD:EE:FF`)
   - **"Button X ESP-NOW Target Relay"**: Select which relay to toggle (Relay 1-4)

3. **Example Configuration**:
   - Switch 1, Button 1 → Target MAC: Switch 2's MAC, Target Relay: Relay 1
   - Switch 2, Button 1 → Target MAC: Switch 3's MAC, Target Relay: Relay 1
   - Switch 3, Button 1 → Target MAC: Switch 1's MAC, Target Relay: Relay 1

### 5. Test

Press a button on one switch. The configured relay on the target switch should toggle.

## Configuration Entities

### Switches
- **ESP-NOW Enabled**: Master switch to enable/disable ESP-NOW functionality

### Text Sensors (Read-only)
- **ESP-NOW MAC Address**: Shows this device's MAC address (for sharing with other devices)
- **Button X ESP-NOW Target MAC Address**: Shows the currently configured target MAC

### Inputs (Configuration)
- **Button X ESP-NOW Target MAC Address**: Text input to set the target device MAC address
  - Format: `AA:BB:CC:DD:EE:FF` (uppercase or lowercase)
  - Set to `00:00:00:00:00:00` to disable ESP-NOW for this button (falls back to local relay)

### Selects (Configuration)
- **Button X ESP-NOW Target Relay**: Select which relay (1-4) to toggle on the target device

## How It Works

1. **Button Press**: When a button is pressed on a switch
2. **ESP-NOW Check**: The system checks if ESP-NOW is enabled and a target MAC is configured
3. **Send Command**: If configured, sends an ESP-NOW message to the target device with:
   - Command: `0x01` (toggle)
   - Relay number: 1-4
4. **Receive Command**: The target device receives the ESP-NOW message
5. **Toggle Relay**: The target device toggles the specified relay

## Message Format

ESP-NOW messages use a simple 2-byte format:
- Byte 0: Command (`0x01` = toggle relay)
- Byte 1: Relay number (1-4)

## Troubleshooting

### Button doesn't toggle remote relay
1. Check that **"ESP-NOW Enabled"** is ON
2. Verify the target MAC address is correct (check "ESP-NOW MAC Address" on target device)
3. Check that the MAC address format is correct: `AA:BB:CC:DD:EE:FF`
4. Check device logs for ESP-NOW send/receive messages

### Button toggles local relay instead
- This is the fallback behavior when ESP-NOW is not configured
- Make sure the target MAC address is not `00:00:00:00:00:00`
- Verify ESP-NOW is enabled

### ESP-NOW messages not received
1. Ensure all devices are on the same WiFi network (ESP-NOW works over WiFi)
2. Check that devices are within range
3. Verify the MAC address is correct
4. Check device logs for receive errors

## Notes

- ESP-NOW works independently of Home Assistant - switches can communicate even if Home Assistant is offline
- Configuration changes require the device to be connected to Home Assistant
- MAC addresses are stored persistently and survive reboots
- If a target MAC is not configured (all zeros), the button will toggle the local relay (original behavior)

