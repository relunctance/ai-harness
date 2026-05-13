# DeerFlow 2.0 调研

## 项目信息

- **GitHub**: https://github.com/bytedance/deer-flow
- **Stars**: 67.2k
- **定位**: Super Agent Harness（从深度研究框架演变为多 Agent 协作平台）
- **License**: MIT

---

## 核心架构

```
DeerFlow 2.0
│
├── Lead Agent（主 Agent）
│   └── 通过 task_tool 托身 Subagent
│
├── Subagents（子 Agent）
│   ├── general-purpose（通用任务）
│   ├── bash（命令执行）
│   └── custom（自定义）
│
├── Sandbox（隔离执行环境）
│   └── 支持本地 Bash / AIO Sandbox / Docker
│
├── Memory（长期记忆）
│   └── 基于文件（memory.json）的 RAG
│
└── LangGraph Checkpointer（持久化）
        ├── memory（内存，重启丢失）
        ├── sqlite（本地文件）
        └── postgres（PostgreSQL）
```

---

## 多 Agent 分析

### DeerFlow 的 Subagent 不是独立 Agent

DeerFlow 的"多 Agent"实际上是**主 Agent + 从属 Subagent**模式：

```
Lead Agent
    │
    ├── task_tool("research X") → Subagent-A（从属执行单元）
    │       └── 结果通过 tool return 传递给 Lead
    │
    └── task_tool("implement Y") → Subagent-B（从属执行单元）
            └── 结果通过 tool return 传递给 Lead
```

**关键限制**：
- Subagent **没有独立身份**，只是主 Agent 的工具
- Subagent 之间**不直接通信**，只能通过主 Agent 中转
- 所有任务由 Lead Agent 统一调度
- **没有** inbox、Task Dependencies、Team 模板

### DeerFlow 不是真正的多 Agent 系统

| 能力 | DeerFlow 2.0 | ClawTeam-OpenClaw |
|------|-------------|-------------------|
| 独立 Agent | ❌ 只有主 + Subagent | ✅ 任意数量独立 Agent |
| Agent 间通信 | ❌ 无 | ✅ inbox send/peek/broadcast |
| Task Dependencies | ❌ 无 | ✅ --blocked-by + auto-unblock |
| Git Worktree 隔离 | ❌ 无 | ✅ 每个 Agent 强制隔离 |
| Team 模板 | ❌ 无 | ✅ TOML 模板 |
| 分布式 | ❌ 单节点 | ✅ 跨机器（NFS/P2P） |

---

## 持久化后端（确认结果）

| 数据库 | 支持状态 | 说明 |
|--------|---------|------|
| **PostgreSQL** | ✅ 已支持 | checkpointer + database 统一配置 |
| **SQLite** | ✅ 已支持 | 单节点部署 |
| **MySQL** | ❌ 不支持 | 代码里没有任何 MySQL 支持 |
| **Redis** | ⚠️ 计划中 | Stream Bridge 的 Redis 模式（Phase 2，未实现） |
| **MongoDB** | ❌ 不支持 | 代码里没有任何 MongoDB 支持 |

### 实际配置只有 3 种

```yaml
# config.yaml
database:
  backend: memory    # 重启丢失
  # backend: sqlite  # 本地文件
  # backend: postgres # PostgreSQL
```

---

## Workflow 持久化

DeerFlow 2.0 使用 **LangGraph Checkpointer**：

```python
# langgraph.json
{
  "checkpointer": {
    "type": "memory",  // 或 sqlite, postgres
    "connection_string": ".deer-flow/checkpoints.db"
  }
}
```

**与 Temporal 的对比**：

| 维度 | DeerFlow 2.0 | Temporal |
|------|-------------|----------|
| 持久化范围 | LangGraph 会话状态 | 整个 Workflow + Activity |
| 跨服务/跨进程 | ❌ 单节点 | ✅ 多节点 + 多进程 |
| 故障恢复 | ✅ 自动恢复会话状态 | ✅ 完整状态 + 历史记录 |
| 重试策略 | 有限（LangGraph retry） | ✅ Activity 级 + 超时 + 补偿 |
| 与 Agent 集成 | ✅ 原生 | ❌ 需要适配 |
| 部署复杂度 | 低（Docker 一键） | 高（独立 Server + DB） |

