# Expresso MIDI Preset Mod

## Overview

This project adds MIDI program change (PC) preset recall and save functionality to the PedalPCB Expresso expression multiplexer. The Expresso is an expression pedal multiplexer that allows switching between three independent expression outputs using a footswitch.

This modification enables the device to:
- Store and recall up to 128 presets via MIDI program change messages
- Save the current configuration to a preset slot with a long footswitch press
- Maintain all original expression pedal multiplexing functionality

## Hardware Platform

- **Microcontroller**: ATmega328P running on internal 8MHz oscillator
- **Framework**: Arduino
- **Build System**: PlatformIO

## Functional Requirements

1. **Expression Pedal Control**: Continuously read the expression pedal and update the active channel's digital potentiometer
2. **Channel Switching**: Short press footswitch to cycle through active channels (A→B→C→A)
3. **Preset Recall**: Receive MIDI PC (0-127) to load a preset from EEPROM and write all three digipot values
4. **Preset Save**: Long press footswitch (>1 second) to save current three digipot values to EEPROM at the last-received PC slot
   - If no PC received since power-on, long press does nothing
5. **Channel Indication**: LEDs indicate the currently active channel

## MIDI Specification

- **Input**: MIDI Program Change (PC) messages (status byte 0xCn where n = channel)
- **Baud Rate**: 31250 (MIDI standard)
- **Channel**: Configurable or fixed (to be determined during implementation)

## Data Structure

### EEPROM Layout

The device stores 128 presets in EEPROM, with each preset occupying 3 bytes:

```
Preset N (3 bytes at address N*3):
  Byte 0: Channel A wiper position (0-255)
  Byte 1: Channel B wiper position (0-255)
  Byte 2: Channel C wiper position (0-255)

Total: 128 presets × 3 bytes = 384 bytes
```

## Building and Uploading

This project uses PlatformIO. To build and upload:

```bash
# Build the project
pio run

# Build and upload to board
pio run --target upload
```

## Testing

Unit tests can be run using PlatformIO's test framework:

```bash
pio test
```

## Open Questions

- MIDI channel: fixed (e.g., channel 1) or configurable?
- LED behavior on preset load: flash, or just show active channel?
- Initial state at power-on: load preset 0, or start with all zeros?

## License

TBD
