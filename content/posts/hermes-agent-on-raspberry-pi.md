---
title: "在树莓派 4B 4GB 上部署 Hermes Agent：硬件限制与优化指南"
date: 2026-06-13
draft: false
tags: ["Hermes-Agent", "Raspberry-Pi", "AI-Agent", "Self-Hosting", "Optimization"]
---

## 为什么要在树莓派上跑 AI Agent？

你可能看过不少 Hermes Agent 的教程，要么在云服务器上部署，要么在主力开发机上跑。把这么个 AI 智能体塞进一块巴掌大的树莓派里，听起来像是自找麻烦。

初期确实有点折腾。但跑顺之后，我觉得这是目前性价比最高的个人 AI Agent 方案。电费忽略不计，7×24 在线，安静无噪音。这篇文章分享我在树莓派 4B（4GB 版本）上部署和运行 Hermes Agent 的真实体验，包括硬件瓶颈、性能调优和一些省钱技巧。

## 硬件规格与真实表现

| 项目 | 参数 |
|------|------|
| 设备 | Raspberry Pi 4 Model B Rev 1.5 |
| 内存 | 4GB LPDDR4（实际可用约 3.7Gi） |
| 存储 | 32GB SD 卡（剩余约 12GB） |
| 系统 | Ubuntu 24.04.4 LTS |
| Hermes | v0.16.0 |
| 运行时温度 | 42°C（闲置） |

整体结论：**4GB 是及格线，2GB 版本不建议尝试。**

Hermes Agent 本身只占 200-300MB 内存，真正的内存大户是浏览器自动化（Chromium）。不跑浏览器任务时剩余 2.7Gi 绰绰有余，但如果你同时开浏览器截图 + terminal 编译 + 多个子任务，内存会逼近 3GB。

CPU 不是瓶颈，因为 LLM 调用全部走远程 API，树莓派只需要处理工具调用和文本流，ARM Cortex-A72 四核在闲置时负载不到 0.1。但如果你想在 Pi 上跑本地模型（比如通过 llama.cpp 跑量化模型），情况完全不同——4B 没有 GPU，纯 CPU 推理速度只有几 token/s，基本不可用。

真正的短板是 SD 卡的 IO 速度。频繁的文件读写（git 大仓库、大量日志写入）偶尔能感觉到卡顿。有条件的话建议用 SSD 通过 USB 3.0 外接。

国内用户还需要解决网络问题。我用 Clash + mihomo（HTTP 7890 端口）做代理，通过 `http_proxy` 环境变量让 Hermes 走代理访问 GitHub 和 DeepSeek 的 API。国内可以直接访问的平台（阿里云百炼、MiniMax）不需要走代理。

## 部署过程

Hermes 的安装出乎意料地顺利，官方提供了一键安装脚本：

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

v0.16.0 在 Ubuntu 24.04 上即装即用，ARM64 支持很完整。如果安装后要配 OpenCode CLI 或 GitNexus，Python 3.11+ 和 Node.js v22 基本覆盖了所有需求。需要注意 GitNexus 的 Tree-sitter 原生插件在 ARM64 上没有预编译二进制，需要从源码编译，大概 4 分钟能编完。

因为不在本地跑模型，需要配好远程 LLM 提供商。我目前的主力方案：

| 用途 | Provider | 模型 | 原因 |
|------|----------|------|------|
| 主力对话 | OpenCode Go | deepseek-v4-flash | 速度快，日常够用 |
| 复杂任务 | DeepSeek | deepseek-v4-pro | 深度推理 |
| 视觉识别 | Xiaomi MiMo | mimo-v2.5 | 图片分析 |
| 编码子代理 | DeepSeek | deepseek-v4-flash | 走直连通道 |

路由策略：日常走 OpenCode Go 的 flash，复杂编码和推理切 DeepSeek pro，需要看图切 MiMo。

## 运行表现

这台 Pi 已经连续跑了 3 天多，Hermes 没崩过一次。日常包括 QQ 聊天、OpenCode 编码子代理、GitHub 仓库管理、定时记忆同步、Hugo 博客部署。最让我意外的是 QQ Gateway 的稳定性——作为 24 小时在线的聊天机器人服务器，表现得非常靠谱。

几个实际遇到过的卡点：
1. **浏览器任务**：Hermes 的 browser 工具需要启动 Chromium，在 Pi 上加载页面要多等几秒，截图明显比 x86 慢
2. **大量文件操作**：git 大仓库或批量文件处理时，SD 卡 IO 瓶颈显现
3. **并发子任务**：同时跑 3 个子代理，内存从 1GB 飙升到 2.5GB 左右，还在安全范围内

## 优化指南

### 1. Provider 选择：API 优先

树莓派 4B 不适合跑本地 LLM。把推理外包给远程 API，树莓派专心做编排和执行，是最合理的分工。

选 API 提供商主要看延迟和模型多样性。DeepSeek 在国内有节点，延迟比 OpenAI 好不少，同一个账号下 flash 和 pro 切换也方便。

### 2. 模型路由：按任务分级

```
简单任务（聊天、查询）        → flash
中等任务（代码审查、单文件修改）→ flash
复杂任务（多文件重构、深度分析）→ pro
视觉任务（图片理解）          → vision
```

DeepSeek v4 flash 的成本只有 pro 的十分之一，日常 80% 的场景 flash 完全够用，别所有任务都上 pro。

### 3. 代理策略

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

国内平台不需要代理，注意区分即可。用 `clashctl` 可以方便地切换节点。

### 4. 省 Token 的几个习惯

- **Session 压缩**：Hermes 内置的 context compression 默认开启，接近 token 上限时自动压缩历史，省掉不必要的重发
- **技能复用**：重复的工作流写成 skill，每次加载就行，不用从头描述
- **YOLO 模式按需开**：不想每次执行命令都确认，用 `/yolo` 在 session 里临时开关，不建议全局开启

### 5. 记忆与技能

Hermes 的记忆系统在 Pi 上跑得很轻量。`memory_store.db` 只有 64KB，`config.yaml` 16KB，几乎不占资源。我设置了每天凌晨 2 点的 cron 自动同步记忆到 GitHub 私仓，既能备份也能跨设备迁移。

Skills 是 Hermes 最被低估的功能。每解决一个重复问题就写成 skill，下次直接加载。写 skill 的要点：触发条件写清楚、步骤带命令、包含验证步骤、踩过的坑也写进去。

## 总结

树莓派 4B 4GB 跑 Hermes Agent 完全可行。RAM 是硬门槛（至少 4GB），CPU 不是瓶颈，存储 IO 和网络（国内用户）是需要留意的点。

如果你手头正好有一块吃灰的树莓派 4B，不妨试试。不需要新硬件，不需要云服务器，一块小板子加一个 API key 就能拥有一个 7×24 在线的 AI 助手。我这台的日常：白天写代码查资料，晚上跑定时任务，凌晨安静得像不存在。电费大概跟一个 USB 小风扇差不多。
