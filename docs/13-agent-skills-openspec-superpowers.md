# Agent Skills vs OpenSpec vs Superpowers 深度对比

> Date: 2026-05-13
> Status: Research Complete

---

## 一句话定位

| 项目 | 定位 | Stars |
|------|------|-------|
| **Agent Skills** | AI 编程的「工程技能库」— 覆盖全生命周期 | ~4k |
| **OpenSpec** | AI 编程的「Spec 驱动开发框架」— 先对齐再做 | 47.4k |
| **Superpowers** | AI 编程的「完整开发方法论」— 让 AI 能自主工作数小时 | ~8k |

---

## 核心对比矩阵

| 维度 | Agent Skills | OpenSpec | Superpowers |
|------|-------------|----------|-------------|
| **生命周期覆盖** | Define→Plan→Build→Verify→Review→Ship | Propose→Spec→Design→Tasks→Apply→Verify→Archive | Brainstorm→Plan→Build→Test→Review→Ship |
| **入口方式** | 7 个 slash commands (`/spec`, `/build` 等) | `/opsx:propose` + 扩展命令集 | 技能自动触发（不用手动调命令） |
| **技能触发机制** | 手动调用 / 自动按场景激活 | 手动调用 | **自动触发**（核心差异化） |
| **Human-in-loop** | 每个 phase 都需要 human sign-off | 在 proposal/design 阶段 human review | 先对齐 spec，然后 AI 自主执行数小时 |
| **Artifact 管理** | 无持久化，聊天内 | 变更文件夹（proposal/specs/design/tasks） | 有 plan 文档和 design doc |
| **子 Agent 支持** | 无 | 无 | **有**（subagent-driven-development） |
| **Anti-rationalization** | ✅ 有（表格形式） | ❌ | ✅ 有（Red Flags 表格） |
| **工具支持数** | 8+ | 25+ | 8 |
| **TDD 强调** | ✅ (`/test`, `test-driven-development`) | ❌ | ✅ (`test-driven-development`) |
| **零依赖** | ❌ (npm 包) | ❌ (npm 包) | ✅ (零外部依赖) |

---

## 各维度详解

### 1. 技能触发机制（最核心差异）

**Agent Skills** — 手动触发 + 场景激活

```
你: /build
AI: 开始构建...

你: （构建 UI 组件）
AI: 自动激活 frontend-ui-engineering skill
```

- 手动 slash commands 是主要入口
- 某些 skill 会按场景自动激活

**OpenSpec** — 手动驱动，artifact 导向

```
你: /opsx:propose add-user-auth
AI: 创建 openspec/changes/add-user-auth/
    ✓ proposal.md
    ✓ specs/
    ✓ design.md
    ✓ tasks.md

你: /opsx:apply
AI: 执行 tasks.md 中的任务
```

- 每个变更有独立文件夹
- 流程清晰但需要手动推进每一步

**Superpowers** — **自动触发**（最激进）

```
你: "Let's add user authentication"
AI: (自动激活 brainstorming skill)
    "What are you really trying to build?"
    "Tell me about your users..."
```

- `using-superpowers` skill 是 bootstrap
- **只要有 1% 可能性 skill 适用就必须检查**
- 一旦 spec 对齐，AI 可以自主工作数小时不偏离

### 2. Human-in-Loop 时机

| | Agent Skills | OpenSpec | Superpowers |
|--|--|--|--|
| **Spec 对齐** | `/spec` 阶段 | `proposal.md` + `design.md` | Brainstorm 阶段多次对齐 |
| **任务执行** | 每步需 human 确认 | `/opsx:apply` 后自主执行 | Human 说 "go" 后自主执行 |
| **Review** | `/review` 强制质量门禁 | `/opsx:verify` | `requesting-code-review` skill |

**Superpowers 最激进**：Human 确认 spec 后，AI 用 subagent 模式自主工作几个小时。

### 3. 技能数量与专注度

- **Agent Skills**: 22 个技能，精细化分工（`test-driven-development`, `api-and-interface-design`, `frontend-ui-engineering`, `security-and-hardening`...）
- **OpenSpec**: 核心是 spec/artifacts 管理，技能不是重点
- **Superpowers**: ~15 个核心技能，**每个都很深入**（如 `systematic-debugging` 有 6 个子文档讲 debug 方法论）

