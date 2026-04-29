# 蓝牙 API

openvela 蓝牙框架提供完整的蓝牙协议栈接口，支持经典蓝牙（BR/EDR）和低功耗蓝牙（BLE），涵盖从底层连接管理到上层应用规范。

## 核心协议

- **[GAP](bt_gap.md)**（通用访问规范）— 设备发现、连接管理、配对与安全
- **[GATT](bt_gatt.md)**（通用属性规范）— BLE 数据属性读写与通知
- **[设备管理](bt_device.md)** — 远程设备配对、连接、属性查询

## BLE 接口

- **[BLE 扫描](bt_le_scan.md)** — BLE 设备发现与广播数据接收
- **[BLE 广播](bt_le_advertiser.md)** — BLE 广播数据发送与管理

## 音频与媒体

- **[A2DP](bt_a2dp.md)**（高级音频分发）— 高质量立体声音乐传输
- **[AVRCP](bt_avrcp.md)**（音视频远程控制）— 播放控制、切歌、音量调节
- **[HFP](bt_hfp.md)**（免提规范）— 蓝牙通话功能

## 数据与外设

- **[HID](bt_hid.md)**（人机接口设备）— 键盘、鼠标、游戏手柄
- **[SPP](bt_spp.md)**（串口仿真）— 数据透传
- **[PAN](bt_pan.md)**（个人局域网）— 网络共享与蓝牙组网

```eval_rst

.. toctree::
    :maxdepth: 2

    bt_gap
    bt_device
    bt_gatt
    bt_le_scan
    bt_le_advertiser
    bt_a2dp
    bt_avrcp
    bt_hfp
    bt_hid
    bt_spp
    bt_pan

```
