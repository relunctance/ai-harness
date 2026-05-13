# Hermes Agent Self-Evolution 调研

## 项目信息

- **GitHub**: https://github.com/NousResearch/hermes-agent-self-evolution
- **Stars**: 3.1k
- **Forks**: 334
- **Commits**: 7
- **开发者**: Nous Research（teknium1）
- **定位**: Evolutionary self-improvement for Hermes Agent
- **核心技术**: DSPy + GEPA (Genetic-Pareto Prompt Evolution)
- **License**: MIT
- **认证**: ICLR 2026 Oral

---

## 核心定位

> Evolutionary self-improvement for Hermes Agent — optimize skills, prompts, and code using DSPy + GEPA.

Hermes Agent Self-Evolution 是一个**独立的优化 pipeline**，运行在 hermes-agent 之上，通过**反射式进化搜索**自动改进 Agent 的 skills、tool descriptions、system prompts 和 code。

**关键特性**：
- **No GPU training required** — 全部通过 API 调用操作（变异文本、评估结果、选择最优变体）
- **~$2-10 per optimization run** — 极低优化成本
- **Reads execution traces** — 理解"为什么"失败，而不只是"失败了"

---

## 核心架构

### 优化循环

```
┌─────────────────────────────────────────────┐
│  1. SELECT TARGET                           │
│     - Pick a skill, prompt section, or tool  │
│     - Load current version as baseline       │
│                                             │
│  2. BUILD EVALUATION DATASET                │
│     - Mine session_db for real usage        │
│     - Or use hand-crafted test cases        │
│     - Split: train / validation / test      │
│                                             │
│  3. WRAP AS DSPy MODULE                    │
│     - Skill text → dspy.Signature           │
│     - Agent workflow → dspy.ReAct           │
│     - Tool selection → dspy.Predict         │
│                                             │
│  4. RUN OPTIMIZER                          │
│     - Primary: dspy.GEPA (reflective)      │
│     - Fallback: dspy.MIPROv2 (bayesian)    │
│     - Code: Darwinian Evolver (external)    │
│                                             │
│  5. EVALUATE & COMPARE                     │
│     - Run on held-out test set              │
│     - Compare: accuracy, cost, latency      │
│                                             │
│  6. DEPLOY (with approval)                 │
│     - Git branch + PR with diff + metrics   │
│     - Human review & merge                  │
└─────────────────────────────────────────────┘
```

### 数据流

```
SessionDB (real conversations)
    │
    ▼
Evaluation Dataset Builder
    │
    ├──► DSPy Module Wrapper
    │        │
    │        ▼
    │    GEPA Optimizer ◄── Execution Traces (from batch_runner)
    │        │                    ▲
    │        ▼                    │
    │    Candidate Variants ──► batch_runner (parallel eval)
    │        │
    │        ├──► Constraint Validation (tests, char limits)
    │        │
    │        ▼
    │    Best Valid Variant
    │        │
    ▼        ▼
Git Branch + PR (with diff, metrics, before/after)
    │
    ▼
Human Review & Merge
```

---

## 三种优化引擎

| Engine | What It Optimizes | License | Notes |
|--------|------------------|---------|-------|
| **DSPy + GEPA** | Skills, prompts, instructions, tool descriptions | MIT | Primary engine. Reads traces to understand WHY failures happen. ICLR 2026 Oral. |
| **DSPy MIPROv2** | Few-shot examples, instruction text | MIT | Fallback optimizer (bayesian) |
| **Darwinian Evolver** | Code files, algorithms, tool implementations | AGPL v3 | External CLI only |

**GEPA 优势**：
- Works with as few as **3 examples**
- Reads execution traces to understand **WHY** failures happen
- Outperforms both RL and previous DSPy optimizers

---

## 五个优化阶段

| Phase | Target | Engine | Status |
|-------|--------|--------|--------|
| **Phase 1** | Skill files (SKILL.md) | DSPy + GEPA | ✅ Implemented |
| **Phase 2** | Tool descriptions | DSPy + GEPA | 🔲 Planned |
| **Phase 3** | System prompt sections | DSPy + GEPA | 🔲 Planned |
| **Phase 4** | Tool implementation code | Darwinian Evolver | 🔲 Planned |
| **Phase 5** | Continuous improvement loop | Automated pipeline | 🔲 Planned |

### Phase 1: Skill 文件（最高价值，最低风险）

- **What**: SKILL.md 文件 — Agent 遵循的程序性指令
- **How**: 将 skill 文本包装为 DSPy module，通过 batch_runner 评估，用 GEPA 进化
- **Why it works**: Skills 是纯文本，易于变异，可直接测量（Agent 遵循此 skill 时是否正确完成任务？）

### Phase 4: Code 进化（最高价值，最高风险）

- **What**: Tool 实现代码、辅助函数
- **How**: Darwinian Evolver + GitBasedOrganism，通过 pytest + batch_runner 测试
- **Risk**: 代码变更可能破坏功能 — 需要强测试套件作为 guardrails

