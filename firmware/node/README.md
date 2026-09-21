# Sensor Node Firmware

ESP32 firmware for a LongLink sensor node.

## What the node does each cycle

1. Wake up from deep sleep
2. Read all sensors and the battery level
3. Build a packet
4. Wait for its assigned time slot (plus a small random delay)
5. Send the packet over LoRa
6. Wait for the gateway's acknowledgement, and retry if none arrives
7. Go back to deep sleep for one hour

## Draft packet format (will change as the project grows)

| Field | Purpose |
|---|---|
| Node ID | Which node sent this |
| Sequence number | Detects missing packets |
| Sensor values | The readings |
| Battery level | Warns before the node dies |
| CRC | Detects corrupted packets |

## Code structure

Each sensor has its own module (`.hpp` = interface, `.cpp` = code) in `src/sensors/`:
`bmp180`, `ds18b20`, `tof` (VL53L0X), `mpu6050`, and `gps_sim` (simulated for now).
`packet` builds the packet and CRC, `radio_sim` simulates the radio, and
`config.hpp` holds the pins and settings for both the ESP32 and the ESP32-S3.

## Packet flags byte

| Bit | Meaning |
|---|---|
| 0 | Node was moved |
| 1 | Packet carries a GPS position |
| 2-5 | BMP180, DS18B20, VL53L0X, MPU-6050 gave a real reading |

## Hardware and pin connections

Two boards are supported. `config.hpp` picks the pins automatically:
- **ESP32 DevKit** (ESP32-WROOM-32): the real hardware build
- **ESP32-S3**: the Cirkit Designer simulation

All sensors run from 3.3 V. Every wire below goes from the device pin to the ESP32 pin shown.

| Device | Device pin | ESP32 (real) | ESP32-S3 (simulation) | Notes |
|---|---|---|---|---|
| BMP180 (BME280 later) | VCC | 3V3 | 3V3 | Same four wires for either sensor |
| | GND | GND | GND | |
| | SDA | GPIO 21 | GPIO 8 | I2C, address 0x77 |
| | SCL | GPIO 22 | GPIO 9 | |
| MPU-6050 | VCC | 3V3 | 3V3 | |
| | GND | GND | GND | |
| | AD0 | GND | GND | Sets I2C address 0x68 |
| | SDA | GPIO 21 | GPIO 8 | |
| | SCL | GPIO 22 | GPIO 9 | |
| | INT | GPIO 27 | GPIO 5 | Motion interrupt. In the simulation a pushbutton to GND stands in for it |
| VL53L0X | VCC | 3V3 | 3V3 | |
| | GND | GND | GND | |
| | SDA | GPIO 21 | GPIO 8 | I2C, address 0x29 |
| | SCL | GPIO 22 | GPIO 9 | XSHUT is not used; leave it unconnected |
| DS18B20 | VCC | 3V3 | 3V3 | |
| | GND | GND | GND | |
| | DQ (data) | GPIO 4 | GPIO 4 | 4.7 kΩ resistor between DQ and 3V3 |

The BMP180, MPU-6050 and VL53L0X share the same SDA and SCL wires; their
different addresses keep them apart.

### Simulation stand-ins (not needed on real hardware)

If a sensor is missing from the simulator, a potentiometer stands in for it:
outer pins to 3V3 and GND, middle pin to the GPIO below.

| Stand-in for | ESP32-S3 pin |
|---|---|
| VL53L0X (distance) | GPIO 1 |
| DS18B20 (soil temperature) | GPIO 2 |

### Planned (not in the code yet)

The GPS and the LoRa radio are simulated in code for now. Their real wiring:

| Device | Device pin | ESP32 pin | Notes |
|---|---|---|---|
| NEO-6M GPS | VCC | 3V3 | Most modules also accept 5 V |
| | GND | GND | |
| | TX | GPIO 16 | GPS transmit goes to ESP32 receive |
| | RX | GPIO 17 | GPS receive goes to ESP32 transmit |
| LoRa SX1276/78 | VCC | 3V3 | 3.3 V only, never 5 V |
| | GND | GND | |
| | SCK | GPIO 18 | SPI clock |
| | MISO | GPIO 19 | |
| | MOSI | GPIO 23 | |
| | NSS (CS) | GPIO 5 | |
| | RST | GPIO 14 | |
| | DIO0 | GPIO 26 | Packet-received interrupt |

### Pin rules followed

- GPIO 6-11 are used by the ESP32's flash memory and are left alone
- GPIO 34-39 are input-only, so none are used as outputs
- GPIO 27 (real) and GPIO 5 (simulation) are used for the motion wake-up because they can wake the chip from deep sleep

## Status

completed 
