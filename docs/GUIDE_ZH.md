# 虾滑 OpenClaw 工作区治理 — 完整指南

> 🚨 **高危预警** — 这不是普通工具 Skill，而是系统级架构升级。部署前务必备份工作区！

---

## 目录

- [一、这是什么](#一这是什么)
- [二、适用场景与非目标](#二适用场景与非目标)
- [三、安装与激活](#三安装与激活)
- [四、部署前评估](#四部署前评估)
- [五、Phase 1 — 事实分层基础](#五phase-1--事实分层基础)
- [六、Phase 2 — 多 Agent 治理](#六phase-2--多-agent-治理)
- [七、Phase 3 — 检索路由](#七phase-3--检索路由)
- [八、Phase 4 — 保鲜自动化](#八phase-4--保鲜自动化)
- [九、脚本详细说明](#九脚本详细说明)
- [十、参考文档索引](#十参考文档索引)
- [十一、部署后验证](#十一部署后验证)
- [十二、FAQ](#十二faq)

---

## 一、这是什么

面向**复杂 OpenClaw 工作区**的治理型 Skill。当你的工作区已经有多个 Agent、大量文档、记忆层相互打架、旧文件冒充当前事实的时候，这套框架帮你把混乱压回可控状态。

### 核心能力

| 能力 | 解决什么问题 |
|------|------------|
| **多 Agent 协作架构** | Agent 自己改规则自己审批的闭环 |
| **分层记忆系统** | 所有文档平等对待导致的冲突 |
| **语义检索路由** | 对所有问题无脑用 memory_search |
| **审批与安全治理** | 核心文件被无声修改 |
| **保鲜纪律** | 文档过期了没人知道 |
| **收口标准** | 改进阶段无限扩展不收敛 |

### 角色说明

本框架定义了 5 个通用角色（不绑定具体模型）：

| 角色 | 职责 | 最少需要几个 Agent |
|------|------|------------------|
| **Coordinator** | 任务拆分、路径选择、收口 | 可与 Implementer 合并（1 个即可） |
| **Implementer** | 执行文件/代码/脚本改动 | 可与 Coordinator 合并 |
| **Reviewer** | 检查实现质量 | 至少 1 个独立于 Implementer |
| **Policy Auditor** | 审治理/流程/规则变更 | 可选（单 Agent 时由人工替代） |
| **Consensus Peer** | 高风险结构变更前的同级会审 | 可选 |

### 事实分层模型

所有信息分为 7 层，冲突时上层赢：

```
┌─────────────────────────────────────┐
│ 1. Canon（规则层）                    │ ← AGENTS.md、安全限制
├─────────────────────────────────────┤
│ 2. Bridge（当前态层）                  │ ← SYSTEM_OVERVIEW.md
├─────────────────────────────────────┤
│ 3. Health（健康层）                    │ ← current-health.json
├─────────────────────────────────────┤
│ 4. Runtime Snapshot（运行时层）         │ ← runtime-snapshot.md
├─────────────────────────────────────┤
│ 5. Structured Memory（结构化记忆层）    │ ← TOPICS_INDEX.md
├─────────────────────────────────────┤
│ 6. Long-term Memory（长期记忆层）       │ ← MEMORY.md
├─────────────────────────────────────┤
│ 7. Historical（历史层）                │ ← 旧计划、归档文档
└─────────────────────────────────────┘
```

---

## 二、适用场景与非目标

### 适用场景

- ✅ 多个 Agent 或多角色执行路径已出现
- ✅ 文档数量超过 20 个且增长中
- ✅ 同一个问题从不同文件得到矛盾答案
- ✅ semantic search 有用但经常返回过时结果
- ✅ 不知道哪些文档还需要维护

### 非目标

- ❌ 自动拍板或自动改规则
- ❌ 发布私有角色设定或内部组织文化
- ❌ 全自动多 Agent 编排（v1 是手册+脚本，不是编排引擎）
- ❌ 把所有旧文档都保持活跃状态

### 不适合的情况

- 工作区只有 1 个 Agent + 几个文件 → 太早了，先长复杂度再说
- 刚开始用 OpenClaw → 先跑起来，等遇到痛点再部署

---

## 三、安装与激活

### 安装

```bash
# 方式一：ClawHub
clawhub install openclaw-workspace-governance

# 方式二：直接克隆
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

### 激活

安装后，这个 Skill 会出现在 `<available_skills>` 列表中。OpenClaw 会在匹配任务时自动加载 `SKILL.md`。

验证安装成功：

```bash
ls ~/.openclaw/workspace/skills/openclaw-workspace-governance/SKILL.md
```

如果文件存在，安装完成。

### 备份

**部署前必须备份**：

```bash
cp -r ~/.openclaw/workspace ~/.openclaw/workspace-backup-$(date +%Y%m%d)
```

---

## 四、部署前评估

**不要跳过这一步。** 先搞清楚你的工作区现在是什么状况，再决定从哪个 Phase 开始。

### 4.1 快速摸底

回答以下问题：

| # | 问题 | 你的答案 |
|---|------|---------|
| 1 | 当前有几个 Agent？各自什么角色？ | |
| 2 | `docs/` 目录下有多少个 .md 文件？ | |
| 3 | 有没有 `memory/` 目录？里面有什么？ | |
| 4 | AGENTS.md 里有没有定义事实分层或检索策略？ | |
| 5 | 有没有文档标注过 `historical-reference` 等状态标签？ | |
| 6 | 最近一个月内，Agent 是否给过矛盾或过时的回答？ | |
| 7 | 有没有 freshness / working set 相关的检查机制？ | |

### 4.2 判断起点

根据摸底结果选起点：

| 你的状况 | 建议起点 |
|---------|---------|
| 基本空白，没有分层也没有标签 | Phase 1 开始 |
| 有 memory/ 但没有分层规则 | Phase 1 开始，但可以复用现有 memory 文件 |
| 有分层但没有多 Agent | Phase 1 确认 → Phase 3 |
| 多 Agent 但没有写审分离 | Phase 1 确认 → Phase 2 |
| 什么都有但文档过期不可见 | Phase 1 确认 → Phase 4 |

### 4.3 文档现状扫描

用自带脚本扫一遍你的 docs 目录：

```bash
python3 skills/openclaw-workspace-governance/scripts/doc-status-scan.py \
  --root docs --include-unlabeled
```

输出会告诉你：
- 有多少文件已经标了状态标签
- 有多少文件是 unlabeled（默认当 live 处理）
- unlabeled 文件列表 → 这些是你 Phase 1 要处理的

**如果 unlabeled 文件超过 10 个，你的工作区已经需要 Phase 1 了。**

---

## 五、Phase 1 — 事实分层基础

> 预计时间：30-60 分钟
> 前置条件：无
> 目标：Agent 能区分"什么是当前事实"和"什么是历史背景"

这是一切的基础。没有事实分层，后面的治理、路由、保鲜都没有根基。

### 5.1 创建桥接文件

桥接文件是回答"现在什么是真的"的短文件。它不需要面面俱到，但必须**经常更新**（建议 3 天以内）。

```bash
mkdir -p memory/health
```

**创建 `memory/SYSTEM_OVERVIEW.md`：**

```markdown
# System Overview

Updated: 2026-03-25

## 当前系统配置

- OpenClaw 版本：（运行 `openclaw --version`）
- Agent 数量：（填数量和各自角色）
- 记忆后端：（qmd / 文件 / 无 / 其他）
- 通信渠道：（飞书 / Discord / Telegram / 其他）

## 当前运行状态

- 整体状态：正常 / 有问题 / 降级运行
- 活跃的 cron 任务：（列出关键定时任务）
- 已知问题：（当前未修复的问题，没有就写"无"）

## 活跃工作

- （列出当前进行中的主要任务，每项一行）

## 近期变更

- （最近 3-5 天内的重要变更）
```

**创建 `memory/health/current-health.json`：**

```json
{
  "updatedAt": "2026-03-25T12:00:00+08:00",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "agentCount": 1,
      "writeReviewSeparation": false,
      "note": "单 Agent，无写审分离"
    },
    "memory": {
      "status": "ok",
      "backend": "qmd",
      "note": "记忆系统正常"
    },
    "retrieval": {
      "status": "unknown",
      "note": "未配置检索路由策略"
    },
    "automation": {
      "status": "ok",
      "cronHealthy": true,
      "note": "cron/heartbeat 正常"
    },
    "maintenance": {
      "status": "unknown",
      "workingSetDefined": false,
      "note": "未定义 working set"
    }
  },
  "summary": [
    "基础运行正常",
    "治理、检索、维护机制待部署"
  ]
}
```

> **关键**：`overall` 字段要如实填写。`unknown` 比假装 `ok` 更好。

### 5.2 给现有文档打标签

这是 Phase 1 最耗时但最重要的一步。每个旧文档都要做一个决定。

#### 判断规则

| 情况 | 标签 | 放在文件哪里 |
|------|------|------------|
| 这份文档记录的是已经过去的计划/阶段/决策 | `Status: historical-reference` | 文件第 1-3 行 |
| 文档部分有用但整体不再代表当前事实 | `Status: needs-refresh` | 文件第 1-3 行 |
| 安全限制、危险操作清单等（不常更新但仍有效） | `Status: special-case active safety restriction` | 文件第 1-3 行 |
| 文档仍然代表当前事实且需要持续维护 | 不加标签（默认 live） | — |

#### 操作步骤

1. 列出所有 docs 目录下的 .md 文件：
   ```bash
   find docs/ -name "*.md" -type f | sort
   ```

2. 逐个打开，问自己：**如果 Agent 把这份文件当作当前事实来回答问题，会不会给出错误答案？**
   - 会 → 标 `historical-reference` 或 `needs-refresh`
   - 不会 → 保持 live

3. 在文件开头加标签：
   ```markdown
   Status: historical-reference

   # 原来的标题
   ...
   ```

#### 实际操作示例

**场景**：你的 `docs/` 下有这些文件：

```
docs/
  setup-guide-v1.md          ← 半年前写的初始搭建指南
  architecture-plan.md        ← 三个月前的架构计划，一半已落地一半已放弃
  daily-news-config.md        ← 当前还在用的新闻推送配置
  debug-log-2025-12.md        ← 去年 12 月的调试记录
  dangerous-commands.md       ← 危险命令清单，仍然有效
```

处理结果：

| 文件 | 决定 | 理由 |
|------|------|------|
| `setup-guide-v1.md` | `historical-reference` | 初始搭建已完成，不代表当前配置 |
| `architecture-plan.md` | `needs-refresh` | 部分有效但整体不可信 |
| `daily-news-config.md` | 保持 live | 当前在用 |
| `debug-log-2025-12.md` | `historical-reference` | 纯历史记录 |
| `dangerous-commands.md` | `special-case active safety restriction` | 不常更新但必须保持权威 |

#### 量大怎么办

如果有 30+ 个文件，不要一次全做。分批：
1. 第一轮：把明显的历史文档标 `historical-reference`（5 分钟看一眼就能判断的）
2. 第二轮：处理不确定的，统一标 `needs-refresh`
3. 第三轮：在后续使用中逐步把 `needs-refresh` 升级为 live 或降级为 `historical-reference`

### 5.3 在 AGENTS.md 中建立事实分层规则

在你的 AGENTS.md 中添加以下段落（根据你的实际情况调整文件路径）：

```markdown
## 事实分层

回答问题时按以下优先级查找信息。两层冲突时，上层赢：

### 第 1 层：规则层（Canon）
- `AGENTS.md` — 角色定义、审批规则、操作限制
- 安全限制类文档（标有 `special-case active safety restriction`）
- 回答：规则是什么、谁负责什么、什么不能做

### 第 2 层：当前态层（Bridge）
- `memory/SYSTEM_OVERVIEW.md` — 当前系统状态总览
- `memory/health/current-health.json` — 健康状态
- 回答：现在什么是真的、当前状态是什么

### 第 3 层：记忆层（Memory）
- `MEMORY.md` — 长期记忆
- `memory/topics/` — 主题卡片（如有）
- `memory/YYYY-MM-DD.md` — 日记
- 回答：之前发生过什么、上次怎么做的

### 第 4 层：历史层（Historical）
- 标有 `historical-reference` 的文档
- 标有 `needs-refresh` 的文档（使用时需交叉验证）
- 用途：仅作为背景参考，不得作为当前状态的依据

### 规则
- 标有 `historical-reference` 的文件**不得**用于回答"现在怎么样"类问题
- 标有 `needs-refresh` 的文件可以参考但必须与更高层信息交叉验证
- 不确定某文件是否代表当前事实时，优先查第 2 层桥接文件
```

### 5.4 验证 Phase 1

问 Agent 这些问题：

| 测试 | 问题 | 期望行为 | ☐ |
|------|------|---------|---|
| T1 | "当前系统状态是什么？" | 先读 SYSTEM_OVERVIEW.md，不从旧文档拼凑 | |
| T2 | "这个旧文档里的配置还有效吗？"（指一个 historical 文件） | 识别为历史文档，引导查看当前态层 | |
| T3 | "帮我查一下上周做了什么" | 用记忆层（日记/memory_search），不去读 SYSTEM_OVERVIEW | |

3 个都通过 → Phase 1 完成 ✅

> **常见失败**：Agent 忽略了分层规则，还是去读旧文档。
> **修复**：检查 AGENTS.md 中的分层段落是否在文件靠前位置（Agent 上下文有限时可能读不到靠后的内容）。

---

## 六、Phase 2 — 多 Agent 治理

> 预计时间：30-60 分钟
> 前置条件：Phase 1 完成
> 目标：建立写审分离，核心文件修改有审批流程
> ⚠️ 如果你只有 1 个 Agent，请看 [6.5 单 Agent 降级方案](#65-单-agent-降级方案)

### 6.1 评估你需要的部署形态

| 形态 | 适合 | 角色分配 |
|------|------|---------|
| **2-agent** | 中等复杂度工作区 | Agent 1: Coordinator + Implementer / Agent 2: Reviewer |
| **3-agent** | 经常有代码+文档+审查任务 | Coordinator / Implementer / Reviewer |
| **4-agent** | 规则/流程变更频繁 | + Policy Auditor |
| **5-agent** | 大型系统，结构性改动频繁 | + Consensus Peer |

**选择原则**：从最小够用的形态开始。2-agent 就能覆盖大多数场景。

### 6.2 定义角色映射

在 AGENTS.md 中添加角色定义。**关键是要映射到你的具体 Agent**：

```markdown
## 角色定义

| 角色 | 职责 | 对应 Agent | 模型 |
|------|------|-----------|------|
| Coordinator（主管） | 任务拆分、路径选择、收口 | 虾滑 | claude-opus-4 |
| Implementer（执行） | 文件/代码改动 | GPT | gpt-5.4 |
| Reviewer（审查） | 实现质量检查 | 纠察 | gpt-5.3-codex |
| Policy Auditor（政委） | 规则/治理审核 | 政委 | gpt-5.4 |

> 上面是示例。替换成你自己的 Agent 名称和模型。
> 同一个模型可以担任不同角色，但同一份工作不能由同一个 Agent 既写又审。
```

### 6.3 定义变更路由

变更路由决定了"什么样的改动走什么审批路线"。这是防止 Agent 暗改规则的核心。

```markdown
## 变更路由

### 风险分级

| 级别 | 判断条件 | 示例 |
|------|---------|------|
| **低风险** | 不改行为、不改权限、不改规则 | 格式调整、注释修改、日志记录 |
| **中风险** | 改操作指导或默认行为 | 文档重写影响默认行为、检索策略调整 |
| **高风险** | 改规则、改角色边界、改审批要求 | 修改 AGENTS.md、修改权限边界、修改自动化范围 |

### 路由矩阵

| 变更类型 | 风险 | 路线 | 需要人工审批？ |
|---------|------|------|-------------|
| 普通实现 | 低 | Implementer → Reviewer | 否 |
| 普通实现 | 中 | Implementer → Reviewer | 改默认行为时需要 |
| 普通实现 | 高 | Implementer → Reviewer → Policy Auditor | 是 |
| 文档改动 | 低 | Implementer → Reviewer | 否 |
| 文档改动 | 中 | Implementer → Reviewer → Policy Auditor | 改操作含义时需要 |
| 治理/规则 | 中 | Policy Auditor → 人工审批 → 执行 | 是 |
| 治理/规则 | 高 | Consensus Peer → Policy Auditor → 人工审批 → 执行 | 是 |
| 核心文件 | 任何 | Policy Auditor → 人工明确授权 → 执行 | 必须 |

### 核心文件清单

以下文件的修改**必须**经过 Policy Auditor 审核和人工授权：
- `AGENTS.md`
- `USER.md`
- `MEMORY.md`
- `HEARTBEAT.md`
- `IDENTITY.md`
- （根据你的工作区补充）
```

### 6.4 定义降级运行规则

当某个 Agent 不可用时，不要假装分离仍然存在：

```markdown
## 降级运行

当可用 Agent 少于部署形态要求时：

### 降级声明（必须写在执行记录中）
- 当前可用 Agent 数量
- 哪些角色被合并
- 独立性损失说明
- 影响范围

### 降级规则
- 低风险工作：可以继续，记录降级状态
- 中风险工作：需要人工确认后继续
- 高风险/治理工作：必须等 Agent 恢复或获得人工审批
- 绝不假装自审等于独立审查
```

### 6.5 单 Agent 降级方案

如果你只有 1 个 Agent，Phase 2 的核心是**靠人工补位**：

```markdown
## 单 Agent 治理

### 可以自行完成
- 低风险实现工作
- 日志记录、格式调整
- 信息查询和回答

### 需要人工审批
- 任何改变 AGENTS.md 的操作
- 任何改变默认行为的操作
- 任何改变自动化范围的操作

### 执行时声明
每次执行中风险以上工作时，Agent 应主动声明：
"⚠️ 单 Agent 运行，本次变更缺少独立审查。需要你确认后我再执行。"
```

### 6.6 验证 Phase 2

| 测试 | 操作 | 期望行为 | ☐ |
|------|------|---------|---|
| T4 | 要求 Agent 修改 AGENTS.md | Agent 走审核流程（或请求人工授权），不直接改 | |
| T5 | 要求 Agent 修一个普通 bug | 直接走 Implementer → Reviewer 流程 | |
| T6 | 在只有 1 个 Agent 时要求治理变更 | Agent 声明降级运行，请求人工审批 | |

---

## 七、Phase 3 — 检索路由

> 预计时间：20-40 分钟
> 前置条件：Phase 1 完成
> 目标：Agent 按问题类型选择正确的检索方式，不再无脑 memory_search

### 7.1 理解检索路由的核心问题

**没有检索路由时的典型症状**：
- 问"规则是什么"→ Agent 去搜记忆，搜到旧版规则
- 问"现在系统状态"→ Agent 搜到一个月前的 topic card
- 问"用户偏好"→ Agent 搜到聊天记录里的噪声

**检索路由的本质**：不同问题先问不同的信息源。

### 7.2 创建检索策略文件

**创建 `docs/retrieval-policy.md`：**

```markdown
# 检索策略

Updated: 2026-03-25

## 查询分类与路由

Agent 收到问题后，先判断问题类型，再按对应路线检索。

### 类型 1：治理/规则问题
> "谁负责什么" "这个操作需要审批吗" "什么是核心文件"

检索路线：
1. AGENTS.md
2. 安全限制文档
3. 不使用 memory_search

### 类型 2：系统状态问题
> "当前系统状态" "健康状况" "运行是否正常"

检索路线：
1. memory/SYSTEM_OVERVIEW.md
2. memory/health/current-health.json
3. memory_search 仅作为辅助补充

### 类型 3：运行时问题
> "当前用什么模型" "哪些 cron 在跑"

检索路线：
1. runtime snapshot（如有）
2. 桥接文件
3. memory_search 仅作为发现线索

### 类型 4：用户偏好问题
> "用户喜欢什么风格" "有什么特殊要求"

检索路线：
1. USER.md
2. MEMORY.md 中的偏好段
3. memory_search 仅作为次要补充
4. ⚠️ 聊天记录搜索结果噪声大，不可直接信任

### 类型 5：历史溯源问题
> "上次怎么修的" "这个功能什么时候加的"

检索路线：
1. memory_search 发现候选
2. 找到候选文件后**打开原文验证**
3. 不可只凭搜索片段下结论

### 类型 6：维护/保鲜问题
> "哪些文件过期了" "现在该维护什么"

检索路线：
1. 运行 freshness-check.py
2. 查看 working set 配置
3. 通常不需要 memory_search

## 语义搜索信任度

| 查询类型 | memory_search 的角色 | 信任度 |
|---------|---------------------|-------|
| 治理/规则 | 不使用 | — |
| 系统状态 | 辅助召回 | 中（需与桥接文件交叉验证） |
| 运行时 | 发现线索 | 低 |
| 用户偏好 | 次要补充 | 低（聊天记录噪声大） |
| 历史溯源 | 发现候选 | 中（必须文件验证） |
| 维护/保鲜 | 通常不需要 | — |
```

### 7.3 在 AGENTS.md 中激活

添加一段简短的引用（不要重复整个策略文件）：

```markdown
## 检索策略

回答问题前，先判断问题类型（治理/状态/运行时/偏好/历史/维护），再按 `docs/retrieval-policy.md` 中的路由表选择检索方式。

关键规则：
- 治理问题直接读 AGENTS.md，不用 memory_search
- 状态问题先读桥接文件，不信旧 topic card
- 历史问题用 memory_search 发现后必须打开原文验证
```

### 7.4 处理语义搜索不稳定的情况

如果你的语义搜索后端（如 qmd）经常返回不相关结果，可能是这些原因：

| 症状 | 可能原因 | 排查方式 |
|------|---------|---------|
| 搜什么都返回同一批文件 | 索引缓存过旧 | `openclaw memory status --json` 看索引状态 |
| 搜索结果和直接读文件不一致 | 路径不对齐（split-brain） | 运行 `scripts/semantic-runtime-check.sh` |
| 新文件搜不到 | embedding 队列堆积 | 检查 pending embedding count |

详细诊断流程见 `references/semantic-search-diagnostics.md`。

### 7.5 验证 Phase 3

| 测试 | 问题 | 期望行为 | ☐ |
|------|------|---------|---|
| T7 | "AGENTS.md 里的核心文件清单是什么？" | 直接读 AGENTS.md，不搜记忆 | |
| T8 | "当前系统状态？" | 先读 SYSTEM_OVERVIEW.md | |
| T9 | "上周五做了什么？" | 用 memory_search 发现 → 读原文验证 | |
| T10 | "用户偏好什么语气？" | 读 USER.md，不靠聊天记录 | |

---

## 八、Phase 4 — 保鲜自动化

> 预计时间：30-45 分钟
> 前置条件：Phase 1 完成
> 目标：关键文件过期时自动发现，而不是等到 Agent 给出错误答案才发现

### 8.1 设计你的 Working Set

Working set = 必须持续保鲜的最小文件集合。不是"所有重要文件"，而是"过期了会直接导致 Agent 犯错的文件"。

#### 评估标准

| 文件 | 是否进 working set | 理由 |
|------|------------------|------|
| memory/health/current-health.json | ✅ | 过期了 Agent 会报告错误的健康状态 |
| memory/SYSTEM_OVERVIEW.md | ✅ | 过期了 Agent 会回答过时的系统状态 |
| AGENTS.md | ✅ | 变更后如果不更新，Agent 行为和规则不一致 |
| MEMORY.md | ⚠️ 看情况 | 长期记忆更新慢是正常的，7 天阈值 |
| docs/old-plan.md | ❌ | 历史文档，不需要保鲜 |

#### 分组阈值设计

不同类型的文件用不同的过期阈值：

| 分组 | 阈值 | 包含文件类型 | 理由 |
|------|------|------------|------|
| `health` | 3 天 | 健康状态、一致性检查 | 这类信息老化最快 |
| `bridge` | 3 天 | SYSTEM_OVERVIEW 等当前态文件 | 直接影响"现在怎么样"的回答 |
| `runtime` | 3 天 | 运行时快照 | 运行时状态变化快 |
| `entry` | 5 天 | AGENTS.md、入口索引 | 变更不频繁但过期影响大 |
| `structured` | 7 天 | MEMORY.md、TOPICS_INDEX | 长期信息更新慢是正常的 |

你可以根据自己工作区的更新频率调整。核心原则：**宁可告警多一点，也别漏掉关键过期**。

### 8.2 创建配置文件

```bash
mkdir -p docs
```

**创建 `docs/working-set.json`（根据你的实际文件路径修改）：**

```json
{
  "groups": {
    "health": 3,
    "bridge": 3,
    "runtime": 3,
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

**如何添加更多文件**：每个 entry 需要指定 group（对应阈值分组）和 path（相对于工作区根目录）。

### 8.3 运行首次检查

```bash
python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
  --root ~/.openclaw/workspace \
  --config docs/working-set.json
```

#### 理解输出

```
Freshness check @ 2026-03-25 18:00 +0800
Thresholds:
- health: 3d
- bridge: 3d
- entry: 5d
- structured: 7d

[OK] memory/health/current-health.json
  group=health threshold=3d age=0.25d source=updatedAt via json seen_at=2026-03-25T12:00:00+08:00
[WARN] memory/SYSTEM_OVERVIEW.md
  group=bridge threshold=3d age=4.50d source=header-date via header seen_at=2026-03-21T00:00:00+00:00
[OK] AGENTS.md
  group=entry threshold=5d age=1.20d source=mtime via filesystem seen_at=2026-03-24T06:00:00+08:00
[OK] MEMORY.md
  group=structured threshold=7d age=3.00d source=header-date via header seen_at=2026-03-22T00:00:00+00:00

Summary: 3 OK, 1 WARN, total 4
Warned files:
- memory/SYSTEM_OVERVIEW.md (4.50d > 3d)
```

- `[OK]` = 在阈值内
- `[WARN]` = 超过阈值，需要更新
- `source` = 时间戳从哪里获取的（json 字段 / markdown header / 文件系统 mtime）

#### 时间戳识别规则

脚本按以下优先级提取文件的"最后更新时间"：

1. JSON 文件：读 `updatedAt` 字段
2. Markdown 文件：读前 30 行中的 `Updated:`、`Last updated:`、`Last verified:` 等 header
3. 兜底：使用文件系统的 mtime

**推荐**：在文件中主动维护 `Updated: YYYY-MM-DD` header，比依赖 mtime 更可靠。

### 8.4 加入自动化

#### 方式一：HEARTBEAT.md（推荐）

在 HEARTBEAT.md 中添加：

```markdown
# 保鲜检查
#
# 每次心跳执行：
#   python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
#     --root ~/.openclaw/workspace --config docs/working-set.json --json
#
# 处理规则：
# - 如果有 WARN 项：告诉老大哪些文件过期了，建议更新
# - 如果全 OK：不需要报告（安静通过）
# - 如果脚本执行失败：报告执行错误
```

#### 方式二：Cron Job

```bash
# 每天早上 9 点检查一次
openclaw cron add --schedule "0 9 * * *" --command "python3 skills/openclaw-workspace-governance/scripts/freshness-check.py --root ~/.openclaw/workspace --config docs/working-set.json"
```

### 8.5 当收到 WARN 时怎么办

1. **打开告警的文件**
2. **检查内容是否仍然准确**
3. 如果准确：只更新 `Updated:` 日期即可
4. 如果不准确：更新内容 + 日期
5. 如果文件已不再需要保鲜：从 `working-set.json` 中移除

### 8.6 验证 Phase 4

| 测试 | 操作 | 期望结果 | ☐ |
|------|------|---------|---|
| T11 | 运行 freshness-check.py | 输出正常，能识别出过期文件 | |
| T12 | 故意不更新 SYSTEM_OVERVIEW 4 天 | 检查器报 WARN | |
| T13 | 更新文件后再跑 | WARN 消失，变 OK | |

---

## 九、脚本详细说明

### freshness-check.py

| 项目 | 说明 |
|------|------|
| 用途 | 检查 working set 文件的新鲜度 |
| 依赖 | Python 3.6+，无第三方依赖 |
| 输入 | `--root`（工作区根目录）+ `--config`（JSON 配置文件） |
| 输出 | 人类可读报告（默认）或 JSON（加 `--json`） |
| 退出码 | 0 = 全 OK，2 = 有 WARN |

### semantic-runtime-check.sh

| 项目 | 说明 |
|------|------|
| 用途 | 语义搜索运行时诊断 |
| 依赖 | Bash，你的语义 CLI 需支持 `<cmd> status` 和 `<cmd> query "<text>"` |
| 输入 | `--cmd`（CLI 路径）+ `--config-home` + 可选 `--cache-home` + `--query`（可重复） |
| 退出码 | 0 = 全部通过，2 = 有检查失败 |

### doc-status-scan.py

| 项目 | 说明 |
|------|------|
| 用途 | 扫描文档目录统计 status label 分布 |
| 依赖 | Python 3.6+，无第三方依赖 |
| 限制 | 只扫描每个文件的前 12 行 |
| 输入 | `--root`（扫描目录）+ 可选 `--include-unlabeled` |

---

## 十、参考文档索引

深入阅读用。部署时不需要全读，遇到问题时按需查阅。

| 文档 | 什么时候读 |
|------|----------|
| `references/multi-agent-governance.md` | 设计多 Agent 角色分工时 |
| `references/source-of-truth-layering.md` | 不确定哪层信息应该赢时 |
| `references/query-class-routing.md` | 调整检索策略时 |
| `references/live-vs-historical-docs.md` | 大批量打标签时 |
| `references/freshness-discipline.md` | 设计 working set 和阈值时 |
| `references/semantic-search-diagnostics.md` | 语义搜索返回奇怪结果时 |
| `references/completion-criteria.md` | 判断一个改进阶段是否可以收口时 |

---

## 十一、部署后验证

### 综合验证清单

完成所有你部署的 Phase 后，做一次综合测试：

| # | 测试项 | 期望 | 结果 |
|---|--------|------|------|
| 1 | 问当前状态 | 读桥接文件 | ☐ |
| 2 | 问规则 | 读 AGENTS.md | ☐ |
| 3 | 问历史 | memory_search → 文件验证 | ☐ |
| 4 | 问保鲜 | 跑 freshness-check 或读 working set | ☐ |
| 5 | 要求改 AGENTS.md | 走审批流程 | ☐ |
| 6 | 标有 historical 的文件 | 不被当作当前事实 | ☐ |

**判定标准**：
- 全部 PASS = 部署成功 ✅
- 有 1-2 个 FAIL = 检查对应 Phase 的配置
- 多数 FAIL = 检查 AGENTS.md 中的分层规则是否被正确读取

### 更新 health 文件

部署完成后，更新健康状态：

```json
{
  "updatedAt": "（当前时间）",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "note": "事实分层已建立，变更路由已定义"
    },
    "retrieval": {
      "status": "ok",
      "note": "检索路由策略已激活"
    },
    "maintenance": {
      "status": "ok",
      "workingSetDefined": true,
      "note": "freshness-check 已配置"
    }
  }
}
```

---

## 十二、FAQ

**Q: 部署顺序可以调整吗？**
A: Phase 1 必须最先做。Phase 2/3/4 可以按需选择，不必全部部署。

**Q: 一次部署所有 Phase 还是分批？**
A: 建议分批。先 Phase 1，用几天观察效果，再推进后续 Phase。

**Q: 部署后发现 Agent 行为没变？**
A: 最常见原因是 AGENTS.md 中新加的规则在文件靠后位置。Agent 上下文有限时可能没读到。把关键规则往前移。

**Q: 可以只用部分功能吗？**
A: 完全可以。最小有效单元是 Phase 1（事实分层）。其他 Phase 都是增量。

**Q: 怎么回滚？**
A: 删除添加的桥接文件和配置，移除 AGENTS.md 中新增的段落。备份还在的话直接恢复。

**Q: working set 应该多大？**
A: 尽量小。4-8 个文件是常见大小。超过 15 个就要反问自己是不是把不该保鲜的文件也加进去了。
