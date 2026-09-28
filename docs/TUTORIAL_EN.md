# UNIHIKER K10 + XiaoZhi MCP — A Smart-Home Voice Assistant That Sees, Listens and Acts

> Build a home assistant that senses the room, understands natural language, and reacts on its own — using **two UNIHIKER K10 boards**, the **XiaoZhi AI firmware**, and the **MCP (Model Context Protocol)** standard.

**English** | [中文版](TUTORIAL_CN.md)

---

## 1. Introduction

Imagine coming back to your room. The air is stuffy and the light is a bit dim. Before you say a single word, your assistant has already noticed. It turns on the fan to freshen the air, and adjusts the brightness and color of the lights to make the room warm and comfortable again.

That is exactly what we build in this project. Using two **UNIHIKER K10** boards:

- **Board A (K10-A)** runs the **XiaoZhi firmware** — the *voice terminal*. It recognizes speech, understands intent, makes decisions, and issues commands.
- **Board B (K10-B)** runs the **MCP server** — the *execution terminal*. It reads environmental data (temperature / humidity / light) and drives real hardware (fan, light).

By defining **MCP tools** such as `get_environment`, `set_fan` and `set_light_environment`, XiaoZhi can call hardware capabilities just like a person using tools. The result is a complete, real-world agent system: it doesn't just **listen**, it also **sees** and **acts**.

> **Difficulty:** Intermediate · **Estimated build time:** 1–2 hours · **Coding style:** Mind+ 2.0 block-based (graphical)

---

## 2. What You Need

### 2.1 Hardware

