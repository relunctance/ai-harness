# OpenHarness 调研

## 项目信息

- **GitHub**: https://github.com/HKUDS/OpenHarness
- **Stars**: 12.4k
- **Forks**: 2.1k
- **Commits**: 383
- **组织**: HKUDS（香港大学）
- **定位**: Open Agent Harness with a Built-in Personal Agent -- ohmo
- **License**: MIT

---

## 核心定位

> OpenHarness delivers core lightweight agent infrastructure: tool-use, skills, memory, and multi-agent coordination.
> ohmo is a personal AI agent built on OpenHarness.

**一句话总结**：轻量级 Agent 基础设施 + 个人助手 ohmo（支持 Feishu/Slack/Telegram/Discord）。

**运行在现有 Claude Code / Codex 订阅上，不需要额外 API key。**

---

## 核心架构（10 个子系统）

```
openharness/
├── engine/          # 🧠 Agent Loop — query → stream → tool-call → loop
├── tools/           # 🔧 43 Tools — file I/O, shell, search, web, MCP
├── skills/          # 📚 Knowledge — on-demand skill loading (.md files)
├── plugins/         # 🔌 Extensions — commands, hooks, agents, MCP servers
├── permissions/     # 🛡️ Safety — multi-level modes, path rules
├── hooks/           # ⚡ Lifecycle — PreToolUse/PostToolUse event hooks
├── commands/        # 💬 54 Commands — /help, /commit, /plan, /resume
├── mcp/             # 🌐 MCP — Model Context Protocol client
├── memory/          # 🧠 Memory — persistent cross-session knowledge
├── tasks/           # 📋 Tasks — background task management
├── coordinator/     # 🤝 Multi-Agent — subagent spawning, team coordination
├── prompts/         # 📝 Context — system prompt assembly, CLAUDE.md
├── config/          # ⚙️ Settings — multi-layer config
├── ui/              # 🖥️ React TUI — backend protocol + frontend
├── sandbox/         # 🔒 Sandbox isolation
├── swarm/           # 🐝 Swarm coordination (tmux/iterm2 panes)
└── autopilot/        # 🚗 Autopilot dashboard
```

---

## Agent Loop（核心执行循环）

```python
while True:
    response = await api.stream(messages, tools)

    if response.stop_reason != "tool_use":
        break  # Model is done

    for tool_call in response.tool_uses:
        # Permission check → Hook → Execute → Hook → Result
        result = await harness.execute_tool(tool_call)

    messages.append(tool_results)
    # Loop continues — model sees results, decides next action
```

**模型决定做什么，Harness 处理如何做。**

---

## 工具系统（43+ Tools）

| 类别 | 工具 | 说明 |
|------|------|------|
| **文件 I/O** | Bash, Read, Write, Edit, Glob, Grep | 核心文件操作 |
| **搜索** | WebFetch, WebSearch, ToolSearch, LSP | Web 和代码搜索 |
| **Agent** | Agent, SendMessage, TeamCreate, TeamDelete | Subagent 托身和协调 |
| **任务** | TaskCreate/Get/List/Update/Stop/Output | 后台任务管理 |
| **MCP** | MCPTool, ListMcpResources, ReadMcpResource | MCP 协议集成 |
| **模式** | EnterPlanMode, ExitPlanMode, Worktree | 工作流模式切换 |
| **调度** | CronCreate/List/Delete, RemoteTrigger | 定时和远程触发 |
| **Meta** | Skill, Config, Brief, Sleep, AskUser | 知识加载、配置、交互 |

---

## 多 Agent 协调（Swarm System）

### 核心组件

```
swarm/
├── types.py           # BackendType, TeammateIdentity, TeammateSpawnConfig, SpawnResult
├── registry.py        # BackendRegistry — 检测 tmux/iterm2/subprocess 可用性
├── mailbox.py        # MailboxMessage — 文件-based 异步消息队列
├── in_process.py     # InProcessBackend — 进程内 asyncio Task 执行
├── subprocess_backend.py  # SubprocessBackend — 子进程执行
├── spawn_utils.py    # Spawn 工具函数
├── team_lifecycle.py # 团队生命周期管理
└── worktree.py       # Git worktree 隔离
```

### Backend 类型

| Backend | 说明 |
|---------|------|
| **subprocess** | 子进程执行（所有平台） |
| **in_process** | asyncio Task 进程内执行 |
| **tmux** | tmux pane 可视化 |
| **iterm2** | iTerm2 pane 可视化 |

### Mailbox 消息队列

**文件-based 异步消息队列**：

```
~/.openharness/teams/<team>/agents/<agent_id>/inbox/<timestamp>_<message_id>.json
```

**消息类型**：
- `user_message` — 用户消息
- `permission_request/response` — 权限请求
- `shutdown` — 关闭信号
- `idle_notification` — 空闲通知

### Team 工具

| 工具 | 说明 |
|------|------|
| `team_create` | 创建内存中的团队 |
| `team_delete` | 删除团队 |
| `send_message` | 向运行中的 agent 发送消息 |
| `agent` | 生成 subagent subprocess |

---

## Skills 系统

**兼容 anthropics/skills** — 使用相同的 `.md` 格式：

