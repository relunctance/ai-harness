# Ai-Harness

**多 Agent 自动化协作基础设施**

## 背景

[OpenSpec](./docs/01-openspec-overview.md) 是一个出色的规范层框架，但它缺少任务调度、Agent 间通信、自动化流水线能力。

Ai-Harness 旨在补全这些缺失，构建真正的多 Agent 自动化协作系统。

## 文档

- [OpenSpec 概述](./docs/01-openspec-overview.md) — 什么是 OpenSpec，核心工作流，命令一览
- [OpenSpec 能力分析](./docs/02-openspec-capabilities.md) — 任务拆分、调度、多 Agent 协作能力分析
- [Ai-Harness 愿景](./docs/03-ai-harness-vision.md) — 项目定位、目标能力、架构方向

## 核心思路

继承 OpenSpec 规范层，补全：

| 缺失能力 | 解决方案 |
|----------|----------|
| 任务调度 | 任务调度中心（Cron + 队列） |
| Agent 通信 | Agent 网关（消息总线） |
| 自动化流水线 | 任务队列 + 事件驱动 |

## 当前阶段

架构讨论阶段。详见 [愿景文档](./docs/03-ai-harness-vision.md#待调研组件)。

需要调研的组件：
- 消息队列（Redis Pub/Sub / NATS / Kafka）
- 任务队列（Bull / Celery / 自研）
- 调度器（node-cron / Quartz / 自研）
- Agent 运行时（Docker / CLI）
- 状态存储（Redis / PostgreSQL / 文件系统）
- API 网关（Fastify / Express / 自研）
- 部署方式（Docker Compose / Kubernetes）
- Agent 协议（WebSocket / gRPC / HTTP Long-poll）
