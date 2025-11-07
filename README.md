# -------Mod
- 万界修正群--了不起的修仙模拟器/觅长生--联机Mod，已经是第三代修复的联机Mod了
- QQ群：115405294(开发者/自建服爱好者) 746876665(普通用户)
# -------WinServer_For_Docker

- 客户端v1.19、v1.20系列对应的服务器端版本号v1.12，基础功能可以聊天、发送物品
- 客户端v6.4 对应的服务器端版本号v4.6
- 如需自建服务器主要客户端版本号和服务器版本号一致

# -------在Doker中部署win11系统来运行Server
##  docker compose
```docker compose
services:
  windows-11:
    image: ghcr.io/aspnmy/xchat_server:v1.12_amazingcultivationsimulator_win11
    container_name: windows
    environment:
      DISK_SIZE: "30G"
      VERSION: "11l"
      RAM_SIZE: "4G"
      CPU_CORES: "4"
      LANGUAGE: "Chinese"
      REGION: "en-US"
      KEYBOARD: "en-US"
      USERNAME: "aspnmy" #默认镜像默认名
      PASSWORD: "caixia" #默认镜像默认密码
    devices:
      - /dev/kvm
      - /dev/net/tun
    cap_add:
      - NET_ADMIN
    ports:
      - 7818:8181/tcp
      - 7818:8181/udp
      - 7777:8006
      - 7489:3389/tcp
      - 7489:3389/udp
    volumes:
      - win11:/storage
    restart: always
    stop_grace_period: 2m
volumes:
  win11:
```
- 使用上面的文件在本地docker环境中安装winxp系统
- 运行 IP+7777 访问winXP的桌面
- 复制项目中Server中对应的服务器端到winXP中，运行对应服务器端
##  docker cli
``` dokcer cli
docker volume create win11
docker run --name windows -e DISK_SIZE=10G -e VERSION=11l -e RAM_SIZE=4G -e CPU_CORES=4 -e LANGUAGE=Chinese -e REGION=en-US -e KEYBOARD=en-US -e USERNAME=aspnmy -e PASSWORD=caixia --device /dev/kvm --device /dev/net/tun --cap-add NET_ADMIN -p 7818:8181 -p 7777:8006 -p 7489:3389/tcp -p 7489:3389/udp -v win11:/storage --restart always --stop-timeout 2m ghcr.io/aspnmy/xchat_server:v1.12_amazingcultivationsimulator_win11

```

## 如何发布自己的XChat-server服务器给大家使用

###  - 1、自建XChat-server服务器注册到ServerLists公共节点上(需要一点编程能力)
将自建的服务器的ServerHost、ServerPort注册到UpdateServerLists接口上
  - 公共节点URl：（US） https://api.caixiagame.us.earth-oline.org (默认)
  - 公共节点URl：（CN） https://api.caixiagame.cn.earth-oline.org
  - 公共节点URl：（TW） https://api.caixiagame.tw.earth-oline.org
  - 推荐使用US的默认节点，其他节点只用于连接不到默认节点时的备用，数据互为镜像，只为边缘节点加速使用
  - 参考下方注册接口的使用形式注册自建的Server到公共ServerLists上
### - 2、从 https://github.com/aspnmy/Docker_XChat_Server/releases 下载对应的服务器端

### - 3、`/api?type=ServerSets` 所有需要存储与服务器的功能，都需要从此接口获取配置数据，Steam客户端版本号v3.5开始，部分UI数据缓存在服务器端，不进行`/api?type=ServerSets`验证的情况下，默认功能只能实现“聊天室发送消息，发送物品”的基础能力；
在验证`/api?type=ServerSets`以后，可以进一步使用 发送NPC 、 跨地图事件、 加载公共地图事件、跨用户结婚、公共福利院、藏经阁(不需要第三方mod支持)、公共摆摊市场、发布悬赏、万界阁(基于服务器端的Mod修改器，可以生成Server_ModLists中所拥有的Mod中的物品ID写入到用户的存档文件中，并且云端保留唯一一个存档文件，不和Steam云存档冲突)

### - 4、 当前可以使用Steam MOD客户端下载地址
  - https://steamcommunity.com/sharedfiles/filedetails/?id=3557958666
## 接口概述