### 4. Anti-Slop 机制（防止 AI 敷衍）

**Agent Skills** 有 anti-rationalization 表格：

```
❌ "This is just a simple change" → 停止，检查是否有适用 skill
❌ "I can skip tests for now"   → 停止，TDD 是强制要求
```

**Superpowers** 有 Red Flags 表格：

```
❌ "This is just a question"    → 问题也是任务，检查 skill
❌ "Let me gather info first"   → 停止，skill 告诉你怎么收集
```

**OpenSpec** 无此机制。

### 5. 子 Agent / 并行执行

只有 **Superpowers** 有成熟的 subagent 支持：

- `subagent-driven-development` skill
- 主 agent 协调多个子 agent 并行工作
- 子 agent 的输出被主 agent 审查后才继续

### 6. 工具支持

| | Agent Skills | OpenSpec | Superpowers |
|--|--|--|--|
| Claude Code | ✅ | ✅ | ✅ (官方 marketplace) |
| Cursor | ✅ | ✅ | ✅ |
| GitHub Copilot | ✅ | ✅ | ✅ |
| Windsurf | ✅ | ✅ | ❌ |
| Gemini CLI | ✅ | ✅ | ✅ |
| Codex | ✅ | ✅ | ✅ |
| Kiro | ✅ | ❌ | ❌ |
| OpenCode | ✅ | ✅ | ✅ |
| 通用（Markdown） | ✅ | ✅ | ✅ |

---

## 关键洞察

### 1. 触发机制决定使用体验

```
Agent Skills:     你告诉 AI → AI 执行 （你驱动）
OpenSpec:         你告诉 AI → AI 创建 artifacts → 你 review → AI 执行
Superpowers:      AI 观察你的行为 → 自动激活技能 → 你对齐 spec → AI 自主执行
```

Superpowers 最接近「AI 原生工作流」——你只需要说出目标，AI 自动选择技能。

### 2. Human 介入程度

```
Superpowers 最少介入: spec 对齐后 AI 自主工作数小时
OpenSpec 中等介入:   每步 artifact 都需要 human review
Agent Skills 最多介入: 每个 phase 都有 slash command 需要 human 触发
```

### 3. 深度 vs 广度

- **Agent Skills**: 广度优先，22 个技能覆盖全生命周期
- **Superpowers**: 深度优先，每个 skill 都有大量反学习表格和 pressure testing
- **OpenSpec**: 独特赛道，专注 spec 管理而非具体技能

---

## 选型建议

| 场景 | 推荐 |
|------|------|
| 你想要开箱即用的完整工程流程 | **Agent Skills** |
| 你想让 AI 自主工作更久，减少介入 | **Superpowers** |
| 你重视需求变更的规范管理和可追溯性 | **OpenSpec** |
| 你的团队有严格的质量门禁要求 | **Agent Skills** (`/review`, `/test`) |
| 你是个人开发者，想要 AI 像个热情的初级工程师 | **Superpowers** |
| 你需要 25+ 工具的支持 | **OpenSpec** |
| 你讨厌配置，想要 AI 自动选择技能 | **Superpowers** |
| 你想要 TDD 强制的开发流程 | **Superpowers** 或 **Agent Skills** |

---

## 与 ai-harness 项目的契合度

根据之前 ai-harness 的调研：

| 组件 | 契合的框架 | 原因 |
|------|-----------|------|
| **ClawTeam** (多 Agent 协作) | **Superpowers** | 已有 subagent-driven-development |
| **OpenSpec** (Spec 管理) | **OpenSpec** | 本身就是 Spec-Driven |
| **OpenHarness** (Agent 编排) | **Agent Skills** | 覆盖全生命周期，适合编排 |
| **Temporal** (工作流) | **Superpowers** | 自主执行 + 子 agent 协调 |

**Superpowers** 与 ai-harness 的 ClawTeam 理念最接近——都强调子 agent 的协调与自主执行。
