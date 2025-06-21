用循环压测的方法，针对特定接口进行性能测试。可以使用如 Apifox、postman 等软件的自动测试模块，也可以使用轻量级的 Locust 等软件包，这里主要使用 Locust。

## 用户行为测试———以注册用户为例

当软件知名度得到提升的时候，注册用户数量将迎来增长，现在测试短时间大量的访问这个接口的情况

### 测试准备

- 使用 Apifox 的自动测试

![](../assets/M7Sfbj5EqoaZIgxQZgCcxfUwnRc.png)

### 测试结果

![](../assets/BZzNbMa27oYLkzxPxMJcp4p5nyw.png)

以下是重要数据分析

#### 90% 响应时间

- 数值: 469 ms
- 90% 的请求能够在 469 毫秒内得到响应。这是衡量系统性能的重要指标之一，反映了大多数用户的实际体验

#### 平均响应时间

- 数值: 418 ms
- 系统处理一个注册请求的平均时间为 418 毫秒。对于注册接口来说，这个响应时间在可接受范围内

#### 请求失败率

- 数值: 1.62%
- 在所有请求中，有 1.62% 的请求失败。是一个低的失败率。进一步分析发现，设备性能达到峰值才会导致失败，说明接口很稳定

## 管理员查询行为测试————以查看用户对设备的权限为例

### 测试准备

因为参数位于 URL，使用 Locust 脚本进行性能测试

```python
from locust import HttpUser, task, between
import random
import json

## 管理员 token
ADMIN_TOKEN = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ0b2tlbl90eXBlIjoiYWNjZXNzIiwiZXhwIjoxNzUwMDcwNzMyLCJpYXQiOjE3NTAwNjcxMzIsImp0aSI6IjU5MGUwOTNmNjY5YzRmZWNhNTkxNWRkNmVkZWZlOGE1IiwidXNlcl9pZCI6MX0.C-zVLZn5PIXF4Ju_JRDioAwgkSIjv1tNXx37e3rGn6o"

## 用户ID范围
USER_ID_RANGE = range(1, 3000)

class DevicePermissionTest(HttpUser):
    wait_time = between(1, 3)  ## 每次请求之间等待时间（秒）

    @task
    def check_user_permissions(self):
        user_id = random.choice(list(USER_ID_RANGE))
        url = f"/api/v1/permissions/user/{user_id}/"

        headers = {
            "Authorization": f"Bearer {ADMIN_TOKEN}",
            "Content-Type": "application/json"
        }

        with self.client.get(url, headers=headers, catch_response=True) as response:
            try:
                data = response.json()
                
                ## 判断 success 字段是否存在且为 True
                if isinstance(data, dict) and data.get("success") is True:
                    print(f"[{user_id}] 成功获取权限信息，data: {data['data']}")
                    ## 标记为成功
                else:
                    ## 接口返回了错误或结构异常
                    msg = data.get("message", "未知错误")
                    response.failure(f"接口返回失败：{msg} - 响应内容: {data}")
                    print(f"[{user_id}] 请求失败: {data}")
                    
            except json.JSONDecodeError:
                ## JSON 解析失败
                response.failure("响应内容不是有效的 JSON")
                print(f"[{user_id}] 响应内容不是有效的 JSON，原始内容：{response.text}")
```

![](../assets/KL3QbTLIdolV5axWCRgcat3knZc.png)

### 测试结果

进入 http://localhost:8089/ 测试

![](../assets/UkBIbn3Gno78zux6QatcV1gynmh.png)

- 图像中出现的曲线尖峰，来自于设备性能的变化，总体上看，接口的性能很好，没有失败案例