---

## Guardrails（护栏）

每个进化变体必须通过：

1. **Full test suite** — `pytest tests/ -q` 必须 100% 通过
2. **Size limits** — Skills ≤15KB，tool descriptions ≤500 chars
3. **Caching compatibility** — 不能在对话中间改变
4. **Semantic preservation** — 不能偏离原始目的
5. **PR review** — 所有变更必须经过人工 review，从不直接 commit

---

## 与 Hermes Agent 的关系

```
hermes-agent-self-evolution/  (独立 repo)
    │
    ├── evolution/             # 主要 package
    │   ├── core/             # 共享基础设施
    │   │   ├── dataset_builder.py    # Eval dataset 生成
    │   │   ├── fitness.py           # Fitness functions (LLM-as-judge)
    │   │   ├── constraints.py       # Constraint validators
    │   │   ├── benchmark_gate.py     # Benchmark gating
    │   │   └── pr_builder.py        # 自动生成 PR
    │   ├── skills/            # Phase 1: Skill 进化
    │   ├── tools/              # Phase 2: Tool description 进化
    │   ├── prompts/            # Phase 3: System prompt 进化
    │   ├── code/               # Phase 4: Code 进化
    │   └── monitor/             # Phase 5: Continuous loop
    ├── datasets/               # 生成的 eval datasets
    └── tests/                  # 测试套件
```

**关键**：`hermes-agent-self-evolution` operates ON `hermes-agent`，不是 part of it。

| Hermes Component | Role |
|-----------------|------|
| `batch_runner.py` | Evaluation harness — parallel test runs |
| `agent/trajectory.py` | Collect execution traces for GEPA |
| `hermes_state.py` (SessionDB) | Mine real usage for eval datasets |
| `skills/` | Primary optimization targets |
| `tools/registry.py` | Tool descriptions to optimize |
| `agent/prompt_builder.py` | System prompt components |
| `tests/` | Guardrails |

---

## 快速开始

```bash
# Install
git clone https://github.com/NousResearch/hermes-agent-self-evolution.git
cd hermes-agent-self-evolution
pip install -e ".[dev]"

# Point at hermes-agent repo
export HERMES_AGENT_REPO=~/.hermes/hermes-agent

# Evolve a skill (synthetic eval data)
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source synthetic

# Or use real session history
python -m evolution.skills.evolve_skill \
    --skill github-code-review \
    --iterations 10 \
    --eval-source sessiondb
```

---

## 与 Darwin Skill 的对比

| 维度 | Darwin Skill | Hermes Self-Evolution |
|------|-------------|---------------------|
| **目标** | 改进任何 LLM 应用 | 专精 Hermes Agent |
| **引擎** | 评估 → 改进 → 实测验证 | DSPy + GEPA |
| **进化方式** | 反射式循环 | Genetic-Pareto Prompt Evolution |
| **数据来源** | 自定义测试集 | SessionDB 真实数据 |
| **Guardrails** | 用户自定义 | 强制测试套件 + char limits + PR review |
| **Phase** | 单阶段 | 5 阶段递进 |
| **成本** | 取决于 LLM 调用 | ~$2-10 per run |
| **License** | AGPL v3 | MIT |

**关键区别**：
- Darwin Skill 是**通用的**评估优化框架
- Hermes Self-Evolution 是**专用的**优化 pipeline，深度集成 Hermes 基础设施
- Hermes Self-Evolution 有更系统的 Guardrails（pytest + char limits + semantic preservation + PR review）

---

## 对 Ai-Harness 的借鉴

### 最值得借鉴的点

1. **GEPA 反射式进化** — 理解"为什么"失败，不只是"失败了"
2. **SessionDB 作为真实数据源** — 从真实使用中提取 eval 数据
3. **Phase 递进策略** — 每个 phase 必须证明自己才进入下一个
4. **系统性 Guardrails** — 测试套件 + char limits + semantic preservation + PR review

### Ai-Harness 如何整合

```python
# 概念：Ai-Harness 的自我改进循环
class AiHarnessSelfImprovement:
    """
    结合 Hermes Self-Evolution 的 GEPA 进化
    与 Darwin Skill 的评估框架
    """
    
    def __init__(self, harness):
        self.harness = harness
        self.session_db = SessionDB()
        self.evaluator = DarwinEvaluator()
        self.optimizer = GEPASelfImprover()
    
    def learn_from_task(self, task, result):
        # 1. 从任务执行中提取模式
        # 2. 用 Darwin 评估器评估结果
        # 3. 用 GEPA 优化 skill/prompt
        # 4. 通过 guardrails 后部署
```

### 核心启示

Hermes Self-Evolution 证明了：
1. **Skill 优化是可行的** — 不需要 GPU，纯 API 调用
2. **真实数据很重要** — SessionDB >>> 合成数据
3. **自动化 + 人工 review** — 自动化变异，human-in-the-loop 审批
4. **渐进式 Phase** — 每个阶段建立在上一个阶段的基础上
