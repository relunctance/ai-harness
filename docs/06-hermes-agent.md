# Hermes Agent 调研

## 项目信息

- **GitHub**: https://github.com/NousResearch/hermes-agent
- **Stars**: 147k
- **Forks**: 23.1k
- **Commits**: 8,181
- **Branches**: 1,005
- **Issues**: 3.7k
- **PRs**: 5k+
- **开发者**: Nous Research（teknium1, amathxbt 等）
- **定位**: "The agent that grows with you" — 自改进 AI Agent
- **官网**: https://hermes-agent.nousresearch.com

---

## 核心定位

> The self-improving AI agent built by Nous Research.

Hermes 是**唯一一个内置自改进学习循环的 Agent**：
- 从经验中创建 skills
- 使用中自我改进
- 主动提醒持久化知识
- 搜索历史对话
- 跨 session 建立用户模型

---

## 核心能力矩阵

| 能力 | 说明 |
|------|------|
| **自改进学习** | Agent-curated memory, autonomous skill creation, skills self-improve during use |
| **FTS5 搜索** | 跨 session 全文搜索 + LLM 摘要 |
| **多平台网关** | Telegram, Discord, Slack, WhatsApp, Signal, Email, CLI |
| **Cron 调度** | 内置调度器，自然语言定义，任意平台投递 |
| **任意模型** | OpenRouter (200+), Nous Portal, OpenAI, Anthropic, HuggingFace 等 |
| **多运行环境** | Local, Docker, SSH, Singularity, Modal, Daytona, Vercel Sandbox |
| **Subagent 并行** | 孤立 subagent 并行工作流 |
| **MCP 集成** | 连接任意 MCP server |
| **Research 工具** | Batch trajectory generation, Atropos RL, trajectory compression |

---

## 架构详解

### 项目结构

```
hermes-agent/
├── run_agent.py          # AIAgent class — 核心对话循环 (~12k LOC)
├── cli.py                # HermesCLI — 交互 CLI (~11k LOC)
├── hermes_state.py       # SessionDB — SQLite + FTS5 session 存储
├── model_tools.py        # 工具编排，tool 发现和调用
├── toolsets.py           # Toolset 定义
├── batch_runner.py       # 并行批处理
├── cron/
│   ├── scheduler.py      # Cron 调度器 (~1800 LOC)
│   └── jobs.py           # 任务定义
├── agent/                # Agent 内部模块（provider adapters, memory, caching 等）
├── hermes_cli/           # CLI 子命令，setup 向导，plugins 加载
├── tools/
│   └── environments/     # 终端后端（local, docker, ssh, modal, daytona, singularity）
├── gateway/              # 消息网关
│   └── platforms/        # 适配器：telegram, discord, slack, whatsapp, signal,
│                         #   homeassistant, matrix, email, dingtalk, wecom, weixin,
│                         #   feishu, qqbot, bluebubbles, yuanbao, webhook, api_server...
├── skills/               # 内置 skills（27 个分类）
├── plugins/              # 插件系统
│   ├── memory/           # 内存提供者（honcho, mem0, supermemory...）
│   ├── context_engine/   # 上下文引擎插件
│   ├── model-providers/  # 推理后端插件
│   ├── kanban/           # 多 Agent board dispatcher + worker
│   ├── observability/    # 可观测性
│   └── image_gen/       # 图像生成
├── acp_adapter/          # ACP server（VS Code / Zed / JetBrains 集成）
├── acp_registry/         # ACP 注册表
├── tinker-atropos/       # RL 训练环境
└── environments/        # Atropos RL 环境
```

### Agent 循环（run_agent.py）

```python
while (api_call_count < self.max_iterations and self.iteration_budget.remaining > 0) \
        or self._budget_grace_call:
    if self._interrupt_requested: break
    response = client.chat.completions.create(model=model, messages=messages, tools=tool_schemas)
    if response.tool_calls:
        for tool_call in response.tool_calls:
            result = handle_function_call(tool_call.name, tool_call.args, task_id)
            messages.append(tool_result_message(result))
        api_call_count += 1
    else:
        return response.content
```

### Cron 调度器（scheduler.py）

```python
"""
Cron job scheduler - executes due jobs.
Provides tick() which checks for due jobs and runs them.
The gateway calls this every 60 seconds from a background thread.
Uses a file-based lock (~/.hermes/cron/.tick.lock)
so only one tick runs at a time if multiple processes overlap.
"""
```

- **Tick 间隔**：60 秒
- **锁机制**：文件锁防止重复执行
- **自然语言任务**：`"每天早上9点给我发日报"`
- **平台投递**：结果可发送到任意平台

