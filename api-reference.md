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