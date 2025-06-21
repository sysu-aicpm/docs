

这个文档主要用于协定后端与设备交互的接口。设备可以考虑使用 docker 容器或者虚拟机模拟，对外暴露 http 接口，方便后端同学开发。数据传输使用 json 格式。为了实现权限控制，设备模块作为后端的下属模块存在，只接受后端 ip 发送的请求。

实现细节详见 github 仓库：[https://github.com/sysu-aicpm/virtual-device/](https://github.com/sysu-aicpm/virtual-device/)

## 设备类型

对应数据库中  `api_device` 的 `device_type`

- `air_conditioner` 空调
- `refrigerator` 冰箱
- `light` 灯
- `lock` 锁
- `camera` 摄像头

## 查询设备信息

- 路由：/query
- 输入参数：list[key]
- 输出参数：info_dict
- 注释：传入需要的 key，返回一个字典，字典的 key 需要覆盖传入的 key。key 可以包括功耗、状态等

## 控制设备

- 路由：/control
- 输入参数：action（string）、param（dict）
- 输出参数：singal（bool）
- 注释：传入执行的动作 action，并附加需要的参数 param。返回是否成功的标识。如果失败，通过 http 状态码等方式返回错误原因

## 心跳

- 向 `{controller_base_url}/devices/heartbeat/` 发送心跳保活
- 注释：应该周期性向后端程序发送心跳包

心跳包中会包含设备事件

## 局域网发现

- 提供了局域网发现的功能，使用 SSDP，主动在局域网内广播从而被其他主机发现

设备在启动后会开始 SSDP 广播，等待后端发现。

后端在发现设备后，设备会将自身的信息，如设备 ip 地址和端口、设备标识符和名称等发送给后端，后端会将自身地址发送给设备，从而实现后端与设备的对接

之后设备会一直发送心跳给后端，在心跳中记录自身功率、发生事件等信息

## API 文档

粘贴自 [https://github.com/sysu-aicpm/virtual-device/blob/main/docs/API.md](https://github.com/sysu-aicpm/virtual-device/blob/main/docs/API.md)

### HTTP 接口

#### 1. 查询设备信息

**请求**

- 路径：`/query`
- 方法：POST
- Content-Type: application/json
- 请求体：

```json
{
    "keys": ["device_id", "device_type", "power", "status", "temperature", "brightness", "locked", "recording", "resolution"]
}
```

**响应**

- Content-Type: application/json
- 状态码：200（成功）
- 响应体示例（冰箱）：

```json
{
    "device_id": "550e8400-e29b-41d4-a716-446655440000",
    "device_type": "refrigerator",
    "power": 100,
    "status": "online",
    "temperature": 4
}
```

#### 2. 控制设备

**请求**

- 路径：`/control`
- 方法：POST
- Content-Type: application/json

**响应**

- Content-Type: application/json
- 状态码：

  - 200：成功
  - 400：请求格式错误
  - 404：设备不存在
  - 500：内部错误

**各设备支持的控制命令：**

##### 冰箱

```json
{
    "action": "set_temperature",
    "params": {
        "temperature": 4  _// 范围：-20到10℃_
    }
}
```

或

```json
{
    "action": "switch",
    "params": {
        "state": "on"  _// "on" 或 "off"_
    }
}
```

##### 灯

```json
{
    "action": "set_brightness",
    "params": {
        "brightness": 80  _// 范围：0-100_
    }
}
```

或

```json
{
    "action": "switch",
    "params": {
        "state": "on"  _// "on" 或 "off"_
    }
}
```

##### 门锁

```json
{
    "action": "set_lock",
    "params": {
        "state": "lock"  _// "lock" 或 "unlock"_
    }
}
```

##### 摄像头

```json
{
    "action": "set_recording",
    "params": {
        "state": "start"  _// "start" 或 "stop"_
    }
}
```

或

```json
{
    "action": "set_resolution",
    "params": {
        "resolution": "1080p"  _// "720p", "1080p" 或 "4k"_
    }
}
```

### 设备主动发送的数据包

#### 心跳包（包含设备事件）

**请求**

- URL：`{CONTROLLER_URL}/api/v1/devices/heartbeat/`
- 方法：POST
- Content-Type: application/json
- 请求体：

```json
{
    "device_identifier": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2024-03-20T10:30:00.000Z",
    "status": "online",
    "data": {
        "current_power_consumption": 102.5,
        "temperature_change": {
            "temperature": 4.2
        },
        "door_state_change": {
            "door_open": true
        }
    }
}
```

**设备状态说明**

- `online`: 设备在线正常工作
- `offline`: 设备离线
- `error`: 设备出现错误

**各设备事件数据字段**

##### 冰箱

```json
{
    "door_state_change": {
        "door_open": true
    },
    "temperature_change": {
        "temperature": 4.2
    },
    "power_state_change": {
        "power_state": "on"
    }
}
```

##### 灯

```json
{
    "brightness_change": {
        "brightness": 80
    },
    "power_state_change": {
        "power_state": "on"
    }
}
```

##### 门锁

```json
{
    "lock_state_change": {
        "lock_state": "locked"
    },
    "battery_level": {
        "battery": 95.5
    }
}
```

##### 摄像头

```json
{
    "camera_state": {
        "camera_state": "recording"
    },
    "resolution_change": {
        "resolution": "1080p"
    },
    "storage_usage": {
        "storage_used_mb": 250
    }
}
```
