# 蓝牙 A2DP API

openvela 蓝牙 A2DP（高级音频分发）接口，支持音频流的发送（Source）和接收（Sink）。

头文件：#include "bt_a2dp.h"、#include "bt_a2dp_sink.h"、#include "bt_a2dp_source.h"


## openvela 实现说明

- **双角色支持**：Source（音频发送端）和 Sink（音频接收端）
- **编解码器**：支持 SBC 和 AAC
- **传输模式**：支持硬件卸载（Offloading）和非卸载模式


## 同步接口


### bt_a2dp_sink_unregister_callbacks

```c
bool bt_a2dp_sink_unregister_callbacks(bt_instance_t* ins, void* cookie);
```

取消注册回调函数，停止接收状态变更通知。

**参数**：

- `ins` 蓝牙客户端实例。
- `cookie` 用户上下文。


**返回值**：

成功时返回回调 cookie，失败或已注册时返回 NULL。


### bt_a2dp_sink_is_connected

```c
bool bt_a2dp_sink_is_connected(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

检查是否已连接。


### bt_a2dp_sink_is_playing

```c
bool bt_a2dp_sink_is_playing(bt_instance_t* ins, bt_address_t* addr);
```

检查是否正在播放。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

检查是否正在播放。


### bt_a2dp_sink_get_connection_state

```c
profile_connection_state_t bt_a2dp_sink_get_connection_state(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 远程设备蓝牙地址。


**返回值**：



### bt_a2dp_sink_connect

```c
bt_status_t bt_a2dp_sink_connect(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

建立连接。


### bt_a2dp_sink_disconnect

```c
bt_status_t bt_a2dp_sink_disconnect(bt_instance_t* ins, bt_address_t* addr);
```

断开与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

断开连接。


### bt_a2dp_source_unregister_callbacks

```c
bool bt_a2dp_source_unregister_callbacks(bt_instance_t* ins, void* cookie);
```

取消注册回调函数，停止接收状态变更通知。

**参数**：

- `ins` 蓝牙客户端实例。
- `cookie` 用户上下文。


**返回值**：

成功时返回回调 cookie，失败或已注册时返回 NULL。


### bt_a2dp_source_is_connected

```c
bool bt_a2dp_source_is_connected(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

检查是否已连接。


### bt_a2dp_source_is_playing

```c
bool bt_a2dp_source_is_playing(bt_instance_t* ins, bt_address_t* addr);
```

检查是否正在播放。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

检查是否正在播放。


### bt_a2dp_source_get_connection_state

```c
profile_connection_state_t bt_a2dp_source_get_connection_state(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 远程设备蓝牙地址。


**返回值**：



### bt_a2dp_source_connect

```c
bt_status_t bt_a2dp_source_connect(bt_instance_t* ins, bt_address_t* addr);
```

发起与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 蓝牙地址 of the peer device.

**返回值**：

建立连接。


### bt_a2dp_source_disconnect

```c
bt_status_t bt_a2dp_source_disconnect(bt_instance_t* ins, bt_address_t* addr);
```

断开与远程设备的连接。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 远程设备蓝牙地址。


**返回值**：

成功时返回 BT_STATUS_SUCCESS，失败时返回错误码。


### bt_a2dp_source_set_silence_device

```c
bt_status_t bt_a2dp_source_set_silence_device(bt_instance_t* ins, bt_address_t* addr, bool silence);
```

设置静音设备。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 远程设备蓝牙地址。
- `silence` 是否设为静音模式（true 为静音）。


**返回值**：

成功时返回 BT_STATUS_SUCCESS，失败时返回错误码。


### bt_a2dp_source_set_active_device

```c
bt_status_t bt_a2dp_source_set_active_device(bt_instance_t* ins, bt_address_t* addr);
```

设置活跃设备。

**参数**：

- `ins` 蓝牙客户端实例。
- `addr` 远程设备蓝牙地址。


**返回值**：

成功时返回 BT_STATUS_SUCCESS，失败时返回错误码。