---

## 核心能力

| 能力 | 说明 |
|------|------|
| **Subagent 执行** | 通过 task_tool 托身子任务，支持 general-purpose 和 bash 类型 |
| **Sandbox 隔离** | 支持本地 Bash、AIO Sandbox、Docker |
| **Memory** | 基于文件的长期记忆，支持 RAG 注入 |
| **Skills** | 兼容 anthropics/skills 格式的技能扩展 |
| **IM Channels** | 支持飞书、Slack、Discord 等消息通道 |
| **MCP Server** | 支持 Model Context Protocol |
| **Checkpoint** | LangGraph 状态持久化（SQLite/PostgreSQL） |

---

## DeerFlow 2.0 vs OpenHarness

| 能力 | DeerFlow 2.0 | OpenHarness |
|------|-------------|-------------|
| 主控 Agent | ✅ LangGraph Lead Agent | ✅ LangGraph + ohmo + Swarm |
| Subagent/Worker | ✅ task_tool（无独立身份） | ✅ swarm spawn（有独立身份） |
| 多 Agent 管理 | ❌ 无 | ✅ team/agent/task/mailbox |
| Cron 调度 | ❌ 无 | ✅ cron_create/list/delete |
| ohmo 助手 | ❌ 无 | ✅ 飞书/Slack/Discord/Telegram |
| Sandbox | ✅ 本地/AIO/Docker | ✅ sandbox/ |
| Memory | ✅ 文件 | ✅ memory/ |
| IM Channels | ✅ 飞书等 | ✅ ohmo |
| Task Dependencies | ❌ 无 | ❌ 无（用 ClawTeam） |
| Team Templates | ❌ 无 | ❌ 无（用 ClawTeam） |
| Cost Dashboard | ❌ 无 | ❌ 无（用 ClawTeam） |

---

## 结论

### DeerFlow 2.0 不适合做主控 Agent

DeerFlow 2.0 是一个**单主 Agent + 从属 Subagent**的系统，存在以下限制：

| 限制 | 说明 |
|------|------|
| **Subagent 无独立身份** | Subagent 只是工具调用，没有自己的 inbox/mailbox，无法与其他 Agent 直接通信 |
| **没有 Swarm 系统** | 无法管理多 Agent 团队（team/agent/task） |
| **没有 Cron 调度** | 无法定时触发任务 |
| **没有 ohmo** | 无法接入飞书等 IM 助手 |
| **持久化后端有限** | 只支持 memory/sqlite/postgres，不支持跨服务协调 |

### DeerFlow 2.0 的合适定位

| 场景 | 角色 | 说明 |
|------|------|------|
| **Sandbox 执行引擎** | 可选组件 | OpenHarness 内置 Sandbox（主要），DeerFlow 2.0 作为备选 |
| **快速原型验证** | 独立使用 | 单一 Agent 任务，适合快速验证 |

### DeerFlow 2.0 vs Temporal vs ClawTeam

| 能力 | DeerFlow 2.0 | Temporal | ClawTeam |
|------|-------------|----------|----------|
| 持久化 | LangGraph Checkpoint | ✅ 完整 Workflow | ❌ |
| 多 Agent 协调 | ❌ | ❌ | ✅ inbox/Task Dep |
| Cron 调度 | ❌ | ✅ Schedule | ❌ |
| 主控 Agent | ✅ | ❌ | ❌ |

**最终选型**：
- **OpenHarness** — 主控 Agent（Swarm + Cron + ohmo）
- **ClawTeam** — 多 Agent 协调（Task Dependencies + Team Templates + Cost Dashboard）
- **Temporal** — Workflow 持久化
- **DeerFlow 2.0** — 可选 Sandbox 执行引擎
