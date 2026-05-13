# 多 Agent 协作方案对比分析

## 目标

分析 Superpowers、DeerFlow、CubeSandbox 三个开源项目，确定打造多 Agent 协同系统还缺失的核心能力。

---

## 项目概述

### Superpowers
- **GitHub**: https://github.com/obra/superpowers
- **Stars**: 188k
- **定位**: Agentic skills framework + 软件开发方法论
- **支持 Agent**: Claude Code, Codex CLI, Codex App, Factory Droid, Gemini CLI, OpenCode, Cursor, GitHub Copilot CLI

### DeerFlow
- **GitHub**: https://github.com/bytedance/deer-flow
- **Stars**: 67.2k
- **定位**: Super Agent Harness（基于 LangGraph + LangChain）
- **特点**: Sub-agents, Sandbox, Long-Term Memory, Skills, IM Channels, MCP Server

### CubeSandbox
- **GitHub**: https://github.com/TencentCloud/CubeSandbox
- **Stars**: 5.5k
- **定位**: Instant, Concurrent, Secure & Lightweight Sandbox Service
- **特点**: <60ms 冷启动, <5MB 内存开销, KVM 隔离, E2B SDK 兼容

### Everything Claude Code (ECC)
- **GitHub**: https://github.com/affaan-m/everything-claude-code
- **Stars**: 181k
- **定位**: Agent harness performance optimization system
- **特点**: 60 agents, 228 skills, Cross-harness 架构, Instincts 学习系统, AgentShield 安全审计

### Hermes Agent
- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Stars**: 147k
- **定位**: The self-improving AI agent
- **特点**: 自改进学习循环, FTS5 搜索, Cron 调度, 多平台网关(20+), 任意模型(200+), 7 种运行环境

### Hermes Agent Self-Evolution
- **GitHub**: https://github.com/NousResearch/hermes-agent-self-evolution
- **Stars**: 3.1k
- **定位**: Evolutionary self-improvement for Hermes Agent
- **特点**: DSPy + GEPA 进化优化, 5 阶段递进, No GPU, ~$2-10 per run, ICLR 2026 Oral

### Temporal
- **GitHub**: https://github.com/temporalio/temporal
- **Stars**: 20.2k
- **定位**: Durable execution platform（持久化执行平台）
- **特点**: Workflow 状态持久化, 自动重试, 历史记录, 多语言 SDK (Go/Java/Python/TS/.NET)

### Anthropic Skills
- **GitHub**: https://github.com/anthropics/skills
- **Stars**: 133k
- **定位**: Agent Skills 官方参考实现
- **特点**: agentskills.io 标准, SKILL.md 格式规范, 文档/创意/开发/企业技能集

### ClawTeam-OpenClaw
- **GitHub**: https://github.com/win4r/ClawTeam-OpenClaw
- **Stars**: 1.4k
- **定位**: Multi-agent swarm coordination for CLI coding agents
- **特点**: Task Dependencies + auto-unblock, Team Templates (TOML), Git Worktree 隔离, Cost Dashboard, inbox send/peek/broadcast, 支持 OpenClaw/Claude Code/Codex/Hermes/nanobot/Cursor

---

## 能力矩阵

| 能力 | Superpowers | DeerFlow | CubeSandbox | ECC | Hermes | Self-Evolution | Temporal | OpenHarness | ClawTeam | 缺口 |
|------|:-----------:|:--------:|:-----------:|:---:|:------:|:--------------:|:--------:|:-----------:|:--------:|:----:|
| 任务调度（Cron/触发器） | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ✅ (CronCreate) | ❌ | **❌** |
| Agent 间通信协议 | ❌ | ✅ (subagent 中转) | ❌ | ❌ | ✅ (ACP, 单 Agent 控制) | ❌ | ❌ | ✅ (Mailbox) | ✅ (inbox) | **❌** |
| 流水线编排引擎 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | **❌** |
| Subagent 管理 | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ (Swarm) | ✅ (spawn) | ✅ |
| 持久化记忆 | ❌ | ✅ | ❌ | ✅ (instincts) | ✅ (FTS5) | ❌ | ❌ | ✅ (memory/) | ❌ | ✅ |
| 沙箱执行环境 | ❌ | ✅ | ✅ | ❌ | ✅ (7种) | ❌ | ❌ | ✅ (sandbox/) | ❌ | ✅ |
| 多 Coding Agent 支持 | ✅ (8种) | ✅ | ❌ | ✅ (10+) | ✅ | ❌ | ❌ | ✅ | ✅ (6种) | ✅ |
| 规范层（Spec-Driven） | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | **❌** |
| TDD / 代码质量 | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| 安全审计 | ❌ | ❌ | ❌ | ✅ (AgentShield) | ❌ | ❌ | ❌ | ✅ (permissions/) | ❌ | ✅ |
| 自改进学习 | ❌ | ❌ | ❌ | ✅ (instincts) | ✅ | ✅ (GEPA) | ❌ | ❌ | ❌ | ✅ |
| 多平台网关 | ❌ | ✅ (IM) | ❌ | ❌ | ✅ (20+) | ❌ | ❌ | ✅ (Feishu等) | ❌ | ✅ |
| 自动 Skill 进化 | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | **❌** |
| Workflow 持久化执行 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |
| 状态机 / DAG | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | ✅ |
| Task Dependencies | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Team Templates | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ (TOML) | ✅ |
| Cost Dashboard | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

