---
name: openclaw-workspace-governance
description: 🚨【高危预警】系统级架构升级！部署前务必备份工作区！面向复杂 OpenClaw 工作区的治理 Skill：事实分层、多 Agent 治理、检索路由、保鲜自动化。
version: 1.1.0
---

# OpenClaw 工作区治理（Workspace Governance）

> 🚨 **高危预警** — 这不是普通工具 Skill，而是系统级架构升级。部署前务必备份工作区！

面向**复杂 OpenClaw 工作区**的治理型 Skill。帮你压住漂移、明确当前事实层、缩小 live docs 面积，让语义检索更可控。

### 核心能力

| 能力 | 解决什么问题 |
|------|------------|
| **多 Agent 协作架构** | Agent 自己改规则自己审批的闭环 |
| **分层记忆系统** | 所有文档平等对待导致的冲突 |
| **语义检索路由** | 对所有问题无脑用 memory_search |
| **审批与安全治理** | 核心文件被无声修改 |
| **保鲜纪律** | 文档过期了没人知道 |
| **收口标准** | 改进阶段无限扩展不收敛 |

### 文档导航

| 文档 | 读者 | 内容 |
|------|------|------|
| **本文件 (SKILL.md)** | Agent | 可执行 Runbook：自动检测 → 部署 → 日常运维 |
| [完整指南（中文）](docs/GUIDE_ZH.md) | 人 | 原理解释 + 详细部署手册 + FAQ |
| [Complete Guide (English)](docs/GUIDE_EN.md) | Human | Design rationale + deployment guide + FAQ |
| `references/*.md` | Agent（按需） | 7 篇深入参考文档 |

### 包含内容

- 🤖 Agent 可执行 Runbook（自动检测部署状态，逐步引导部署）
- 📖 中英文完整指南（12 章节，含部署前评估、4 Phase 部署手册、验证清单）
- 📚 7 篇治理参考文档（多 Agent 治理、事实分层、检索路由、文档标记、保鲜纪律、语义搜索诊断、收口标准）
- 🔧 3 个可执行脚本（freshness-check、semantic-runtime-check、doc-status-scan）
- 📋 4 个示例模板（working-set、health、system-state-index）

---

## Agent Runbook

> 以下内容面向 Agent。按顺序读，按指令执行。
> 执行任何修改前，先确认用户已备份工作区。
> 如果用户未确认备份，**停下来提醒**，不要继续。

本 Runbook 分三部分：
- **Part 1**：首次部署检测与执行（如果工作区还没部署过）
- **Part 2**：日常运维工作流（部署完成后的持续使用）
- **Part 3**：脚本与参考索引

人类用户如需了解原理和价值，请引导阅读 `docs/GUIDE_ZH.md` 或 `docs/GUIDE_EN.md`。

---

## Part 1：首次部署

### Step 0：检测部署状态

执行以下检查，判断工作区当前处于哪个阶段：

```
检查项 A：ls memory/SYSTEM_OVERVIEW.md
检查项 B：grep -l "事实分层\|Source-of-Truth Layering" AGENTS.md
检查项 C：ls docs/retrieval-policy.md
检查项 D：ls docs/working-set.json
```

| 结果 | 含义 | 动作 |
|------|------|------|
| A 不存在 | 未部署 Phase 1 | 从 Phase 1 开始 |
| A 存在，B 不存在 | Phase 1 未完成 | 从 Phase 1.3 开始 |
| A+B 存在，C 不存在 | Phase 1 完成，Phase 3 未部署 | 跳到 Phase 3（Phase 2 按需） |
| A+B+C 存在，D 不存在 | Phase 1+3 完成，Phase 4 未部署 | 跳到 Phase 4 |
| 全部存在 | 已完整部署 | 跳到 Part 2（日常运维） |

> **在开始任何 Phase 之前**，向用户确认：
> "我检测到工作区尚未部署 [Phase X]。需要我现在开始部署吗？部署前请确认已备份工作区。"

---

### Phase 1：事实分层基础

> 目标：建立"什么是当前事实 vs 什么是历史背景"的区分能力。
> 这是一切的根基，后续所有 Phase 都依赖它。

#### Phase 1.1：创建桥接文件

桥接文件回答"现在什么是真的"。你需要采集当前工作区的信息来填充它们。

