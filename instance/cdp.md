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
```


## DEMO
### X 登录

```python
from playwright.sync_api import sync_playwright, Browser, Page, BrowserContext  
import time  
import requests  
import json  
import os  
  
# X.com登录信息  
  
USERNAME = "" 
PASSWORD = ""          
PHONEOREMAIL = ""  
TWOFACODE = ""  
  
# cookies文件路径  
COOKIES_FILE = f"test/{USERNAME}_x_cookies.json"  
  
def get_2fa_code():  
    """获取2FA验证码"""  
    url = "https://www.henduohao.com/tools/twofaGet"  
    headers = {  
        'accept': 'application/json, text/javascript, */*; q=0.01',  
        'accept-language': 'zh-CN,zh;q=0.9',  
        'cache-control': 'no-cache',  
        'content-type': 'application/x-www-form-urlencoded; charset=UTF-8',  
        'origin': 'https://www.henduohao.com',  
        'pragma': 'no-cache',  
        'referer': 'https://www.henduohao.com/tools/twofa',  
        'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36',  
        'x-requested-with': 'XMLHttpRequest'  
    }  
    data = {'twoFaCode': TWOFACODE}  
      
    try:  
        response = requests.post(url, headers=headers, data=data)  
        response_data = response.json()  
        if response_data['code'] == 0:  
            return response_data['data']  
        else:  
            print(f"获取2FA验证码失败: {response_data['msg']}")  
            return None  
    except Exception as e:  
        print(f"获取2FA验证码时发生错误: {e}")  
        return None  
  
def save_cookies(context):  
    """保存浏览器上下文的cookies到本地文件"""  
    cookies = context.cookies()  
    os.makedirs(os.path.dirname(COOKIES_FILE), exist_ok=True)  
    with open(COOKIES_FILE, 'w', encoding='utf-8') as f:  
        json.dump(cookies, f, ensure_ascii=False, indent=2)  
    print(f"cookies已保存到: {COOKIES_FILE}")  
  
def load_cookies(context, url="https://x.com"):  
    """从本地文件加载cookies到浏览器上下文"""  
    if not os.path.exists(COOKIES_FILE):  
        print(f"未找到cookies文件: {COOKIES_FILE}")  
        return False  
        try:  
        with open(COOKIES_FILE, 'r', encoding='utf-8') as f:  
            cookies = json.load(f)  
        context.add_cookies(cookies)  
        print(f"成功加载cookies")  
        return True  
    except Exception as e:  
        print(f"加载cookies失败: {e}")  
        return False  
  
def login_with_cookies(cdp_url):  
    """使用已保存的cookies登录X.com"""  
    with sync_playwright() as playwright:  
        # 连接到已存在的Chrome实例  
        browser = playwright.chromium.connect_over_cdp(cdp_url)  
          
        # 获取浏览器的第一个上下文  
        default_context = browser.contexts[0]  
          
        # 获取上下文中的第一个页面  
        page = default_context.pages[0]  
          
        # 尝试加载cookies  
        cookies_loaded = load_cookies(default_context)  
          
        # 访问X.com  
        print("正在访问X.com...")  
        page.goto("https://x.com/")  
        time.sleep(2)  # 等待页面加载  
        # 检查是否已经登录  
        if "登录" in page.content() or "Sign in" in page.content():  
            print("cookies登录失败，尝试重新登录...")  
            login_x()  
        else:  
            print("使用cookies成功登录X.com")  
              
            # 打印页面标题和URL  
            print(f'当前页面: {page.title()}')  
            print(f'当前URL: {page.url}')  
  
            # 截图保存  
            print("正在保存页面截图...")  
            page.screenshot(path="test/x_login_with_cookies_screenshot.png")  
              
            # 等待一段时间以便观察  
            time.sleep(3)  
            print("使用cookies登录测试完成")  
  
