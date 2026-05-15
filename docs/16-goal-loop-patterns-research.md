# AI Agent 目标与循环模式调研

> 调研日期：2026-05-15
> 来源：Claude Code / Hermes Agent / Karpathy autoresearch / X.com 社区讨论

---

## 一、Claude Code `/goal` 目标系统

**文档**：[Keep Claude working toward a goal](https://code.claude.com/docs/en/goal)

### 核心机制

`/goal` 命令设置一个完成条件，Claude 会跨回合持续工作直到条件满足。

```
用户输入: /goal <condition>
     ↓
Claude 执行一轮
     ↓
小型快速模型（Judge）检查条件是否满足
     ↓
满足 → 目标清除，自动结束
不满足 → Claude 继续下一轮
```

### 关键设计

| 要素 | 说明 |
|------|------|
| **Judge 模型** | 每次回复后调用小型快速模型（如 Haiku）判断条件 |
| **判断依据** | 仅基于 Claude 已输出的内容，不运行命令或读取文件 |
| **条件写法** | 需要是 Claude 产出能证明的事情，如 "all tests pass" |
| **Turn 限制** | 可选 `stop after N turns` 防止无限循环 |
| **自动清除** | 条件满足后目标自动清除 |

### 与其他自主模式的对比

| 模式 | 下一轮何时开始 | 何时停止 |
|------|---------------|---------|
| `/goal` | 上一轮结束 | 模型确认条件满足 |
| `/loop` | 时间间隔到达 | 手动停止或 Claude 判断完成 |
| Stop Hook | 上一轮结束 | 自定义脚本/prompt 决定 |

**组合使用**：`/goal` + Auto Mode = 每轮自动批准工具调用 + 无需逐轮提示

### 条件有效写法

```bash
/goal all tests in test/auth pass and the lint step is clean
/goal CHANGELOG.md has an entry for every PR merged this week
/goal implement the design in DESIGN.md and stop after 20 turns
```

### 需求条件

1. **可测量的最终状态**：测试结果、构建退出码、文件数量
2. **明确的检查方式**：如 `npm test exits 0`
3. **约束条件**：如 "no other test file is modified"

---

## 二、Claude Code Hooks 系统

**文档**：[Hooks reference](https://code.claude.com/docs/en/hooks)

### 核心机制

Hooks 在 Claude Code 生命周期的特定点自动执行用户定义的逻辑。

```
Hook 事件触发 → 发送 JSON context (stdin 或 POST)
     ↓
匹配器检查 (Matcher)
     ↓
Handler 执行 (shell command / HTTP / MCP tool / LLM prompt / Agent)
     ↓
退出码决定结果
```

### Hook 类型

| Handler 类型 | 说明 |
|-------------|------|
| `command` | 运行 shell 命令，stdin 接收 JSON |
| `http` | 发送 HTTP POST 请求 |
| `mcp_tool` | 调用 MCP 服务器工具 |
| `prompt` | 发送 prompt 给 LLM 做单轮评估 |
| `agent` | 启动带工具访问的子 agent 做多轮验证 |

### 关键 Hook 事件

| 事件 | 触发时机 |
|------|---------|
| `SessionStart` | 每次会话开始 |
| `PreToolUse` | 每次工具调用前 |
| `PostToolUse` | 每次工具调用后 |
| `Stop` | Claude 响应结束后 |
| `StopFailure` | 因错误停止时 |
| `UserPromptSubmit` | 用户 prompt 提交时 |

### Stop Hook（循环控制核心）

```json
{
  "decision": "block",
  "reason": "Must pass tests before stopping"
}
```

返回 `decision: "block"` 阻止 Claude 停止，reason 作为继续的指导。

**退出码 2** = 阻止 Claude 停止，继续对话

### 异步 Hook

```json
{
  "async": true,
  "additionalContext": "background task result"
}
```

异步 hook 不能阻止，但可以在下一轮提供额外 context。

---

## 三、Hermes Agent Persistent Goals

**文档**：[Persistent Goals](https://hermes-agent.nousresearch.com/docs/user-guide/features/goals)

### 核心机制

直接受 Codex CLI `/goal` 启发，独立实现。给 Hermes 一个持久化的目标，跨会话持续工作。

```
用户输入: /goal <text>
     ↓
Claude 执行一轮
     ↓
Judge 模型检查（goal + 最后 4KB 回复）
     ↓
{"done": <bool>, "reason": "<一句话理由>"}
     ↓
done=true → 目标清除
done=false → 追加 continuation prompt，继续
```

### Judge 设计

| 要素 | 说明 |
|------|------|
| **Judge 调用** | 每次回复后调用 auxiliary 模型 |
| **输入** | 目标文本 + 最后 ~4KB 回复内容 |
| **输出** | 严格 JSON：`{"done": bool, "reason": "..."}` |
| **保守策略** | 仅在回复明确确认完成时才标记 done |
| **容错** | Judge 错误时视为 continue，不阻塞进度 |

### 持久化

- 目标状态存储在 `SessionDB.state_meta`，key = `goal:<session_id>`
- **跨会话**：关闭笔记本后明天可以继续
- Turn 预算：默认 20 轮续续（可配置）

### 命令

```bash
/goal <text>           # 设置/替换目标
/goal                  # 查看状态
/goal status           # 查看状态
/goal pause            # 暂停循环
/goal resume           # 恢复（重置 turn 计数器）
/goal clear            # 清除目标
```

### 安全机制

| 机制 | 说明 |
|------|------|
| **Mid-run 安全** | `/goal status/pause/clear` 在 agent 运行时安全执行 |
| **新目标拒绝** | Mid-run 时设置新目标会被拒绝，需先 `/stop` |
| **用户抢占** | 任何真实消息都会中断 continuation loop |
| **Resume 重置** | `/goal resume` 重置 turn 计数器 |

### 缓存效率

20 轮 goal 的缓存成本 = 20 轮普通对话。因为 continuation prompt 只是追加到历史，不影响 system prompt 或 cache。

---

## 四、Karpathy Autoresearch

**仓库**：[github.com/karpathy/autoresearch](https://github.com/karpathy/autoresearch)

### 核心机制

给 AI agent 一个小型但真实的 LLM 训练环境，让它通宵自主实验——修改代码、训练 5 分钟、检查改进、重复。

```
AI Agent 修改 train.py
     ↓
固定 5 分钟 wall-clock 预算
     ↓
评估指标: val_bpb (validation bits per byte，越低越好)
     ↓
有改进 → 接受修改
无改进 → 回退
     ↓
循环继续
```

### 架构

| 文件 | 作用 |
|------|------|
| `prepare.py` | 固定常量、数据准备、运行时工具 |
| `train.py` | agent 唯一修改的文件（GPT 模型、Muon+AdamW 优化器、训练循环）|
| `program.md` | AI agent 的基线指令 |

### 评估指标

```python
metric = "val_bpb"  # validation bits per byte
goal = "lower is better"
budget = "5 minutes wall-clock"
```

### 特点

1. **极简接口**：agent 只编辑一个文件 `train.py`
2. **固定预算**：5 分钟训练 + 评估，快速迭代
3. **单一指标**：val_bpb 简单直接
4. **自主循环**：agent 自己决定改什么、怎么改

---

## 五、Ralph Loop（社区实践）

### Geoffrey Huntley：Self-Healing Ralph Loop

**来源**：[X.com](https://x.com/GeoffreyHuntley/status/2012708172491030589)

> "a self healing ralph loop that detected authentication/security problems through putting the entire system under test where it automatically resolved the security issue, implemented missing functionality and automatically deployed it to production without ci via sudo then verified that the problem was resolved"

**关键特性**：
- 自动检测认证/安全问题
- 将整个系统置于测试下
- 自动解决安全问题
- 实现缺失功能
- 无需 CI 直接通过 sudo 部署到生产
- 验证问题已解决

### Trey Goff：Codex Hooks 上的 Ralph Loop

**来源**：[X.com](https://x.com/thetreygoff/status/2032120438650740950)

> "They added some basic hooks yesterday, so I ported the ralph loop over to codex and improved it a bit. prevent early exits, make the agents actually complete the task you assigned so you don't have to babysit them"

**GitHub**：`github.com/treygoff24/autonomous-loop`

**改进点**：
- 防止 agent 过早退出
- 确保 agent 真正完成分配的任务
- 不需要"盯着"agent 工作

### Ralph Loop 模式总结

```
设置目标
     ↓
Agent 执行
     ↓
检查点验证 (Hook)
     ↓
失败 → 自动修复 + 重试验证
     ↓
成功 → 部署 / 完成
     ↓
发现新问题 → 继续修复循环
```

---

## 六、Tobi Lütke：Context Engineering

**来源**：[X.com](https://x.com/tobi/status/1935533422589399127)

> 推文内容未能获取（X.com 访问受限）

**已知背景**：
- Tobi Lütke 是 Shopify CEO / Rails 作者
- 关注 AI Agent 的上下文管理
- "Context engineering" 强调如何组织 prompt 和上下文给 AI

**推测要点**（需验证）：
- 如何设计有效的 system prompt
- 上下文信息的组织方式
- 给 AI 清晰的结构化上下文 vs 自由发挥

---

## 七、模式对比与总结

### 循环/目标模式分类

| 类型 | 代表 | 评估方式 | 循环控制 |
|------|------|---------|---------|
| **Goal-Judge** | Claude Code `/goal`, Hermes `/goal` | 小型模型判断条件 | 自动继续直到满足 |
| **Hook-Block** | Claude Code Stop Hook | 自定义脚本/prompt | 阻止停止继续工作 |
| **Budget-Iter** | Karpathy autoresearch | 固定指标阈值 | 固定时间/迭代预算 |
| **Self-Heal** | Ralph Loop | 端到端测试验证 | 自动修复+重试 |

### 关键洞察

1. **Judge 模式优于硬编码条件**：Claude/Hermes 用小型模型判断，灵活且便宜
2. **Continuation Prompt 是关键**：追加 user-role message 比修改 system prompt 更高效
3. **Hook 是基础设施**：Codex/Claude 的 hook 系统是实现各种循环模式的基础
4. **安全机制必不可少**：Turn 预算、容错、用户抢占防止无限循环
5. **Self-Healing 是高级形态**：Ralph Loop 在真实场景验证 + 自动修复 + 部署

### 设计建议

1. **优先使用 Judge 模式**：设置目标 + 小模型判断，比硬编码条件更灵活
2. **Hook 是循环控制的核心**：Stop Hook 阻止停止，PreToolUse 验证行为
3. **必须有限制机制**：Turn 预算、时间限制、退出条件
4. **容错设计**：Judge 失败时继续而非阻塞
5. **用户可抢占**：任何输入都能中断自动循环

---

## 八、参考资料

| 来源 | 链接 |
|------|------|
| Claude Code Goals | https://code.claude.com/docs/en/goal |
| Claude Code Hooks | https://code.claude.com/docs/en/hooks |
| Hermes Persistent Goals | https://hermes-agent.nousresearch.com/docs/user-guide/features/goals |
| Karpathy Autoresearch | https://github.com/karpathy/autoresearch |
| Trey Goff Autonomous Loop | https://github.com/treygoff24/autonomous-loop |
| Geoffrey Huntley Ralph Loop | https://x.com/GeoffreyHuntley/status/2012708172491030589 |
| Trey Goff Codex Loop | https://x.com/thetreygoff/status/2032120438650740950 |
| Tobi Lütke Context Engineering | https://x.com/tobi/status/1935533422589399127 |