**操作 1：采集信息**

执行以下命令并记录结果：

```bash
# OpenClaw 版本
openclaw --version 2>/dev/null || echo "unknown"

# 目前有多少个 agent 配置
ls ~/.openclaw/agents/ 2>/dev/null | head -20 || echo "none"

# memory 目录结构
ls -la memory/ 2>/dev/null || echo "no memory dir"

# 当前 cron 任务
# 使用 cron(action=list) 工具获取
```

**操作 2：创建目录**

```bash
mkdir -p memory/health
```

**操作 3：创建 `memory/SYSTEM_OVERVIEW.md`**

用采集到的信息填充以下模板。不确定的字段填 `unknown`，不要瞎猜：

```markdown
# System Overview

Updated: [今天的日期，格式 YYYY-MM-DD]

## 当前系统配置

- OpenClaw 版本：[填采集结果]
- Agent 数量：[填数量]，角色：[列出已知角色，如"主 Agent"、"代码 Agent"等]
- 记忆后端：[填 qmd / 文件系统 / 无 / unknown]
- 通信渠道：[填当前活跃的渠道，如 飞书 / Discord / Telegram]

## 当前运行状态

- 整体状态：[healthy / has-issues / unknown]
- 活跃 cron 任务：[列出，或"无"]
- 已知问题：[列出，或"无"]

## 活跃工作

- [向用户确认当前在做什么，或写"待确认"]
```

**操作 4：创建 `memory/health/current-health.json`**

```json
{
  "updatedAt": "[今天的 ISO 时间戳，如 2026-03-25T12:00:00+08:00]",
  "overall": "[根据实际情况填 healthy / has-issues / unknown]",
  "domains": {
    "governance": {
      "status": "[ok 如果有多 Agent 且有审批流程，否则 unknown]",
      "agentCount": [填数字],
      "writeReviewSeparation": [true 如果已有写审分离，否则 false],
      "note": "[简述当前治理状况]"
    },
    "memory": {
      "status": "[ok 如果记忆系统在跑，否则 unknown]",
      "backend": "[qmd / files / none / unknown]",
      "note": "[简述]"
    },
    "retrieval": {
      "status": "unknown",
      "note": "检索路由策略尚未配置"
    },
    "automation": {
      "status": "[ok 如果 cron/heartbeat 正常，否则 unknown]",
      "cronHealthy": [true/false],
      "note": "[简述]"
    },
    "maintenance": {
      "status": "unknown",
      "workingSetDefined": false,
      "note": "working set 尚未定义"
    }
  }
}
```

> **Impact**：创建这两个文件后，你回答"当前系统状态"类问题时，应优先读取这两个文件，而不是从散落的旧文档中拼凑。

#### Phase 1.2：给现有文档打标签

**目的**：让旧文档停止假装自己是当前事实。

**操作 1：扫描现有文档**

```bash
python3 [skill_dir]/scripts/doc-status-scan.py --root docs --include-unlabeled 2>/dev/null
```

如果 `docs/` 目录不存在或为空，跳过此步。

**操作 2：对每个 unlabeled 文件做判断**

逐个读取文件内容（前 30 行即可），用以下规则判断：

| 判断问题 | 如果答案是"是" | 操作 |
|---------|-------------|------|
| 这个文件记录的是已完成的计划、过去的阶段、旧的决策？ | 是 | 在文件第 1 行插入 `Status: historical-reference`，空一行后接原内容 |
| 文件部分有用但整体不再可信？ | 是 | 在文件第 1 行插入 `Status: needs-refresh` |
| 文件是安全限制、危险操作清单等？（不常更新但仍有效） | 是 | 在文件第 1 行插入 `Status: special-case active safety restriction` |
| 文件仍然代表当前事实且需要持续维护？ | 是 | 不加标签，保持原样 |

**操作格式**：

```markdown
Status: historical-reference

# 原来的标题
...原来的内容...
```

> **如果文件数量多于 15 个**：不要一次全做。先处理 `docs/` 顶层的明显历史文件，向用户报告进度，分批完成。

