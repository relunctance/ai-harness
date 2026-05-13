# Spec Kit vs OpenSpec 深度对比

> Date: 2026-05-13
> Status: Research Complete

---

## 一句话定位

| 项目 | 定位 | Stars | 维护方 |
|------|------|-------|--------|
| **Spec Kit** | GitHub 官方推出的 Spec-Driven Development **工具链**，规格文档直接生成可执行代码 | ~3k | GitHub 官方 |
| **OpenSpec** | 社区主导的 Spec-Driven Development **框架**，轻量迭代式变更管理 | 47.4k | Fission-AI (社区) |

---

## 核心对比矩阵

| 维度 | Spec Kit | OpenSpec |
|------|----------|----------|
| **出品方** | GitHub 官方 | Fission-AI (独立公司) |
| **架构风格** | Python CLI (`specify`) + Agent prompts | Node.js CLI (`openspec`) + Artifacts |
| **规格文档** | **可执行** — 直接生成实现 | **参考用** — 指导但不直接生成代码 |
| **Phase 划分** | Constitution → Spec → Plan → Tasks → Implement | Proposal → Specs → Design → Tasks → Apply → Archive |
| **变更单位** | 每个 feature 一个目录 (`specs/001-xxx/`) | 每个变更一个目录 (`changes/add-xxx/`) |
| **Human 介入** | 每步都需 human sign-off | 核心 profile 简化为 3 步 |
| **工具支持** | 少（Copilot 官方集成，其他实验性） | 多（25+，覆盖主流 AI 工具） |
| **扩展机制** | Extensions (Read/Write/Process/Integration) + Presets | 无明确扩展机制 |
| **Constitution** | ✅ 有（项目级原则） | ❌ |
| **Plan 生成** | `/speckit.plan` 需要 human 提供 tech stack | `/opsx:propose` 自动生成 design |
| **并行任务** | ✅ (`[P]` 标记) | ❌ |
| **TDD 集成** | 在 tasks/implement 中包含 | ❌ |
| **研究文档** | ✅ (`research.md` 可并行调研) | ❌ |
| **零依赖** | ❌ (Python) | ❌ (Node.js) |

---

## 核心差异详解

### 1. 规格文档的「可执行性」

这是 Spec Kit 与 OpenSpec **最本质的区别**。

**Spec Kit**: 规格文档是**可执行的**

```
spec.md 包含结构化数据模型、API 契约、UI 描述
     ↓ specify CLI 解析
     ↓ 在 /speckit.implement 时
直接生成可工作的代码实现
```

**OpenSpec**: 规格文档是**参考**

```
proposal.md + specs/*.md + design.md
     ↓
Human review
     ↓
AI 根据 spec 自己写代码
```

Spec Kit 的规格文档更接近「**契约**」，定义清楚了 AI 必须遵守的边界；OpenSpec 的规格更接近「**指南**」，给 AI 足够的灵活性。

### 2. Phase 流程对比

**Spec Kit (7 步)**:

```
/speckit.constitution → /speckit.specify → /speckit.clarify → 
/speckit.plan → /speckit.tasks → /speckit.implement
```

- `constitution`: 项目级原则（代码质量、测试标准、UX 一致性）
- `specify`: 描述要做什么（what + why，不含 tech stack）
- `clarify`: 结构化澄清需求（Clarifications section）
- `plan`: **Human 提供** tech stack + 架构选择
- `tasks`: AI 从 plan 自动生成任务列表
- `implement`: AI 执行任务

**OpenSpec (6 步)**:

```
/opsx:propose → /opsx:ff(可选) → /opsx:apply → /opsx:verify → /opsx:archive
          ↺ /opsx:sync(合并到主规范)
```

- `propose`: 创建变更 + 生成全套 artifacts
- `ff` (fast-forward): 快速生成全部 artifacts
- `apply`: 执行 tasks.md 中的任务
- `sync`: 合并 delta specs 到主规范
- `archive`: 归档变更

### 3. Constitution 机制

Spec Kit 独有 `constitution.md` 机制：

> 这是项目的「宪法」，定义了代码质量标准、测试要求、UX 原则、性能要求。所有后续开发都必须遵守。

OpenSpec 没有这层抽象，直接在 artifact 中定义规范。

### 4. 扩展性

**Spec Kit** 有两层扩展：

