# Temporal 调研

## 项目信息

- **GitHub**: https://github.com/temporalio/temporal
- **Stars**: 20.2k
- **Forks**: 1.6k
- **Commits**: 9,008
- **Branches**: 328
- **Tags**: 496
- **定位**: Durable execution platform（持久化执行平台）
- **License**: MIT

---

## 核心定位

> Temporal is a durable execution platform that enables developers to build scalable applications without sacrificing productivity or reliability.

Temporal 是一个**持久化执行平台**，让开发者构建可扩展应用而不牺牲生产力或可靠性。

**历史**：起源于 Uber 的 Cadence fork，由 Cadence 创作者创立 Temporal Technologies。

---

## 核心概念

### Workflows（工作流）

```python
@workflow.defn
class MyWorkflow:
    @workflow.run
    async def run(self, name: str) -> str:
        result = await activity.execute(name)
        return result
```

- **持久化执行** — 即使服务器崩溃，workflow 也会从上次状态恢复
- **无限重试** — 自动处理间歇性故障和重试失败操作
- **状态管理** — 所有状态变化被持久化

### Activities（活动）

```python
@activity.defn
async def my_activity(param: str) -> str:
    # 执行外部操作：数据库调用、API 请求等
    return f"Result: {param}"
```

- **执行单元** — 执行具体业务逻辑
- **可重试** — 失败时自动重试
- **有超时控制** — 可设置 timeout 和 retry policy

### Workers（工作者）

```python
async def main():
    client = await Client.connect("localhost:7233")
    worker = Worker(
        client,
        task_queue="my-task-queue",
        workflows=[MyWorkflow],
        activities=[my_activity],
    )
    await worker.run()
```

- **执行 Workflows 和 Activities**
- **连接到 Temporal server**
- **处理任务队列**

---

## 架构

```
┌─────────────────────────────────────────────────────┐
│                  Temporal Server                    │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │  Workflow   │  │  History    │  │  Matching  │ │
│  │  Executor   │  │  Service    │  │  Service   │ │
│  └─────────────┘  └─────────────┘  └───────────┘ │
│                                                     │
│  ┌─────────────────────────────────────────────┐  │
│  │            Persistence (Cassandra/Postgres) │  │
│  └─────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Worker    │      │   Worker    │      │   Worker    │
│  (Go/Java/  │      │  (Python/   │      │  (Other     │
│   TypeScript)│      │  .NET)      │      │  SDKs)      │
└─────────────┘      └─────────────┘      └─────────────┘
```

### 核心服务

| Service | 功能 |
|---------|------|
| **History Service** | 存储 workflow 执行历史，管理状态 |
| **Matching Service** | 任务队列匹配，分发任务给 workers |
| **Worker Service** | 执行 workflow 和 activity |

### 持久化存储

支持：
- **Cassandra**（默认）
- **PostgreSQL**
- **MySQL**
- **SQLite**（开发用）

---

## 关键特性

### 1. 持久化执行（Durable Execution）

Workflow 的每次状态变化都被记录。即使：
- 服务器崩溃
- 网络中断
- 服务重启

Workflow 都能从**上次状态**恢复执行。

### 2. 自动重试

```python
@activity.defn
@activity.retry(maximum_attempts=5, initial_interval=1)
async def my_activity():
    ...
```

- **Exponential backoff** — 指数退避
- **Maximum attempts** — 最大重试次数
- **Dead letter queue** — 超过重试次数进入 DLQ

### 3. 信号和查询（Signals & Queries）

```python
# Signal（向 running workflow 发送消息）
await handle.signal_send(workflow_id, "update-name", "new_name")

# Query（查询 running workflow 状态）
result = await handle.query("get-status")
```

- **Signals** — 异步发送信号给 running workflow
- **Queries** — 同步查询 workflow 内部状态

### 4. 定时/调度（Timers）

```python
@workflow.run
async def run(self):
    await workflow.sleep(timedelta(hours=24))
    # 执行 delayed 逻辑
```

- **Durable timers** — 即使服务重启也能触发
- **用于延迟执行、周期性任务**

### 5. 分布式事务支持

通过 **Saga 模式**支持分布式事务：

```python
# 补偿事务（Compensation）
async def execute_order(self):
    step1 = await self.reserve_inventory()
    try:
        step2 = await self.process_payment()
    except:
        await step1.compensate()  # 补偿 step1
        raise
```

---

## SDK 支持

| 语言 | 状态 |
|------|------|
| **Go** | GA（正式发布） |
| **Java** | GA |
| **Python** | GA |
| **TypeScript/JS** | GA |
| **.NET** | Beta |
| **PHP** | Beta |
| **Ruby** | Beta |

---

## 与 Ai-Harness 的关系

### Temporal 解决了什么

| 能力 | Temporal | Ai-Harness 目标 |
|------|----------|----------------|
| **持久化执行** | ✅ | ✅ |
| **状态机** | ✅ | ✅ |
| **自动重试** | ✅ | ✅ |
| **任务调度** | ✅ (timers) | ✅ |
| **工作流编排** | ✅ | ✅ |
| **Workflow-as-code** | ✅ | ✅ |

### Temporal 没有解决的

| 能力 | Temporal | 缺口 |
|------|----------|------|
| **Agent ↔ Agent 协议** | ❌ | ❌ |
| **多 Coding Agent 支持** | ❌ | ❌ |
| **Sandbox 执行** | ❌ | ❌ |
| **规范层（Spec-Driven）** | ❌ | ❌ |
| **自然语言任务定义** | ❌ | ❌ |

---

## 关键启示

### Temporal 的核心价值

1. **持久化 > 可靠** — 所有状态变化可恢复
2. **幂等性** — Activity 天然支持幂等设计
3. **透明重试** — 开发者只需定义业务逻辑，重试由框架处理
4. **事件溯源** — History Service 记录完整执行历史

### 对 Ai-Harness 的借鉴

| Temporal 特性 | Ai-Harness 借鉴 |
|--------------|----------------|
| **持久化执行** | Agent workflow 状态持久化 |
| **状态机 + DAG** | Workflow 编排引擎 |
| **Timers** | Cron 调度增强 |
| **Signals** | Agent 间消息协议 |
| **Saga pattern** | 分布式 workflow 补偿 |
| **History log** | 审计和回放 |

### 架构对比

```
Temporal 架构：                    Ai-Harness 目标：
┌──────────────┐                 ┌──────────────────┐
│   Workflow   │                 │  Agent Workflow  │
│   Executor   │                 │  (持久化状态机)   │
└──────┬───────┘                 └────────┬─────────┘
       │                                  │
       ▼                                  ▼
┌──────────────┐                 ┌──────────────────┐
│   History    │                 │   Execution      │
│   Service   │                 │   Log / Audit    │
└──────┬───────┘                 └────────┬─────────┘
       │                                  │
       ▼                                  ▼
┌──────────────┐                 ┌──────────────────┐
│  Matching    │                 │   Agent Mesh     │
│  Service    │                 │   (消息总线)      │
└──────────────┘                 └──────────────────┘
```

**关键差异**：Temporal 是**进程内**的 workflow 执行，A i-Harness 想要的是**跨 Agent** 的协作编排。
