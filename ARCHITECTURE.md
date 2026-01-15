# Architecture

## Hardware Components

### Existing Hardware

The PedalPCB Expresso expression multiplexer includes:

- **Microcontroller**: ATmega328P (8MHz internal oscillator)
- **Digital Potentiometers**: Three MCP41010 chips (10kΩ, 256 steps) controlled via SPI
  - Shared MOSI (PB3) and SCK (PB5) lines
  - Individual chip select pins: CS_A, CS_B, CS_C
- **Expression Pedal Input**: Connected via voltage divider to ADC
- **User Interface**:
  - Single SPST momentary footswitch
  - Three channel indicator LEDs
- **Power Supply**: 5V regulated supply using L78L05 regulator

### Hardware Additions for MIDI

The MIDI modification requires adding a MIDI input circuit:

- **Optocoupler**: 6N138 (or H11L1M) for MIDI signal isolation (6N138 is preferred for MIDI applications)
- **Current Limiting Resistor**: 220Ω between MIDI input and optocoupler LED cathode
- **Pull-up Resistor**: On optocoupler output to ATmega328P
- **Connection**: Optocoupler output to PD0/RX (pin 2)

This provides standard MIDI 5-pin DIN input with proper electrical isolation.

## Pin Assignments

| Function          | ATmega328P Pin | Notes                    |
|-------------------|----------------|--------------------------|
| MOSI              | PB3 (pin 17)   | SPI data out             |
| SCK               | PB5 (pin 19)   | SPI clock                |
| CS_A              | TBD            | Chip select for pot A    |
| CS_B              | TBD            | Chip select for pot B    |
| CS_C              | TBD            | Chip select for pot C    |
| Expression ADC    | TBD (PC0-PC5)  | Expression pedal input   |
| Footswitch        | TBD            | Momentary switch input   |
| LED_A             | TBD            | Channel A indicator      |
| LED_B             | TBD            | Channel B indicator      |
| LED_C             | TBD            | Channel C indicator      |
| MIDI RX           | PD0 (pin 2)    | MIDI input via optocoupler |

**Note**: Pin assignments marked as TBD need to be verified against the actual PCB traces.

## Software Modules

The firmware is organized into the following functional modules:

### 1. SPI Driver
- **Purpose**: Write wiper position values to MCP41010 digital potentiometers
- **Interface**: Standard Arduino SPI library
- **Operation**: Send 2-byte command (command byte + data byte) to selected chip

### 2. ADC Driver
- **Purpose**: Read expression pedal input voltage
- **Interface**: Arduino analogRead() or direct AVR ADC registers
- **Operation**: Convert 10-bit ADC value to 8-bit wiper position (0-255)

### 3. UART Driver
- **Purpose**: Receive MIDI bytes at 31250 baud
- **Interface**: Arduino Serial or direct AVR UART registers
- **Configuration**: 31250 baud, 8 data bits, no parity, 1 stop bit

### 4. MIDI Parser
- **Purpose**: Detect MIDI Program Change messages and extract program number
- **Input**: Raw MIDI bytes from UART
- **Output**: Program number (0-127) when valid PC message received
- **State Machine**: Track MIDI message status and data bytes

### 5. EEPROM Routines
- **Purpose**: Read and write 3-byte preset data
- **Interface**: Arduino EEPROM library or direct AVR EEPROM registers
- **Layout**: 128 presets × 3 bytes = 384 bytes total
- **Operations**:
  - `loadPreset(uint8_t presetNum)`: Read 3 bytes from EEPROM
  - `savePreset(uint8_t presetNum, uint8_t values[3])`: Write 3 bytes to EEPROM

### 6. Footswitch Handler
- **Purpose**: Debounce footswitch and detect short press vs long press
- **Debounce**: Require stable state for ~20-50ms
- **Timing**: 
  - Short press: Release before 1 second threshold
  - Long press: Hold for >1 second
- **State Machine**: Track press, hold, and release events

### 7. Main Loop
- **Responsibilities**:
  - Poll ADC for expression pedal position
  - Update active channel's digital potentiometer
  - Check footswitch state
  - Process incoming MIDI messages
  - Handle preset load/save operations
- **Timing**: Non-blocking design to maintain responsive expression pedal control

## Data Flow

### Expression Pedal Update (Continuous)
```
Expression Pedal → ADC → Scale to 0-255 → Write to Active Digipot
```

### Channel Switch (Short Press)
```
Footswitch (short) → Increment Channel (A→B→C→A) → Update Active LED
```

### Preset Recall (MIDI PC)
```
MIDI Input → UART → MIDI Parser → PC Number → Load from EEPROM → Write All 3 Digipots
```

### Preset Save (Long Press)
```
Footswitch (long) → Read All 3 Digipot Values → Save to EEPROM[Last PC Number]
```

## State Management

The firmware maintains the following state variables:

- **Current Active Channel**: Which channel (A, B, or C) is being controlled by the expression pedal
- **Current Wiper Positions**: Three 8-bit values (0-255) for channels A, B, C
- **Last Received PC**: The most recent MIDI program change number (0-127), or invalid if none received
- **Footswitch State**: Current debounced state and press duration

## Memory Usage

- **EEPROM**: 384 bytes for preset storage (128 presets × 3 bytes)
- **SRAM**: Minimal variables for state management
- **Flash**: Program code and constants

The ATmega328P provides:
- 32KB Flash (program memory)
- 2KB SRAM
- 1KB EEPROM

This project uses a small fraction of available resources.
