<!-- ============================ LANGUAGE SWITCHER ============================ -->
<div align="center">

# 🏠 UNIHIKER K10 + XiaoZhi MCP — Smart-Home Voice Assistant

### A home assistant that **sees** the room, **understands** your words, and **acts** on its own.

[![English](https://img.shields.io/badge/Language-English-2f81f7?style=for-the-badge)](#-english) &nbsp; [![中文](https://img.shields.io/badge/语言-简体中文-e8453c?style=for-the-badge)](#-简体中文)

<br>

![UNIHIKER K10](https://img.shields.io/badge/Board-UNIHIKER%20K10-00A0E9?style=flat-square)
![XiaoZhi](https://img.shields.io/badge/Firmware-XiaoZhi%20AI-7B61FF?style=flat-square)
![MCP](https://img.shields.io/badge/Protocol-MCP-00B894?style=flat-square)
![Mind+](https://img.shields.io/badge/IDE-Mind%2B%202.0-F5A623?style=flat-square)
![License](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey?style=flat-square)

</div>

---

<!-- ================================ ENGLISH ================================ -->
<a id="-english"></a>

## 🇬🇧 English

### What is this?

Two **UNIHIKER K10** boards and one AI model, working together as a **proactive smart-home assistant**. You walk into a stuffy, dimly-lit room — before you say anything, the fan spins up and the lights warm to the right brightness and color.

It is not a voice-command gimmick. The assistant **senses** the environment, **understands** natural language, **decides** what to do, and **acts** through real hardware.

> **Large model + open protocol (MCP) + real hardware = a running smart-space system.**

### 🧰 Hardware used

| Qty | Part | Role |
|:--:|---|---|
| **2×** | [UNIHIKER K10](https://www.dfrobot.com.cn/goods-4014.html) | ESP32-S3 AI dev board — one as voice terminal, one as hardware controller |
| **1×** | [Gravity: 130 DC Motor Fan](https://www.dfrobot.com.cn/goods-1251.html) | Driven from pin P0 via PWM |
| **2×** | [USB-C cable](https://www.dfrobot.com.cn/goods-2977.html) | Power + flashing |

**Software:** [Mind+ 2.0](https://mindplus.cc/en/download.html) · [XiaoZhi firmware](https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai) · [XiaoZhi console](https://xiaozhi.me/)

### 🎯 What the project does

| Capability | How it works |
|---|---|
| **Voice interaction** | K10-A runs XiaoZhi firmware, talks to the cloud LLM, understands intent — no hard-coded keywords |
| **Environment sensing** | K10-B exposes `get_environment` — onboard temp / humidity read back to the AI |
| **Fan control** | `set_fan(on)` → PWM on P0 (1023 = on, 0 = off) |
| **Scene lighting** | `set_light_environment(mode, …)` → `auto` / `study` / `relax` / `night` / `off` / `manual` |
| **Proactive behavior** | A role prompt turns the assistant from a command-follower into a thoughtful caretaker |

### 🏗️ Architecture

```
┌──────────────────────────┐        WebSocket / MCP        ┌──────────────────────────┐
│   UNIHIKER K10-A         │  ───────────────────────────▶ │   UNIHIKER K10-B         │
│   XiaoZhi firmware       │   get_environment / set_fan    │   Mind+ program          │
│   (Voice / AI terminal)  │   set_light_environment        │   (Execution terminal)   │
│                          │  ◀───────────────────────────  │                          │
│  • Speech recognition    │        tool result (JSON)      │  • Temp / humidity / lux │
│  • Intent understanding  │                                │  • P0 → fan (PWM)        │
│  • Decision making       │                                │  • Onboard RGB LEDs      │
└──────────────────────────┘                                └──────────────────────────┘
                                        ▲
                                  ┌─────┴──────┐
                                  │  MCP layer │  Model Context Protocol
                                  └────────────┘
```

**MCP (Model Context Protocol)** is the open standard that lets the AI call hardware capabilities the same way it calls tools. Add a tool, describe it in plain language, and the assistant figures out when to use it.

### 🚀 Quick start

```text
1. Flash K10-A with the XiaoZhi firmware   →  https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai
2. Configure Wi-Fi + add the device on     →  https://xiaozhi.me/
3. Open Mind+ 2.0 → Upload mode → load the UNIHIKER K10 library and the XiaoZhi MCP client library
4. Register 3 MCP tools on K10-B:          get_environment · set_fan · set_light_environment
5. Copy the MCP endpoint from the console  →  paste it into your Mind+ blocks
6. Upload to K10-B, paste the role prompt  →  done ✅
```

👉 **Full step-by-step tutorial: [`docs/TUTORIAL_EN.md`](docs/TUTORIAL_EN.md)** — every step with screenshots.

### 📂 How to use the materials

| I want to… | Go to |
|---|---|
| Build it from scratch | [`docs/TUTORIAL_EN.md`](docs/TUTORIAL_EN.md) / [`docs/TUTORIAL_CN.md`](docs/TUTORIAL_CN.md) |
| Just grab the role prompt | TUTORIAL §5.8 — copy straight from the code block |
| Look at a specific screenshot | [`images/`](images/) — all files are semantically named |
| Understand *why* it works | TUTORIAL §3 Architecture & §4 Working Principle |
| Port it to another board | TUTORIAL §3 — MCP is board-agnostic |
| Fix something that broke | TUTORIAL §8 Troubleshooting table |

> **Note:** the demo video is not bundled here — GitHub rejects files over 100 MB. The written tutorial plus screenshots are fully self-contained.

### ⚠️ Requirements & gotchas

- Both K10 boards must be on the **same Wi-Fi network** (SSID + password identical).
- Tool **descriptions matter more than you think** — that text is the only manual your AI gets. Be explicit about when to call the tool.
- `set_light_environment` mode strings are **case-sensitive**.
- Set P0 as an output **before** PWM writes, or the fan will stay silent.

---

<br>

<!-- ================================ 中文 ================================ -->
<a id="-简体中文"></a>

## 🇨🇳 简体中文

### 这是什么？

两块**行空板 K10**，加一个大模型，组成一个**会主动响应的智能家居语音管家**。你推开房门，空气有点闷、光线有点暗——还没等你开口，风扇已经转起来，灯光也调到了合适的亮度和色温。

它不是「语音指令」的噱头。这个管家会**感知**环境、**理解**人话、**做出决策**，并通过真实硬件**执行**动作。

> **大模型 + 开放协议（MCP） + 真实硬件 = 可运行的智能空间系统。**

### 🧰 用了什么硬件

| 数量 | 元件 | 作用 |
|:--:|---|---|
| **2 块** | [行空板 K10](https://www.dfrobot.com.cn/goods-4014.html) | ESP32-S3 AI 开发板——一块做语音端，一块做硬件控制端 |
| **1 个** | [Gravity: 130 直流电机风扇](https://www.dfrobot.com.cn/goods-1251.html) | 由 P0 引脚 PWM 驱动 |
| **2 根** | [USB-C 数据线](https://www.dfrobot.com.cn/goods-2977.html) | 供电 + 烧录 |

**软件：** [Mind+ 2.0](https://mindplus.cc/en/download.html) · [小智固件](https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai) · [小智控制台](https://xiaozhi.me/)

### 🎯 这个项目做了什么

| 能力 | 实现方式 |
|---|---|
| **语音交互** | K10-A 运行小智固件，对接云端大模型，理解自然语言意图——不靠固定关键词 |
| **环境感知** | K10-B 暴露 `get_environment` 工具——板载温湿度数据回传给 AI |
| **风扇控制** | `set_fan(on)` → P0 输出 PWM（1023 = 开，0 = 关） |
| **场景灯光** | `set_light_environment(mode, …)` → `auto` / `study` / `relax` / `night` / `off` / `manual` |
| **主动行为** | 一段角色提示词，把「执行命令的机器」变成「体贴的管家」 |

### 🏗️ 系统架构

```
┌──────────────────────────┐        WebSocket / MCP        ┌──────────────────────────┐
│   行空板 K10-A           │  ───────────────────────────▶ │   行空板 K10-B           │
│   小智固件               │   get_environment / set_fan    │   Mind+ 程序             │
│   （语音 / AI 端）       │   set_light_environment        │   （执行端）             │
│                          │  ◀───────────────────────────  │                          │
│  • 语音识别              │        工具返回结果(JSON)      │  • 温湿度 / 光线         │
│  • 意图理解              │                                │  • P0 → 风扇 (PWM)       │
│  • 决策判断              │                                │  • 板载 RGB 灯           │
└──────────────────────────┘                                └──────────────────────────┘
                                        ▲
                                  ┌─────┴──────┐
                                  │  MCP 层    │  Model Context Protocol
                                  └────────────┘
```

**MCP（模型上下文协议）**是让 AI 用「调用工具」的方式调用硬件能力的开放标准。你只要新增一个工具、用大白话描述它，AI 自己就会判断什么时候该用它。

### 🚀 快速开始

```text
1. 给 K10-A 刷入小智固件        →  https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai
2. 配网 + 在小智控制台添加设备  →  https://xiaozhi.me/
3. 打开 Mind+ 2.0 → 上传模式 → 加载行空板 K10 库 + 小智 MCP 客户端库
4. 在 K10-B 上注册 3 个 MCP 工具：get_environment · set_fan · set_light_environment
5. 从控制台复制 MCP 接入点      →  粘贴进 Mind+ 积木
6. 上传到 K10-B，粘贴角色提示词 →  完成 ✅
```

👉 **完整分步教程：[`docs/TUTORIAL_CN.md`](docs/TUTORIAL_CN.md)** —— 每一步都有截图。

### 📂 资料怎么用

| 我想…… | 去哪里 |
|---|---|
| 从零照着做一遍 | [`docs/TUTORIAL_CN.md`](docs/TUTORIAL_CN.md) / [`docs/TUTORIAL_EN.md`](docs/TUTORIAL_EN.md) |
| 只拿角色提示词 | 教程 §5.8 —— 直接从代码块复制 |
| 看某一张截图 | [`images/`](images/) —— 文件已语义化命名 |
| 搞懂它*为什么*能工作 | 教程 §3 架构 与 §4 工作原理 |
| 移植到别的开发板 | 教程 §3 —— MCP 与具体板子无关 |
| 出问题了要排查 | 教程 §8 常见问题排查表 |

> **说明：** 演示视频未随仓库提供——GitHub 限制单文件 100 MB。文字教程 + 截图已足够完整复现。

### ⚠️ 注意事项与坑

- 两块 K10 必须在**同一个 WiFi** 下（名称与密码完全一致）。
- **工具描述比你想象的更重要**——这段文字是 AI 唯一的说明书，要写清楚「什么时候调用它」。
- `set_light_environment` 的 `mode` 字符串**区分大小写**。
- 一定要**先**把 P0 设为输出模式，再写 PWM，否则风扇不会转。

---

<div align="center">

### 📜 License

Documentation licensed under **CC BY-NC-SA 4.0** — see [LICENSE](LICENSE).

Built by [**mengyali878-jpg**](https://github.com/mengyali878-jpg) · Questions & ideas welcome via **Issues**

<sub>⭐ If this project helped you build something, a star is appreciated.</sub>

</div>
