# 部署手册（中文）

> 这份手册教你**从零开始部署**工作区治理。按顺序做，每步都有具体文件内容。
> 
> ⚠️ 部署前请备份你的工作区：`cp -r ~/.openclaw/workspace ~/.openclaw/workspace-backup-$(date +%Y%m%d)`

---

## 目录

- [Phase 1：最小部署（30 分钟见效）](#phase-1最小部署30-分钟见效)
- [Phase 2：多 Agent 治理](#phase-2多-agent-治理)
- [Phase 3：语义搜索与检索路由](#phase-3语义搜索与检索路由)
- [Phase 4：保鲜自动化](#phase-4保鲜自动化)
- [部署后验证](#部署后验证)
- [常见问题排查](#常见问题排查)

---

## Phase 1：最小部署（30 分钟见效）

目标：让 Agent 能区分"什么是当前事实"和"什么是历史背景"。

### 第 1 步：创建桥接文件

桥接文件回答"现在什么是真的"。在你的工作区创建：

```bash
mkdir -p memory/health
```

**文件：`memory/SYSTEM_OVERVIEW.md`**

```markdown
# System Overview

Updated: 2026-03-25

## 当前状态

- Agent 数量：1（主 Agent）
- 记忆后端：（填你的，如 qmd / 文件 / 无）
- 主要通信渠道：（填你的，如 飞书 / Discord / Telegram）
- OpenClaw 版本：（填你的，运行 `openclaw --version` 获取）

## 活跃工作

- （列出当前进行中的主要任务）

## 已知问题

- （列出已知但未修复的问题，没有就写"无"）
```

**文件：`memory/health/current-health.json`**

```json
{
  "updatedAt": "2026-03-25T12:00:00+08:00",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "note": "单 Agent 运行，无多角色治理需求"
    },
    "memory": {
      "status": "ok",
      "note": "基础记忆系统运行中"
    },
    "automation": {
      "status": "ok",
      "note": "cron/heartbeat 正常"
    }
  }
}
```

### 第 2 步：给旧文档打标签

找到你工作区里**不再代表当前事实的旧文档**，在文件开头加一行标签：

```markdown
Status: historical-reference

# 旧的部署计划
...
```

常用标签：
- `historical-reference` — 有参考价值但不代表当前事实
- `needs-refresh` — 部分过时，读的时候要注意
- 没标签 = 默认当作活跃文档

**实际操作**：翻一遍你的 `docs/` 目录，给每个旧文件开头加标签。不确定的先标 `needs-refresh`。

### 第 3 步：在 AGENTS.md 中添加事实分层规则

在你的 `AGENTS.md`（或等效的 Agent 指令文件）中加入以下段落：

```markdown
## 事实分层

回答问题时按以下优先级查找：
1. **规则层**：AGENTS.md、安全限制文档 → 回答"规则是什么"
2. **当前态层**：memory/SYSTEM_OVERVIEW.md、memory/health/ → 回答"现在什么是真的"
3. **记忆层**：MEMORY.md、memory/topics/ → 回答"之前发生过什么"
4. **历史层**：标有 `historical-reference` 的文档 → 仅作为背景参考

两层冲突时，上层赢。标有 `historical-reference` 的文件不得作为当前状态的依据。
```

### 第 4 步：验证

问你的 Agent 这几个问题，看它是否回答正确：

| 问题 | 期望行为 |
|------|---------|
| "当前系统状态是什么？" | 先读 `memory/SYSTEM_OVERVIEW.md`，不是从旧文档里拼凑 |
| "这个旧计划还有效吗？"（指一个你已标 historical 的文件） | 回答"这是历史文档，当前状态请看 SYSTEM_OVERVIEW" |

**Phase 1 完成标志**：Agent 能区分当前态和历史态，不再用旧文档回答"现在怎么样"。

---

## Phase 2：多 Agent 治理

> 如果你只有 1 个 Agent，可以跳过这个 Phase，但建议至少读一遍原则。

目标：建立写审分离，防止 Agent 自己改规则自己审批。

### 第 1 步：定义角色

在 `AGENTS.md` 中添加：

```markdown
## 角色分工

| 角色 | 职责 | 对应 Agent / 模型 |
|------|------|------------------|
| Coordinator（主管） | 任务拆分、路径选择、收口 | （填你的主 Agent） |
| Implementer（执行） | 文件/代码/脚本改动 | （填你的代码 Agent） |
| Reviewer（审查） | 检查实现质量 | （填你的审查 Agent） |
| Policy Auditor（政委） | 审核规则/流程/治理变更 | （填你的审核 Agent） |

### 基本规则
- 高风险工作不得由同一角色既实现又做最终审查
- 治理/规则变更必须经过 Policy Auditor
- 核心文件（AGENTS.md、MEMORY.md 等）修改需要人工授权
```

### 第 2 步：定义变更路由

在 `AGENTS.md` 中添加：

```markdown
## 变更路由

| 变更类型 | 路线 |
|---------|------|
| 普通实现（bug修复、功能开发） | Coordinator → Implementer → Reviewer |
| 文档改动 | Coordinator → Implementer → Reviewer |
| 治理/规则/流程改动 | Coordinator → Policy Auditor → 人工审批 → 执行 |
| 核心文件修改 | Coordinator → Policy Auditor → 人工明确授权 → 执行 |
```

### 第 3 步：降级运行声明

如果你只有 2 个 Agent 或临时只有 1 个可用，在执行记录中声明：

```markdown
## 降级运行声明
- 当前可用 Agent：1（仅主 Agent）
- 独立性损失：无独立审查
- 影响：高风险改动需要人工审批
```

**Phase 2 完成标志**：Agent 不再自己改规则自己审批。治理变更有明确的审核路径。

---

## Phase 3：语义搜索与检索路由

目标：让 Agent 按问题类型选择正确的检索方式。

### 第 1 步：创建检索策略文件

**文件：`docs/retrieval-policy.md`**

```markdown
# 检索策略

Updated: 2026-03-25

## 按查询类型路由

| 查询类型 | 首先检索 | 语义搜索角色 |
|---------|---------|------------|
| 治理/规则 | AGENTS.md、安全限制文档 | 不使用 |
| 系统状态 | memory/SYSTEM_OVERVIEW.md、memory/health/ | 辅助召回 |
| 运行时信息 | docs/runtime-snapshot.md（如有） | 仅发现 |
| 用户偏好 | USER.md、MEMORY.md 中的偏好段 | 仅次要 |
| 历史溯源 | memory/ 日志 → 文件验证 | 发现层 |
| 维护/保鲜 | working set 文件列表 | 通常不需要 |

## 规则
- 语义搜索（memory_search）不作为治理和规则问题的主要来源
- 当前态问题优先读桥接文件，不信旧 topic card
- 历史问题用语义搜索发现候选，但要用文件原文验证
```

### 第 2 步：在 AGENTS.md 中引用

```markdown
## 检索策略

回答问题前先判断问题类型，再按 `docs/retrieval-policy.md` 中的路由表选择检索方式。
不要对所有问题都无脑用 memory_search。
```

**Phase 3 完成标志**：Agent 在回答治理问题时先看 AGENTS.md 而不是搜记忆；回答状态问题时先看桥接文件。

---

## Phase 4：保鲜自动化

目标：让关键文件的过期自动可见。

### 第 1 步：创建你的 working set 配置

复制示例并根据你的工作区修改：

```bash
cp skills/openclaw-workspace-governance/assets/examples/working-set.example.json \
   docs/working-set.json
```

编辑 `docs/working-set.json`，改成你的实际文件路径：

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

说明：
- `groups` 里的数字是天数阈值（超过就告警）
- `health` / `bridge` 建议 3 天
- `entry` 建议 5 天
- `structured` 建议 7 天

### 第 2 步：跑一次检查

```bash
python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
  --root ~/.openclaw/workspace \
  --config docs/working-set.json
```

预期输出类似：

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

Summary: 3 OK, 1 WARN, total 4
Warned files:
- memory/SYSTEM_OVERVIEW.md (4.50d > 3d)
```

### 第 3 步：（可选）加入 heartbeat

在你的 `HEARTBEAT.md` 中添加：

```markdown
# 保鲜检查
# 每次心跳运行：
#   python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
#     --root ~/.openclaw/workspace --config docs/working-set.json
# 如果有 WARN，提醒老大哪些文件过期了
```

**Phase 4 完成标志**：`freshness-check.py` 能跑通，过期文件自动可见。

---

## 部署后验证

完成以上 Phase 后，用这些问题测试你的 Agent：

| # | 问题 | 期望行为 | 结果 |
|---|------|---------|------|
| 1 | "当前系统状态是什么？" | 先读 SYSTEM_OVERVIEW.md | ☐ PASS / ☐ FAIL |
| 2 | "AGENTS.md 里的审核规则是什么？" | 直接读 AGENTS.md，不靠语义搜索 | ☐ PASS / ☐ FAIL |
| 3 | "上周做了什么？" | 用 memory_search 发现，再用文件验证 | ☐ PASS / ☐ FAIL |
| 4 | "哪些文件需要更新？" | 跑 freshness-check.py 或读 working set | ☐ PASS / ☐ FAIL |

4 个都 PASS = 部署成功 ✅

---

## 常见问题排查

**Agent 还是用旧文档回答当前状态**
→ 检查旧文档是否已标 `historical-reference`
→ 检查 AGENTS.md 中事实分层规则是否已添加

**freshness-check.py 报 file missing**
→ 检查 `working-set.json` 里的路径是否正确（相对于 `--root`）

**Agent 自己改了 AGENTS.md 没经过审核**
→ 检查 AGENTS.md 中是否写了"核心文件修改需要人工授权"
→ 如果只有 1 个 Agent，这条规则需要人工监督执行

**不知道从哪个 Phase 开始**
→ 从 Phase 1 开始。Phase 1 是一切的基础，30 分钟就能完成。
→ Phase 2-4 按需部署，不需要一次全做。