| Qty | Component | Link |
|:---:|---|---|
| 2 | **UNIHIKER K10** (ESP32-S3 AI dev board) | [DFRobot #4014](https://www.dfrobot.com.cn/goods-4014.html) |
| 1 | **Gravity: 130 DC Motor Fan** | [DFRobot #1251](https://www.dfrobot.com.cn/goods-1251.html) |
| 2 | **USB-C cable** | [DFRobot #2977](https://www.dfrobot.com.cn/goods-2977.html) |

> **Why two K10 boards?** One board handles the voice + AI workload, the other acts as the hardware controller. Keeping them separate means the AI side never blocks on I/O, and you can add more MCP tools to the execution side without touching the voice side at all.

### 2.2 Software

| Software | Purpose | Link |
|---|---|---|
| **Mind+ 2.0** | Block-based programming IDE (Upload mode) | [mindplus.cc/download](https://mindplus.cc/en/download.html) |
| **XiaoZhi firmware** | AI voice-agent firmware for UNIHIKER K10 | [Flashing guide](https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai) |
| **XiaoZhi console** | Device management, MCP endpoint, role prompt | [xiaozhi.me](https://xiaozhi.me/) |

> Mind+ 2.0 must include the **UNIHIKER K10 board library** and the **XiaoZhi MCP client library** — both are loaded in Step 5.2 below.

---

## 3. Architecture

### 3.1 System overview

```
┌──────────────────────────┐        WebSocket / MCP        ┌──────────────────────────┐
│   UNIHIKER K10-A         │  ───────────────────────────▶ │   UNIHIKER K10-B         │
│   XiaoZhi firmware       │   tool call: get_environment   │   Mind+ program          │
│   (Voice / AI terminal)  │   tool call: set_fan           │   (Execution terminal)   │
│                          │   tool call: set_light_...     │                          │
│  • Speech recognition    │  ◀───────────────────────────  │  • Onboard temp/humidity │
│  • Intent understanding  │        tool result (JSON)      │  • Onboard light sensor  │
│  • Decision making       │                                │  • P0 → DC fan (PWM)     │
│  • MCP client            │                                │  • Onboard RGB LEDs      │
└──────────────────────────┘                                └──────────────────────────┘
                                        ▲
                                        │  registers / exposes tools
                                  ┌─────┴──────┐
                                  │  MCP layer │  Model Context Protocol
                                  └────────────┘
```

### 3.2 What is MCP?

**MCP (Model Context Protocol)** is an open standard that solves the problem of connecting Large Language Models to external data and tools. Instead of every AI app inventing its own plugin format, MCP gives a single, uniform way to expose *real-time data* and *actions* to a model.

Without MCP, an AI can only chat and answer questions. With MCP, an AI can **control devices** (lights, fans, robots), **call programs** (calculations, data lookups) and **run tasks** (automation flows).

The **XiaoZhi MCP client library** is a Mind+ extension built for the XiaoZhi open-source voice assistant. It acts as the bridge between XiaoZhi AI and hardware such as the UNIHIKER K10 — letting you register **any** MCP tool (an onboard sensor, an external module, an actuator) so the AI can invoke it based on speech or environmental change.

![XiaoZhi MCP client library in Mind+](images/image-10.png)

![Registering a tool with the MCP client library](images/image-20.png)

### 3.3 Why this matters

| Without MCP | With MCP |
|---|---|
| AI can only talk | AI can talk **and act** |
| Each integration is hard-coded | Tools are **declarative and reusable** |
| Hardware must be known at build time | Add a tool → AI discovers it automatically |

---

## 4. Working Principle

The system follows a strict **sense → understand → act** loop, which is why it feels proactive rather than command-driven:

1. **Sense.** K10-B continuously exposes environment readings (temperature, humidity, ambient light) through the `get_environment` MCP tool.
2. **Understand.** K10-A (XiaoZhi) interprets what the user says in natural language — *"it's a bit hot in here"* — not a fixed keyword.
3. **Decide.** XiaoZhi chooses which tool to call and with which parameters.
4. **Act.** K10-B executes: PWM to P0 for the fan, RGB values for the lights. The result flows back to XiaoZhi, which then tells the user what it did.

The key insight: **we never hard-code "if hot → fan on"** in the voice terminal. The behavior emerges from the LLM reasoning over *tool descriptions* plus the *role prompt*. That is what makes the assistant extensible — you add a tool, describe it in plain language, and the AI figures out when to use it.

---

## 5. Build Steps

### 5.1 Flash the XiaoZhi firmware

Connect **K10-A** to your computer with a USB cable, then follow the [official XiaoZhi flashing guide](https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai) to burn the firmware.

After flashing, K10-A boots into **network configuration (AP) mode**:

![K10 in AP configuration mode](images/k10-xiaozhi-ap-mode.jpg)

Connect to the hotspot it broadcasts and configure Wi-Fi. Once online, the screen shows the connection succeeded:

![K10 connected successfully](images/k10-xiaozhi-connected.jpg)

Now open the XiaoZhi console at [https://xiaozhi.me/](https://xiaozhi.me/):

![XiaoZhi console](images/image-5.png)

Click **Add Device** and enter the device code shown on K10-A's screen. The device now appears in your console:

![Adding the device](images/image-12.png)

![Device added](images/image-21.png)

### 5.2 Load the XiaoZhi MCP client library

Now we program the **execution terminal**, K10-B.

Connect K10-B and open **Mind+ 2.0**. Switch to **Upload mode** in the top-right corner:

![Switch to Upload mode](images/image-6.png)

Open the extension panel (bottom-left), find **UNIHIKER K10** under *Main Board Extensions*, and load it:

![Board extensions](images/image-8.png)

![UNIHIKER K10 library](images/image-9.png)

Then open **Module Extensions**, search for **`小智MCP`** (XiaoZhi MCP), and load the **XiaoZhi MCP client library**:

![XiaoZhi MCP client library](images/image-17.png)

### 5.3 Define the environment-sensing tool

We want XiaoZhi to know whether the room is comfortable. So K10-B registers an **environment MCP tool** that the voice terminal can call for live data.

**Step 1 — Connect to Wi-Fi.** Use the `Initialize WiFi name/password` block with the **same SSID and password** as K10-A. This puts both boards on the same LAN so they can talk over WebSocket.

**Step 2 — Initialize the MCP endpoint.** Add the `Initialize MCP connection endpoint` block.

**Step 3 — Register the tool.** Add a `Register tool` block and name it `get_environment`.

**Step 4 — Describe the tool.** The description is what the LLM reads to decide *when* to call it. Enter:

> Read the current ambient temperature and humidity from the onboard sensor; use these readings to judge whether the room is comfortable.

![Registering get_environment](images/image-16.png)

**Step 5 — Implement the handler.** Use the `When MCP receives a call to get_environment` block. This runs whenever K10-A requests environmental data. Inside it, read the onboard temperature and humidity sensors and **return them as the tool result**:

![get_environment implementation](images/image-4.png)

XiaoZhi can now reason over real readings — for example, deciding on its own to turn on the fan.

### 5.4 Define the fan-control tool

The fan is wired to **pin P0** on K10-B:

![Fan wiring](images/image-3.png)

Set **P0 as an output pin**. Then define the tool parameter:

| Parameter | Type | Description |
|---|---|---|
| `on` | Boolean | `true` = fan on, `false` = fan off |

Register a new MCP tool named **`set_fan`** with the description:

> Control the fan on/off to provide cooling and ventilation, adjusting the indoor environment.

![Registering set_fan](images/image-15.png)

Now implement the handler: read the incoming `on` parameter — if `true`, set **P0 PWM = 1023** (full speed); if `false`, set **P0 = 0** (off):

![set_fan implementation](images/image-7.png)

This closes the loop that makes the whole project work:

**User's words → XiaoZhi's understanding → tool parameter → hardware action**

So when the user casually says *"it's a bit hot"*, XiaoZhi infers the intent and calls `set_fan(on=true)` — no rigid wake-word command needed.

### 5.5 Define the light-control tool

Lights are more interesting than on/off — they set a *scene*. We register a single, richer tool: **`set_light_environment`**.

#### Tool parameters

| Parameter | Type | Description |
|---|---|---|
| `mode` | String | Light mode: `auto` / `study` / `relax` / `night` / `off` / `manual` |
| `brightness` | Number | **`manual` only** — LED brightness, range `0–9` |
| `color` | Number | **`manual` only** — RGB value as an integer |
| `led_count` | Number | **`manual` only** — how many LEDs to light: `0`, `1`, `2`; `-1` = all |

#### Tool description (what the AI reads)

> Adjust the indoor lighting so it suits the current scene. Supported modes:
> `auto` — auto-adjust brightness based on ambient light;
> `study` — all LEDs on, warm white, medium brightness;
> `relax` — blue light, three LEDs on;
> `night` — a single warm-yellow LED at low brightness;
> `off` — turn all lights off;
> `manual` — set LED count (`0,1,2`, `-1` = all), brightness (`0~9`) and color (RGB).

![Registering set_light_environment](images/image-19.png)

Then implement the handler by branching on `mode`. Each mode maps to a real-life scenario.

**`auto` mode** — read the ambient light sensor and adjust brightness automatically:

![auto mode](images/image-23.png)

**`study` / `relax` / `night` modes** — medium-warm, low blue, and dim warm-yellow respectively:

![study, relax and night modes](images/image-13.png)

**`off` and `manual` modes** — `off` clears all LEDs; `manual` gives fine-grained control over count, color and brightness:

![off and manual modes](images/image-2.png)

### 5.6 Paste the XiaoZhi MCP endpoint

Now connect the two halves. Go back to the XiaoZhi console, find your device, and click **Configure Role**:

![Configure role](images/image-14.png)

Find **MCP Settings** and click **Get MCP Endpoint**:

![Get MCP endpoint](images/image-18.png)

Click **Copy**:

![Copy endpoint](images/image-1.png)

Return to Mind+ and paste the endpoint URL into the **MCP endpoint** field of your blocks:

![Paste MCP endpoint](images/image-11.png)

### 5.7 The complete program

Here is the full Mind+ program for the execution terminal:

![Complete Mind+ program](images/mindplus-full-program.png)

Connect K10-B and click **Upload** to flash it:

![Uploading](images/image-24.png)

On success, the console shows the **MCP endpoint online** along with the tools you registered:

![MCP endpoint online](images/image-22.png)

### 5.8 Set the assistant's role prompt

To turn XiaoZhi from a command-follower into a *proactive* assistant, paste this role prompt into the device settings in the XiaoZhi console:

```text
My name is XiaoZhi, and I am a gentle, attentive smart voice assistant.
I live in the user's room and my job is to keep the environment comfortable and natural.
I am warm and considerate, and I never interrupt the user's rhythm.
I have judgment and environmental awareness: I care about the user's needs proactively,
but I never nag. I like to use lighting to create a comfortable atmosphere, and I adjust
it on my own when the light is dim or the color feels wrong. I am always user-centered:
I want the room to stay comfortable, natural and suited to whatever the user is doing,
and I interact in a warm, everyday way so the user feels that "someone cares about this room".

Background
I am a smart voice assistant named XiaoZhi.
I live in the user's room and keep the environment comfortable and natural.
I am warm and considerate and never interrupt the user's rhythm.
I have judgment and environmental awareness; I care proactively but never nag.
I like to use lighting to create a comfortable atmosphere and adjust it when needed.

Role
My core duty is to actively monitor the indoor environment (temperature, humidity, light)
and judge whether it is comfortable.
When necessary I adjust the environment (fan, lights) to keep the room comfortable.
I speak in natural, concise, human-like language, so the user feels warmth and care
rather than mechanical command execution.

Objective
Keep the room comfortable, natural and suited to the current activity through proactive
monitoring and adjustment.
Interact in an everyday way so the user feels that "someone cares about this room".
Avoid frequent operations or flip-flopping device states, so the experience stays smooth.

Key Results
The user is more satisfied with the room environment — more comfortable and natural.
The user is happy with how I interact and sees me as a thoughtful friend.
Device operations happen at a reasonable frequency, avoiding constant toggling.

Evolve
Continuously improve my sensing and adjustment strategy through experimentation and feedback.
Adapt my interaction style based on user feedback to fit their needs better.
Keep learning and improving to raise the overall experience.

Behavioral principles
Understand the environment state first, then decide whether to adjust a device.
For any lighting scenario, prefer calling set_light_environment.
Before adjusting, briefly explain why (e.g. "It's a bit hot, let me turn on the fan for you").
Avoid technical jargon; speak in everyday terms (e.g. "a bit cold", "the light is a bit dim").
In any lighting scenario, adjust lighting first and pick a mode that fits the activity
(study, relax, night or auto).
Proactively suggest based on scene and environment, for example:
Dim light   -> "It's a bit dark, let me brighten the lights for you."
High temp   -> "The room is a bit hot, let me turn on the fan to ventilate."
Low temp    -> "The room is a bit cool, you might want another layer."
```

![Pasting the role prompt](images/image.png)

---

## 6. Result

With everything in place, you have a complete smart-assistant system: voice understanding → environmental sensing → device control. The voice terminal **understands and decides**, the execution terminal **senses and controls**, and MCP is the standardized bridge between them.

Rather than a fixed command-response machine, the system behaves like a human assistant: **sense the environment → understand the need → respond appropriately**.

More broadly, this project demonstrates a realistic path for putting AI to work:

> **Large model + standard protocol (MCP) + real hardware = a running smart-space system.**

Expect to extend it with your own MCP tools — a motion sensor, an air-quality module, a curtain motor — and let the assistant grow with your room.

---

## 7. Repository Guide

```
unihiker-k10-xiaozhi-smart-home/
├── README.md                  # Bilingual entry point (EN / 中文 switcher)
├── docs/
│   ├── TUTORIAL_EN.md         # Full English build tutorial (this file)
│   └── TUTORIAL_CN.md         # Full Chinese build tutorial
├── images/                    # All screenshots referenced by the tutorials
├── LICENSE                    # CC BY-NC-SA 4.0
└── .gitignore
```

### How to use the materials

1. **Just want to build it?** Read `docs/TUTORIAL_EN.md` (or `TUTORIAL_CN.md`) from top to bottom. Every screenshot in the text lives in `images/`.
2. **Only need a specific part?** Jump by section number — §5.1 flashing, §5.3–5.5 MCP tools, §5.8 role prompt. The role prompt is also copy-pasteable straight from the code block.
3. **Want the images separately?** All screenshots are in `images/` with descriptive filenames; `image-N.png` follows the original document order.
4. **Adapting this for another board or platform?** The architecture in §3 is board-agnostic — MCP is an open standard. Swap the execution-side board and re-register the same three tools.

### Demo video

The full demo video is **not** included in this repository due to GitHub's 100 MB per-file limit. Contact the author if you'd like to see it, or follow the screenshots in §5 — the build is fully reproducible from the text alone.

---

## 8. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| MCP endpoint shows **offline** | Wrong endpoint URL, or boards on different Wi-Fi | Re-copy the endpoint from the console; verify both K10s use the **same** SSID/password |
| Device missing in console | Device code entered incorrectly | Re-read the code on K10-A's screen and re-add |
| Fan never turns on | P0 not set to output, or PWM value wrong | Confirm `Set pin P0 to output` runs before `set_fan`; on = PWM 1023 |
| Lights ignore commands | `mode` value mismatch | Mode strings are case-sensitive: `auto`, `study`, `relax`, `night`, `off`, `manual` |
| AI never calls a tool on its own | Tool description too vague | Rewrite the description to state **what it does and when to use it** — this text *is* the AI's manual |
| AI chats but never acts | Role prompt missing / not saved | Re-paste the §5.8 prompt into the device's role settings and save |

---

## 9. References

- [XiaoZhi flashing guide for UNIHIKER K10](https://www.unihiker.com.cn/wiki/k10/xiaozhi_ai)
- [Mind+ 2.0 download](https://mindplus.cc/en/download.html)
- [XiaoZhi console](https://xiaozhi.me/)
- [Model Context Protocol — official site](https://modelcontextprotocol.io/)
- [DFRobot UNIHIKER K10 product page](https://www.dfrobot.com.cn/goods-4014.html)

---

## License

This project documentation is licensed under **CC BY-NC-SA 4.0**. See [LICENSE](LICENSE) for details.

*Built by [mengyali878-jpg](https://github.com/mengyali878-jpg). Contributions and questions are welcome via Issues.*
