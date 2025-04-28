---
layout: default
title: MCP工具
---

# MCP工具

## 目录

- [文件操作](#文件操作)
  - [读取文件](#读取文件)
  - [写入文件](#写入文件)
  - [编辑文件](#编辑文件)
  - [列出目录](#列出目录)
  - [搜索文件内容](#搜索文件内容)
  - [搜索文件名](#搜索文件名)
  - [删除文件](#删除文件)
- [命令执行](#命令执行)
  - [执行Shell命令](#执行shell命令)
- [使用示例](#使用示例)
  - [使用Agent与MCP服务器](#使用agent与mcp服务器)

## 文件操作

### 读取文件（read_file）

**功能**: 读取指定文件的内容

**参数**:
- `path`: 文件路径
- `offset`: 起始行偏移（从0开始，可选）
- `limit`: 读取的最大行数（可选）

**返回**: 文件内容

### 写入文件（write_file）

**功能**: 写入内容到指定文件

**参数**:
- `path`: 文件路径
- `content`: 要写入的内容
- `append`: 是否追加到文件末尾（默认为False）

**返回**: 操作结果

### 编辑文件（edit_file）

**功能**: 编辑指定文件的内容

**参数**:
- `path`: 文件路径
- `operations`: 编辑操作列表，每个操作包含:
  - `type`: 操作类型（'insert'或'replace'）
  - `start_line`: 开始行号
  - `end_line`: 结束行号（仅用于替换）
  - `content`: 要插入或替换的内容

**返回**: 操作结果

### 列出目录（list_directory）

**功能**: 列出目录内容，不读取文件内容

**参数**:
- `path`: 目录路径，默认为当前目录
- `recursive`: 是否递归列出子目录（默认为False）
- `include_hidden`: 是否包含隐藏文件（默认为False）

**返回**: 目录结构信息，包括:
- `path`: 目录路径
- `total_items`: 总项目数
- `total_dirs`: 总目录数
- `total_files`: 总文件数
- `items`: 项目列表

### 搜索文件内容（grep_search）

**功能**: 使用正则表达式在文件中搜索匹配

**参数**:
- `pattern`: 搜索模式（正则表达式）
- `path`: 要搜索的目录或文件路径，默认为当前目录
- `recursive`: 是否递归搜索子目录（默认为True）
- `case_sensitive`: 是否区分大小写（默认为False）
- `include_pattern`: 要包含的文件模式，如"*.py"（可选）
- `exclude_pattern`: 要排除的文件模式，如"*.log"（可选）
- `max_results`: 最大结果数量（默认为50）

**返回**: 搜索结果，包括:
- `pattern`: 搜索模式
- `total_matches`: 总匹配数
- `truncated`: 结果是否被截断
- `matches`: 匹配列表

### 搜索文件名（search_files）

**功能**: 使用模糊匹配查找与给定模式匹配的文件名

**参数**:
- `pattern`: 文件名模式（支持模糊匹配）
- `path`: 要搜索的目录路径，默认为当前目录
- `max_results`: 最大结果数量（默认为10）

**返回**: 搜索结果，包括:
- `pattern`: 搜索模式
- `total_matches`: 总匹配数
- `truncated`: 结果是否被截断
- `files`: 匹配的文件列表

### 删除文件（delete_file）

**功能**: 删除指定路径的文件或目录

**参数**:
- `path`: 要删除的文件或目录路径
- `recursive`: 是否递归删除目录内容（删除目录时必须为True，默认为False）

**返回**: 删除结果，包括:
- `path`: 删除的路径
- `success`: 是否成功
- `items_deleted`: 删除的项目数

## 命令执行

### 执行Shell命令（run_command）

**功能**: 执行Shell命令

**参数**:
- `command`: 要执行的Shell命令
- `timeout`: 执行超时时间（秒，可选）
- `allow_nonzero_exit_code`: 是否允许非零退出码（默认为False）

**返回**: 命令执行结果（标准输出）

## 使用示例

### 使用Agent与MCP服务器

**功能**: 通过Agent使用MCP服务器执行任务的完整示例

**demo1**:
使用agent框架

```python
import asyncio
import logfire

from pydantic_ai import Agent
from pydantic_ai.mcp import MCPServerHTTP
from pydantic_ai.models.openai import OpenAIModel
from pydantic_ai.providers.openai import OpenAIProvider


logfire.configure(
    service_name='my_logfire_service',
    send_to_logfire=False,
)

model = OpenAIModel(
    'anthropic/claude-3.5-sonnet',
    provider=OpenAIProvider(
        base_url='https://openrouter.ai/api/v1',
        api_key='',
    ),
)

server = MCPServerHTTP(url='https://mcp-xxxx.llmsandbox.shop/sse?token=xxxx')

agent = Agent(model, mcp_servers=[server])
Agent.instrument_all()

async def main():
    async with agent.run_mcp_servers():
        result = await agent.run('编写一个植物大战僵尸的html小游戏',deps=5)
    print(result.all_messages())

if __name__ == '__main__':
    asyncio.run(main())
```

**demo2**
使用jsonrpc调用
```python
"""  
MCP客户端测试脚本  
  
用于测试沙盒客户端MCP服务是否正常工作  
"""  
import asyncio  
import json  
import sys  
import traceback  
from urllib.parse import urljoin  
  
import httpx  
import httpx_sse  
from loguru import logger  
  
# 设置日志  
logger.remove()  
logger.add(sys.stderr, level="INFO")  
  
# 全局事件队列，用于在不同任务间共享事件  
event_queue = asyncio.Queue()  
init_complete = asyncio.Event()  # 初始化完成事件标志  
  
  
async def mcp_connection(base_url: str = "http://localhost:80"):  
    """  
    测试MCP连接并执行简单工具调用  
  
    Args:        base_url: MCP服务的基础URL  
    """    sse_url = f"{base_url}/sse"  
    logger.info(f"连接到SSE端点: {sse_url}")  
  
    # 创建初始化请求  
    init_request = {  
        "jsonrpc": "2.0",  
        "method": "initialize",  
        "params": {  
            "protocolVersion": "1.0",  
            "capabilities": {},  
            "clientInfo": {  
                "name": "TestClient",  
                "version": "1.0.0"  
            }  
        },  
        "id": "init-1"  
    }  
  
    # 创建一个简单的工具调用请求  
    tool_request = {  
        "jsonrpc": "2.0",  
        "method": "tools/call",  
        "params": {  
            "name": "run_command",  
            "arguments": {  
                "command": "ls -la",  
                "timeout": 10,  
                "allow_nonzero_exit_code": False  
            }  
        },  
        "id": "tool-1"  
    }  
  
    # 创建任务组来并行处理SSE连接和发送消息  
    async with asyncio.TaskGroup() as tg:  
        # 启动SSE连接任务  
        sse_task = tg.create_task(connect_to_sse(sse_url, base_url))  
  
        # 等待获取session_uri  
        session_uri = await event_queue.get()  
        logger.info(f"使用服务器返回的会话URL: {session_uri}")  
  
        # 发送初始化请求  
        logger.info("发送初始化请求...")  
        logger.info(f"初始化请求内容: {json.dumps(init_request, ensure_ascii=False, indent=2)}")  
  
        async with httpx.AsyncClient() as client:  
            response = await client.post(  
                session_uri,  
                json=init_request,  
                timeout=30  
            )  
  
            logger.info(f"初始化响应状态码: {response.status_code}")  
            logger.info(f"初始化响应内容: {response.text}")  
  
            if response.status_code != 202:  
                logger.error(f"初始化请求未被接受: {response.text}")  
                return  
  
            # 等待初始化完成事件  
            logger.info("等待服务器初始化完成...")  
            await asyncio.wait_for(init_complete.wait(), timeout=10)  
            logger.info("服务器初始化已完成，准备发送工具调用请求")  
  
            # 发送工具调用请求  
            logger.info("发送工具调用请求...")  
            logger.info(f"请求内容: {json.dumps(tool_request, ensure_ascii=False, indent=2)}")  
  
            response = await client.post(  
                session_uri,  
                json=tool_request,  
                timeout=30  
            )  
  
            logger.info(f"工具调用响应状态码: {response.status_code}")  
            logger.info(f"工具调用响应内容: {response.text}")  
  
            if response.status_code == 202:  
                logger.info("请求被接受，现在应该会执行工具")  
            else:  
                logger.error(f"请求未被接受: {response.text}")  
  
        # 等待一段时间以接收工具执行结果  
        await asyncio.sleep(10)  
  
  
async def connect_to_sse(sse_url: str, base_url: str):  
    """  
    连接到SSE端点并监听事件  
  
    Args:        sse_url: SSE端点URL  
        base_url: 服务器基础URL，用于构造完整的消息URI  
    """    logger.info(f"开始连接SSE: {sse_url}")  
  
    async with httpx.AsyncClient() as client:  
        try:  
            async with httpx_sse.aconnect_sse(client, 'GET', sse_url, timeout=30) as event_source:  
                logger.info("SSE连接已建立")  
  
                # 监听SSE事件  
                async for event in event_source.aiter_sse():  
                    if event.event == "endpoint":  
                        logger.info(f"收到endpoint事件: {event.data}")  
                        # 构造完整的URL  
                        full_uri = urljoin(base_url, event.data)  
                        logger.info(f"构造完整URL: {full_uri}")  
                        # 将endpoint URL放入队列，以便另一个任务使用  
                        await event_queue.put(full_uri)  
                    elif event.event == "message":  
                        logger.info(f"收到消息事件: {event.data}")  
                        # 解析JSON响应  
                        try:  
                            response = json.loads(event.data)  
                            logger.info(f"解析后的消息: {json.dumps(response, ensure_ascii=False, indent=2)}")  
  
                            # 检查初始化响应  
                            if "result" in response and response.get("id") == "init-1":  
                                logger.info("收到初始化完成响应，标记初始化已完成")  
                                init_complete.set()  # 设置初始化完成事件  
  
                            # 检查是否是工具调用的结果  
                            if "result" in response and response.get("id") == "tool-1":  
                                logger.info(f"收到工具调用结果: {response['result']}")  
                              
                            # 检查是否是工具列表结果  
                            if "result" in response and response.get("id") == "list-tools":  
                                logger.info(f"收到工具列表: {json.dumps(response['result'], ensure_ascii=False, indent=2)}")  
                              
                            # 检查是否是运行命令结果  
                            if "result" in response and response.get("id") == "run-cmd":  
                                logger.info(f"收到命令执行结果: {response['result']}")  
                        except json.JSONDecodeError:  
                            logger.error(f"无法解析JSON消息: {event.data}")  
                    else:  
                        logger.info(f"收到其他事件: {event.event} - {event.data}")  
  
        except Exception as e:  
            logger.error(f"SSE连接错误: {str(e)}")  
            logger.error(traceback.format_exc())  
  
  
async def all_tools(base_url: str = "http://localhost:8088"):  
    """  
    测试所有可用的工具  
  
    Args:        base_url: MCP服务的基础URL  
    """    sse_url = f"{base_url}/sse"  
    logger.info(f"连接到SSE端点: {sse_url}")  
  
    # 初始化请求  
    init_request = {  
        "jsonrpc": "2.0",  
        "method": "initialize",  
        "params": {  
            "protocolVersion": "1.0",  
            "capabilities": {},  
            "clientInfo": {  
                "name": "TestClient",  
                "version": "1.0.0"  
            }  
        },  
        "id": "init-1"  
    }  
  
    # 获取可用工具列表  
    list_tools_request = {  
        "jsonrpc": "2.0",  
        "method": "tools/list",  
        "id": "list-tools"  
    }  
  
    # 创建任务组来并行处理SSE连接和发送消息  
    async with asyncio.TaskGroup() as tg:  
        # 启动SSE连接任务  
        sse_task = tg.create_task(connect_to_sse(sse_url, base_url))  
  
        # 等待获取session_uri  
        session_uri = await event_queue.get()  
        logger.info(f"使用服务器返回的会话URL: {session_uri}")  
  
        # 发送初始化请求  
        logger.info("发送初始化请求...")  
  
        async with httpx.AsyncClient() as client:  
            response = await client.post(  
                session_uri,  
                json=init_request,  
                timeout=30  
            )  
  
            if response.status_code != 202:  
                logger.error(f"初始化请求未被接受: {response.text}")  
                return  
  
            # 等待初始化完成事件  
            logger.info("等待服务器初始化完成...")  
            await asyncio.wait_for(init_complete.wait(), timeout=10)  
            logger.info("服务器初始化已完成，准备获取工具列表")  
  
            logger.info("获取可用工具列表...")  
  
            response = await client.post(  
                session_uri,  
                json=list_tools_request,  
                timeout=30  
            )  
  
            logger.info(f"响应状态码: {response.status_code}")  
  
            if response.status_code == 202:  
                logger.info("工具列表请求已发送")  
            else:  
                logger.error(f"工具列表请求失败: {response.text}")  
  
        # 等待一段时间以接收工具列表  
        await asyncio.sleep(10)  
  
  
async def fetch_tools_and_run_command(base_url: str = "http://localhost:8866", token: str = "test_key"):  
    """  
    连接到SSE服务器，获取所有工具，然后执行一个command工具  
        Args:  
        base_url: MCP服务的基础URL  
        token: 用于验证请求  
    """    sse_url = f"{base_url}/sse?token={token}"  
    logger.info(f"连接到SSE端点: {sse_url}")  
      
    # 重置事件标志  
    init_complete.clear()  
      
    # 初始化请求  
    init_request = {  
        "jsonrpc": "2.0",  
        "method": "initialize",  
        "params": {  
            "protocolVersion": "1.0",  
            "capabilities": {},  
            "clientInfo": {  
                "name": "TestClient",  
                "version": "1.0.0"  
            },  
            "mcpServers": {  
                "test-server": {  
                    "url": f"{base_url}/sse",  
                    "env": {  
                        "tiken": token  
                    }  
                }  
            }  
        },  
        "id": "init-1"  
    }  
      
    # 发送"已初始化"通知，确保完成初始化流程  
    initialized_notification = {  
        "jsonrpc": "2.0",  
        "method": "notifications/initialized"  
    }  
      
    # 获取工具列表请求  
    list_tools_request = {  
        "jsonrpc": "2.0",  
        "method": "tools/list",  
        "id": "list-tools"  
    }  
      
    # 命令执行请求  
    run_command_request = {  
        "jsonrpc": "2.0",  
        "method": "tools/call",  
        "params": {  
            "name": "run_command",  
            "arguments": {  
                "command": "ls -a",  
                "timeout": 10,  
                "allow_nonzero_exit_code": False  
            }  
        },  
        "id": "run-cmd"  
    }  
      
    # 创建任务组来并行处理SSE连接和发送消息  
    async with asyncio.TaskGroup() as tg:  
        # 启动SSE连接任务  
        sse_task = tg.create_task(connect_to_sse(sse_url, base_url))  
          
        # 等待获取session_uri  
        session_uri = await event_queue.get()  
        logger.info(f"使用服务器返回的会话URL: {session_uri}")  
          
        async with httpx.AsyncClient() as client:  
            # 步骤1: 发送初始化请求  
            logger.info("步骤1: 发送初始化请求...")  
            response = await client.post(  
                session_uri,  
                json=init_request,  
                timeout=30  
            )  
              
            if response.status_code != 202:  
                logger.error(f"初始化请求未被接受: {response.text}")  
                return  
            # 发送"已初始化"通知  
            logger.info("发送'已初始化'通知...")  
            response = await client.post(  
                session_uri,  
                json=initialized_notification,  
                timeout=30  
            )  
              
            if response.status_code != 202:  
                logger.error(f"'已初始化'通知未被接受: {response.text}")  
              
            # 等待初始化完成事件  
            logger.info("等待服务器初始化完成...")  
            await asyncio.wait_for(init_complete.wait(), timeout=10)  
            logger.info("服务器初始化已完成")  
              
            # 步骤2: 获取所有工具  
            logger.info("步骤2: 获取所有可用工具...")  
            response = await client.post(  
                session_uri,  
                json=list_tools_request,  
                timeout=30  
            )  
              
            logger.info(f"工具列表请求响应状态码: {response.status_code}")  
              
            if response.status_code != 202:  
                logger.error(f"工具列表请求失败: {response.text}")  
                return  
            # 等待工具列表响应  
            await asyncio.sleep(2)  
              
            # 步骤3: 执行command工具  
            logger.info("步骤3: 执行command工具...")  
            response = await client.post(  
                session_uri,  
                json=run_command_request,  
                timeout=30  
            )  
              
            logger.info(f"命令执行请求响应状态码: {response.status_code}")  
              
            if response.status_code != 202:  
                logger.error(f"命令执行请求失败: {response.text}")  
                return  
            logger.info("命令执行请求已发送，等待结果...")  
          
        # 等待接收命令执行结果  
        await asyncio.sleep(5)  
          
    logger.info("测试完成")  
  
  
if __name__ == "__main__":  
    logger.info("启动MCP客户端测试...")  
  
    # 获取命令行参数中的服务地址  
    base_url = "https://mcp-xxx.llmsandbox.shop"  
    token = 'xxxxx'  
    # 执行测试  
    try:  
        # 仅执行fetch_tools_and_run_command，忽略之前的测试  
        asyncio.run(fetch_tools_and_run_command(base_url, token))  
    except Exception as e:  
        logger.error(f"测试过程中发生错误: {str(e)}")  
        logger.error(f"错误详情: {traceback.format_exc()}")  
  
    logger.info("测试完成")
```