# Ai-Harness 愿景

## 项目定位

**多 Agent 自动化协作基础设施** — 用已有开源组件（不自研），实现"需求→拆解→评审→设计方案→开发→测试→验收"的多 Agent 协作，**人在关键节点把关**。

## 解决的问题

OpenSpec 解决了"AI 编程需求模糊"的问题，但缺少任务调度、Agent 间通信、自动化流水线能力。

**解决方案**：用已有组件补全，而不是自研。

## 阶段一 MVP 组件

| 组件 | 解决什么问题 |
|------|-------------|
| **OpenHarness** | 主控 Agent + ohmo 飞书助手 + Swarm 多 Agent 协调 + Mailbox 消息队列 |
| **OpenSpec** | 规范层，spec-driven 开发 |
| **Superpowers** | TDD 方法论，brainstorming，parallel agent dispatch |
| **CubeSandbox** | 沙箱执行，<60ms 冷启动 |
| **Temporal** | Workflow 持久化，自动重试，失败恢复 |
| **GitHub Actions** | CI/CD 自动化 |

## 目标能力

### 1. 需求理解与拆解
- OpenHarness Agent Loop 理解用户需求
- Superpowers brainstorming 澄清模糊需求
- 拆解成可执行的小任务

### 2. 规范驱动开发
- OpenSpec 定义任务规范
- Superpowers writing-plans 分解任务
- spec-driven 执行

### 3. 多 Agent 并行开发
- OpenHarness Swarm 协调多 Agent
- CubeSandbox 隔离执行环境
- Superpowers subagent-driven 并行分派

### 4. TDD 测试驱动
- Superpowers TDD 流程
- RED-GREEN-REFACTOR 强制循环
- GitHub Actions CI 全绿

### 5. Workflow 持久化
- Temporal 流水线状态持久化
- 失败自动重试
- 完整执行历史

### 6. 人在关键节点把关
- ohmo 飞书/Slack/Discord/Telegram 接收通知
- Human 审批设计方案
- Human 验收签字

## 不做什么

- ❌ 不做自己的 AI 模型
- ❌ 不做 IDE 或编辑器
- ❌ 不自研消息队列/任务队列/调度器
- ❌ 不重复造轮子

## 架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      Human                                  │
│         飞书/审批 → 验收签字 · 需求澄清 · 关键审批            │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  OpenHarness (主控 Agent)                                    │
│  · ohmo — 飞书/Slack/Discord/Telegram 个人助手               │
│  · Agent Loop — query → stream → tool-call → loop           │
│  · Swarm — team/agent/send_message 多 Agent 协调             │
│  · Mailbox — 文件-based 异步消息队列                         │
│  · CronCreate/List/Delete — 任务调度                        │
└────────────┬────────────────────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────────────────────────┐
│  OpenSpec (规范层)                                           │
│  Superpowers (开发方法论)                                    │
│  CubeSandbox (沙箱执行)                                      │
│  Temporal (Workflow 持久化)                                  │
│  GitHub Actions (CI/CD)                                      │
└─────────────────────────────────────────────────────────────┘
```

## 下一步

1. 确认组件清单（当前是否合适）
2. 选定第一个试点任务
3. 验证现有组件能否跑通端到端流程
