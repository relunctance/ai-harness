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

---

## 能力矩阵

| 能力 | Superpowers | DeerFlow | CubeSandbox | ECC | Hermes | Self-Evolution | Temporal | OpenHarness | 缺口 |
|------|:-----------:|:--------:|:-----------:|:---:|:------:|:--------------:|:--------:|:-----------:|:----:|
| 任务调度（Cron/触发器） | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ | ✅ (CronCreate) | **❌** |
| Agent 间通信协议 | ❌ | ✅ (subagent 中转) | ❌ | ❌ | ✅ (ACP, 单 Agent 控制) | ❌ | ❌ | ✅ (Mailbox) | **❌** |
| 流水线编排引擎 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | **❌** |
| Subagent 管理 | ✅ | ✅ | ❌ | ✅ | ✅ | ❌ | ❌ | ✅ (Swarm) | ✅ |
| 持久化记忆 | ❌ | ✅ | ❌ | ✅ (instincts) | ✅ (FTS5) | ❌ | ❌ | ✅ (memory/) | ✅ |
| 沙箱执行环境 | ❌ | ✅ | ✅ | ❌ | ✅ (7种) | ❌ | ❌ | ✅ (sandbox/) | ✅ |
| 多 Coding Agent 支持 | ✅ (8种) | ✅ | ❌ | ✅ (10+) | ✅ | ❌ | ❌ | ✅ | ✅ |
| 规范层（Spec-Driven） | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | **❌** |
| TDD / 代码质量 | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| 安全审计 | ❌ | ❌ | ❌ | ✅ (AgentShield) | ❌ | ❌ | ❌ | ✅ (permissions/) | ✅ |
| 自改进学习 | ❌ | ❌ | ❌ | ✅ (instincts) | ✅ | ✅ (GEPA) | ❌ | ❌ | ✅ |
| 多平台网关 | ❌ | ✅ (IM) | ❌ | ❌ | ✅ (20+) | ❌ | ❌ | ✅ (Feishu等) | ✅ |
| 自动 Skill 进化 | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ❌ | **❌** |
| Workflow 持久化执行 | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| 状态机 / DAG | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |

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

### 缺口 1️⃣：任务调度中心（Scheduler）

```
现状：没有项目支持定时触发、事件驱动、任务依赖图
需要：Cron + 事件触发 + 优先级队列 + DAG 调度
```

**为什么缺失**：Superpowers/DeerFlow/CubeSandbox 都是"人驱动"或"请求驱动"，没有后台任务调度能力。

**影响**：
- 无法自动化周期性任务（如每日构建、巡检）
- 无法实现事件驱动（代码提交触发评审）
- 无法管理任务依赖（DAG 执行顺序）

---

### 缺口 2️⃣：Agent 间通信协议（Agent Mesh）

```
现状：DeerFlow 只有 IM 通道（人→Agent），没有 Agent↔Agent 协议
需要：消息总线 + 任务分发 + 状态同步 + Agent 发现机制
```

**为什么缺失**：现有项目都是"人调用 Agent"，不是"Agent 调用 Agent"。

**影响**：
- Agent 之间无法直接协作
- 无法实现任务接力（A agent 完成 → 通知 B agent 继续）
- 无法共享上下文状态

---

### 缺口 3️⃣：流水线编排引擎（Workflow Orchestrator）

```
现状：没有项目有状态机 + 条件分支 + 循环 + 人工审批 gate
需要：Long-running workflow + compensation/回滚 + 人机协作
```

**为什么缺失**：DeerFlow 的 subagent 是"并行执行 + 结果汇总"，没有复杂流程控制。

**影响**：
- 无法实现条件分支（if/else）
- 无法实现循环（retry/until）
- 无法人工审批后继续（human-in-the-loop）
- 无法补偿事务（compensation）

---

### 缺口 4️⃣：规范层 + 执行层 合一

```
现状：OpenSpec 只有规范层，Superpowers 只有方法论，都无法自动执行
需要：Spec-Driven → 自动执行 → 验证 → 反馈闭环
```

**为什么缺失**：OpenSpec 产生的 design.md 是静态文档，需要人工执行。Superpowers 是方法论，没有规范输入。

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

**为什么缺失**：Superpowers 是"给每个 agent 装技能"，不是"多 agent 协作"。

**影响**：
- 每个 agent 独立工作，无法协作
- 无法跨 agent 共享任务状态
- 无法统一调度多 agent

---

## 建议架构

```
┌──────────────────────────────────────────────────────────────┐
│                         Ai-Harness                            │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Scheduler   │  │  Agent Mesh  │  │  Workflow    │      │
│  │  (定时/触发)  │  │  (消息总线)   │  │ Orchestrator │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                  │                  │               │
│         └──────────────────┼──────────────────┘               │
│                            │                                  │
│  ┌─────────────────────────┼─────────────────────────────┐   │
│  │                  Agent Runtime                           │   │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  │   │
│  │  │ Claude  │  │ Cursor  │  │Copilot  │  │  ...    │  │   │
│  │  │  Code   │  │         │  │         │  │         │  │   │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                  │
│         ┌──────────────────┼──────────────────┐                │
│         │                  │                  │                 │
│    ┌────▼────┐    ┌──────▼──────┐   ┌──────▼──────┐       │
│    │ OpenSpec│    │   Memory    │   │   Sandbox   │       │
│    │ (规范层) │    │  (持久化)    │   │Deer/Cube    │       │
│    └──────────┘    └─────────────┘   └─────────────┘       │
└──────────────────────────────────────────────────────────────┘
```

### 核心模块说明

| 模块 | 职责 | 技术选型（待定） |
|------|------|-----------------|
| **Scheduler** | Cron 调度、事件触发、任务队列、DAG | Bull / Celery / 自研 |
| **Agent Mesh** | Agent 间消息总线、任务分发、状态同步 | Redis Pub/Sub / NATS / 自研 |
| **Workflow Orchestrator** | 状态机、条件分支、循环、人工审批 | Temporal / 自研 |
| **Agent Runtime** | 统一接口抽象多 Agent | OpenSpec Agent Protocol |
| **Sandbox** | 代码隔离执行环境 | DeerFlow Sandbox / CubeSandbox |
| **Memory** | 跨会话持久记忆 | Redis / PostgreSQL |
| **OpenSpec Engine** | 规范层执行 | 继承 OpenSpec |

---

## 技术选型（待调研）

以下组件需要调研确定：

1. **消息队列**：Redis Pub/Sub vs NATS vs Kafka
2. **任务队列**：Bull (Redis) vs Celery vs 自研
3. **调度器**：node-cron vs Quartz vs 自研
4. **Workflow 引擎**：Temporal vs Prefect vs 自研
5. **Agent 运行时**：Docker 容器化 vs 直接调用 CLI
6. **状态存储**：Redis vs PostgreSQL vs 文件系统
7. **API 网关**：Fastify vs Express vs 自研
8. **部署方式**：Docker Compose vs Kubernetes
9. **Agent 协议**：WebSocket vs gRPC vs HTTP Long-poll