- **Community Extensions**: 社区贡献的插件（docs/code/process/integration/visibility 类别）
- **Presets**: 预配置的流程模板（lean/scaffold/self-test）

**OpenSpec** 无明确扩展机制，但支持自定义 artifacts 结构。

### 5. 并行研究能力

Spec Kit 的 `research.md` 允许：

```
在 plan 阶段，AI 可以并行调研不确定的技术细节
每个 research 任务作为独立子任务并行执行
```

OpenSpec 没有这个能力。

### 6. 工具支持

| 工具 | Spec Kit | OpenSpec |
|------|----------|----------|
| GitHub Copilot | ✅ (官方集成) | ✅ |
| Claude Code | 实验性 | ✅ |
| Cursor | 实验性 | ✅ |
| Codex | ❌ | ✅ |
| Gemini CLI | ❌ | ✅ |
| Windsurf | ❌ | ✅ |
| OpenCode | ❌ | ✅ |
| Kiro | ❌ | ✅ |

Spec Kit 只支持 Copilot 官方集成，其他工具实验性或不支持。OpenSpec 覆盖更广。

---

## 优缺点分析

### Spec Kit

**优点**:
- ✅ 规格文档可直接生成代码，减少翻译损失
- ✅ Constitution 机制确保项目级原则被遵守
- ✅ GitHub 官方维护，可信度高
- ✅ 支持并行研究和任务
- ✅ 并行执行标记 (`[P]`) 优化开发流程
- ✅ TDD 结构化集成在 tasks 中

**缺点**:
- ❌ 只有 Copilot 是官方支持，其他工具实验性
- ❌ Phase 多（7 步），对于简单变更可能过于繁琐
- ❌ Python 依赖，在非 Python 项目中可能不习惯
- ❌ plan 阶段需要 human 提供 tech stack（灵活性降低）
- ❌ 扩展机制复杂（Extensions + Presets），学习成本高

### OpenSpec

**优点**:
- ✅ 工具覆盖广（25+），适合多工具团队
- ✅ 轻量灵活，迭代友好
- ✅ 支持 brownfield（存量项目）
- ✅ 社区活跃，stars 最高
- ✅ Core profile 简化为 3 步，易上手

**缺点**:
- ❌ 规格文档不可执行，依赖 AI 理解质量
- ❌ 无 Constitution 机制
- ❌ 无并行研究能力
- ❌ 无并行任务标记
- ❌ 无 TDD 强制要求

---

## 选型建议

| 场景 | 推荐 |
|------|------|
| 你的团队主要用 GitHub Copilot | **Spec Kit** (官方支持) |
| 你想要规格文档直接生成代码 | **Spec Kit** |
| 你想要严格的项目级原则 | **Spec Kit** (Constitution) |
| 你用多种 AI 工具（Claude/Cursor/Codex 等） | **OpenSpec** |
| 你的项目需要快速迭代 | **OpenSpec** |
| 你做存量项目改造 | **OpenSpec** (brownfield-first) |
| 你需要并行调研技术细节 | **Spec Kit** (research.md) |
| 你想要更轻量的框架 | **OpenSpec** |

---

## Spec Kit 的野心

Spec Kit 把自己定位为「**Executable Spec**」——规格文档不只是文档，而是可以直接生成代码的「**程序**」。

这比 OpenSpec 的思路更激进：
- OpenSpec: specs 是 AI 的「**参考指南**」
- Spec Kit: specs 是「**代码模板 + 约束规则**」

但 Spec Kit 目前只有 Copilot 官方支持，其他工具的支持状态是「实验性」，实际效果待验证。

---

## 与 ai-harness 的契合度

Spec Kit 和 OpenSpec 都属于「Spec-Driven Development」赛道，但各有侧重：

| ai-harness 组件 | 契合的框架 | 原因 |
|-----------------|-----------|------|
| **OpenSpec** (Spec 管理) | **OpenSpec** | 本身就是 Spec-Driven，理念一致 |
| **ClawTeam** (多 Agent) | **Superpowers** | Subagent 协调能力 |
| **OpenHarness** (编排) | **Spec Kit** 或 **Agent Skills** | 全生命周期覆盖 |

**如果 ai-harness 要整合 Spec-Driven Development**:
- 轻量方案 → 集成 OpenSpec（工具覆盖广）
- 深度方案 → 集成 Spec Kit（规格可执行）