---

## 各项目核心能力详解

### Superpowers

**核心理念**：Subagent 驱动开发 + 强制技能检查

**核心工作流**：
1. `brainstorming` — 写代码前先澄清需求，生成设计文档
2. `writing-plans` — 分解成 2-5 分钟的小任务
3. `subagent-driven-development` — 每个任务分派给独立 subagent，两阶段 review
4. `test-driven-development` — RED-GREEN-REFACTOR
5. `finishing-a-development-branch` — 完成后验证测试，提交/PR

**强制技能检查**：agent 执行任何 task 前自动检查相关 skills，不是建议，是强制。

**贡献者**：Jesse Vincent (obra), 11 位核心贡献者

---

### DeerFlow

**核心理念**：从 Deep Research 进化为 Super Agent Harness

**核心架构**（基于 LangGraph + LangChain）：

```
Lead Agent
    ├── Sub-Agent 1 (并行)
    ├── Sub-Agent 2 (并行)
    └── Sub-Agent N (并行)
            │
            ▼
    ┌───────────────┐
    │   Sandbox     │
    │ Local/Docker/ │
    │ Kubernetes    │
    └───────────────┘
            │
            ▼
    ┌───────────────┐
    │    Memory     │
    │ (Long-Term)   │
    └───────────────┘
```

**核心能力**：

| 能力 | 说明 |
|------|------|
| **Sub-Agents** | Lead agent 可动态 spawn 子 agent，每个有独立 context |
| **Sandbox** | Local / Docker / K8s 三种隔离执行模式 |
| **Long-Term Memory** | 跨会话持久记忆，用户 profile、偏好、知识积累 |
| **Skills** | Markdown skill 扩展系统，按需加载 |
| **IM Channels** | Telegram / Slack / Feishu / WeChat / WeCom / DingTalk |
| **MCP Server** | 支持 MCP 协议扩展 |
| **Tracing** | LangSmith + Langfuse 可观测性 |

**部署规模参考**：

| 场景 | 最低配置 | 推荐配置 |
|------|----------|----------|
| 本地评估 | 4 vCPU, 8GB RAM | 8 vCPU, 16GB RAM |
| Docker 开发 | 4 vCPU, 8GB RAM, 25GB SSD | 8 vCPU, 16GB RAM |
| 长运行服务器 | 8 vCPU, 16GB RAM, 40GB SSD | 16 vCPU, 32GB RAM |

**IM Channels 配置**：

```yaml
channels:
  langgraph_url: http://localhost:8001/api
  gateway_url: http://localhost:8001
  
  feishu:
    enabled: true
    app_id: $FEISHU_APP_ID
    app_secret: $FEISHU_APP_SECRET
  
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN
  
  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN
    app_token: $SLACK_APP_TOKEN
```

---

### CubeSandbox

**核心理念**：硬件级隔离 + 毫秒级冷启动

**架构组件**：

| 组件 | 职责 |
|------|------|
| **CubeAPI** | 高并发 REST API Gateway（Rust），兼容 E2B SDK |
| **CubeMaster** | 集群编排器，接收 API 请求分发到 Cubelets |
| **CubeProxy** | 反向代理，E2B 协议兼容，路由到沙箱实例 |
| **Cubelet** | 计算节点本地调度，管理所有沙箱实例生命周期 |
| **CubeVS** | eBPF 虚拟交换机，内核级网络隔离和安全策略 |
| **CubeHypervisor** | KVM MicroVM 虚拟化层 |
| **CubeShim** | containerd Shim v2 API 集成 |

**性能指标**：