> **Impact**：标记完成后，你在回答问题时必须遵守以下规则：
> - 标有 `historical-reference` 的文件 → 不可用于回答"现在怎么样"类问题
> - 标有 `needs-refresh` 的文件 → 可参考但必须与桥接文件交叉验证
> - 标有 `special-case active safety restriction` 的文件 → 始终有效，不可降级

#### Phase 1.3：在 AGENTS.md 中建立事实分层规则

**操作**：在 AGENTS.md 中找到一个合适的位置（建议在文件前半部分，`## Memory` 段落之前或之后），插入以下内容：

> ⚠️ 不要覆盖 AGENTS.md！只做增量插入。
> 先读取当前 AGENTS.md 的完整内容，找到插入位置，用 edit 工具精确插入。

```markdown
## 事实分层（Source-of-Truth Layering）

回答问题时按以下优先级查找信息。两层冲突时，上层赢：

### 第 1 层：规则层（Canon）
- `AGENTS.md` — 角色定义、审批规则、操作限制
- 标有 `special-case active safety restriction` 的文档
- 回答：规则是什么、谁负责什么、什么不能做

### 第 2 层：当前态层（Bridge）
- `memory/SYSTEM_OVERVIEW.md` — 当前系统状态总览
- `memory/health/current-health.json` — 健康状态
- 回答：现在什么是真的、当前状态如何

### 第 3 层：记忆层（Memory）
- `MEMORY.md` — 长期记忆
- `memory/topics/` — 主题卡片（如有）
- `memory/YYYY-MM-DD.md` — 日记
- 回答：之前发生过什么、上次怎么做的

### 第 4 层：历史层（Historical）
- 标有 `historical-reference` 的文档
- 标有 `needs-refresh` 的文档（使用时需交叉验证）
- 用途：仅作为背景参考，不得作为当前状态的依据

### 分层规则
- 标有 `historical-reference` 的文件**不得**用于回答"现在怎么样"类问题
- 标有 `needs-refresh` 的文件可以参考但必须与第 2 层交叉验证
- 不确定某文件是否代表当前事实时，优先查第 2 层桥接文件
```

> **Impact**：插入此规则后，你的检索行为必须改变——
> 收到"当前系统状态"类问题时，第一步是读 `memory/SYSTEM_OVERVIEW.md`，不是 memory_search。
> 收到规则类问题时，第一步是读 AGENTS.md，不是搜记忆。

#### Phase 1.4：验证

自行执行以下验证：

1. 确认 `memory/SYSTEM_OVERVIEW.md` 存在且内容非空
2. 确认 `memory/health/current-health.json` 存在且 JSON 合法
3. 确认 AGENTS.md 中包含"事实分层"段落
4. 如果处理过文档标签：确认至少有 1 个文件被正确标记

向用户报告：
"Phase 1 部署完成。已创建桥接文件 [列出]，标记了 [N] 个历史文档，在 AGENTS.md 中添加了事实分层规则。从现在开始，我回答状态问题会优先读桥接文件。"

---

### Phase 2：多 Agent 治理（可选）

> 跳过条件：如果工作区只有 1 个 Agent 且用户没有扩展计划，可以跳过。
> 但即使只有 1 个 Agent，也建议至少添加"核心文件保护"规则（见 Phase 2.3）。

#### Phase 2.1：检测现有角色

```bash
# 检查 AGENTS.md 中是否已有角色定义
grep -i "角色\|Role.*Definition\|Coordinator\|Implementer\|Reviewer\|Policy Auditor" AGENTS.md
```

如果已有角色定义，向用户确认是否需要补充治理规则。如果没有，继续。

#### Phase 2.2：询问用户角色映射

向用户提问：

"Phase 2 需要定义角色分工。请告诉我：
1. 当前有哪些 Agent？各自叫什么名字、用什么模型？
2. 哪个 Agent 负责写代码/改文件？（Implementer）
3. 哪个 Agent 负责审查？（Reviewer）
4. 是否需要独立的规则审核角色？（Policy Auditor）
5. 如果只有 1 个 Agent，是否希望我添加核心文件保护规则？"

**等待用户回复后**，根据回复填充以下模板。

#### Phase 2.3：插入角色与路由规则

在 AGENTS.md 中，在"事实分层"段落之后插入：

> ⚠️ 以下内容需要根据用户回复填充具体的 Agent 名称和模型。方括号 [] 内的内容必须替换。

