# Ai-Harness

**多 Agent 自动化协作基础设施**

## 背景

[OpenSpec](./docs/01-openspec-overview.md) 是一个出色的规范层框架，但它缺少任务调度、Agent 间通信、自动化流水线能力。

[Ai-Harness](./docs/03-ai-harness-vision.md) 旨在用**已有开源组件**补全这些缺失，构建真正的多 Agent 自动化协作系统。

**约束**：不自研，用已有调研的组件。

## 目标

用已有组件，实现"需求→拆解→评审→设计方案→开发→测试→验收"的多 Agent 协作，**人在关键节点把关**。

## 阶段一 MVP 组件清单

| 阶段 | 组件 | 解决什么问题 |
|------|------|-------------|
| **主控 Agent** | **OpenHarness** | 需求理解、Agent Loop、Swarm 管理、Cron 调度、ohmo 飞书助手、Sandbox、Memory |
| **多 Agent 协调** | **ClawTeam-OpenClaw** | Task Dependencies、Team 模板、Git Worktree 隔离、Cost Dashboard、inbox 通信 |
| **规范层** | **OpenSpec** | 任务规范定义、spec-driven 开发、标准命令 |
| **开发方法论** | **Superpowers** | brainstorming（需求澄清）、writing-plans（任务分解）、subagent-driven（并行开发）、TDD（测试驱动）、requesting-code-review（评审） |
| **沙箱执行** | **CubeSandbox** | 代码在隔离环境中执行，<60ms 冷启动，<5MB 内存 |
| **Workflow 持久化** | **Temporal** | 流水线状态持久化、自动重试、失败恢复、历史记录 |
| **CI/CD** | **GitHub Actions** | 自动化测试、构建、部署 |
| **Human** | **你** | 验收签字、关键决策、需求澄清 |

## 架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      Human                                  │
│         飞书/审批 → 验收签字 · 需求澄清 · 关键审批            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  OpenHarness (主控 Agent)                                  │
│  · ohmo — 飞书/Slack/Discord/Telegram 个人助手              │
│  · Agent Loop — query → stream → tool-call → loop         │
│  · Swarm — team/agent/send_message/task 管理              │
│  · CronCreate/List/Delete — 任务调度                       │
│  · Sandbox — 隔离执行环境                                  │
│  · Memory — 长期记忆                                       │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  ClawTeam-OpenClaw (多 Agent 协调)                         │
│  · clawteam spawn — 托身独立 Worker Agents                 │
│  · Task Dependencies — --blocked-by + auto-unblock         │
│  · Team Templates — TOML 模板                              │
│  · inbox send/peek/broadcast — Agent 间消息通信             │
│  · Cost Dashboard — 实时 token/cost 追踪                   │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  OpenSpec (规范层)  │  Superpowers (TDD)  │  CubeSandbox (沙箱)  │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  Temporal (Workflow 持久化)  │  GitHub Actions (CI/CD)       │
└─────────────────────────────────────────────────────────────┘
```

## 核心流程

```
Human: "做一个用户登录功能"
    │
    ▼
DeerFlow 2.0 (需求理解)
    │ Lead Agent 接收需求
    │ 拆解 → [注册任务, 登录任务, 登出任务, 记住我功能]
    │
    ▼
ClawTeam-OpenClaw (多 Agent 协调)
    │ clawteam launch auth-team --goal "实现登录功能"
    │ ├── Task Dependencies: API schema → auth + DB → frontend → tests
    │ └── inbox 通信: "Here's the OpenAPI spec", "Auth endpoints ready"
    │
    ▼
OpenSpec (规范定义)
    │ spec → 具体的 task definitions
    │
    ▼
Superpowers (开发方法论)
    │ brainstorming → writing-plans → subagent-driven
    │
    ├──► Agent-A: 实现注册功能 (Sandbox + Git Worktree)
    ├──► Agent-B: 实现登录功能 (Sandbox + Git Worktree)
    ├──► Agent-C: 编写测试 (TDD)
    │
    ▼
Temporal (流水线编排 + 持久化)
    │ 状态持久化 / 自动重试 / 历史记录
    │
    ▼
