---
layout: default
title: Chrome DevTools Protocol (CDP)
---

# Chrome DevTools Protocol (CDP)

Chrome DevTools Protocol (CDP)是一个允许客户端与Chromium浏览器实例通信的协议，通过此协议可以检查、调试和控制Chromium浏览器的行为。在沙盒环境中，CDP提供了强大的浏览器自动化能力。

## CDP概述

CDP允许通过WebSocket连接与Chrome浏览器进行通信，提供以下主要功能：

- **浏览器控制**：打开页面、导航、刷新、管理标签等
- **DOM操作**：查询和修改DOM元素
- **JavaScript执行**：在页面上下文中执行JavaScript代码
- **网络监控**：拦截、监控和修改网络请求
- **性能分析**：收集性能指标和时间线事件
- **截图和打印**：获取页面截图或PDF输出

## 沙盒中的CDP实现

在沙盒环境中，CDP功能通过多种方式提供，包括直接CDP API以及更高级的自动化框架如Playwright。

## Playwright连接CDP与非无头浏览器操作

Playwright不仅可以启动新的浏览器实例，还可以连接到现有的CDP端点或以非无头模式运行，便于调试和可视化操作过程。

### 连接到现有的CDP端点

```python
from playwright.sync_api import sync_playwright

def connect_to_cdp():
    with sync_playwright() as p:
        # 连接到已经运行的Chrome实例的CDP端点
        browser = p.chromium.connect_over_cdp("https://cdp-xxxx.llmsandbox.shop/cdp?token=xxxx")
        
        # 获取默认上下文
        default_context = browser.contexts[0]
        
        # 获取已打开的页面
        pages = default_context.pages
        page = pages[0] if pages else default_context.new_page()
        
        # 在已连接的浏览器上执行操作
        page.goto('https://example.com')
        page.screenshot(path='screenshot_from_connected_browser.png')
        
        # 不要关闭浏览器，因为它是外部启动的
        browser.close()
```
## Playwright使用示例

Playwright是一个强大的浏览器自动化库，在内部使用CDP，但提供了更简洁的API。以下是Playwright连接远程浏览器操作的示例：

### 基本页面导航

```python
from playwright.sync_api import sync_playwright

def browse_example():
    with sync_playwright() as p:
		browser = p.chromium.connect_over_cdp("https://cdp-xxxx.llmsandbox.shop/cdp?token=xxxx")
        page = browser.new_page()
        page.goto('https://example.com')
        title = page.title()
        content = page.content()
        browser.close()
        return title, content
```

### 截图和元素交互

```python
from playwright.sync_api import sync_playwright

def interact_example():
    with sync_playwright() as p:
	    browser = p.chromium.connect_over_cdp("https://cdp-xxxx.llmsandbox.shop/cdp?token=xxxx")
        page = browser.new_page()
        page.goto('https://example.com/login')
        
        # 填写表单
        page.fill('input#username', 'myuser')
        page.fill('input#password', 'mypassword')
        
        # 点击按钮
        page.click('button#login')
        
        # 等待导航完成
        page.wait_for_load_state('networkidle')
        
        # 截图
        page.screenshot(path='screenshot.png')
        browser.close()
```


## 使用场景

CDP和Playwright在沙盒环境中的典型使用场景包括：

1. **Web自动化测试**：模拟用户交互，验证网站功能
2. **网页内容抓取**：获取结构化数据
3. **截图和可视化**：生成页面截图或可视化报告

## Browser-Use库集成

Browser-Use是一个高级浏览器自动化库，可以与CDP端点集成，提供LLM驱动的浏览器代理功能。它允许您创建智能代理来执行基于任务的浏览器操作。

### Browser-Use集成示例

```python
import asyncio

from browser_use import BrowserContextConfig, Browser, Agent, BrowserConfig
from browser_use.browser.context import BrowserContext
from langchain_openai import ChatOpenAI


async def run():
    # 配置browser-use浏览器上下文
    context_config = BrowserContextConfig(
        locale='en-US',
        user_agent='Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36',
        highlight_elements=False,  # 启用高亮以获得更好的可视化效果
    )
    config = BrowserConfig(
        cdp_url=f"https://cdp-xxxx.llmsandbox.shop/cdp?token=xxxx"
    )

    # 初始化浏览器和上下文
    browser = Browser(config=config)
    browser_context = BrowserContext(browser=browser, config=context_config)
    
    # 配置LLM用于代理决策
    llm = ChatOpenAI(
        api_key='YOUR_API_KEY',
        base_url='https://openrouter.ai/api/v1',
        model="anthropic/claude-3.5-sonnet",
    )

    # 创建代理执行任务
    agent = Agent(
        browser=browser,
        browser_context=browser_context,
        task='查询最新的关税影响，并总结对策',
        llm=llm,
    )

    # 运行代理，最多执行10个步骤
    await agent.run(max_steps=10)

if __name__ == '__main__':
    asyncio.run(run())