| 指标 | Docker Container | Traditional VM | CubeSandbox |
|------|------------------|-----------------|-------------|
| 隔离级别 | Low (Shared Kernel) | High (Dedicated) | Extreme (KVM + eBPF) |
| 冷启动 | 200ms+ | Seconds | **<60ms** |
| 内存开销 | Low | High | **<5MB** |
| 部署密度 | High | Low | **Extreme (千级/节点)** |

**E2B SDK 兼容**：零成本迁移，只需改 URL 环境变量：
```python
import os
from e2b_code_interpreter import Sandbox

with Sandbox.create(template=os.environ["CUBE_TEMPLATE_ID"]) as sandbox:
    result = sandbox.run_code("print('Hello from Cube Sandbox!')")
```

---

## 五大核心缺口

> ⚠️ **注意**：随着 MVP 组件调研完成，部分缺口已被现有组件填补，以下是更新后的评估。

### 缺口 1️⃣：任务调度中心（Scheduler）

```
现状：定时触发、事件驱动、任务依赖图
需要：Cron + 事件触发 + 优先级队列 + DAG 调度
```

**已被填补**：
- **Task Dependencies** — ClawTeam 支持 `--blocked-by` + auto-unblock
- **Cron 调度** — Temporal 支持定时触发

**仍缺失**：
- 事件驱动触发（代码提交 → 自动触发任务）
- 优先级队列
- 完整 DAG 可视化

**影响**：
- 无法实现 Git Hook 触发评审
- 任务优先级无法控制

---

### 缺口 2️⃣：Agent 间通信协议（Agent Mesh）

```
现状：DeerFlow 只有 IM 通道（人→Agent），没有 Agent↔Agent 协议
需要：消息总线 + 任务分发 + 状态同步 + Agent 发现机制
```

**✅ 已被 ClawTeam 填补**：
- `inbox send/peek/broadcast` — 真正的 Agent↔Agent 通信
- `clawteam spawn` — 任务分发
- `task wait` — 状态同步

**结论**：缺口已填补，ClawTeam 是真正的多 Agent 协调系统。

---

### 缺口 3️⃣：流水线编排引擎（Workflow Orchestrator）

```
现状：状态机 + 条件分支 + 循环 + 人工审批 gate
需要：Long-running workflow + compensation/回滚 + 人机协作
```

**✅ 已被 Temporal 填补**：
- Workflow 状态持久化
- Activity 自动重试 + 超时
- 补偿机制（Saga 模式）
- 人机协作（Signal/Query）

**结论**：缺口已填补，Temporal 是生产级的流水线编排引擎。

---

### 缺口 4️⃣：规范层 + 执行层 合一

```
现状：OpenSpec 只有规范层，Superpowers 只有方法论，都无法自动执行
需要：Spec-Driven → 自动执行 → 验证 → 反馈闭环
```

**仍缺失**：
- OpenSpec 的 design.md 需要人工执行
- Superpowers 的方法论需要人工驱动 subagent
- 没有组件能把"规范文档"自动转化为"可执行任务"

**影响**：
- 规范和执行脱节
- 无法验证执行是否符合规范
- 无法从执行结果反馈改进规范

---

### 缺口 5️⃣：统一多 Agent 管理层

```
现状：Superpowers 支持 8 种 Agent 但没有共享状态/通信
需要：统一接口抽象多 Agent（Claude Code/Cursor/Copilot 等）
```

**✅ 已被 ClawTeam 部分填补**：
- `clawteam spawn` 支持多种 Agent（OpenClaw/Claude Code/Codex/Hermes/nanobot/Cursor）
- Git Worktree 隔离

**仍缺失**：
- 统一的任务队列（ClawTeam 用文件 + tmux，规模有限）
- 跨机器协调（NFS/P2P 还在规划中）

---

### 缺口评估总结

| 缺口 | 状态 | 填补组件 |
|------|------|---------|
| 任务调度中心 | ⚠️ 部分填补 | Temporal（Cron）、ClawTeam（Task Dependencies） |
| Agent 间通信协议 | ✅ 已填补 | ClawTeam（inbox） |
| 流水线编排引擎 | ✅ 已填补 | Temporal |
| 规范层 + 执行层 | ❌ 仍缺失 | 无 |
| 统一多 Agent 管理层 | ⚠️ 部分填补 | ClawTeam |

---

## 建议架构

