# API参考

本文档提供沙盒管理系统API的详细参考。系统提供RESTful API，返回JSON格式的数据。

## 认证

所有API请求都需要通过认证。请在HTTP请求头中添加API密钥：

```
X-API-KEY: <您的API密钥>
```

## 通用响应格式

所有API响应采用统一的格式：

```json
{
  "success": true|false,
  "data": {...},
  "error": {
    "code": "错误代码",
    "message": "错误消息",
    "details": {}
  },
  "timestamp": 1625097600000
}
```

## 沙盒管理

### 创建沙盒

- **接口说明**: 创建新的沙盒实例
- **HTTP方法**: POST
- **请求路径**: `/api/v1/sandboxes`
- **请求体**: 
  ```json
  {
    "userId": "用户ID",
    "mcpVersion": "1.0(可选)",
    "imageId": "镜像ID(可选) , 只有当镜像状态是AVAILABLE镜像才可用",
    "instanceSpecification": "实例规格，如1u1g（可选）"
  }
  ```
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "sandboxId": "沙盒ID",
      "mcpUrl": "MCP服务URL",
      "vncUrl": "VNC服务URL",
      "cdpUrl": "CDP服务URL",
      "status": "沙盒状态"
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 201 Created

### 获取沙盒列表

- **接口说明**: 获取沙盒列表
- **HTTP方法**: GET
- **请求路径**: `/api/v1/sandboxes`
- **请求参数**:
  - `userId`: 用户ID（可选）
  - `status`: 沙盒状态过滤（可选）
  - `page`: 页码（默认值：1）
  - `size`: 每页大小（默认值：20）
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "total": 10,
      "pages": 1,
      "currentPage": 1,
      "sandboxes": [
        {
          "sandboxId": "沙盒ID",
          "userId": "用户ID",
          "instanceId": "EC2实例ID",
          "status": "沙盒状态",
          "publicIp": "公网IP",
          "instanceType": "实例类型",
          "tags": {},
          "createdAt": "2023-01-01T00:00:00Z",
          "lastModified": "2023-01-01T00:00:00Z"
        }
      ]
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

### 获取沙盒详情

- **接口说明**: 获取沙盒详情
- **HTTP方法**: GET
- **请求路径**: `/api/v1/sandboxes/{sandboxId}`
- **路径参数**:
  - `sandboxId`: 沙盒ID
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "sandboxId": "沙盒ID",
      "mcpUrl": "MCP服务URL",
      "vncUrl": "VNC服务URL",
      "cdpUrl": "CDP服务URL",
      "status": "沙盒状态",
      "apiKeyId": "API密钥ID"
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

### 停止沙盒

- **接口说明**: 停止沙盒
- **HTTP方法**: POST
- **请求路径**: `/api/v1/sandboxes/{sandboxId}/stop`
- **路径参数**:
  - `sandboxId`: 沙盒ID
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "sandboxId": "沙盒ID",
      "userId": "用户ID",
      "status": "STOPPED",
      "publicIp": "公网IP",
      "uptime": 3600,
      "message": "沙盒已停止",
      "lastUpdated": "2023-01-01T12:00:00",
      "imageId": "镜像ID"
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

### 获取沙盒状态

- **接口说明**: 获取沙盒状态
- **HTTP方法**: GET
- **请求路径**: `/api/v1/sandboxes/{sandboxId}/status`
- **路径参数**:
  - `sandboxId`: 沙盒ID
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "sandboxId": "沙盒ID",
      "userId": "用户ID",
      "status": "RUNNING",
      "publicIp": "公网IP",
      "uptime": 3600,
      "message": "沙盒运行中",
      "lastUpdated": "2023-01-01T12:00:00",
      "imageId": "镜像ID"
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

## 镜像管理

### 获取镜像列表

