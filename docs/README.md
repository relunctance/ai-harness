# Ai-Harness 文档

> 多 Agent 自动化协作基础设施

## 核心选型指南

| 你的目标 | 推荐组合 |
|---------|---------|
| 多 Agent 并行开发 | ClawTeam-OpenClaw + Temporal |
| Spec 规范管理 | OpenSpec |
| 规格直接生成代码 | Spec Kit |
| AI 自主工作数小时 | Superpowers |
| 全生命周期质量 | Agent Skills |
| Human 实时控制 | OpenHarness |
| Workflow 持久化 | Temporal |

**快速导航**: [15-component-selection-guide.md](./15-component-selection-guide.md) — 完整选型决策树

---

## 文档索引

### 概览与愿景
| 文档 | 说明 |
|------|------|
| [03-ai-harness-vision](./03-ai-harness-vision.md) | 项目定位、目标能力、架构方向 |
| [15-component-selection-guide](./15-component-selection-guide.md) | **选型指南** — 根据目标选择组件 |

### Spec 类工具深度调研
| 文档 | 说明 |
|------|------|
| [01-openspec-overview](./01-openspec-overview.md) | OpenSpec 概述：核心工作流、命令、哲学 |
| [02-openspec-capabilities](./02-openspec-capabilities.md) | OpenSpec 能力分析：任务拆分、调度、多 Agent 协作 |
| [13-agent-skills-openspec-superpowers](./13-agent-skills-openspec-superpowers.md) | Agent Skills vs OpenSpec vs Superpowers 三方对比 |
| [14-spec-kit-vs-openspec](./14-spec-kit-vs-openspec.md) | Spec Kit vs OpenSpec 深度对比（规格可执行性为核心差异） |

### 协调与自主执行
| 文档 | 说明 |
|------|------|
| [04-multi-agent-landscape](./04-multi-agent-landscape.md) | 多 Agent 协作方案：Superpowers / DeerFlow / CubeSandbox / OpenHarness / ClawTeam |
| [11-openharness](./11-openharness.md) | OpenHarness 调研：主控 Agent + ohmo 助手 |
| [12-deerflow-2](./12-deerflow-2.md) | DeerFlow 2.0 调研：Subagent 持久化后端分析 |

### 单点工具调研
| 文档 | 说明 |
|------|------|
| [05-everything-claude-code](./05-everything-claude-code.md) | Everything Claude Code 调研：181k stars |
| [06-hermes-agent](./06-hermes-agent.md) | Hermes Agent 调研：147k stars 自改进 Agent |
| [07-hermes-agent-self-evolution](./07-hermes-agent-self-evolution.md) | Hermes Agent Self-Evolution：DSPy + GEPA 进化优化 |
| [08-temporal](./08-temporal.md) | Temporal 调研：Workflow 持久化平台 |
| [09-anthropic-skills](./09-anthropic-skills.md) | Anthropic Skills 调研：133k stars 官方仓库 |
| [10-重新审视Agent间通信](./10-重新审视Agent间通信.md) | 已调研项目的 Agent 通信模式分析与缺口 |

---

## 快速导航

- [OpenSpec 官网](https://github.com/Fission-AI/OpenSpec)
- [Spec Kit 官网](https://github.com/github/spec-kit)
- [Superpowers](https://github.com/obra/superpowers)
- [Agent Skills](https://github.com/addyosmani/agent-skills)
- [DeerFlow](https://github.com/bytedance/deer-flow)
- [CubeSandbox](https://github.com/TencentCloud/CubeSandbox)
- [Everything Claude Code](https://github.com/affaan-m/everything-claude-code)
- [Hermes Agent](https://github.com/NousResearch/hermes-agent)
- [Hermes Agent Self-Evolution](https://github.com/NousResearch/hermes-agent-self-evolution)
- [Temporal](https://github.com/temporalio/temporal)
- [Anthropic Skills](https://github.com/anthropics/skills)
- [ClawTeam-OpenClaw](https://github.com/win4r/ClawTeam-OpenClaw)
- [OpenHarness](https://github.com/AI-Engineering/OPEN_HARNESS)

---

## 架构图

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

## 专家团系统 (Expert Teams)

基于「角色 + Skills 组合」的子 Agent 定制系统。

### 核心组件

| 组件 | 仓库 | 用途 |
|------|------|------|
| **Role-Creator** | [relunctance/role-creator](https://github.com/relunctance/role-creator) | 创建单个 role 目录结构 |
| **Team-Assembler** | [relunctance/team-assembler](https://github.com/relunctance/team-assembler) | 组合多个 roles 为完整团队 |
| **role-installer** | [relunctance/role-installer](https://github.com/relunctance/role-installer) | 安装专家团（clone 仓库 + skills） |
| **role-skill-manager** | [relunctance/role-skill-manager](https://github.com/relunctance/role-skill-manager) | 管理 role 的 skills（添加/删除/升级） |
| **inbox-router** | [relunctance/inbox-router](https://github.com/relunctance/inbox-router) | sub-agent 协调通信（任务分配/汇报/等待） |

### 架构图

```
Human → 主理人 (OpenHarness)
           ↓
      Inbox-Router
           ↓ dispatch / report / wait-for
    ┌──────┼──────┐
    ↓      ↓      ↓
  Role A  Role B  Role C
           ↓
      汇报给主理人
```

### 目录结构

```
~/expert-teams/<team_name>/
├── SKILL.md              ← 主理人入口 skill
├── config.yaml           ← 团队全局配置
├── common_skills/       ← 公共 skills
├── roles/                ← 子 agent 目录
│   ├── lisi/
│   │   ├── SKILL.md
│   │   ├── config.yaml   ← 可覆盖团队配置
│   │   └── skills/        ← 该 role 的 skills
│   ├── wangwu/
│   └── zhangsan/
└── scripts/
```

### 使用流程

1. **创建 Role**: `Role-Creator` 创建单个 role
2. **调试 Role**: 单独调试优化每个 role
3. **组装团队**: `Team-Assembler` 组合多个 roles
4. **安装团队**: `role-installer` 一键安装
5. **管理 Skills**: `role-skill-manager` 添加/升级 skills
6. **协调工作**: `inbox-router` 分配任务、接收汇报

### 相关组件

- **OpenHarness**: 主理人（任务拆解、协调）
- **ClawTeam**: Git Worktree 隔离、Task Dependencies
- **Superpowers**: subagent-driven 开发模式
- **Temporal**: 任务持久化、失败重试
