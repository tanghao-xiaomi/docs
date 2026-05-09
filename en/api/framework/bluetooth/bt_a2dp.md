\[ English | [简体中文](../../../../zh-cn/api/framework/bluetooth/bt_a2dp.md) \]

# Bluetooth A2DP API

The openvela Bluetooth A2DP (Advanced Audio Distribution Profile) interface supports audio stream sending (Source) and receiving (Sink).

Header files: #include "bt_a2dp.h", #include "bt_a2dp_sink.h", #include "bt_a2dp_source.h"


## openvela Implementation Notes

- **Dual-role support**: Source (audio sender) and Sink (audio receiver)
- **Codecs**: Supports SBC and AAC
- **Transport modes**: Supports hardware offloading and non-offloading modes


## Synchronous Interfaces


### bt_a2dp_sink_unregister_callbacks

```c
bool bt_a2dp_sink_unregister_callbacks(bt_instance_t* ins, void* cookie);
```

Unregister callback functions and stop receiving state change notifications.

**Parameters**:

- `ins` Bluetooth client instance.
- `cookie` User context.


**Returns**:

Returns the callback cookie on success, or NULL on failure or if already registered.


### bt_a2dp_sink_is_connected

```c
bool bt_a2dp_sink_is_connected(bt_instance_t* ins, bt_address_t* addr);
```

Check whether the A2DP Sink is connected to the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns true if connected, false otherwise.


### bt_a2dp_sink_is_playing

```c
bool bt_a2dp_sink_is_playing(bt_instance_t* ins, bt_address_t* addr);
```

Check whether audio is currently playing.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns true if playing, false otherwise.


### bt_a2dp_sink_get_connection_state

```c
profile_connection_state_t bt_a2dp_sink_get_connection_state(bt_instance_t* ins, bt_address_t* addr);
```

Get the A2DP Sink connection state with the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the remote device.


**Returns**:

Returns the current connection state.


### bt_a2dp_sink_connect

```c
bt_status_t bt_a2dp_sink_connect(bt_instance_t* ins, bt_address_t* addr);
```

Initiate an A2DP Sink connection to the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.


### bt_a2dp_sink_disconnect

```c
bt_status_t bt_a2dp_sink_disconnect(bt_instance_t* ins, bt_address_t* addr);
```

Disconnect the A2DP Sink connection from the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.


### bt_a2dp_source_unregister_callbacks

```c
bool bt_a2dp_source_unregister_callbacks(bt_instance_t* ins, void* cookie);
```

Unregister callback functions and stop receiving state change notifications.

**Parameters**:

- `ins` Bluetooth client instance.
- `cookie` User context.


**Returns**:

Returns the callback cookie on success, or NULL on failure or if already registered.


### bt_a2dp_source_is_connected

```c
bool bt_a2dp_source_is_connected(bt_instance_t* ins, bt_address_t* addr);
```

Check whether the A2DP Source is connected to the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns true if connected, false otherwise.


### bt_a2dp_source_is_playing

```c
bool bt_a2dp_source_is_playing(bt_instance_t* ins, bt_address_t* addr);
```

Check whether audio is currently playing.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns true if playing, false otherwise.


### bt_a2dp_source_get_connection_state

```c
profile_connection_state_t bt_a2dp_source_get_connection_state(bt_instance_t* ins, bt_address_t* addr);
```

Get the A2DP Source connection state with the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the remote device.


**Returns**:

Returns the current connection state.


### bt_a2dp_source_connect

```c
bt_status_t bt_a2dp_source_connect(bt_instance_t* ins, bt_address_t* addr);
```

Initiate an A2DP Source connection to the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the peer device.

**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.


### bt_a2dp_source_disconnect

```c
bt_status_t bt_a2dp_source_disconnect(bt_instance_t* ins, bt_address_t* addr);
```

Disconnect the A2DP Source connection from the remote device.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the remote device.


**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.


### bt_a2dp_source_set_silence_device

```c
bt_status_t bt_a2dp_source_set_silence_device(bt_instance_t* ins, bt_address_t* addr, bool silence);
```

Set the device to silence mode.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the remote device.
- `silence` Whether to enable silence mode (true for silent).


**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.


### bt_a2dp_source_set_active_device

```c
bt_status_t bt_a2dp_source_set_active_device(bt_instance_t* ins, bt_address_t* addr);
```

Set the active device for A2DP Source.

**Parameters**:

- `ins` Bluetooth client instance.
- `addr` Bluetooth address of the remote device.


**Returns**:

Returns BT_STATUS_SUCCESS on success, or an error code on failure.