```
┌──────────────────────────────────────────────────────────────┐
│                         Ai-Harness                            │
│                                                              │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  DeerFlow 2.0 (主控 Agent)                │ │
│  │  Lead Agent — 需求理解、任务拆解                          │ │
│  │  task_tool — 托身 Subagent 执行                          │ │
│  │  Sandbox — 隔离执行环境                                  │ │
│  │  Memory — 长期记忆                                       │ │
│  │  Checkpointer — LangGraph 状态持久化                     │ │
│  └──────────────────────────┬─────────────────────────────┘ │
│                               │                               │
│  ┌──────────────────────────▼─────────────────────────────┐ │
│  │              ClawTeam-OpenClaw (多 Agent 协调)           │ │
│  │  clawteam spawn — 托身独立 Worker Agents                 │ │
│  │  Task Dependencies — --blocked-by + auto-unblock         │ │
│  │  Team Templates — TOML 模板                              │ │
│  │  inbox send/peek/broadcast — Agent 间消息通信             │ │
│  │  Cost Dashboard — 实时 token/cost 追踪                   │ │
│  └──────────────────────────┬─────────────────────────────┘ │
│                               │                               │
│         ┌────────────────────┼────────────────────┐         │
│         │                    │                    │         │
│  ┌──────▼──────┐    ┌────────▼───────┐   ┌───────▼──────┐   │
│  │  OpenSpec   │    │  Superpowers   │   │ CubeSandbox  │   │
│  │  (规范层)   │    │  (TDD 方法论)   │   │  (沙箱执行)   │   │
│  └─────────────┘    └────────────────┘   └──────────────┘   │
│                               │                               │
│         ┌────────────────────┼────────────────────┐         │
│         │                    │                    │         │
│  ┌──────▼──────┐    ┌────────▼───────┐   ┌───────▼──────┐   │
│  │  Temporal   │    │ GitHub Actions │   │    Human     │   │
│  │ (持久化)     │    │   (CI/CD)     │   │  (验收签字)   │   │
│  └─────────────┘    └────────────────┘   └──────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

### 核心模块说明

> ✅ = 已有 MVP 组件，⚠️ = 部分填补，❌ = 仍缺失

| 模块 | 职责 | 技术选型 | 状态 |
|------|------|---------|------|
| **主控 Agent** | 需求理解、任务拆解、Subagent 托身 | DeerFlow 2.0 | ✅ |
| **多 Agent 协调** | Task Dependencies、inbox 通信、Team 模板 | ClawTeam-OpenClaw | ✅ |
| **规范层** | Spec-Driven 任务定义 | OpenSpec | ✅ |
| **开发方法论** | TDD、brainstorming、parallel dispatch | Superpowers | ✅ |
| **沙箱执行** | 代码隔离执行，<60ms 冷启动 | CubeSandbox | ✅ |
| **Workflow 持久化** | 状态持久化、自动重试、失败恢复 | Temporal | ✅ |
| **CI/CD** | 自动化测试、构建、部署 | GitHub Actions | ✅ |
| **Scheduler** | Cron 定时触发 | Temporal | ⚠️ 需 Temporal Cloud |
| **事件驱动** | Git Hook 触发、代码提交触发 | ❌ 仍缺失 | ❌ |
| **优先级队列** | 任务优先级控制 | ❌ 仍缺失 | ❌ |

---

## 技术选型（已确定）

经过完整调研，MVP 组件选型已确定：

| 组件 | 选型 | 调研文档 |
|------|------|---------|
| 主控 Agent | **DeerFlow 2.0** | [12-deerflow-2.md](./12-deerflow-2.md) |
| 多 Agent 协调 | **ClawTeam-OpenClaw** | （见 README） |
| 规范层 | **OpenSpec** | [01-openspec-overview.md](./01-openspec-overview.md) |
| 开发方法论 | **Superpowers** | [04-multi-agent-landscape.md](./04-multi-agent-landscape.md) |
| 沙箱执行 | **CubeSandbox** | [04-multi-agent-landscape.md](./04-multi-agent-landscape.md) |
| Workflow 持久化 | **Temporal** | [08-temporal.md](./08-temporal.md) |
| CI/CD | **GitHub Actions** | — |

### 待补充的缺口

以下能力 MVP 阶段仍未有成熟开源组件，需后续评估：

1. **事件驱动触发** — Git Hook → 自动触发任务（如提交代码 → 自动评审）
2. **优先级队列** — 任务优先级控制
3. **完整 DAG 可视化** — 任务依赖关系可视化
4. **规范层 + 执行层 闭环** — OpenSpec design.md → 自动任务执行
