# AI Agent 开发框架/工具 选型指南

> Date: 2026-05-13
> Status: synthesized from all research

---

## 你的目标是什么？

### 目标 A：构建「多 Agent 自动化协作流水线」
> 多个 AI Agent 并行工作，协调完成复杂任务，人在关键节点把关

**核心组件**: OpenHarness + ClawTeam-OpenClaw + Temporal

### 目标 B：规范 AI 编程输出质量
> 让 AI 在写代码之前先理解需求，形成可追溯的规范文档

**核心组件**: OpenSpec 或 Spec Kit

### 目标 C：让 AI 能自主工作数小时不偏离
> AI 自动选择技能，Human 对齐目标后 AI 自主执行

**核心组件**: Superpowers

### 目标 D：AI 编程全生命周期质量保障
> 从需求到上线，每步都有工程化质量门禁

**核心组件**: Agent Skills

---

## 组件全景图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        你的需求                                     │
│   规范 │ 协调 │ 自主 │ 质量 │ 持久化 │ 工具支持                        │
└─────────────────────────────────────────────────────────────────────┘
        │       │       │       │         │
        ▼       ▼       ▼       ▼         ▼
    ┌────────┬────────┬────────┬────────┬────────┐
    │OpenSpec│ClawTeam│Super-  │Agent   │Temporal │
    │Spec Kit│OpenClaw│powers  │Skills  │         │
    └────────┴────────┴────────┴────────┴────────┘
```

---

## Spec 类工具：OpenSpec vs Spec Kit vs Agent Skills

### 一句话对比

| 工具 | 规格文档 | 触发方式 | 工具支持 |
|------|---------|---------|---------|
| **OpenSpec** | 参考指南（不直接生成代码） | 手动 `/opsx:*` | 25+ |
| **Spec Kit** | 可执行契约（直接生成代码） | 手动 `/speckit.*` | 仅 Copilot 官方 |
| **Agent Skills** | 工程技能库（22 个技能） | 手动 + 场景自动 | 8+ |

### OpenSpec vs Spec Kit 核心差异

**OpenSpec**: 规格是「**指南**」，AI 理解后自由发挥

```
/opsx:propose "用户登录功能"
AI 生成:
  ✓ proposal.md — 为什么做
  ✓ specs/     — 功能需求
  ✓ design.md  — 技术方案
  ✓ tasks.md   — 实现清单
  ↓
Human review
  ↓
/opsx:apply → AI 自己写代码
```

**Spec Kit**: 规格是「**契约**」，直接约束代码生成

```
/speckit.specify "用户登录功能"
AI 生成:
  ✓ spec.md (含数据模型、API 契约、UI 描述)
  ✓ contracts/api-spec.json
  ↓
/speckit.plan (Human 提供 tech stack)
  ↓
/speckit.implement → **直接从 spec 生成可执行代码**
```

### 决策树

```
你的项目用 Copilot 吗？
  ├─ 是 → Spec Kit（官方集成，规格直接生成代码）
  └─ 否 → OpenSpec（工具无关，25+ 支持）
        ├─ 想要轻量简单 → OpenSpec Core Profile
        └─ 想要完整规范 → OpenSpec Expanded Workflow
```

### Agent Skills 的定位

Agent Skills **不是 Spec 框架**，而是**工程技能库**。

```
当你要做 X，Agent Skills 告诉你：
  /spec    → 写规范前先定义清楚
  /plan    → 拆解成小任务
  /build   → 增量实现（一薄片一薄片）
  /test    → 测试是证明，不是检查
  /review  → 质量门禁
  /ship    → 上线检查清单
```

**Agent Skills 适合**: 团队有严格工程流程，需要每个 phase 都有强制质量门禁。

---

## 协调类工具：ClawTeam-OpenClaw vs OpenHarness

### 核心问题：谁来当主控？

| 场景 | 推荐 |
|------|------|
| Human 想要随时介入每个决策 | OpenHarness（Human 实时控制） |
| Human 对齐目标后让 AI 自主跑 | ClawTeam-OpenClaw（Swarm 自治） |

### OpenHarness

**定位**: 主控 Agent 基础设施

```
Human ←→ OpenHarness Agent ←→ 工具/其他 Agent
         ↓
      ohmo（飞书/Slack 通知）
```

**优点**:
- 实时 Human-in-loop
- Cron 调度
- 飞书/Slack/Discord/Telegram 通知
- Agent Loop 持久化

**缺点**:
- 多 Agent 协调能力弱（主要是 inbox 消息）
- 适合「1 个主控 + 多个工具」，不适合「多 Agent 并行开发」

### ClawTeam-OpenClaw

**定位**: Swarm 多 Agent 协调

```
Human ←→ ClawTeam 主控
         ↓ (Swarm 模式)
      [Worker Agent A] ←→ [Worker Agent B] ←→ [Worker Agent C]
         ↓
      Git Worktree 隔离