```markdown
## 角色定义与变更路由

### 角色映射

| 角色 | 职责 | Agent | 模型 |
|------|------|-------|------|
| Coordinator | 任务拆分、路径选择、收口 | [填 Agent 名] | [填模型] |
| Implementer | 文件/代码改动执行 | [填 Agent 名] | [填模型] |
| Reviewer | 实现质量审查 | [填 Agent 名] | [填模型] |
| Policy Auditor | 治理/规则审核 | [填 Agent 名，如无则写"由用户担任"] | [填模型] |

> 同一模型可担任不同角色，但同一份工作不得由同一 Agent 既写又审。

### 变更路由

| 变更类型 | 风险级别 | 路线 | 需要人工审批？ |
|---------|---------|------|-------------|
| 普通实现（bug修复、功能开发） | 低 | Implementer → Reviewer | 否 |
| 普通实现 | 中（改默认行为） | Implementer → Reviewer | 是 |
| 文档改动 | 低 | Implementer → Reviewer | 否 |
| 文档改动 | 中（改操作含义） | Implementer → Reviewer → Policy Auditor | 是 |
| 治理/规则变更 | 任何 | Policy Auditor → 人工审批 → 执行 | 是 |
| 核心文件修改 | 任何 | Policy Auditor → 人工明确授权 → 执行 | 必须 |

### 核心文件清单

以下文件的修改**必须**经过审核和人工授权：
- `AGENTS.md`
- `USER.md`
- `MEMORY.md`
- `HEARTBEAT.md`
- `IDENTITY.md`

### 降级运行

当可用 Agent 少于角色要求时：
- 在执行记录中声明降级状态和独立性损失
- 低风险工作可继续，中风险需人工确认，高风险必须等 Agent 恢复或获人工审批
- 绝不假装自审等于独立审查
```

**如果只有 1 个 Agent**，使用简化版：

```markdown
## 核心文件保护

以下文件的修改必须先向用户说明变更内容并获得明确授权：
- `AGENTS.md`
- `USER.md`
- `MEMORY.md`
- `HEARTBEAT.md`
- `IDENTITY.md`

执行中风险以上工作时，主动声明：
"⚠️ 单 Agent 运行，本次变更缺少独立审查。需要你确认后我再执行。"
```

> **Impact**：插入此规则后——
> - 你修改 AGENTS.md 前必须走审批流程（或请求人工授权）
> - 你执行治理变更前必须经过 Policy Auditor（或人工）审核
> - 降级运行时你必须主动声明

#### Phase 2.4：验证

1. 确认 AGENTS.md 中包含角色定义或核心文件保护规则
2. 尝试描述"如果用户要求修改 AGENTS.md，你会怎么做"——应该包含审批步骤

向用户报告完成。

---

### Phase 3：检索路由

> 目标：按问题类型选择正确的检索方式。

#### Phase 3.1：创建检索策略文件

**操作**：创建 `docs/retrieval-policy.md`

```markdown
# 检索策略

Updated: [今天的日期]

## 查询分类与路由

收到问题后，先判断类型，再按对应路线检索。

### 类型 1：治理/规则
> "谁负责什么" "需要审批吗" "什么是核心文件"

路线：
1. 读 AGENTS.md
2. 读安全限制文档
3. **不使用** memory_search

### 类型 2：系统状态
> "当前系统状态" "健康状况" "运行是否正常"

路线：
1. 读 memory/SYSTEM_OVERVIEW.md
2. 读 memory/health/current-health.json
3. memory_search 仅作为辅助补充

### 类型 3：运行时
> "当前用什么模型" "哪些 cron 在跑"

路线：
1. 读 runtime snapshot（如有）
2. 读桥接文件
3. memory_search 仅作为发现线索

### 类型 4：用户偏好
> "用户喜欢什么风格" "有什么特殊要求"

路线：
1. 读 USER.md
2. 读 MEMORY.md 中的偏好段
3. memory_search 仅次要补充
4. ⚠️ 聊天记录搜索噪声大，不可直接信任

### 类型 5：历史溯源
> "上次怎么修的" "这功能什么时候加的"

路线：
1. memory_search 发现候选
2. **打开候选文件原文验证**
3. 不可只凭搜索片段下结论

### 类型 6：维护/保鲜
> "哪些文件过期了" "现在该维护什么"

路线：
1. 运行 freshness-check.py（如已配置）
2. 读 working set 配置
3. 通常不需要 memory_search
```

