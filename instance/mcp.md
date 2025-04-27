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

### 读取文件

**功能**: 读取指定文件的内容

**参数**:
- `path`: 文件路径
- `offset`: 起始行偏移（从0开始，可选）
- `limit`: 读取的最大行数（可选）

**返回**: 文件内容

### 写入文件

**功能**: 写入内容到指定文件

**参数**:
- `path`: 文件路径
- `content`: 要写入的内容
- `append`: 是否追加到文件末尾（默认为False）

**返回**: 操作结果

### 编辑文件

**功能**: 编辑指定文件的内容

**参数**:
- `path`: 文件路径
- `operations`: 编辑操作列表，每个操作包含:
  - `type`: 操作类型（'insert'或'replace'）
  - `start_line`: 开始行号
  - `end_line`: 结束行号（仅用于替换）
  - `content`: 要插入或替换的内容

**返回**: 操作结果

### 列出目录

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

### 搜索文件内容

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

### 搜索文件名

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

### 删除文件

**功能**: 删除指定路径的文件或目录

**参数**:
- `path`: 要删除的文件或目录路径
- `recursive`: 是否递归删除目录内容（删除目录时必须为True，默认为False）

**返回**: 删除结果，包括:
- `path`: 删除的路径
- `success`: 是否成功
- `items_deleted`: 删除的项目数

## 命令执行

### 执行Shell命令

**功能**: 执行Shell命令

**参数**:
- `command`: 要执行的Shell命令
- `timeout`: 执行超时时间（秒，可选）
- `allow_nonzero_exit_code`: 是否允许非零退出码（默认为False）

**返回**: 命令执行结果（标准输出）

## 使用示例

### 使用Agent与MCP服务器

**功能**: 通过Agent使用MCP服务器执行任务的完整示例

**示例代码**:

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