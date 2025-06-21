
[https://github.com/sysu-aicpm/mcp-server](https://github.com/sysu-aicpm/mcp-server)

## demo

## 安装依赖

```bash
pip install fastmcp requests
```

## 使用方法

### 启动服务器

```bash
python mcp_server.py --backend http://backend-api-address:8000/api/v1 --token your_auth_token
```

your_auth_token 令牌需要在通过 `POST /auth/login` 登录到后端后获取

参数说明:

- `--backend`: 后端 API 地址，默认为 `http://localhost:8000`
- `--token`: 认证令牌，用于访问后端 API 的授权

### 对话测试

你可以使用 cursor 或者 cline(VSCode 插件)与大模型对话，在其中调用 MCP 服务器。

```json
{
  "mcpServers": {
    "cpm-smarthome": {
      "disabled": false,
      "timeout": 60,
      "transportType": "stdio",
      "command": "python",
      "args": [
        "/path/to/mcp_server.py",
        "--token",
        "your_auth_token"
      ]
    }
  }
}
```

### 可用工具

服务器提供以下工具供大模型使用:

1. `get_config()`: 获取当前配置信息

   - 返回后端 API 地址和认证令牌配置状态
   - 注意：认证令牌不会返回明文
2. `get_device_overview()`: 获取所有设备的概览信息

   - 返回系统中所有设备的基本信息列表
   - 包括设备 ID、名称、状态等信息
3. `get_device_detail(device_id)`: 获取特定设备的详细信息

   - 返回指定设备的详细状态信息
   - 包括设备状态、功耗、运行时间、日志等
4. `control_device(device_id, action, parameters)`: 控制特定设备执行操作

   - 用于向指定设备发送控制命令
   - 具体参数请参考上方设备类型说明
5. `get_device_type_docs(device_type)`: 获取设备类型的控制文档

   - 不指定设备类型时返回所有设备类型的概览
   - 指定设备类型时返回该类型的详细控制文档

## 后端 API 接口

本服务器连接到以下后端 API 接口:

1. 控制设备: `POST /api/v1/devices/{device_id}/control/`
2. 查询设备概要: `GET /api/v1/devices/overview/`
3. 查询设备详情: `GET /api/v1/devices/{device_id}/detail/`