#### Phase 3.2：在 AGENTS.md 中激活

在 AGENTS.md 的"事实分层"段落之后（或角色定义之后）插入：

```markdown
## 检索策略

回答问题前，先判断问题类型（治理/状态/运行时/偏好/历史/维护），按 `docs/retrieval-policy.md` 中的路由表检索。

关键规则：
- 治理问题 → 直接读 AGENTS.md，不用 memory_search
- 状态问题 → 先读桥接文件，不信旧 topic card
- 历史问题 → memory_search 发现后必须打开原文验证
```

> **Impact**：激活后——
> - 收到治理类问题时，你不再调用 memory_search，而是直接读 AGENTS.md
> - 收到状态类问题时，你先读 SYSTEM_OVERVIEW.md 再考虑搜索
> - 收到历史类问题时，memory_search 的结果必须经过文件验证才能用于回答

#### Phase 3.3：验证

1. 确认 `docs/retrieval-policy.md` 存在
2. 确认 AGENTS.md 中包含"检索策略"段落
3. 自检：描述收到"当前系统状态"问题时你的检索顺序——应该是"先读 SYSTEM_OVERVIEW"而不是"先 memory_search"

向用户报告完成。

---

### Phase 4：保鲜自动化

> 目标：关键文件过期时自动发现。

#### Phase 4.1：设计 working set

**操作 1：列出候选文件**

```bash
# 已有的桥接文件
ls memory/SYSTEM_OVERVIEW.md memory/health/current-health.json 2>/dev/null

# 核心指令文件
ls AGENTS.md MEMORY.md 2>/dev/null

# 运行时快照（如有）
ls docs/runtime-snapshot*.md 2>/dev/null

# 主题索引（如有）
ls memory/TOPICS_INDEX.md 2>/dev/null
```

**操作 2：向用户确认**

展示候选列表，问用户：
"以下是我建议纳入保鲜监控的文件。请确认或调整：
- [列出文件] — 阈值 [N] 天
需要增删吗？"

#### Phase 4.2：创建配置文件

根据确认的文件列表，创建 `docs/working-set.json`：

```json
{
  "groups": {
    "health": 3,
    "bridge": 3,
    "entry": 5,
    "structured": 7
  },
  "workingSet": [
    {"group": "health", "path": "memory/health/current-health.json"},
    {"group": "bridge", "path": "memory/SYSTEM_OVERVIEW.md"},
    {"group": "entry", "path": "AGENTS.md"},
    {"group": "structured", "path": "MEMORY.md"}
  ]
}
```

> 根据用户确认的实际列表调整 `workingSet` 数组。每个 entry 需要 `group`（对应阈值分组）和 `path`（相对于工作区根目录）。

#### Phase 4.3：首次运行

```bash
python3 [skill_dir]/scripts/freshness-check.py \
  --root [workspace_root] \
  --config docs/working-set.json
```

向用户报告结果。如果有 WARN 项，说明哪些文件过期了。

#### Phase 4.4：设置持续检查

向用户提供两个选项：

**选项 A：加入 HEARTBEAT.md**

在 HEARTBEAT.md 末尾追加：

```markdown
# 保鲜检查
# 每次心跳执行 freshness-check.py
# 有 WARN 项时提醒老大，全 OK 时静默通过
```

**选项 B：创建 Cron Job**

使用 cron 工具创建定时任务。

让用户选择，然后执行。

> **Impact**：配置完成后——
> - 过期文件会被自动发现并报告
> - 你在心跳/定时检查中发现过期文件时，应主动提醒用户更新

#### Phase 4.5：验证

1. 确认 `docs/working-set.json` 存在且 JSON 合法
2. 确认 freshness-check.py 能正常运行
3. 确认持续检查已配置（HEARTBEAT 或 Cron）

向用户报告完成。

---

### 部署完成

所有已部署的 Phase 完成后，执行最终验证：

1. 更新 `memory/health/current-health.json`，将已部署的 domain 状态改为 `ok`
2. 更新 `memory/SYSTEM_OVERVIEW.md` 中的"近期变更"段落
3. 向用户报告完整的部署摘要