- **接口说明**: 获取镜像列表
- **HTTP方法**: GET
- **请求路径**: `/api/v1/sandbox-images`
- **请求参数**:
  - `userId`: 用户ID（必填）
  - `status`: 镜像状态过滤（可选）
  - `page`: 页码（默认值：1）
  - `size`: 每页大小（默认值：20）
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "total": 10,
      "pages": 1,
      "currentPage": 1,
      "images": [
        {
          "imageId": "镜像ID",
          "awsImageId": "AWS镜像ID",
          "name": "镜像名称",
          "description": "镜像描述",
          "status": "镜像状态",
          "sourceInstanceId": "源实例ID",
          "sourceSandboxId": "源沙盒ID",
          "userId": "用户ID",
          "region": "区域",
          "isDefault": false,
          "storageSize": 8,
          "createdAt": "2023-01-01T12:00:00",
          "lastUpdated": "2023-01-01T12:00:00"
        }
      ]
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

### 获取镜像详情

- **接口说明**: 获取镜像详情
- **HTTP方法**: GET
- **请求路径**: `/api/v1/sandbox-images/{imageId}`
- **路径参数**:
  - `imageId`: 镜像ID
- **响应**:
  ```json
  {
    "success": true,
    "data": {
      "imageId": "镜像ID",
      "awsImageId": "AWS镜像ID",
      "name": "镜像名称",
      "description": "镜像描述",
      "status": "镜像状态",
      "sourceInstanceId": "源实例ID",
      "sourceSandboxId": "源沙盒ID",
      "userId": "用户ID",
      "region": "区域",
      "isDefault": false,
      "storageSize": 8,
      "createdAt": "2023-01-01T12:00:00",
      "lastUpdated": "2023-01-01T12:00:00"
    },
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

### 删除镜像

- **接口说明**: 删除镜像
- **HTTP方法**: POST
- **请求路径**: `/api/v1/sandbox-images/{imageId}/delete`
- **路径参数**:
  - `imageId`: 镜像ID
- **响应**:
  ```json
  {
    "success": true,
    "data": true,
    "timestamp": 1625097600000
  }
  ```
- **状态码**: 200 OK

## 错误码

- `400`: 请求参数错误
- `401`: 未授权
- `403`: 无权限访问
- `404`: 资源不存在
- `500`: 服务器内部错误 

```python
"""
 sandbox demo 
"""
import aiohttp  
import asyncio  
import time  
import json  
  