def login_x(cdp_url):  
    with sync_playwright() as playwright:  
        # 连接到已存在的Chrome实例  
        browser = playwright.chromium.connect_over_cdp(  
            cdp_url)  
  
        # 获取浏览器的第一个上下文  
        default_context = browser.contexts[0]  
  
        # 获取上下文中的第一个页面  
        page = default_context.pages[0]  
  
        # 访问X.com  
        print("正在访问X.com...")  
        page.goto("https://x.com/")  
        time.sleep(2)  # 等待页面加载  
  
        # 检查是否已经登录  
        if "登录" in page.content() or "Sign in" in page.content():  
            print("准备登录X.com...")  
  
            # 点击登录按钮  
            page.click('a[href="/login"]')  
            time.sleep(2)  # 等待登录页面加载  
  
            # 输入用户名  
            print("正在输入用户名...")  
            page.fill('input[autocomplete="username"]', USERNAME)  
            page.press('input[autocomplete="username"]', 'Enter')  
            time.sleep(2)  # 等待下一步加载  
  
            # 可能需要处理电话验证步骤  
            if "phone number" in page.content():  
                print("检测到需要验证电话，正在处理...")  
                # 如果有电话号码输入框，则输入  
                if page.query_selector('input[data-testid="ocfEnterTextTextInput"]'):  
                    page.fill('input[data-testid="ocfEnterTextTextInput"]', PHONEOREMAIL)  
                    # page.click('div[data-testid="ocfEnterTextNextButton"]')  
                    page.click('button:has-text("Next")')  
                    time.sleep(2)  
  
            # 输入密码  
            print("正在输入密码...")  
            page.fill('input[autocomplete="current-password"]', PASSWORD)  
            time.sleep(2)  
  
            # 尝试多种方式定位登录按钮  
            print("尝试点击登录按钮...")  
              
            # 方法1：等待登录按钮出现，最长等待10秒  
            try:  
                page.wait_for_selector('div[data-testid="LoginForm_Login_Button"]', timeout=10000)  
                print("找到登录按钮，尝试点击")  
            except Exception as e:  
                print(f"等待登录按钮超时: {e}")  
              
            # 方法2：打印页面内容，帮助调试  
            button_text = page.evaluate('''() => {  
                // 尝试查找所有可能的登录按钮  
                const buttons = Array.from(document.querySelectorAll('button')).filter(b =>                    b.textContent.includes('Log in') ||   
                    b.textContent.includes('Sign in') ||   
                    b.textContent.includes('登录')  
                );                return buttons.map(b => ({text: b.textContent, html: b.outerHTML}));            }''')  
            print(f"找到的可能登录按钮: {button_text}")  
              
            # 方法3：尝试多种选择器  
            login_selectors = [  
                'button:has-text("Log in")',  
                'button:has-text("Sign in")',  
                'button:has-text("登录")'  
            ]  
              
            for selector in login_selectors:  
                if page.query_selector(selector):  
                    print(f"尝试点击选择器: {selector}")  
                    try:  
                        page.click(selector)  
                        print(f"成功点击: {selector}")  
                        break  
                    except Exception as e:  
                        print(f"点击 {selector} 失败: {e}")  
              
            time.sleep(5)  # 等待登录完成  
            # 检查是否需要2FA验证  
            if "verification code" in page.content().lower() or "验证码" in page.content():  
                print("检测到需要2FA验证，正在获取验证码...")  
                verification_code = get_2fa_code()  
                  
                if verification_code:  
                    print(f"获取到2FA验证码: {verification_code}")  
                    # 尝试定位2FA输入框  
                    verification_selectors = [  
                        'input[autocomplete="one-time-code"]',  
                        'input[name="verfication_code"]',  
                        'input[placeholder*="code"]',  
                        'input[data-testid="ocfEnterTextTextInput"]'  
                    ]  
                      
                    for selector in verification_selectors:  
                        if page.query_selector(selector):  
                            print(f"找到2FA输入框: {selector}")  
                            page.fill(selector, verification_code)  
                            break  
                    # 尝试点击验证按钮  
                    verification_button_selectors = [  
                        'button:has-text("Verify")',  
                        'button:has-text("Next")',  
                        'button:has-text("确认")',  
                        'button:has-text("验证")'  
                    ]  
                      
                    for selector in verification_button_selectors:  
                        if page.query_selector(selector):  
                            print(f"点击验证按钮: {selector}")  
                            page.click(selector)  
                            break  
                    time.sleep(5)  # 等待验证完成  
                else:  
                    print("未能获取2FA验证码，登录可能会失败")  
  
            time.sleep(5)  
            # 验证是否登录成功  
            if "home" in page.url:  
                print("登录成功！")  
                # 保存cookies到本地  
                save_cookies(default_context)  
            else:  
                print("登录失败，请检查用户名和密码是否正确。")  
                # 打印当前URL和页面标题，帮助调试  
                print(f"当前URL: {page.url}")  
                print(f"页面标题: {page.title()}")  
                save_cookies(default_context)  
        else:  
            print("已经登录X.com")  
            # 保存cookies到本地  
            save_cookies(default_context)  
  
        # 打印页面标题和URL  
        print(f'当前页面: {page.title()}')  
        print(f'当前URL: {page.url}')  
  
        # 截图保存  
        print("正在保存页面截图...")  
        page.screenshot(path="test/x_login_screenshot.png")  
  
        # 等待一段时间以便观察  
        time.sleep(3)  
        print("测试完成")  
  
  
if __name__ == '__main__':  
    cdp_url = ''  
    login_x(cdp_url)  
    # 尝试使用cookies登录  
    # login_with_cookies(cdp_url)
```