```
~/.openharness/skills/<skill>/SKILL.md
~/.claude/skills/<skill>/SKILL.md
~/.agents/skills/<skill>/SKILL.md
<project>/.openharness/skills/<skill>/SKILL.md
```

**内置 Skills**：
- commit, review, debug, plan, test, simplify
- pdf, xlsx（来自 anthropics/skills）
- 40+ 更多

---

## ohmo 个人助手

```
ohmo init           # 初始化 ~/.ohmo workspace
ohmo config         # 配置 channel 和 provider
ohmo gateway start  # 启动 gateway

支持 channel：
- Telegram
- Slack
- Discord
- Feishu（飞书）
```

**关键概念**：
```
~/.ohmo/
├── soul.md         # 长期 agent 个性和行为
├── identity.md     # ohmo 身份
├── user.md         # 用户 profile
├── BOOTSTRAP.md    # 首次运行引导
└── memory/         # 个人记忆
```

---

## Provider 兼容性

| Provider | 格式 | 典型后端 |
|----------|------|---------|
| **Claude 官方** | Anthropic-Compatible | api.anthropic.com |
| **Moonshot/Kimi** | Anthropic-Compatible | api.moonshot.cn |
| **GLM** | Anthropic-Compatible | custom endpoint |
| **MiniMax** | Anthropic-Compatible | custom endpoint |
| **OpenAI** | OpenAI-Compatible | api.openai.com |
| **DeepSeek** | OpenAI-Compatible | api.deepseek.com |
| **Codex** | Subscription bridge | 本地订阅 |
| **Copilot** | OAuth | GitHub OAuth |
| **Ollama** | OpenAI-Compatible | localhost:11434 |

---

## 权限系统

| 模式 | 行为 | 适用场景 |
|------|------|---------|
| **Default** | 写/执行前询问 | 日常开发 |
| **Auto** | 允许所有 | 沙箱环境 |
| **Plan Mode** | 阻止所有写操作 | 大型重构、评审优先 |

---

## 与 Hermes Agent 的对比

| 维度 | Hermes Agent | OpenHarness |
|------|-------------|-------------|
| **Stars** | 147k | 12.4k |
| **架构** | Python CLI + gateway | Python CLI + TUI |
| **Agent Loop** | ✅ | ✅ |
| **工具数** | 90+ | 43+ |
| **Skills** | ✅ | ✅ (兼容 anthropics) |
| **多 Agent 协调** | ✅ (ACP + delegate_task) | ✅ (Swarm + Mailbox) |
| **Subagent 通信** | ACP (单向) | Mailbox (文件队列) |
| **Workflow 持久化** | ❌ | ❌ |
| **Cron 调度** | ✅ | ✅ (CronCreate/List/Delete) |
| **多平台 Gateway** | ✅ (20+) | ✅ (Feishu/Slack/Discord/Telegram) |
| **Self-Evolution** | ✅ | ❌ |
| **沙箱隔离** | ✅ (7种环境) | ✅ (sandbox/) |

---

## 对 Ai-Harness 的价值

### OpenHarness 解决了什么

1. **多 Agent 协调基础设施** — Swarm system + Mailbox 消息队列
2. **Team 管理** — team_create/agent/send_message 工具链
3. **个人助手** — ohmo 运行在 Feishu/Slack/Discord/Telegram
4. **Skills 兼容** — 兼容 anthropics/skills 标准

### OpenHarness 没解决的

1. **Workflow 持久化** — 没有 Temporal 那样的持久化执行
2. **规范层** — 没有 OpenSpec 那样的 spec-driven 开发
3. **TDD 方法论** — 没有 Superpowers 那样的 TDD 流程
4. **Self-Evolution** — 没有 GEPA 那样的自动进化

---

## 整合到 MVP 架构

**OpenHarness 可以替代部分组件**：

| 原计划组件 | OpenHarness 替代方案 |
|-----------|---------------------|
| Hermes Agent (主控) | OpenHarness (已有 Agent Loop + delegate) |
| DeerFlow (Subagent) | OpenHarness Swarm (team + agent + send_message) |
| Hermes ACP (平台) | OpenHarness ohmo (Feishu/Slack/Discord/Telegram) |

**整合后架构**：

```
Human (Feishu/Slack/Discord/Telegram ← ohmo)
    │
    ▼
OpenHarness (主控 Agent)
    │
    ├── OpenSpec (规范层)
    ├── Superpowers (TDD 方法论)
    ├── CubeSandbox (沙箱执行) — 或 OpenHarness sandbox/
    └── Temporal (Workflow 持久化)
```

---

## 关键发现

1. **OpenHarness 的 Swarm + Mailbox 是最完整的开源多 Agent 协调实现**
   - 比 Hermes ACP 更面向多 Agent（Mailbox 消息队列 vs ACP 单向控制）
   - 比 DeerFlow subagent 更结构化（Team 抽象 + 多种 Backend）

2. **ohmo 个人助手已经支持 Feishu** — 直接可以用来做 Human-in-the-loop 交互

3. **兼容 anthropics/skills** — Skills 系统可以直接复用

4. **缺少的部分**：
   - Workflow 持久化（Temporal 可以补全）
   - 规范层（OpenSpec 可以补全）
   - TDD 方法论（Superpowers 可以补全）