# Configuration  
BASE_URL = ""  # 接口地址
API_KEY = ""  # 实际使用中请替换为有效的API Key  
HEADERS = {  
    "Content-Type": "application/json",  
    "X-API-Key": API_KEY  
}  
  
  
async def create_sandbox(session):  
    """创建沙盒实例"""  
    url = f"{BASE_URL}/v1/sandboxes"  
    payload = {  
        "userId": "v3_domain_test",  
        # "instanceSpecification": "2u4g"  
    }  
  
    async with session.post(url, headers=HEADERS, json=payload) as response:  
        if response.status != 201:  
            error_text = await response.text()  
            raise Exception(f"创建沙盒失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        response_data = await response.json()  
        return response_data.get("data", {}).get("sandboxId")  
  
  
async def get_sandbox_status(session, sandbox_id):  
    """获取沙盒状态"""  
    url = f"{BASE_URL}/v1/sandboxes/{sandbox_id}/status"  
  
    async with session.get(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"获取沙盒状态失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        response_data = await response.json()  
        return response_data.get("data", {}).get("status")  
  
  
async def get_sandbox_details(session, sandbox_id):  
    """获取沙盒详情"""  
    url = f"{BASE_URL}/v1/sandboxes/{sandbox_id}"  
  
    async with session.get(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"获取沙盒详情失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        return await response.json()  
  
  
async def get_sandbox_list(session, user_id="v3_domain_test", status="RUNNING", page=1, size=20):  
    """获取沙盒列表"""  
    url = f"{BASE_URL}/v1/sandboxes?userId={user_id}&status={status}&page={page}&size={size}"  
  
    async with session.get(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"获取沙盒列表失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        return await response.json()  
  
  
async def stop_sandbox(session, sandbox_id):  
    """停止沙盒"""  
    url = f"{BASE_URL}/v1/sandboxes/{sandbox_id}/stop"  
  
    async with session.post(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"停止沙盒失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        return await response.json()  
  
  
async def poll_until_running(session, sandbox_id, interval=2):  
    """轮询沙盒状态直到RUNNING"""  
    print(f"开始轮询沙盒 {sandbox_id} 的状态...")  
    while True:  
        status = await get_sandbox_status(session, sandbox_id)  
        print(f"当前沙盒状态: {status}")  
  
        if status == "RUNNING":  
            # 获取沙盒详情以获取URL  
            details = await get_sandbox_details(session, sandbox_id)  
            return details  
  
        await asyncio.sleep(interval)  
  
  
async def get_image_details(session, image_id):  
    """获取镜像详情"""  
    url = f"{BASE_URL}/v1/sandbox-images/{image_id}"  
  
    async with session.get(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"获取镜像详情失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        return await response.json()  
  
  
async def poll_until_image_available(session, image_id, interval=2):  
    """轮询镜像状态直到可用"""  
    print(f"开始轮询镜像 {image_id} 的状态...")  
    while True:  
        image_details = await get_image_details(session, image_id)  
        status = image_details.get("data", {}).get("status")  
        print(f"当前镜像状态: {status}")  
  
        if status == "AVAILABLE":  
            print(f"镜像已就绪: {image_id}")  
            return image_details  
  
        await asyncio.sleep(interval)  
  
  
async def delete_image(session, image_id):  
    """删除镜像"""  
    url = f"{BASE_URL}/v1/sandbox-images/{image_id}/delete"  
  
    async with session.post(url, headers=HEADERS) as response:  
        if response.status != 200:  
            error_text = await response.text()  
            raise Exception(f"删除镜像失败，状态码: {response.status}, 错误信息: {error_text}")  
  
        return await response.json()  
  
  
async def main():  
    """主函数"""  
    async with aiohttp.ClientSession() as session:  
        try:  
            print("开始创建沙盒...")  
            sandbox_id = await create_sandbox(session)  
            print(f"沙盒创建成功，ID: {sandbox_id}")  
  
            details = await poll_until_running(session, sandbox_id)  
  
            # 从响应体中提取URL  
            data = details.get("data", {})  
            if data:  
                print(f"沙盒实例: {data}")  
                print(f"沙盒已准备就绪")  
                print(f'沙盒ID: {data["sandboxId"]}')  
                print(f'mcp地址: {data["mcpUrl"]},可通过cursor或者其他agent使用')  
                print(f'vnc地址: {data["vncUrl"]}, 可浏览器直接访问')  
                print(f'cdp地址: {data["cdpUrl"]}, 可通过playwright访问')  
                # 获取沙盒列表  
                print("\n获取沙盒列表...")  
                sandbox_list = await get_sandbox_list(session)  
                print(f"沙盒列表: {json.dumps(sandbox_list, indent=2, ensure_ascii=False)}")  
  
                # 再次获取沙盒详情  
                print(f"\n再次获取沙盒 {sandbox_id} 详情...")  
                sandbox_details = await get_sandbox_details(session, sandbox_id)  
                print(f"沙盒详情: {json.dumps(sandbox_details, indent=2, ensure_ascii=False)}")  
  
                # 停止沙盒  
                print(f"\n停止沙盒 {sandbox_id}...")  
                stop_result = await stop_sandbox(session, sandbox_id)  
                print(f"停止沙盒结果: {json.dumps(stop_result, indent=2, ensure_ascii=False)}")  
  
                # 获取镜像ID并轮询镜像状态  
                image_id = stop_result.get("data", {}).get("imageId")  
                if image_id:  
                    print(f"\n检测到镜像ID: {image_id}，开始等待镜像就绪...")  
                    image_details = await poll_until_image_available(session, image_id)  
                    print(f"镜像详情: {json.dumps(image_details, indent=2, ensure_ascii=False)}")  
  
                    # 删除镜像  
                    print(f"\n删除镜像 {image_id}...")  
                    delete_result = await delete_image(session, image_id)  
                    print(f"删除镜像结果: {json.dumps(delete_result, indent=2, ensure_ascii=False)}")  
                else:  
                    print("停止沙盒后未找到有效的镜像ID")  
  
                return data  
            else:  
                print("未创建沙盒实例")  
                return None  
  
        except Exception as e:  
            print(f"发生错误: {e}")  
            return None  
  
  
if __name__ == "__main__":  
    # 运行异步主函数  
    asyncio.run(main())
```