### 消息网关（gateway/）

支持 20+ 平台适配器：
```
telegram, discord, slack, whatsapp, signal,
homeassistant, matrix, mattermost, email, sms,
dingtalk, wecom, weixin, feishu, qqbot,
bluebubbles, yuanbao, webhook, api_server...
```

**单一 gateway 进程**驱动所有平台。

### Skills 系统（skills/）

27 个分类，涵盖：
```
autonomous-ai-agents, creative, data-science, devops,
diagramming, dogfood, domain, email, gaming, gifs,
github, inference-sh, mcp, media, mlops, note-taking,
productivity, red-teaming, research, smart-home,
social-media, software-development, yuanbao...
```

software-development 下含：
```
debugging-hermes-tui-commands
hermes-agent-skill-authoring
node-inspect-debugger
plan
python-debugpy
requesting-code-review
spike
subagent-driven-development
systematic-debugging
test-driven-development
writing-plans
```

### 自改进学习循环

**三种记忆机制**：
1. **Session Memory** — SQLite + FTS5，跨 session 全文搜索
2. **Skill Memory** — 从经验中自主创建 skills
3. **User Modeling** — Honcho dialectic 用户建模

**关键文件**：
- `hermes_state.py` — SessionDB，FTS5 搜索
- `agent/` — memory, caching, compression 模块

### 运行环境（tools/environments/）

| 后端 | 特点 |
|------|------|
| **Local** | 直接本地执行 |
| **Docker** | 容器隔离 |
| **SSH** | 远程执行 |
| **Singularity** | HPC 环境 |
| **Modal** | Serverless，按需唤醒，接近零空闲成本 |
| **Daytona** | Serverless，休眠/唤醒 |
| **Vercel Sandbox** | Serverless |

### 插件系统（plugins/）

```
plugins/
├── memory/              # honcho, mem0, supermemory...
├── context_engine/     # 上下文引擎
├── model-providers/    # 推理后端（openrouter, anthropic, gmi...）
├── kanban/             # 多 Agent board dispatcher + worker
├── observability/      # 可观测性
├── image_gen/          # 图像生成
└── <others>/           # disk-cleanup, google_meet, spotify...
```

---

## 能力对比

| 维度 | Hermes | Superpowers | DeerFlow | ECC | Ai-Harness 目标 |
|------|:------:|:-----------:|:--------:|:---:|:---------------:|
| **自改进学习** | ✅ | ❌ | ❌ | ✅ (instincts) | ✅ |
| **Cron 调度** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **多平台网关** | ✅ (20+) | ❌ | ✅ (IM) | ❌ | ✅ |
| **多 Agent 协作** | ⚠️ | ✅ | ❌ | ✅ | ✅ |
| **Subagent** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **任意模型** | ✅ (200+) | ❌ | ❌ | ❌ | ✅ |
| **多运行环境** | ✅ (7种) | ❌ | ✅ | ❌ | ✅ |
| **FTS5 搜索** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Workflow 状态机** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **规范层** | ❌ | ❌ | ❌ | ❌ | ✅ |

> ⚠️ DeerFlow 的"多 Agent"是 Subagent 系统，不是真正的多 Agent 协作。主控 Agent 已选型 OpenHarness。

---

## 关键启示

### Hermes 做得最好的

1. **自改进学习循环** — 不是被动的知识检索，是主动的 skill 创建和优化
2. **Cron 调度 + 自然语言** — `hermes cron "每天早上9点发日报"` 即可
3. **多平台统一网关** — 20+ 平台一个进程
4. **FTS5 跨 session 搜索** — 对话历史不只是记忆，是可搜索的知识库
5. **Serverless 运行环境** — Modal/Daytona 接近零空闲成本

### Hermes 的局限性

1. **没有 Workflow 状态机** — subagent 是并行的，但没有条件分支/循环/人工审批
2. **没有规范层** — 没有类似 OpenSpec 的 Spec-Driven 开发
3. **不是多 Agent 协作平台** — 虽然有 subagent 和 kanban 插件，但本质还是"一个 Agent 带多个 worker"

### 对 Ai-Harness 的借鉴

| Hermes 特性 | Ai-Harness 借鉴方式 |
|------------|---------------------|
| Cron scheduler | 内置任务调度（Hermes 的 scheduler.py 只有 ~1800 LOC） |
| FTS5 session search | 跨 session 知识检索 |
| 自改进 skills | 从执行历史中自动优化 workflow |
| 多平台 gateway | 统一消息总线 |
| 多运行环境 | 抽象 Sandbox 接口（主要用 CubeSandbox） |
