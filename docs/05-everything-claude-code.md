# Everything Claude Code (ECC) 调研

## 项目信息

- **GitHub**: https://github.com/affaan-m/everything-claude-code
- **Stars**: 181k
- **Forks**: 21k
- **贡献者**: 170+
- **认证**: Anthropic Hackathon Winner
- **最新版本**: v2.0.0-rc.1

---

## 项目定位

> The performance optimization system for AI agent harnesses.

ECC 是一个**跨 Agent 平台的配置包 + 规则集 + 工具链**，不是独立的执行平台。

支持：Claude Code, Codex, Cursor, OpenCode, Gemini, Kiro, Qwen, Trae 等 10+ 主流 Agent IDE。

---

## 核心能力矩阵

| 能力 | 数量/说明 |
|------|-----------|
| **Agents** | 60 个专业化 subagent |
| **Skills** | 228 个工作流定义 |
| **Commands** | 75 个维护中的 slash 命令 |
| **Legacy shims** | 75+ 退役命令兼容层 |
| **Rules** | 34 套（common + 10 语言） |
| **Hook 事件** | 15 种 |
| **Hook 脚本** | 20+ 个 Node.js 脚本 |
| **语言生态** | TypeScript, Python, Go, Rust, Java, Kotlin, Swift, PHP, Perl, C++, HarmonyOS |

---

## 核心能力详解

### 1. Multi-Agent 编排

ECC 提供完整的多 Agent 协作命令：

```bash
/multi-plan        # 多 Agent 协作规划
/multi-execute     # 编排执行
/multi-backend     # 后端多服务编排
/multi-frontend    # 前端多服务编排
/multi-workflow    # 通用多服务工作流
```

**注意**：这些命令需要安装 `ccg-workflow` 运行时：
```bash
npx ccg-workflow
```
该运行时提供：
- `~/.claude/bin/codeagent-wrapper`
- `~/.claude/.ccg/prompts/*`

---

### 2. Continuous Learning v2（Instincts 系统）

ECC 实现了类 Darwin Skill 的自主学习机制：

| 命令 | 功能 |
|------|------|
| `/instinct-status` | 查看学到的 instincts（含置信度） |
| `/instinct-import` | 导入他人 instincts |
| `/instinct-export` | 导出 instincts 分享 |
| `/evolve` | 聚类 instincts 成正式 skills |
| `/prune` | 删除 30 天过期的 instincts |

**Instinct 格式**：
- 从 session 中自动提取模式
- 含置信度评分
- 可进化学为正式 SKILL.md

**与 Darwin Skill 的区别**：
- Darwin Skill：评估 → 改进 → 实测验证
- ECC Instincts：从 session 自动提取 → 置信度评分 → 进化成 skill

---

### 3. AgentShield 安全审计

ECC 内置安全审计工具（Anthropic Hackathon 获奖项目）：

```bash
npx ecc-agentshield scan        # 快速扫描
npx ecc-agentshield scan --fix  # 自动修复安全 issue
npx ecc-agentshield scan --opus # 三个 Opus 4.6 agent 红蓝对抗
npx ecc-agentshield init        # 从零生成安全配置
```

**扫描范围**：
- Secrets 检测（14 种模式）
- 权限审计
- Hook 注入分析
- MCP 服务器风险画像
- Agent 配置审查

**数据**：1282 tests, 98% coverage, 102 rules

---

### 4. Skill Creator

从 git 历史自动生成 SKILL.md：

```bash
/skill-create                    # 分析当前仓库
/skill-create --instincts       # 同时生成 instincts
```

也支持 GitHub App（高级功能）：
- 分析任意 issue
- push 触发自动分析
- 10k+ commits 支持

---

### 5. Cross-Harness 架构

ECC 2.0 的 Rust 控制平面：

```bash
ecc status --markdown --write status.md    # 生成状态快照
ecc work-items upsert <item>              # 手动添加工作项
ecc work-items sync-github --repo owner/repo  # 同步 GitHub PR/issue
ecc sessions                               # 会话管理
ecc status --exit-code                     #  readiness 失败时自动化退出码
```

**工作项来源**：Linear / GitHub / 手动

---

### 6. 跨平台支持

