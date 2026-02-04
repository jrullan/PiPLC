# Explorer HAT Pro I/O Provider Guide

## Overview

The Explorer HAT Pro provider gives PiPLC access to all hardware I/O on the [Pimoroni Explorer HAT Pro](https://shop.pimoroni.com/products/explorer-hat) add-on board for Raspberry Pi. Unlike the generic GPIO provider, this provider has a fixed hardware mapping with no pin configuration required — select the provider type and start programming.

The board provides digital inputs, digital outputs (via ULN2003A Darlington driver), LEDs, capacitive touch pads (CAP1208 over I2C), analog inputs (ADS1015 ADC over I2C), and two motor outputs (DRV8833 dual H-bridge with software PWM).

## Requirements

- Raspberry Pi with Explorer HAT Pro attached
- PiPLC ARM64 `.deb` package (Debian Trixie)
- I2C enabled on the Raspberry Pi (`sudo raspi-config` > Interface Options > I2C)

## Hardware Setup

1. Attach the Explorer HAT Pro to the Raspberry Pi GPIO header.
2. Enable I2C if not already enabled:
   ```bash
   sudo raspi-config nonint do_i2c 0
   ```
3. Verify the board is detected:
   ```bash
   i2cdetect -y 1
   ```
   You should see devices at addresses `0x28` (CAP1208 touch controller) and `0x48` (ADS1015 ADC).

## PLC Memory Map

The Explorer HAT Pro maps to fixed PLC addresses. No `.ioconfig` pin configuration is needed.

### Digital Inputs (`I:` region)

| Address | Hardware | Notes |
|---------|----------|-------|
| `I:0/0` | Digital Input 1 | GPIO 23 — 5V-tolerant buffered input |
| `I:0/1` | Digital Input 2 | GPIO 22 |
| `I:0/2` | Digital Input 3 | GPIO 24 |
| `I:0/3` | Digital Input 4 | GPIO 25 |
| `I:0/4` | Touch Pad 1 | CAP1208 channel 5 (I2C) |
| `I:0/5` | Touch Pad 2 | CAP1208 channel 6 |
| `I:0/6` | Touch Pad 3 | CAP1208 channel 7 |
| `I:0/7` | Touch Pad 4 | CAP1208 channel 8 |
| `I:0/8` | Crocodile Clip 5 | CAP1208 channel 1 (I2C) |
| `I:0/9` | Crocodile Clip 6 | CAP1208 channel 2 |
| `I:0/10` | Crocodile Clip 7 | CAP1208 channel 3 |
| `I:0/11` | Crocodile Clip 8 | CAP1208 channel 4 |

### Digital Outputs (`O:` region)

| Address | Hardware | Notes |
|---------|----------|-------|
| `O:0/0` | Digital Output 1 | GPIO 6 — via ULN2003A sink driver |
| `O:0/1` | Digital Output 2 | GPIO 12 |
| `O:0/2` | Digital Output 3 | GPIO 13 |
| `O:0/3` | Digital Output 4 | GPIO 16 |
| `O:0/4` | LED 1 (Blue) | GPIO 4 |
| `O:0/5` | LED 2 (Yellow) | GPIO 17 |
| `O:0/6` | LED 3 (Red) | GPIO 27 |
| `O:0/7` | LED 4 (Green) | GPIO 5 |
| `O:0/8` | Motor 1 Forward | GPIO 19 — PWM-modulated by speed word |
| `O:0/9` | Motor 1 Backward | GPIO 20 — PWM-modulated by speed word |
| `O:0/10` | Motor 2 Forward | GPIO 21 — PWM-modulated by speed word |
| `O:0/11` | Motor 2 Backward | GPIO 26 — PWM-modulated by speed word |

### Analog Inputs (`N:` region, read)

| Address | Hardware | Range | Notes |
|---------|----------|-------|-------|
| `N:0` | ADC Channel 1 | 0 – 2047 | ADS1015 12-bit, 0–3.3V |
| `N:1` | ADC Channel 2 | 0 – 2047 | |
| `N:2` | ADC Channel 3 | 0 – 2047 | |
| `N:3` | ADC Channel 4 | 0 – 2047 | |
| `N:4` | Motor 1 Speed | 0 – 100 | Readback of current duty cycle |
| `N:5` | Motor 2 Speed | 0 – 100 | Readback of current duty cycle |

### Analog Outputs (`N:` region, write)

| Address | Hardware | Range | Notes |
|---------|----------|-------|-------|
| `N:4` | Motor 1 Speed | 0 – 100 | PWM duty cycle percentage |
| `N:5` | Motor 2 Speed | 0 – 100 | PWM duty cycle percentage |

## Selecting the Provider

### On the Raspberry Pi (local)

1. Open the I/O Config panel (View > I/O Config).
2. Select **Explorer HAT Pro** from the Provider Type combo box.
3. Click **Apply** (engine must be stopped).

No further pin configuration is needed. The digital pin and analog channel tables are disabled since all mappings are fixed by the hardware.

### From a Remote Windows/Linux Editor

1. Connect to the Raspberry Pi engine (Connection > Connect to Remote Engine).
2. Open the I/O Config panel.
3. Select **Explorer HAT Pro** and click **Apply**.

The provider type is available on all platforms for configuring remote engines. The hardware driver runs on the Raspberry Pi side.

### Via Command Line

Create an `.ioconfig` file:

```json
{
    "version": 1,
    "provider": "explorer_hat_pro"
}
```

Load it when starting the engine:

```bash
piplc-engine --ioconfig /path/to/explorer-hat.ioconfig
```

## Motor Control

Motors are controlled through a combination of direction bits and speed words.

### Direction

Set motor direction using the output bits:

| Desired Motion | O:0/8 (M1 Fwd) | O:0/9 (M1 Bwd) | Behavior |
|----------------|-----------------|-----------------|----------|
| Forward | ON | OFF | Motor 1 runs forward |
| Backward | OFF | ON | Motor 1 runs backward |
| Coast (free) | OFF | OFF | Motor 1 coasts to stop |
| Brake | ON | ON | Motor 1 brakes (both driven) |

Motor 2 uses `O:0/10` (forward) and `O:0/11` (backward) identically.

### Speed

Set motor speed by writing a value 0–100 to the speed word:

- `N:4` — Motor 1 speed (0% = stopped, 100% = full speed)
- `N:5` — Motor 2 speed

The software PWM runs at approximately 1 kHz. The direction bits gate which GPIO receives the PWM signal.

### Example: Run Motor 1 Forward at 75%

```
Rung 0:  |--[ XIC I:0/0 ]--[ OTE O:0/8 ]--|   (Input 1 enables forward)
Rung 1:  |--[ XIC I:0/0 ]--[ MOV 75 N:4 ]--|   (Set speed to 75%)
Rung 2:  |--[ XIO I:0/0 ]--[ MOV 0  N:4 ]--|   (Clear speed when stopped)
```

When Input 1 is pressed, Motor 1 runs forward at 75% duty cycle. When released, the speed is set to 0 and the direction bit clears.

## Analog Input (ADC)

The four ADC channels read voltage on the Explorer HAT Pro's analog input pads (labeled 1–4 on the board). The ADS1015 returns a 12-bit value. In PiPLC, the raw 12-bit value (0–2047) is stored directly in `N:0` through `N:3`.

### Example: Threshold Detection

```
Rung 0:  |--[ GRT N:0 1000 ]--[ OTE O:0/4 ]--|   (Blue LED on if ADC1 > 1000)
```

### Voltage Conversion

To convert the raw ADC value to millivolts (0–3300 mV range):

```
Rung 0:  |--[ MUL N:0 3300 N:10 ]--|   (raw * 3300)
Rung 1:  |--[ DIV N:10 2047 N:11 ]--|   (÷ 2047 = millivolts)
```

## Touch Pads and Crocodile Clips

The CAP1208 provides 8 capacitive touch channels over I2C. The board labels them 1–8: touch pads 1–4 (`I:0/4` – `I:0/7`, CAP1208 channels 5–8) and crocodile clips 5–8 (`I:0/8` – `I:0/11`, CAP1208 channels 1–4). The crocodile clips can sense touch on conductive objects connected to the clips.

All 8 channels behave like digital inputs — ON when touched, OFF when released. The CAP1208 handles debouncing internally.

### Example: Touch-Activated LED

```
Rung 0:  |--[ XIC I:0/4 ]--[ OTE O:0/4 ]--|   (Touch pad 1 lights blue LED)
Rung 1:  |--[ XIC I:0/5 ]--[ OTE O:0/5 ]--|   (Touch pad 2 lights yellow LED)
```

## Safe State Behavior

When the PLC engine stops or faults, `applySafeState()` is called automatically:

- All digital outputs (1–4) are driven LOW
- All LEDs are turned OFF
- Motor speeds are set to 0%
- Motor direction enables are cleared
- The PWM thread stops modulating

This ensures motors and outputs are de-energized on any fault condition.

## Troubleshooting

### "I2C device not found" on initialize

- Verify I2C is enabled: `sudo raspi-config nonint get_i2c` (should return `0`)
- Check the board is detected: `i2cdetect -y 1`
- Ensure `/dev/i2c-1` exists and the user has permission (add user to `i2c` group: `sudo usermod -aG i2c $USER`)

### Touch pads not responding

- The CAP1208 requires direct skin contact with the conductive pads on the board.
- If the product ID check fails during initialization, the provider will log an error. Check `journalctl -u piplc-engine` for details.

### Motor runs but speed doesn't change

- Verify the speed word (`N:4` or `N:5`) is being written each scan. A `MOV` instruction must continuously set the value.
- The direction bit (`O:0/8–11`) must also be ON for the corresponding motor direction — the PWM only modulates GPIOs that have their enable bit set.

### ADC readings are noisy or zero

- Check wiring to the analog input pads on the board.
- The ADS1015 uses single-shot conversion mode. Each channel is read sequentially during the scan cycle, which adds a small amount of scan time.

## See Also

- [Memory Regions](../reference/memory-regions.md) — PLC address format reference
- [Instruction Reference](../reference/instructions.md) — All 30 instruction types
- [GPIO Architecture](../architecture/gpio.md) — I/O provider architecture
- [Getting Started](../getting-started.md) — Installation and setup
