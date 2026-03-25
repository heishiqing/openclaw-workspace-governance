# 虾滑 OpenClaw 工作区治理 — 详细指南

> 🚨 **高危预警** — 这不是普通工具 Skill，而是系统级架构升级。部署前务必备份工作区！

---

## 目录

- [这是什么](#这是什么)
- [系统能力详解](#系统能力详解)
- [适用场景](#适用场景)
- [非目标](#非目标)
- [安装](#安装)
- [快速开始](#快速开始)
- [核心工作流](#核心工作流)
- [脚本说明](#脚本说明)
- [参考文档索引](#参考文档索引)
- [示例模板索引](#示例模板索引)
- [FAQ](#faq)

---

## 这是什么

面向**复杂 OpenClaw 工作区**的治理型 Skill。它不是为了制造更多流程，而是为了在复杂度已经上来之后，帮你：

- 压住漂移（governance drift / doc drift / retrieval drift）
- 明确当前事实层（谁说了算）
- 缩小 live docs 面积（不是所有文档都需要保持活跃）
- 让语义检索变得更可控（按查询类型分配信任度）
- 让维护可持续（working set + freshness checks）

---

## 系统能力详解

### 多 Agent 协作架构

建立以 Supervisor 为中心的调度机制，实现"写审分离"：

- **Coordinator** — 定义问题、选择路径、负责收口
- **Implementer** — 执行文件/脚本/操作改动
- **Reviewer** — 检查实现质量，揪明显问题
- **Policy Auditor** — 审治理、流程、规则变化
- **Consensus Peer** — 高风险结构变更前的可选同级会审

支持 2-agent 到 5-agent 的灵活部署形态。核心规则：高风险工作里，不要让同一角色既实现又做最终审查。

### 分层记忆系统

从无序堆砌走向逐层蒸馏：

| 层级 | 用途 | 典型文件 |
|------|------|---------|
| Canon | 规则、审批、硬边界 | AGENTS.md、治理文档 |
| Bridge | 回答"现在什么是真的" | SYSTEM_OVERVIEW.md |
| Health | 运行健康和一致性状态 | current-health.json |
| Runtime Snapshot | 某一时点的运行事实 | runtime-snapshot.md |
| Structured Memory | 中等持久度的主题卡片 | TOPICS_INDEX.md |
| Long-term Memory | 高持久事实 | MEMORY.md |
| Historical | 背景、旧阶段、审计 | 旧版计划、归档文档 |

两层冲突时，不要平均——必须决定谁赢。

### 语义检索与路由

不同问题类型命中不同层：

| 查询类型 | 优先检索层 | 语义搜索角色 |
|---------|-----------|------------|
| 治理/规则 | Canon | 仅背景 |
| 系统状态 | Bridge + Health | 辅助召回 |
| 运行时 | Runtime Snapshot | 仅发现 |
| 用户偏好 | 手工维护的 Profile | 仅次要 |
| 历史溯源 | 证据发现 + 文件验证 | 发现层 |
| 维护/保鲜 | Working Set | 通常不需要 |

### 审批与安全治理

- 核心文件修改必须经过独立评审与人工授权
- 内置防偷懒协议（代码交付契约）：要求完整交付，禁止 TODO 占位符
- 治理/政策/流程变更必须过 Policy Auditor
- 降级运行时必须声明独立性损失

### 保鲜纪律

- 定义最小 working set（必须保鲜的文件集合）
- 按文件类型分组设置阈值（health/bridge 用短阈值，topic cards 用长阈值）
- 自动化检查器把"谁该更新了"变成可重复执行的检查

### 收口标准

每个阶段明确三类状态：
- **Landed** — 能力已存在、已验证、已接入实际运行
- **Improving** — 已工作但需要更长观察期
- **Deferred** — 明确推迟，不是遗忘

主线够完整就收口，不要无限扩相。

---

## 适用场景

当你的工作区已经出现以下信号时使用：

- ✅ 多个 Agent 或多角色执行路径已出现
- ✅ 文档、记忆、状态层越来越多
- ✅ 回答"现在到底什么是真的"开始变难
- ✅ semantic search 有用但不再能无脑相信
- ✅ live docs 过多、working set 不清、维护失控

⚠️ 这是进阶 Skill。如果你的工作区还很简单（1 个 agent、几个文件），不需要这个。

---

## 非目标

这份 Skill **不会**：

- 替你自动拍板或自动改规则
- 发布私有角色 lore 或内部组织文化
- 打包个人记忆、日记或私有运行历史
- 在 v1 里做全自动多 Agent 编排
- 把所有旧文档都维持成"活文档"

它提供的是治理 playbook，不是无限自动化引擎。

---

## 安装

### 方式一：通过 ClawHub

```bash
clawhub install openclaw-workspace-governance
```

### 方式二：直接克隆

```bash
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

### 安装后

1. **备份你的工作区**（重要！）
2. 阅读 `SKILL.md` — 这是 Agent 会读取的主指令文档
3. 从「快速开始」的 6 步流程开始，不要直接改文件
4. 按需阅读 `references/` 下的详细参考文档

### 包结构

```
SKILL.md                          # 主指令文档（Agent 读取的入口）
docs/
  GUIDE_ZH.md                     # 中文详细指南（本文件）
  GUIDE_EN.md                     # 英文详细指南
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

---

## 快速开始

先别急着改文件。主线只有 6 步：

1. **分类问题** — 判断你遇到的是哪一类 drift（governance / current-state / retrieval / doc / maintenance / phase）
2. **选择治理路径** — 区分实现改动、文档改动、治理改动，走不同的路线
3. **找到权威事实层** — 确定哪一层应该回答你的问题
4. **收紧文档边界** — 给文档打标签（live / historical / needs-refresh），定义最小活跃集
5. **用真实问题验证** — 至少测 system-state、governance、maintenance、historical 各一个
6. **主动收口** — 主线够完整就关闭，转入维护观察期

如果工作区现在很乱，**先减少歧义，不要先加流程**。

---

## 核心工作流

详细的 9 步工作流请参阅 `SKILL.md`，这里列出要点：

1. 先分类问题类型（6 种 drift）
2. 按变更类型选治理路径（implementation / documentation / governance）
3. 建立权威事实分层（7 层，冲突时有明确优先级）
4. 按查询类型决定检索顺序（7 种 query class）
5. 给文档打角色边界（live / historical / needs-refresh / special-case）
6. 建 working set 和 freshness checks
7. 诊断 semantic-search runtime（路径对齐、split-brain、按类信任）
8. 用代表性问题验证（PASS / PASS WITH ESCALATION / FAIL）
9. 主动收尾（landed / improving / deferred）

---

## 脚本说明

### freshness-check.py

检查 working set 中文件的新鲜度。

```bash
python3 scripts/freshness-check.py \
  --root . \
  --config assets/examples/working-set.example.json

# JSON 输出
python3 scripts/freshness-check.py \
  --root . \
  --config assets/examples/working-set.example.json \
  --json
```

配置文件定义分组阈值和文件列表，参见 `assets/examples/working-set.example.json`。

### semantic-runtime-check.sh

语义搜索运行时诊断。假设你的 CLI 支持 `<cmd> status` 和 `<cmd> query "<text>"` 子命令。

```bash
bash scripts/semantic-runtime-check.sh \
  --cmd /path/to/your-semantic-cli \
  --config-home ~/.config/your-backend \
  --cache-home ~/.cache/your-backend \
  --query "system state" \
  --query "weekly review status"
```

### doc-status-scan.py

扫描文档目录，统计 status label 分布。注意：只扫描每个文件的前 12 行。

```bash
python3 scripts/doc-status-scan.py --root docs
python3 scripts/doc-status-scan.py --root docs --json
python3 scripts/doc-status-scan.py --root docs --include-unlabeled
```

---

## 参考文档索引

| 文档 | 内容 |
|------|------|
| `references/multi-agent-governance.md` | 角色分离、变更路由、降级运行、审批边界 |
| `references/source-of-truth-layering.md` | 7 层事实模型、冲突解决规则 |
| `references/query-class-routing.md` | 7 种查询类型、检索优先级、语义搜索角色 |
| `references/live-vs-historical-docs.md` | 文档状态标记、最小活跃集、特殊安全文档处理 |
| `references/freshness-discipline.md` | Working set、分组阈值、刷新顺序 |
| `references/semantic-search-diagnostics.md` | 路径对齐、split-brain、按类信任判断 |
| `references/completion-criteria.md` | Landed/Improving/Deferred、收口标准、维护观察期 |

---

## 示例模板索引

| 模板 | 用途 |
|------|------|
| `assets/examples/working-set.example.json` | freshness-check.py 的配置文件模板 |
| `assets/examples/working-set.example.md` | 工作集文档模板 |
| `assets/examples/current-health.example.json` | 健康状态 JSON 模板 |
| `assets/examples/system-state-index.example.md` | 系统状态统一索引模板 |

---

## FAQ

**Q: 我的工作区只有 1 个 Agent，能用吗？**
A: 能用。但要明确标注"本来这里该有第二审"，不要假装自审等价。高风险改动需要人工审批。

**Q: 和 AGENTS.md 是什么关系？**
A: 这个 Skill 提供的是通用治理框架。你的 AGENTS.md 是你工作区自己的治理规则（canon 层）。两者互补，不冲突。

**Q: 语义搜索不靠谱怎么办？**
A: 不要给 blanket trust label。按查询类型分别评估信任度，对不靠谱的类型降级为"仅辅助召回"或"仅发现"。详见 `references/semantic-search-diagnostics.md`。

**Q: 可以部分采用吗？**
A: 完全可以。按需读取 references，不需要一次性全部部署。先从"分类问题 + 建立事实分层"开始最有效。