| IDE | Agents | Commands | Skills | Hooks | Rules |
|-----|:------:|:--------:|:------:|:-----:|:-----:|
| Claude Code | 60 | 75 | 228 | 8 | 34 |
| Cursor | 48 | Shared | Shared | 15 | 34 |
| Codex | Shared | Instruction-based | 32 | None | Instruction-based |
| OpenCode | 12 | 35 | 37 | 11 | 13 |

**DRY Adapter 模式**：Cursor 的 hook adapter 将 stdin JSON 转换为 Claude Code 格式，复用 `scripts/hooks/*.js`。

---

### 7. Token 优化

ECC 提供 token 消耗优化指南：

```json
// ~/.claude/settings.json
{
  "model": "sonnet",  // 默认用 sonnet，省 60%
  "env": {
    "MAX_THINKING_TOKENS": "10000",  // 减少 70% hidden thinking
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"  // 更早压缩
  }
}
```

**原则**：
- 默认 sonnet，复杂架构/调试才用 opus
- 用 `/clear` 在无关任务间重置（免费）
- 用 `/compact` 在逻辑断点手动压缩

---

### 8. 规则系统

```
rules/
├── common/           # 语言无关原则（always install）
├── typescript/      # TS/JS
├── python/          # Python
├── golang/          # Go
├── swift/           # Swift
├── php/             # PHP
├── arkts/           # HarmonyOS / ArkTS
└── cpp/             # C++
```

每套规则包含：coding-style, git-workflow, testing, performance, patterns, hooks, agents, security。

---

## Dashboard GUI

```bash
npm run dashboard
# 或
python3 ./ecc_dashboard.py
```

功能：
- Tab 界面：Agents, Skills, Commands, Rules, Settings
- Dark/Light 主题切换
- 字体定制
- 项目 logo
- 跨组件搜索和过滤

---

## 局限性

ECC 本质是**配置包 + 规则集 + 工作流定义**，不是执行平台。

### 已解决
| 能力 | 状态 |
|------|------|
| 多 Agent 协作 | ✅ 手动编排 |
| 持久记忆 | ✅ Instincts 系统 |
| 安全审计 | ✅ AgentShield |
| TDD/代码质量 | ✅ Rules + Hooks |
| 跨平台抽象 | ✅ 10+ 平台 |

### 未解决（缺口）
| 能力 | 状态 |
|------|------|
| 任务调度 | ❌ 无 |
| Agent↔Agent 协议 | ❌ 无 |
| 流水线状态机 | ❌ 无 |
| 沙箱执行 | ❌ 依赖宿主 IDE |
| 规范层（Spec-Driven） | ❌ 无 |

---

## 与其他项目对比

| 维度 | Superpowers | DeerFlow | CubeSandbox | ECC | Ai-Harness 目标 |
|------|:-----------:|:--------:|:-----------:|:---:|:---------------:|
| **定位** | 方法论 | 执行平台 | 沙箱服务 | 配置包 | 协作基础设施 |
| **多 Agent** | ✅ | ❌ | ❌ | ✅ | ✅ |
| **调度** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **协议通信** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **状态机** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **沙箱** | ❌ | ✅ | ✅ | ❌ | ✅ |
| **持久记忆** | ❌ | ✅ | ❌ | ✅ | ✅ |
| **安全** | ❌ | ❌ | ❌ | ✅ | 待定 |
| **规范层** | ❌ | ❌ | ❌ | ❌ | ✅ |

> ⚠️ DeerFlow 的"多 Agent"是 Subagent 系统，不是真正的多 Agent 协作。主控 Agent 已选型 OpenHarness。

---

## 关键启示

### ECC 可借鉴
1. **Instincts 系统** — 从 session 自动提取模式，置信度评分，进化成 skill
2. **AgentShield** — 红蓝对抗审计模式
3. **Cross-Harness 抽象** — 统一接口抽象多 IDE
4. **Dashboard GUI** — 可视化组件管理

### Ai-Harness 差异点
ECC 是"给 Agent 装配件"，Ai-Harness 是"让 Agent 协作"。

ECC 不解决：
- Agent 之间如何通信
- 任务如何定时触发
- 工作流如何持久化状态

这些正是 Ai-Harness 要填补的空白。