---

## Part 2：日常运维工作流

> 以下是部署完成后的持续使用指南。
> 当你被触发处理治理相关问题时，按以下流程操作。

### 2.1 先分类问题

遇到治理相关问题时，先判断是哪一类 drift：

| 类型 | 信号 | 首要动作 |
|------|------|---------|
| governance drift | 角色/审批/规则边界在漂 | 检查 AGENTS.md 中的角色和路由定义 |
| current-state drift | "现在什么是真的"不清楚 | 刷新桥接文件 |
| retrieval drift | 搜索结果互相打架 | 按 `references/semantic-search-diagnostics.md` 诊断 |
| doc drift | 旧文档冒充当前事实 | 打标签，缩小 live 面积 |
| maintenance drift | 不知道该维护什么 | 运行 freshness-check.py |
| phase drift | 流程文档越来越多但不收口 | 按 `references/completion-criteria.md` 评估是否可以关闭 |

### 2.2 按变更类型选路线

遵循 AGENTS.md 中定义的变更路由矩阵。
如果 AGENTS.md 中没有定义路由（只做了 Phase 1），使用默认规则：
- 普通改动 → 直接做
- 改规则/改行为 → 先向用户说明再做
- 改核心文件 → 必须用户授权

### 2.3 检索时遵循路由策略

遵循 `docs/retrieval-policy.md` 中的查询分类路由。
核心原则：不同问题先问不同的信息源。

### 2.4 维护桥接文件

当以下情况发生时，主动更新桥接文件：
- 系统配置发生变更 → 更新 SYSTEM_OVERVIEW.md
- 健康状态变化 → 更新 current-health.json
- 完成重大任务 → 更新 SYSTEM_OVERVIEW.md 的"活跃工作"段

更新时同步修改 `Updated:` 日期。

### 2.5 响应保鲜告警

当 freshness-check 报告 WARN 时：
1. 打开告警文件
2. 检查内容是否仍准确
3. 准确 → 只更新日期
4. 不准确 → 更新内容 + 日期
5. 不再需要保鲜 → 从 working-set.json 中移除

### 2.6 主动收口

当一个改进阶段的主线已落地时：
- 标记 **landed**（已落地）/ **improving**（观察中）/ **deferred**（明确推迟）
- 转入维护观察期
- 不要继续生产新的流程文档

详见 `references/completion-criteria.md`。

---

## Part 3：脚本与参考索引

### 脚本

| 脚本 | 用途 | 运行方式 |
|------|------|---------|
| `scripts/freshness-check.py` | 检查 working set 文件新鲜度 | `python3 [script] --root [workspace] --config [config.json]` |
| `scripts/semantic-runtime-check.sh` | 语义搜索运行时诊断 | `bash [script] --cmd [cli] --config-home [path] --query "text"` |
| `scripts/doc-status-scan.py` | 扫描文档 status label 分布 | `python3 [script] --root docs --include-unlabeled` |

### 参考文档

遇到具体问题时按需阅读，不需要提前全部读完：

| 文档 | 何时阅读 |
|------|---------|
| `references/multi-agent-governance.md` | 设计/调整多 Agent 角色分工时 |
| `references/source-of-truth-layering.md` | 不确定哪层信息应该赢时 |
| `references/query-class-routing.md` | 调整检索策略时 |
| `references/live-vs-historical-docs.md` | 大批量给文档打标签时 |
| `references/freshness-discipline.md` | 设计 working set 和阈值时 |
| `references/semantic-search-diagnostics.md` | 语义搜索返回奇怪结果时 |
| `references/completion-criteria.md` | 判断改进阶段是否可以收口时 |

### 示例模板

| 模板 | 用途 |
|------|------|
| `assets/examples/working-set.example.json` | freshness-check 配置模板 |
| `assets/examples/working-set.example.md` | working set 文档模板 |
| `assets/examples/current-health.example.json` | 健康状态 JSON 模板 |
| `assets/examples/system-state-index.example.md` | 系统状态索引模板 |

---

> **人类用户**：如需了解本 Skill 的设计原理、系统能力详解、适用场景分析和 FAQ，请阅读 `docs/GUIDE_ZH.md`（中文）或 `docs/GUIDE_EN.md`（English）。