GitHub Actions (CI/CD)
    │ 测试全绿 / 构建通过
    │
    ▼
Human 验收签字 ✅
```

## 组件角色分工

| 组件 | 来源 | 调研文档 |
|------|------|---------|
| DeerFlow 2.0 | https://github.com/bytedance/deer-flow | [12-deerflow-2](./docs/12-deerflow-2.md) |
| ClawTeam-OpenClaw | https://github.com/win4r/ClawTeam-OpenClaw | (见下方说明) |
| OpenSpec | https://github.com/Fission-AI/OpenSpec | [01-openspec-overview](./docs/01-openspec-overview.md) |
| Superpowers | https://github.com/obra/superpowers | [04-multi-agent-landscape](./docs/04-multi-agent-landscape.md) |
| CubeSandbox | https://github.com/TencentCloud/CubeSandbox | [04-multi-agent-landscape](./docs/04-multi-agent-landscape.md) |
| Temporal | https://github.com/temporalio/temporal | [08-temporal](./docs/08-temporal.md) |

## 不做什么

- ❌ 不做自己的 AI 模型
- ❌ 不做 IDE 或编辑器
- ❌ 不自研消息队列/任务队列/调度器
- ❌ 不重复造轮子

## 当前阶段

架构讨论阶段。详见 [多 Agent 协作方案对比](./docs/04-multi-agent-landscape.md)。

## 文档索引

| 文档 | 说明 |
|------|------|
| [OpenSpec 概述](./docs/01-openspec-overview.md) | OpenSpec 核心工作流、命令、哲学 |
| [OpenSpec 能力分析](./docs/02-openspec-capabilities.md) | 任务拆分/调度/多 Agent 协作能力分析 |
| [Ai-Harness 愿景](./docs/03-ai-harness-vision.md) | 项目定位、目标能力、架构方向 |
| [多 Agent 协作方案对比](./docs/04-multi-agent-landscape.md) | Superpowers / DeerFlow / CubeSandbox / OpenHarness / ClawTeam 深度分析 |
| [Everything Claude Code](./docs/05-everything-claude-code.md) | 181k stars 的 Agent 工具集 |
| [Hermes Agent](./docs/06-hermes-agent.md) | 147k stars 自改进 Agent |
| [Hermes Agent Self-Evolution](./docs/07-hermes-agent-self-evolution.md) | DSPy + GEPA 进化优化 |
| [Temporal](./docs/08-temporal.md) | 20k stars 持久化执行平台 |
| [Anthropic Skills](./docs/09-anthropic-skills.md) | 133k stars Agent Skills 官方仓库 |
| [重新审视 Agent 间通信](./docs/10-重新审视Agent间通信.md) | 已调研项目的 Agent 通信模式分析与缺口 |
| [OpenHarness](./docs/11-openharness.md) | 12k stars 多 Agent 协调基础设施 + ohmo 个人助手 |
| [DeerFlow 2.0](./docs/12-deerflow-2.md) | DeerFlow 2.0 调研：Subagent 不是真正的多 Agent，持久化后端分析 |

## 参考项目

- [DeerFlow 2.0](https://github.com/bytedance/deer-flow) — 主控 Agent + Subagent + Sandbox + Memory
- [ClawTeam-OpenClaw](https://github.com/win4r/ClawTeam-OpenClaw) — 真正的多 Agent 协调 + Task Dependencies + Team 模板
- [OpenSpec](https://github.com/Fission-AI/OpenSpec) — 规范层框架
- [Superpowers](https://github.com/obra/superpowers) — Agent skills framework + TDD 方法论
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox) — KVM 沙箱，<60ms 冷启动
- [Temporal](https://github.com/temporalio/temporal) — 20k stars 持久化执行平台
- [Hermes Agent](https://github.com/NousResearch/hermes-agent) — 147k stars 自改进 AI Agent
- [OpenHarness](https://github.com/HKUDS/OpenHarness) — 多 Agent 协调基础设施 + ohmo
- [DeerFlow](https://github.com/bytedance/deer-flow) — 原始版本（1.x 分支）
