# Anthropic Skills 调研

## 项目信息

- **GitHub**: https://github.com/anthropics/skills
- **Stars**: 133k
- **Forks**: 15.7k
- **定位**: Public repository for Agent Skills（Anthropic 官方技能仓库）
- **License**: Apache 2.0（部分技能是 source-available）

---

## 核心定位

> This repository contains Anthropic's implementation of skills for Claude.

Anthropic 官方的 **Claude Skills** 参考实现仓库，展示 Claude 技能系统的可能性。

**关键参考**：
- 技能规范定义（agentskills.io 标准）
- 技能创建模板
- 各类任务的最佳实践

---

## 目录结构

```
anthropics/skills/
├── .claude-plugin/       # Claude Code 插件配置
├── skills/               # 技能实现
│   ├── algorithmic-art/
│   ├── brand-guidelines/
│   ├── canvas-design/
│   ├── claude-api/
│   ├── doc-coauthoring/
│   ├── docx/             # Word 文档处理
│   ├── frontend-design/
│   ├── internal-comms/
│   ├── mcp-builder/      # MCP Server 生成
│   ├── pdf/              # PDF 处理
│   ├── pptx/             # PPT 处理
│   ├── skill-creator/
│   ├── slack-gif-creator/
│   ├── theme-factory/
│   ├── web-artifacts-builder/
│   ├── webapp-testing/
│   └── xlsx/             # Excel 处理
├── spec/                 # Agent Skills 规范
│   └── agent-skills-spec.md
└── template/             # 技能创建模板
```

---

## Agent Skills 规范（agent-skills-spec.md）

### 技能格式

每个技能是一个文件夹，包含 `SKILL.md`：

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

### 前置matter 要求

| 字段 | 说明 |
|------|------|
| `name` | 唯一标识符（lowercase, hyphens） |
| `description` | 完整描述：技能做什么、何时使用 |

### 技能内容

```markdown
# Skill Name

[主指令]

## Examples
[使用示例]

## Guidelines
[使用指南]
```

---

## 技能分类

### 开发和技术类

| 技能 | 说明 |
|------|------|
| **claude-api** | 调用 Claude API |
| **mcp-builder** | 构建 MCP Server |
| **webapp-testing** | Web 应用测试 |
| **frontend-design** | 前端设计 |

### 文档处理类

| 技能 | 说明 |
|------|------|
| **docx** | Word 文档创建和编辑 |
| **pdf** | PDF 处理 |
| **pptx** | PowerPoint 创建 |
| **xlsx** | Excel 电子表格 |
| **doc-coauthoring** | 文档协作 |

### 创意和设计类

| 技能 | 说明 |
|------|------|
| **algorithmic-art** | 算法艺术生成 |
| **canvas-design** | Canvas 设计 |
| **web-artifacts-builder** | Web 工件构建 |
| **theme-factory** | 主题工厂 |

### 企业和通信类

| 技能 | 说明 |
|------|------|
| **brand-guidelines** | 品牌指南 |
| **internal-comms** | 内部通信 |
| **slack-gif-creator** | Slack GIF 创建 |

---

## 使用方式

### Claude Code

```bash
# 添加为 marketplace 插件
/plugin marketplace add anthropics/skills

# 安装技能集
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

### Claude.ai

已对付费用户开放所有技能。

### Claude API

支持通过 API 上传和使用自定义技能。

---

## 技能创建模板

```markdown
---
name: my-skill-name
description: A clear description of what this skill does and when to use it
---

# My Skill Name

[Add your instructions here that Claude will follow when this skill is active]

## Examples
- Example usage 1
- Example usage 2

## Guidelines
- Guideline 1
- Guideline 2
```

---

## 与 Darwin Skill 的关系

| 维度 | Darwin Skill | Anthropic Skills |
|------|------------|-----------------|
| **格式** | SKILL.md + 评估 + 改进 | SKILL.md（纯指令） |
| **规范** | agentskills.io | agentskills.io（相同） |
| **自改进** | ✅ | ❌ |
| **评估框架** | ✅ | ❌ |
| **使用场景** | 通用 LLM | Claude 专用 |
| **开源程度** | AGPL v3 | Apache 2.0 / Source-available |

---

## 对 Ai-Harness 的借鉴

### 技能规范参考

Anthropic Skills 是最权威的 **SKILL.md 格式参考**：

```markdown
---
name: <lowercase-hyphens>
description: <完整描述，包含何时使用>
---

# Skill Name

## 指令内容
[Claude 遵循的具体指令]

## Examples
[使用示例]

## Guidelines
[使用指南]
```

### 技能分类参考

| 分类 | 示例 |
|------|------|
| **开发/技术** | claude-api, mcp-builder, webapp-testing |
| **文档处理** | docx, pdf, pptx, xlsx |
| **创意/设计** | algorithmic-art, canvas-design |
| **企业通信** | brand-guidelines, internal-comms |

### 技能创建最佳实践

1. **description 要完整** — 说明何时使用，不是简单重复 name
2. **Examples 要具体** — 真实的可运行示例
3. **Guidelines 要可操作** — 不是泛泛的原则，而是具体指导
4. **内容要模块化** — 避免过长的指令，拆分成合理单元

---

## 与 agentskills.io 的关系

> For information about the Agent Skills standard, see agentskills.io.

Anthropic Skills 实现了 **agentskills.io** 规范定义的技能标准。这个标准是开放的，任何 LLM Agent 都可以实现。

**agentskills.io 定义的核心内容**：
- SKILL.md 格式规范
- 技能元数据字段
- 技能注册和发现机制
