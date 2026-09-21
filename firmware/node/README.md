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

## Hardware and pin connections


## Status

completed 