```

**优点**:
- Task Dependencies（`--blocked-by` + auto-unblock）
- Git Worktree 强制隔离
- Team Templates（一键启动团队）
- inbox 消息广播

**缺点**:
- 没有内置持久化（Temporal 补）
- Human 不能实时介入每个决策

### 决策

```
你的场景是「人在循环中」还是「AI 自主执行」？
  ├─ 人在循环中 → OpenHarness
  └─ AI 自主执行 → ClawTeam-OpenClaw
        ├─ 需要失败恢复 → + Temporal
        └─ 需要持久化 → + Temporal
```

---

## 自主执行：Superpowers

**核心能力**: 技能自动触发 + Subagent 协调

```
你说: "Let's add dark mode"

Superpowers 自动触发:
  brainstorming skill → "What are you really trying to build?"
  ↓ (Human 对齐 spec)
  writing-plans skill → 创建实现计划
  ↓
  subagent-driven-development → 子 agent 并行执行
  ↓
  requesting-code-review → Human review
```

**适合场景**:
- 个人开发者，想要 AI 像「热情的初级工程师」
- Human 只想对齐目标，不想介入每步细节
- 需要 AI 能自主工作数小时

**不适合场景**:
- 团队有严格审批流程
- 需要实时 Human-in-loop

---

## 持久化：Temporal

**定位**: Workflow 持久化 + 自动重试 + 失败恢复

```
任务执行中崩溃？
  └─ Temporal 自动从上次 checkpoint 恢复

LLM 调用超时？
  └─ Temporal 自动重试（可配置）

需要完整执行历史？
  └─ Temporal 持久化每个 step
```

**注意**: Temporal 是**后端服务**，需要自己部署（docker-compose 或 k8s）。

---

## 工具选择矩阵

| 你的目标 | 必选 | 可选 | 不需要 |
|---------|------|------|--------|
| 多 Agent 并行开发 | ClawTeam-OpenClaw | Temporal | OpenHarness（如果不需要实时 Human） |
| Spec 规范管理 | OpenSpec | - | - |
| 规格直接生成代码 | Spec Kit | - | OpenSpec（如果 Copilot 唯一） |
| AI 自主工作数小时 | Superpowers | - | - |
| 全生命周期质量 | Agent Skills | - | - |
| Workflow 持久化 | Temporal | - | - |
| Human 实时控制 | OpenHarness | - | - |
| 飞书通知 | ohmo (OpenHarness) | - | - |

---

## 实际组合示例

### 组合 1：个人开发提效
```
Superpowers (自动技能) + OpenSpec (规范管理) + CubeSandbox (沙箱执行)
```
适用于：个人想要 AI 自主完成功能，人只做 design review

### 组合 2：团队协作流水线
```
OpenSpec (规范) + ClawTeam-OpenClaw (协调) + Temporal (持久化) + GitHub Actions (CI)
```
适用于：团队多 Agent 并行开发，需要失败恢复

### 组合 3：企业级规范开发
```
Spec Kit (可执行规格) + Agent Skills (质量门禁) + OpenHarness (Human 实时控制)
```
适用于：企业需要严格规范，Human 随时介入

### 组合 4：全功能方案（ai-harness 目标）
```
OpenHarness (主控 + 通知) + ClawTeam-OpenClaw (协调) + OpenSpec (规范) 
+ Superpowers (TDD + Subagent) + Temporal (持久化) + CubeSandbox (沙箱)
```
适用于：完整的多 Agent 协作基础设施

---

## 组件依赖关系

```
                    ┌──────────────────┐
                    │   Human 决策点    │
                    └────────┬─────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    OpenHarness (主控)                       │
│   ohmo (通知) + Agent Loop (控制流) + Cron (调度)            │
└────────────────────────┬────────────────────────────────────┘
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐
│  OpenSpec   │  │  Superpowers │  │  ClawTeam-OpenClaw │
│  (规范层)   │  │  (TDD+自主)  │  │  (多 Agent 协调)   │
└──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘
       │                 │                     │
       │                 │                     │
       ▼                 ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│           Temporal (持久化)  │  CubeSandbox (沙箱)           │
│           失败恢复 + 重试    │  <60ms 冷启动                 │
└─────────────────────────────────────────────────────────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │  GitHub Actions  │
                    │     (CI/CD)      │
                    └──────────────────┘
```

---

## 下一步

1. **确认你的核心目标**（A/B/C/D 或组合）
2. **选定第一个试点组件**（建议从 OpenSpec 开始，因为它最独立）
3. **验证端到端流程**
