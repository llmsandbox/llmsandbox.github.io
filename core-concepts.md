# 核心概念

## 沙盒实例

沙盒实例是系统中的核心资源，它是一个基于AWS EC2的虚拟机实例，提供独立的计算环境。

### 生命周期

沙盒实例有以下几种状态：

1. **PENDING**：沙盒正在创建中，包括EC2实例启动和初始化配置阶段
2. **RUNNING**：沙盒正常运行中，可以访问
3. **STOPPED/TERMINATED**：沙盒已终止/删除
4. **ERROR**: 实例异常

生命周期转换图：

```mermaid
graph TD
    PENDING --> RUNNING
    RUNNING --> STOPPED
    STOPPED --> TERMINATED
    PENDING --> ERROR
    RUNNING --> ERROR
    STOPPED --> ERROR
```

### 沙盒操作时序

```mermaid

sequenceDiagram
    participant 用户
    participant 沙盒服务
    participant AWS EC2

    用户->>沙盒服务: 请求创建沙盒
    activate 沙盒服务
    沙盒服务->>AWS EC2: 调用 RunInstances API 创建EC2实例
    activate AWS EC2
    AWS EC2-->>沙盒服务: 返回实例信息 (Instance ID)
    deactivate AWS EC2
    沙盒服务-->>用户: 返回沙盒ID
    deactivate 沙盒服务

    Note right of AWS EC2: EC2 实例启动和初始化...

    loop
        用户->>沙盒服务: 查询沙盒状态
        activate 沙盒服务
        沙盒服务->>AWS EC2: 查询实例状态
        activate AWS EC2
        AWS EC2-->>沙盒服务: 返回实例状态 (Running, IP地址等)
        deactivate AWS EC2
        沙盒服务-->>用户: 返回沙盒状态 (Ready) 及连接信息
        deactivate 沙盒服务
    end
    用户-->>用户: 保存用户和沙盒的关系


    loop MCP调用
        用户->>AWS EC2: 执行命令
        activate AWS EC2

        AWS EC2-->>用户: 返回操作结果
        deactivate AWS EC2
    end


    用户->>沙盒服务: 请求停止沙盒
    activate 沙盒服务
    沙盒服务->>AWS EC2: 调用 停止实例 API
    activate AWS EC2
    AWS EC2-->>沙盒服务: 确认实例已停止
    沙盒服务->>AWS EC2: 调用 创建镜像 API
    AWS EC2-->>沙盒服务: 镜像创建成功
    deactivate AWS EC2
    沙盒服务-->>用户: 返回停止成功 (镜像ID)
    deactivate 沙盒服务
    用户-->>用户: 保存用户和镜像的关系
```

### 自定义CPU和内存

沙盒实例支持自定义CPU和内存配置：

- **CPU**：可选择不同核心数量
- **内存**：可选择不同内存大小

通过`instanceSpecification`参数指定，格式为`{CPU核心数}u{内存GB数}g
- `1u1g`(默认)：1核心1GB内存 
- `2u4g`：2核心4GB内存
 

### 持久化

沙盒实例使用以下存储方式：

- **快照**：在创建镜像时生成，用于保存实例状态


## 镜像

镜像是创建沙盒实例的蓝图，包含操作系统和预安装的软件。用户停止沙盒时自动生成该沙盒的镜像

### 镜像操作时序
```mermaid
sequenceDiagram
    participant 用户
    participant 沙盒服务
    participant AWS EC2
    用户->>沙盒服务: 请求删除镜像
    activate 沙盒服务
    沙盒服务->>AWS EC2: 调用 删除镜像 API
    activate AWS EC2
    AWS EC2-->>沙盒服务: 确认镜像已删除
    deactivate AWS EC2
    沙盒服务-->>用户: 返回删除镜像成功
    deactivate 沙盒服务
``` 