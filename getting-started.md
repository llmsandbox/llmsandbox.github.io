# 快速开始

本文档将帮助您快速上手沙盒管理系统，包括获取API密钥、创建和管理沙盒实例等基本操作。

## API密钥

在使用沙盒管理系统前，您需要获取API密钥进行身份认证。

### 获取API密钥

1. 联系系统管理员申请API密钥
2. 将获得的API密钥妥善保存，它将用于所有API请求的认证

### 使用API密钥

所有API请求都需要在HTTP头部包含API密钥进行认证：

```
X-API-KEY: <您的API密钥>
```

## 创建沙盒实例

### 基本步骤

1. 准备创建沙盒的必要参数：用户ID、MCP版本、镜像ID等
2. 调用创建沙盒API
3. 等待沙盒创建完成并获取沙盒访问URL

### 示例请求

```http
POST /api/v1/sandboxes HTTP/1.1
Content-Type: application/json
X-API-KEY: your-api-key

{
  "userId": "user123",
  "mcpVersion": "1.0",
  "imageId": "img-default"
}
```

### 示例响应

```json
{
  "success": true,
  "data": {
    "sandboxId": "sb-1234567890",
    "mcpUrl": "https://mcp.example.com/sb-1234567890",
    "vncUrl": "https://vnc.example.com/sb-1234567890",
    "cdpUrl": "https://cdp.example.com/sb-1234567890",
    "status": "CREATING"
  },
  "timestamp": 1625097600000
}
```

## 管理沙盒实例

创建沙盒后，您可以执行以下操作：

1. **获取沙盒列表**：查看您创建的所有沙盒
2. **查看沙盒详情**：获取特定沙盒的详细信息
3. **停止沙盒**：暂停并删除不需要使用的沙盒

详细API使用方法请参考[API参考](api-reference.md)文档。

## 常见问题

### 沙盒创建失败怎么办？

请检查以下几点：
- API密钥是否有效
- 参数是否正确
- 查看API响应中的错误信息

### 如何监控沙盒资源使用情况？

您可以通过[获取沙盒状态API](api-reference.md#get-sandbox-status)查看沙盒的基本状态，或者使用专门的监控API获取更详细的资源使用情况。 