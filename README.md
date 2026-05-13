# Ai-Harness

**多 Agent 自动化协作基础设施**

## 背景

[OpenSpec](./docs/01-openspec-overview.md) 是一个出色的规范层框架，但它缺少任务调度、Agent 间通信、自动化流水线能力。

[Ai-Harness](./docs/03-ai-harness-vision.md) 旨在补全这些缺失，构建真正的多 Agent 自动化协作系统。

## 文档索引

| 文档 | 说明 |
|------|------|
| [OpenSpec 概述](./docs/01-openspec-overview.md) | 什么是 OpenSpec，核心工作流，命令 |
| [OpenSpec 能力分析](./docs/02-openspec-capabilities.md) | 任务拆分/调度/多 Agent 能力分析 |
| [Ai-Harness 愿景](./docs/03-ai-harness-vision.md) | 项目定位、目标能力、架构方向 |
| [多 Agent 协作方案对比](./docs/04-multi-agent-landscape.md) | Superpowers / DeerFlow / CubeSandbox 深度分析 |
| [Everything Claude Code](./docs/05-everything-claude-code.md) | 181k stars 的 Agent 工具集 |
| [Hermes Agent](./docs/06-hermes-agent.md) | 147k stars 自改进 Agent |
| [Hermes Agent Self-Evolution](./docs/07-hermes-agent-self-evolution.md) | DSPy + GEPA 进化优化 |
| [Temporal](./docs/08-temporal.md) | 20k stars 持久化执行平台 |
| [Anthropic Skills](./docs/09-anthropic-skills.md) | 133k stars Agent Skills 官方仓库 |
| [重新审视 Agent 间通信](./docs/10-重新审视Agent间通信.md) | 已调研项目的 Agent 通信模式分析与缺口 |
| [OpenHarness](./docs/11-openharness.md) | 12k stars 多 Agent 协调基础设施 + ohmo 个人助手 |

## 核心思路

继承 OpenSpec 规范层，补全：

| 缺失能力 | 解决方案 |
|----------|----------|
| 任务调度 | 任务调度中心（Cron + 队列） |
| Agent 通信 | Agent 网关（消息总线） |
| 自动化流水线 | 任务队列 + 事件驱动 |

## 当前阶段

架构讨论阶段。详见 [多 Agent 协作方案对比](./docs/04-multi-agent-landscape.md#五大核心缺口)。

## 参考项目

- [OpenSpec](https://github.com/Fission-AI/OpenSpec) — 规范层框架
- [Superpowers](https://github.com/obra/superpowers) — Agent skills framework + TDD 方法论
- [DeerFlow](https://github.com/bytedance/deer-flow) — Super Agent Harness（Sandbox + Sub-agents + Memory）
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) — KVM 沙箱，<60ms 冷启动
- [Everything Claude Code](https://github.com/affaan-m/everything-claude-code) — 181k stars 的跨平台 Agent 工具集
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — 147k stars 自改进 AI Agent
- [Hermes Agent Self-Evolution](https://github.com/NousResearch/hermes-agent-self-evolution) — DSPy + GEPA 进化优化
- [Temporal](https://github.com/temporalio/temporal) — 20k stars 持久化执行平台
- [Anthropic Skills](https://github.com/anthropics/skills) — 133k stars Agent Skills 官方仓库
- [重新审视 Agent 间通信](https://github.com/relunctance/ai-harness/blob/main/docs/10-重新审视Agent间通信.md) — 已调研项目的 Agent 通信模式分析与缺口
- [OpenHarness](https://github.com/HKUDS/OpenHarness) — 12k stars 多 Agent 协调基础设施 + ohmo 个人助手
