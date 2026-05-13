# Ai-Harness 愿景

## 项目定位

**多 Agent 自动化协作基础设施** — 在 OpenSpec 规范层基础上，补全任务调度、Agent 间通信、自动化流水线，构建真正的多 Agent 协作系统。

## 解决的问题

OpenSpec 解决了"AI 编程需求模糊"的问题，但缺少：

| 缺失能力 | 描述 | 后果 |
|----------|------|------|
| 任务调度 | 无法定时自动触发 | 人工干预多，无法自动化 |
| Agent 通信 | Agent 之间无法消息传递 | 协作靠人工桥接 |
| 流水线 | 没有自动执行链 | 无法端到端自动化 |

**Ai-Harness** 在 OpenSpec 基础上补全这三块。

## 目标能力

### 1. OpenSpec 规范引擎（继承）
- 完整的 propose → specs → design → tasks → verify → archive 工作流
- Delta spec 管理
- 跨仓库规范同步

### 2. 任务调度中心
- Cron 驱动的定时任务
- 任务依赖图
- 优先级队列
- 失败重试机制

### 3. Agent 网关
- 支持 Claude Code、Cursor、Windsurf、Copilot 等
- Agent 间消息队列通信
- 任务分发与状态同步

### 4. Workspace 协调
- 跨仓库/跨服务规划
- 多仓库联动变更
- 规范冲突检测与合并

### 5. 一键部署
- Docker Compose / Kubernetes 部署
- 基础设施即代码

## 不做什么

- ❌ 不做自己的 AI 模型
- ❌ 不做 IDE 或编辑器
- ❌ 不重复造 OpenSpec 已经做好的轮子

## 架构方向（待讨论）

```
┌──────────────────────────────────────────────────────────┐
│                      Ai-Harness                          │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  Scheduler    │  │  Agent Mesh   │  │  Spec Engine │ │
│  │  (Cron+Queue) │  │  (Message Bus)│  │  (OpenSpec)  │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘ │
│         │                  │                  │         │
│         └──────────────────┼──────────────────┘         │
│                            │                              │
│         ┌──────────────────┼──────────────────┐          │
│         │                  │                  │          │
│         ▼                  ▼                  ▼          │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │ Claude Code │    │   Cursor   │    │  Copilot   │   │
│  │   Agent     │    │   Agent    │    │   Agent    │   │
│  └────────────┘    └────────────┘    └────────────┘   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

## 待调研组件

以下组件需要调研，确定技术选型：

1. **消息队列**：Redis Pub/Sub vs NATS vs Kafka
2. **任务队列**：Bull (Redis) vs Celery vs 自研
3. **调度器**：node-cron vs Quartz vs 自研
4. **Agent 运行时**：Docker 容器化 vs 直接调用 CLI
5. **状态存储**：Redis vs PostgreSQL vs 文件系统
6. **API 网关**：Fastify vs Express vs 自研
7. **部署方式**：Docker Compose vs Kubernetes
8. **Agent 协议**：WebSocket vs gRPC vs HTTP Long-poll

## 下一步

先讨论清楚整体架构和技术选型，再逐步实现。