本文档详细介绍API配置服务器提供的接口，特别是`UpdateServerLists`接口的使用方法。

## UpdateServerLists接口

### 功能描述

用于更新服务器列表。该接口会：
- 读取ServerLists列表中现有的服务器列表
- 与传入的新服务器数据进行比较（基于ServerHost和ServerPort）
- 只追加不存在的服务器，避免重复添加
- 保存更新后的完整服务器列表

### 接口信息

- **URL**: `/api?type=UpdateServerLists`
- **方法**: `POST`
- **Content-Type**: `application/json`
- **允许跨域**: 是

### 请求参数

请求体必须是一个JSON数组，包含一个或多个服务器对象。每个服务器对象包含以下字段：

| 字段名 | 类型 | 必需 | 描述 |
|-------|------|------|------|
| ServerName | String | 是 | 服务器名称 |
| ServerHost | String | 是 | 服务器主机地址 |
| ServerPort | String | 是 | 服务器端口号 |

### 请求示例

```json
[
  {
    "ServerName": "测试服务器01",
    "ServerHost": "test.server.com",
    "ServerPort": "37667"
  },
  {
    "ServerName": "测试服务器02",
    "ServerHost": "test2.server.com",
    "ServerPort": "37667"
  }
]
```

### 响应格式

接口返回标准的JSON格式，包含错误码和消息：

#### 成功响应

```json
{
  "Error": 200,
  "Message": "服务器列表已更新"
}
```

#### 错误响应

1. **请求格式错误**
   ```json
   {
     "Error": 400,
     "Message": "提交的服务器列表格式不正确"
   }
   ```

2. **服务器错误**
   ```json
   {
     "Error": 500,
     "Message": "更新服务器列表失败，请联系管理员"
   }
   ```

### 特别说明

- 接口会自动过滤掉与现有服务器列表中ServerHost和ServerPort完全相同的服务器条目
- 如果ServerLists列表中没有服务器列表数据，将直接创建新的列表
- 如果ServerLists列表中的数据格式不正确，将使用空数组作为基础进行追加

## 其他可用接口

### 1. ServerLists 接口

用于获取服务器列表。

- **URL**: `/api?type=ServerLists`
- **方法**: `GET`
- **返回格式**: `{"Error": 200, "Message": [服务器列表数组]}`

### 2. ServerSets 接口

用于获取服务器配置（需要认证）。

- **URL**: `/api?type=ServerSets`
- **方法**: `GET`
- **认证头**: `X-Auth-Key`
- **返回格式**: `{"Error": 200, "Message": 配置对象}`

## 使用示例代码

### JavaScript 示例

```javascript
// 更新服务器列表示例
async function updateServerLists() {
  const servers = [
    {
      "ServerName": "我的新服务器",
      "ServerHost": "myserver.com",
      "ServerPort": "37667"
    }
  ];
  
  try {
    const response = await fetch('https://your-api-domain.com/api?type=UpdateServerLists', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(servers)
    });
    
    const data = await response.json();
    console.log('更新结果:', data);
    
    if (data.Error === 200) {
      console.log('服务器列表更新成功');
    } else {
      console.error('更新失败:', data.Message);
    }
  } catch (error) {
    console.error('请求错误:', error);
  }
}

// 调用函数
updateServerLists();
```

### Python 示例

```python
import requests
import json

def update_server_lists():
    url = "https://your-api-domain.com/api?type=UpdateServerLists"
    servers = [
        {
            "ServerName": "Python测试服务器",
            "ServerHost": "python.server.com",
            "ServerPort": "37667"
        }
    ]
    
    try:
        response = requests.post(
            url,
            headers={"Content-Type": "application/json"},
            data=json.dumps(servers)
        )
        data = response.json()
        print(f"更新结果: {data}")
        
        if data.get("Error") == 200:
            print("服务器列表更新成功")
        else:
            print(f"更新失败: {data.get('Message')}")
    except Exception as e:
        print(f"请求错误: {e}")

# 调用函数
update_server_lists()
```

## 故障排除

1. **请求格式错误**：确保请求体是有效的JSON数组格式
2. **更新失败**：检查服务器网络连接和ServerLists列表服务状态
3. **数据重复**：如果添加了相同的服务器信息，接口会自动忽略，这是正常行为

如有其他问题，请联系系统管理员。

  
