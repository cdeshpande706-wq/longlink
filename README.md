# LongLink

**A long-range, low-power sensor data collection network built on LoRa.**

Many remote sensor nodes, each carrying several sensors, send one reading
every hour to a single gateway. The gateway stores the data, shows it on a
dashboard, and alerts a person when something needs attention.

> Status: completed 

## How it works

Sensor nodes → (LoRa, 865-867 MHz) → Gateway → Storage → Dashboard and alerts

1. **Sensor nodes** (ESP32 + LoRa module + sensors) sleep for an hour, wake,
   read all sensors, send one packet, and go back to sleep.
2. **Gateway** (ESP32 + LoRa module + Raspberry Pi) listens for packets,
   checks them, confirms receipt, and saves the data.
3. **Dashboard and alerts** show the latest readings and warn when a node
   goes silent or a reading looks wrong.

## Key features

- Hourly reporting with deep sleep for long battery life
- Custom packet format with error checking (CRC)
- Time slots per node, plus random jitter, so packets don't collide
- Acknowledge-and-retry so no reading is lost
- Clock drift correction for the sleep timer
- Silent-node alarm when a node stops reporting

## Hardware

- ESP32 boards (nodes and gateway)
- LoRa module SX1276/SX1278, 865-867 MHz version
- Raspberry Pi (gateway host)
- Sensors: [ BME280 ,MPU-6050 , VL53L0X , DS18B20 (waterproof probe) , NEO-6M GPS (NEO-M8N as upgrade),  ]
- Power: [ solar]

### Sensors (per node)

| Sensor | Measures | Interface | Why it's here |
|---|---|---|---|
| BME280 | Temperature, humidity, pressure | I2C | Core environment data at low cost |
| MPU-6050 | Motion and tilt (accelerometer + gyro) | I2C | Detects if a node is moved, shaken or tampered with |
| VL53L0X | Distance (time-of-flight) | I2C | Water level, object presence, door open/closed |
| DS18B20 (waterproof probe) | Soil or water temperature | 1-Wire | Cheap, rugged ground/water readings |
| NEO-6M GPS (NEO-M8N as upgrade) | Location and precise UTC time | UART | Map view of nodes and a shared time reference |

The three I2C sensors share the same two wires; the DS18B20 uses one extra pin
and the GPS uses two (serial). A regular reading fits in about 15 bytes, and a
reading that includes a GPS position in about 23, which keeps LoRa airtime
(and battery use) low.

**GPS power strategy:** the GPS draws far more power than the other sensors,
so it is not used every hour. A fix is taken at power-up, again when the
MPU-6050 detects movement, and once a day to refresh the time. Other reports
carry the last known position.

### Considered but not used

| Sensor | Reason |
|---|---|
| ESP32-CAM / Grove Vision AI V2 | Images are too large for LoRa's small packets |
| BME680, MPU-9250, VL53L1X, Chirp! | More expensive upgrades; can be swapped in later |
