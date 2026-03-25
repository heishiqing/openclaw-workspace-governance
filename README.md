<p align="center">
  <img src="https://api.iconify.design/fluent-emoji:brain.svg" width="120" height="120" alt="Brain Icon" />
</p>

<h1 align="center">OpenClaw Workspace Governance</h1>

<p align="center">
  <strong>A System-Level Governance Framework for Complex OpenClaw Workspaces</strong><br/>
  <em>面向复杂 OpenClaw 工作区的系统级治理 Skill</em>
</p>

<p align="center">
  <a href="#-简体中文">🇨🇳 简体中文</a> &nbsp;·&nbsp; <a href="#-english">🇬🇧 English</a>
</p>

---

<h2 id="-简体中文">🇨🇳 简体中文</h2>

> 🚨 **高危预警** — 这不是普通工具 Skill，而是系统级架构升级。部署前务必备份工作区！

### 这是什么

面向**复杂 OpenClaw 工作区**的治理型 Skill。在复杂度已经上来之后，帮你压住漂移、明确当前事实层、缩小 live docs 面积，让语义检索更可控。

### 核心能力

- **多 Agent 协作架构** — 以 Supervisor 为中心的调度机制，写审分离，防止单一 Agent 越权闭环
- **分层记忆系统** — 从原始日志到主题卡片再到长期记忆，逐层蒸馏与质量门控
- **语义检索与路由** — 查询分类（状态/规则/历史等），基于桥接文件、结构化记忆与 qmd 的精准召回优先级
- **审批与安全治理** — 核心文件修改必须经过独立评审与人工授权，内置防偷懒协议
- **保鲜纪律** — working set + 分组阈值 + 自动化检查
- **收口标准** — landed / improving / deferred，主动关闭阶段

### 适用场景

- 多个 Agent 或多角色执行路径已出现
- 文档、记忆、状态层越来越多
- 回答"现在到底什么是真的"开始变难
- semantic search 有用但不再能无脑相信
- live docs 过多、working set 不清、维护失控

⚠️ 进阶 Skill，不是新手入门包。

### 包结构

```
SKILL.md                          # 主指令文档（中文）
references/
  multi-agent-governance.md       # 角色分离与路由
  source-of-truth-layering.md     # 层级优先级
  query-class-routing.md          # 查询分类与检索顺序
  live-vs-historical-docs.md      # 文档状态标记
  freshness-discipline.md         # 保鲜纪律与阈值
  semantic-search-diagnostics.md  # 语义搜索诊断
  completion-criteria.md          # 阶段收口标准
scripts/
  freshness-check.py              # 保鲜检查器
  semantic-runtime-check.sh       # 语义运行时诊断
  doc-status-scan.py              # 文档状态扫描器
assets/examples/
  working-set.example.json        # 保鲜配置模板
  working-set.example.md          # 工作集文档模板
  current-health.example.json     # 健康状态模板
  system-state-index.example.md   # 系统状态索引模板
```

### 安装

```bash
# 通过 ClawHub
clawhub install openclaw-workspace-governance

# 或直接克隆
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

---

<h2 id="-english">🇬🇧 English</h2>

> 🚨 **WARNING** — This is NOT a standard tool skill. It is a system-level architecture upgrade. Back up your workspace before deployment!

### What Is This

A governance skill for **complex OpenClaw workspaces**. It helps you reduce drift, clarify current truth, shrink the live-doc surface, and make semantic retrieval more controllable.

### Core Capabilities

- **Multi-Agent Architecture** — Supervisor-centric dispatch, separation of writing and reviewing, preventing single-agent unauthorized closed loops
- **Layered Memory System** — Progressive distillation from daily notes → topic cards → long-term memory with quality gating
- **Semantic Retrieval Routing** — Query classification (state/rules/history etc.) with precise recall priorities across bridge files, structured memory, and qmd
- **Approval & Governance** — Core file modifications require independent review and human authorization, with built-in anti-laziness protocols
- **Freshness Discipline** — Working set + grouped thresholds + automated checks
- **Completion Criteria** — Landed / improving / deferred — close phases on purpose

### When to Use

- Multiple agents or role-based execution paths already exist
- Growing docs / memory / state layers
- "What is true now?" is getting hard to answer
- Semantic search is useful but not uniformly trustworthy
- Too many live docs, no clear working set, maintenance out of control

⚠️ Advanced skill, not a beginner starter pack.

### Package Structure

```
SKILL.md                          # Main skill instructions (Chinese)
references/
  multi-agent-governance.md       # Role separation & routing
  source-of-truth-layering.md     # Layer precedence
  query-class-routing.md          # Query classification & retrieval order
  live-vs-historical-docs.md      # Doc status labeling
  freshness-discipline.md         # Working set & thresholds
  semantic-search-diagnostics.md  # Backend-aware diagnostics
  completion-criteria.md          # Phase closure discipline
scripts/
  freshness-check.py              # Working set freshness checker
  semantic-runtime-check.sh       # Semantic runtime diagnostic
  doc-status-scan.py              # Doc status label scanner
assets/examples/
  working-set.example.json        # Freshness config template
  working-set.example.md          # Working set documentation template
  current-health.example.json     # Health state template
  system-state-index.example.md   # System state index template
```

### Install

```bash
# Via ClawHub
clawhub install openclaw-workspace-governance

# Or clone directly
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

---

## License

MIT
