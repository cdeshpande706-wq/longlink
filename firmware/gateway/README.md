# Gateway Firmware

ESP32 + LoRa firmware for the LongLink gateway. It is always listening.

## What the gateway does

- Receives packets from all nodes
- Checks the CRC and rejects corrupted packets
- Sends an acknowledgement, including a timing correction for the node
- Passes valid readings to the Raspberry Pi
- Notices when a node has not reported for over an hour and 15 minutes

## Hardware and pin connections


## Status

completed 
