---
layout: default
title: VNC (Virtual Network Computing)
---

# VNC (Virtual Network Computing)

VNC是一种图形化桌面共享系统，使用远程帧缓冲协议(RFB)在网络上传输键盘和鼠标事件以及屏幕更新。在沙盒环境中，VNC提供了远程访问和控制图形化界面的能力。

## VNC概述

VNC系统由以下组件组成：

- **VNC服务器**：运行在被控制的机器上，捕获屏幕并接收输入事件
- **VNC客户端**：连接到服务器的应用程序，显示远程屏幕并发送输入事件
- **VNC查看器**：用于查看和控制远程桌面的客户端应用程序

VNC的优势包括：

- 跨平台兼容性（Windows、Linux、macOS等）
- 与操作系统无关的远程访问
- 可通过浏览器访问（使用适当的代理或WebVNC实现）

## 浏览器中的VNC访问

通过浏览器访问VNC是最常用的访问方式之一，不需要安装专门的VNC客户端软件。浏览器访问VNC_URL可直接查看浏览器访问情况

![[Pasted image 20250427154954.png]]



