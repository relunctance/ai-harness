# OpenSpec 概述

## 是什么

**OpenSpec** 是一个 **Spec-Driven Development (SDD)** 框架，专为 AI 编程助手设计。

GitHub: https://github.com/Fission-AI/OpenSpec
Stars: 47.4k | Forks: 3.3k

## 核心理念

```
fluid not rigid         — 灵活不僵化，无阶段门禁
iterative not waterfall — 迭代不瀑布，边做边学
easy not complex        — 简单不复杂，快速上手
brownfield-first       — 支持存量项目，不仅是新建项目
```

## 解决了什么问题

AI 编程助手很强，但需求模糊、结果不可预测。

OpenSpec 在代码编写之前先让人类和 AI 就"要构建什么"达成一致，形成轻量级的规范层。

## 核心工作流

```
/opsx:propose → /opsx:ff → /opsx:apply → /opsx:verify → /opsx:archive
    │              │            │             │             │
  创建变更      快速生成      执行任务      验证实现      归档合并
  artifacts      全部artifacts
```

## Artifacts

| Artifact | 文件 | 描述 |
|----------|------|------|
| Proposal | `proposal.md` | 为什么做这个变更 |
| Specs | `specs/**/*.md` | 需求和场景（Delta 格式） |
| Design | `design.md` | 技术方案 |
| Tasks | `tasks.md` | 实现 checklist |

## 目录结构

```
openspec/
├── specs/                  # 主规范（系统行为的事实来源）
│   ├── auth/
│   └── ui/
└── changes/               # 变更提案
    ├── add-xxx/
    └── archive/          # 归档的变更
```

## 命令一览

### Core Profile（默认）

| 命令 | 用途 |
|------|------|
| `/opsx:propose` | 创建变更 + 生成全套 artifacts |
| `/opsx:explore` | 探索需求，不创建 artifacts |
| `/opsx:apply` | 执行 tasks.md 中的任务 |
| `/opsx:sync` | 合并 delta specs 到主规范 |
| `/opsx:archive` | 归档变更 |

### Expanded Workflow

| 命令 | 用途 |
|------|------|
| `/opsx:new` | 启动变更脚手架 |
| `/opsx:continue` | 按依赖链逐个创建 artifact |
| `/opsx:ff` | 快速生成全部 artifacts |
| `/opsx:verify` | 验证实现是否符合规范 |
| `/opsx:bulk-archive` | 批量归档多个变更 |
| `/opsx:onboard` | 引导式教程 |

## 支持的工具

25+ AI 工具：Claude Code、Cursor、Windsurf、Copilot 等。
