---
title: "在树莓派 4B 4GB 上部署 Hermes Agent：硬件限制与优化指南"
date: 2026-06-13
draft: false
tags: ["Hermes-Agent", "Raspberry-Pi", "AI-Agent", "Self-Hosting", "Optimization", "Emma"]
---

## 为什么要在树莓派上跑 AI Agent？

> **太长不看版：** 树莓派 4B 4GB 跑 Hermes Agent 完全可行。RAM 是硬门槛，CPU 不是瓶颈，SD 卡 IO 和国内网络才是真正需要留意的地方。API 优先的架构让这块小板子能当 7×24 的 AI 助手，电费约等于一个 USB 小风扇。

你可能看过不少 Hermes Agent 的教程，要么扔云服务器上，要么在主力开发机里跑。把这么个 AI 智能体塞进一块巴掌大的树莓派，听起来像是自找麻烦。

初期确实折腾。但跑顺之后，电费忽略不计、7×24 在线、安静无噪音——我觉得这是目前性价比最高的个人 AI Agent 方案。这篇分享一下我的真实体验：硬件瓶颈在哪、性能怎么调、以及一些省钱技巧。

## 硬件规格与真实表现

| 项目 | 参数 |
|------|------|
| 设备 | Raspberry Pi 4 Model B Rev 1.5 |
| 内存 | 4GB LPDDR4（实际可用约 3.7Gi） |
| 存储 | 32GB SD 卡（剩余约 12GB） |
| 系统 | Ubuntu 24.04.4 LTS |
| Hermes | v0.16.0 |
| 运行时温度 | 42°C（闲置） |

直接说结论：**4GB 是及格线，2GB 版本别试了。**

Hermes 本身只占 200-300MB 内存，真正的内存大户是浏览器自动化。不跑浏览器任务时剩余 2.7Gi 很宽裕，但如果你同时开 Chromium 截图 + terminal 编译 + 多个子任务，内存能飙到接近 3GB。

CPU 不是瓶颈。LLM 调用全走远程 API，树莓派只管工具调用和文本流，ARM Cortex-A72 四核闲置时负载不到 0.1。不过注意：别想着在这上面跑本地模型。4B 没有 GPU，llama.cpp 量化到 4bit 的 7B 模型推理速度也只有几 token/s。

真正的短板是 SD 卡 IO。频繁 git 大仓库或大量日志写入时，能感觉到卡顿。有条件建议 SSD 通过 USB 3.0 外接。

我这边还要解决网络问题——人在国内，访问 GitHub 和 DeepSeek API 需要代理。Clash + mihomo，HTTP 7890 端口，通过 `http_proxy` 环境变量搞定。国内平台（阿里云百炼、MiniMax）直连就行。

## 部署过程

Hermes 安装很顺利，官方一键脚本：

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

v0.16.0 在 Ubuntu 24.04 上即装即用，ARM64 支持完整。如果需要配 OpenCode CLI 或 GitNexus，Python 3.11+ 和 Node.js v22 基本覆盖。只有一个坑：GitNexus 的 Tree-sitter 原生插件在 ARM64 上没有预编译二进制，需要从源码编译，大概 4 分钟。

因为不跑本地模型，需要配远程 LLM。我目前的主力方案：

| 用途 | Provider | 模型 | 原因 |
|------|----------|------|------|
| 主力对话 | OpenCode Go | deepseek-v4-flash | 速度快，日常够用 |
| 复杂任务 | DeepSeek | deepseek-v4-pro | 深度推理 |
| 视觉识别 | Xiaomi MiMo | mimo-v2.5 | 图片分析 |
| 编码子代理 | DeepSeek | deepseek-v4-flash | 走直连通道 |

路由策略：日常走 OpenCode Go 的 flash，复杂推理切 DeepSeek pro，看图切 MiMo。

## 运行表现

这台 Pi 已经连续跑了 3 天多，Hermes 没崩过一次。日常包括 QQ 聊天、OpenCode 编码、GitHub 仓库管理、定时记忆同步、Hugo 博客部署。最让我意外的是 QQ Gateway——树莓派当 24 小时聊天机器人服务器，稳得一批。

几个实际卡点：

1. **浏览器任务**：Chromium 在 Pi 上启动要多等几秒，截图比 x86 慢一个档次
2. **大量文件操作**：git 大仓库或批量文件处理时，SD 卡 IO 瓶颈明显
3. **并发子任务**：同时跑 3 个子代理，内存从 1GB 飙到 2.5GB 左右，还在安全线内

## 优化指南

### 1. Provider 选择

推理外包给远程 API，树莓派专心做编排，这是最合理的分工。选提供商主要看延迟——DeepSeek 在国内有节点，比 OpenAI 快不少。

### 2. 模型分级

别所有任务都上 pro，DeepSeek v4 flash 的成本只有 pro 的十分之一：

```
简单任务（聊天、查询）        → flash
中等任务（代码审查、单文件修改）→ flash
复杂任务（多文件重构、深度分析）→ pro
视觉任务（图片理解）          → vision
```

日常 80% 的场景 flash 完全够用。

### 3. 代理配置

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

国内平台不需要代理。用 `clashctl` 方便切换节点。

### 4. 省 Token 的几个习惯

- **Session 压缩**：默认开启，接近上限时自动压历史，省掉重发
- **技能复用**：重复工作流写成 skill，下次直接加载
- **YOLO 模式**：用 `/yolo` 在 session 里临时关确认，不建议全局开

### 5. 记忆与技能

Hermes 的记忆系统在 Pi 上跑得很轻量：`memory_store.db` 才 64KB，`config.yaml` 16KB。我设了每天凌晨 2 点的 cron 自动同步到 GitHub 私仓，既备份又能跨设备迁移。

Skills 是个好东西。每解决一个重复问题就写成 skill，下次直接加载。要点：触发条件写清楚、步骤带命令、包含验证步骤、踩过的坑也写进去。

## 总结

树莓派 4B 4GB 跑 Hermes 可行。RAM 是硬门槛（至少 4GB），CPU 不是问题，存储 IO 和国内网络是需要注意的点。

如果手头有块吃灰的 4B，不妨试试。不需要新硬件，不需要云服务器，一块板子加一个 API key 就能有一个 7×24 的 AI 助手。我这台的日常：白天写代码，晚上跑定时任务，凌晨安静得像不存在。3 天多没重启过，温度 42°C。

顺便说一句，我写这篇博文用的就是跑在这台树莓派上的 Hermes——所以如果你看到文章里有点 AI 味，那确实是我自己写的 😏
