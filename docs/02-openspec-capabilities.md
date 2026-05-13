# OpenSpec 能力分析

## 回答用户三个问题

### 1. 会拆任务么？

**会**。`tasks.md` 是核心 artifact 之一，按层级编号组织：

```markdown
## 1. Theme Infrastructure
- [ ] 1.1 Create ThemeContext with light/dark state
- [ ] 1.2 Add CSS custom properties for colors
- [ ] 1.3 Implement localStorage persistence

## 2. UI Components
- [ ] 2.1 Create ThemeToggle component
...
```

`/opsx:apply` 逐个执行并标记 checkbox `[x]`。

---

### 2. 具备调度功能么？

**没有**。OpenSpec 是**纯规范层（spec layer）**，不是任务队列或调度系统。

- ❌ 不运行后台进程
- ❌ 不主动触发任何操作
- ❌ 没有定时任务
- ❌ 所有动作由**人工 slash command 或 CLI 触发**

AI 只是执行者，不是调度者。

---

### 3. 多 Agent 协作呢？

**支持有限，主要靠人工桥接**。

#### 支持的部分
- **25+ AI 工具**：Claude Code、Cursor、Windsurf、Copilot 等
- **跨仓库规划**：Workspaces（协调层）
- **变更冲突检测**：`/opsx:bulk-archive` 会检测 spec 冲突

#### 协作模型 — 人工串行

```
人类 在 Claude Code  → /opsx:propose 创建规范
人类 切换到 Cursor   → 读取同一个 openspec/ 目录，继续 /opsx:apply
人类 切换到 Copilot → ...
```

**没有真正的 Agent 间通信协议**。每个 AI 工具独立运行，人工负责"交接棒"。

---

## 能力矩阵

| 能力 | OpenSpec | 实际状态 |
|------|----------|----------|
| 任务拆分 | ✅ | tasks.md 层级清单 |
| 主动调度 | ❌ | 纯规范层，不运行后台 |
| 多Agent协调 | ⚠️ 有限 | 靠人工在工具间切换，无消息传递 |
| 跨仓库规范 | ✅ | Workspaces（规划层） |

## 缺失能力总结

OpenSpec 的定位是**"人工 + AI 的需求协议层"**，解决的是"AI 编程需求模糊、结果不可预测"的问题。

但它缺少：

1. **任务调度**：无法定时自动触发任务
2. **Agent 间通信**：Agent 之间无法直接消息传递
3. **自动化流水线**：没有 CI/CD 式的自动执行链
4. **状态持久化**：任务状态、进度需要人工维护

这些缺失正是 **Ai-Harness** 需要补全的方向。
