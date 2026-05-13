# 重新审视：已调研项目中的 Agent 间通信

## 修正之前的错误

之前说"仍然缺失 Agent↔Agent 协议"是**不准确的**。让我重新梳理：

---

## 已调研项目中的 Agent 通信模式

### 1. Hermes Agent — ACP (Agent Client Protocol)

**ACP 是真正的 Agent 通信协议**，但不面向多 Agent 协作：

- **定位**：Editor ↔ Agent 通信协议（VS Code / Zed / JetBrains）
- **实现**：`acp_adapter/` — 基于 JSON-RPC over stdio
- **会话管理**：`list/load/resume/fork` 会话操作
- **本质**：**单 Agent 远程控制**，不是多 Agent 对等通信

**ACP 暴露的能力**：
```
Initialize → 初始化连接
NewSession → 创建新会话
LoadSession / ResumeSession → 恢复会话
ForkSession → 分叉会话（类 subagent 概念）
SendMessage → 发送消息
```

**缺失**：没有 Agent ↔ Agent 的**对等消息传递**、**服务发现**、**分布式协作**。

### 2. Hermes Agent — delegate_task

**Subagent 托身，但非对等通信**：

```
父 Agent
  └── delegate_task() → 启动子 Agent（独立进程/会话）
         ├── 传递任务描述
         ├── 接收结果返回
         └── 无直接消息通道
```

- **通信模型**：过程调用（parent calls child, child returns result）
- **无持久消息队列**：调用结束即断开
- **无服务发现**：子 Agent 由父 Agent 直接命名指定

### 3. DeerFlow — Subagent System

**更结构化的 subagent 托身**：

```
Lead Agent
  ├── delegating to subagent: general_purpose
  ├── delegating to subagent: bash_agent
  └── waiting for results...
```

- **Subagent Registry**：注册已知 subagent 类型
- **Executor**：后台执行引擎
- **Token Collector**：子 Agent token 用量收集
- **本质**：**主从架构**，Lead Agent 统一调度

**缺失**：subagent 之间不能直接通信，必须通过 Lead Agent 中转。

### 4. Superpowers — dispatching-parallel-agents

**并行任务分派，无持久通信**：

```
Task 1 (Agent A) ─┐
Task 2 (Agent B) ─┼→ 并行执行 → 结果汇总
Task 3 (Agent C) ─┘
```

- **每个 Agent 独立**：独立上下文、独立工作
- **无消息通道**：Agent 之间不直接通信
- **只共享最终产物**：通过文件/git/人工汇总

---

## 为什么说"缺失"是对的

| 维度 | 现状 | 缺口 |
|------|------|------|
| **协议** | ACP（单 Agent 控制） | 没有多 Agent 对等协议 |
| **消息模型** | RPC / 过程调用 | 没有持久化消息队列 |
| **服务发现** | 手动指定名称 | 没有自动发现机制 |
| **状态共享** | 无 | 没有共享状态空间 |
| **拓扑** | 星型（父-子） | 没有网状 / 对等拓扑 |
| **生命周期** | 任务绑定 | 没有长期 Agent 身份 |

**核心缺口**：没有一个项目实现 **Agent Mesh** — Agent 可以自由地、持久地、点对点地相互通信。

---

## 对 Ai-Harness 的启示

### 已有的解决方案

经过调研，MVP 阶段已选定以下组件填补这些能力：

| 需要的能力 | MVP 组件 | 说明 |
|-----------|---------|------|
| **消息总线** | OpenHarness Mailbox / ClawTeam inbox | Agent 间消息传递 |
| **Agent Registry** | OpenHarness Swarm | team/agent 管理 |
| **对等通信** | ClawTeam inbox send/peek/broadcast | Agent↔Agent 直接通信 |
| **Task Dependencies** | ClawTeam | --blocked-by + auto-unblock |
| **持久化执行** | Temporal | Workflow 状态持久化 |

### 仍需关注的缺口

| 缺口 | 说明 |
|------|------|
| **事件驱动触发** | Git Hook → 自动触发任务 |
| **优先级队列** | 任务优先级控制 |
|| **规范层 + 执行层 闭环** | OpenSpec → 自动执行 |

### 参考借鉴

| 项目 | 借鉴点 |
|------|--------|
| **Temporal** | 持久化执行、状态机、Saga 补偿 |
| **Hermes ACP** | 会话管理协议（值得扩展为多 Agent） |
| **DeerFlow Subagent** | Executor + Registry 模式 |
| **NATS / Redis PubSub** | 消息总线参考实现 |
| **Distributed Actors** | OTP / Akka 风格的 Actor 模型 |

---

## 结论

经过完整调研，MVP 阶段已选定组件填补核心缺口。

**已填补的缺口**：

| 缺口 | MVP 组件 |
|------|---------|
| Agent ↔ Agent 对等通信 | ClawTeam inbox / OpenHarness Mailbox |
| 持久化消息总线 | ClawTeam inbox（文件）、OpenHarness Mailbox |
| 动态服务发现 | OpenHarness Swarm（team/agent registry） |
| Task Dependencies | ClawTeam（--blocked-by + auto-unblock） |
| 共享状态空间 | Temporal（Workflow 持久化） |

**仍需关注的缺口**：

| 缺口 | 说明 |
|------|------|
| 事件驱动触发 | Git Hook → 自动触发任务 |
| 优先级队列 | 任务优先级控制 |
| 规范层 + 执行层 闭环 | OpenSpec → 自动执行 |
