\[ English | [简体中文](../../../../zh-cn/api/framework/bluetooth/index.md) \]

# Bluetooth API

The openvela Bluetooth framework provides a complete Bluetooth stack interface, supporting Classic Bluetooth (BR/EDR) and Bluetooth Low Energy (BLE), covering everything from low-level connection management to upper-layer application profiles.

## Core Protocols

- **[GAP](bt_gap.md)** (Generic Access Profile) — Device discovery, connection management, pairing and security
- **[GATT](bt_gatt.md)** (Generic Attribute Profile) — BLE data attribute read/write and notifications
- **[Device Management](bt_device.md)** — Remote device pairing, connection, and property queries

## BLE Interfaces

- **[BLE Scanning](bt_le_scan.md)** — BLE device discovery and broadcast data reception
- **[BLE Advertising](bt_le_advertiser.md)** — BLE advertising data transmission and management

## Audio and Media

- **[A2DP](bt_a2dp.md)** (Advanced Audio Distribution Profile) — High-quality stereo music streaming
- **[AVRCP](bt_avrcp.md)** (Audio/Video Remote Control Profile) — Playback control, track change, volume adjustment
- **[HFP](bt_hfp.md)** (Hands-Free Profile) — Bluetooth call functionality

## Data and Peripherals

- **[HID](bt_hid.md)** (Human Interface Device) — Keyboards, mice, game controllers
- **[SPP](bt_spp.md)** (Serial Port Profile) — Data pass-through
- **[PAN](bt_pan.md)** (Personal Area Network) — Network sharing and Bluetooth networking
