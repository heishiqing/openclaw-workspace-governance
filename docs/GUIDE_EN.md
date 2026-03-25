# OpenClaw Workspace Governance — Complete Guide

> 🚨 **WARNING** — This is NOT a standard tool skill. It is a system-level architecture upgrade. Back up your workspace before deployment!

---

## Table of Contents

- [1. What Is This](#1-what-is-this)
- [2. Scope & Non-goals](#2-scope--non-goals)
- [3. Installation & Activation](#3-installation--activation)
- [4. Pre-Deployment Assessment](#4-pre-deployment-assessment)
- [5. Phase 1 — Source-of-Truth Foundation](#5-phase-1--source-of-truth-foundation)
- [6. Phase 2 — Multi-Agent Governance](#6-phase-2--multi-agent-governance)
- [7. Phase 3 — Retrieval Routing](#7-phase-3--retrieval-routing)
- [8. Phase 4 — Freshness Automation](#8-phase-4--freshness-automation)
- [9. Script Reference](#9-script-reference)
- [10. Reference Document Index](#10-reference-document-index)
- [11. Post-Deployment Validation](#11-post-deployment-validation)
- [12. FAQ](#12-faq)

---

## 1. What Is This

A governance skill for **complex OpenClaw workspaces**. When your workspace already has multiple Agents, dozens of documents, memory layers fighting each other, and old files pretending to be current truth — this framework helps you push the chaos back into a controllable state.

### Core Capabilities

| Capability | Problem It Solves |
|-----------|------------------|
| **Multi-Agent Architecture** | Agent modifies its own rules and self-approves |
| **Layered Memory System** | All documents treated as equal, causing conflicts |
| **Semantic Retrieval Routing** | Blindly using memory_search for every question |
| **Approval & Governance** | Core files silently modified without review |
| **Freshness Discipline** | Documents go stale and nobody notices |
| **Completion Criteria** | Improvement phases expand forever without closure |

### Roles

This framework defines 5 generic roles (not tied to specific models):

| Role | Responsibility | Min Agents Needed |
|------|---------------|-------------------|
| **Coordinator** | Task decomposition, path selection, closure | Can merge with Implementer (1 is enough) |
| **Implementer** | Execute file/code/script changes | Can merge with Coordinator |
| **Reviewer** | Check implementation quality | At least 1 independent from Implementer |
| **Policy Auditor** | Review governance/rule/process changes | Optional (human substitutes in single-agent) |
| **Consensus Peer** | Pre-commit peer review for risky structural changes | Optional |

### Source-of-Truth Layers

All information is organized into 7 layers. When layers conflict, the higher layer wins:

```
┌─────────────────────────────────────┐
│ 1. Canon (Rules)                    │ ← AGENTS.md, safety restrictions
├─────────────────────────────────────┤
│ 2. Bridge (Current State)           │ ← SYSTEM_OVERVIEW.md
├─────────────────────────────────────┤
│ 3. Health                           │ ← current-health.json
├─────────────────────────────────────┤
│ 4. Runtime Snapshot                 │ ← runtime-snapshot.md
├─────────────────────────────────────┤
│ 5. Structured Memory                │ ← TOPICS_INDEX.md
├─────────────────────────────────────┤
│ 6. Long-term Memory                 │ ← MEMORY.md
├─────────────────────────────────────┤
│ 7. Historical                       │ ← Old plans, archived docs
└─────────────────────────────────────┘
```

---

## 2. Scope & Non-goals

### When to Use

- ✅ Multiple agents or role-based execution paths exist
- ✅ More than 20 documents and growing
- ✅ Same question gets contradictory answers from different files
- ✅ Semantic search is useful but often returns stale results
- ✅ Unclear which documents still need maintenance

### Non-goals

- ❌ Automate approval decisions or policy changes
- ❌ Publish private role lore or internal organization culture
- ❌ Full automated multi-agent orchestration (v1 is a playbook + scripts, not an orchestration engine)
- ❌ Keep every old document active

### When NOT to Use

- Workspace has just 1 Agent + a few files → too early, wait for complexity to grow
- Just started using OpenClaw → get running first, deploy this when you hit pain points

---

## 3. Installation & Activation

### Install

```bash
# Option 1: ClawHub
clawhub install openclaw-workspace-governance

# Option 2: Direct clone
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

### Activation

After installation, this skill appears in the `<available_skills>` list. OpenClaw loads `SKILL.md` automatically when matching tasks.

Verify installation:

```bash
ls ~/.openclaw/workspace/skills/openclaw-workspace-governance/SKILL.md
```

If the file exists, installation is complete.

### Backup

**Mandatory before deployment**:

```bash
cp -r ~/.openclaw/workspace ~/.openclaw/workspace-backup-$(date +%Y%m%d)
```

---

## 4. Pre-Deployment Assessment

**Do not skip this.** Understand your workspace's current state before deciding where to start.

### 4.1 Quick Inventory

Answer these questions:

| # | Question | Your Answer |
|---|---------|-------------|
| 1 | How many Agents? What roles? | |
| 2 | How many .md files in `docs/`? | |
| 3 | Is there a `memory/` directory? What's in it? | |
| 4 | Does AGENTS.md define source-of-truth layering or retrieval strategy? | |
| 5 | Are any documents labeled with `historical-reference` or similar? | |
| 6 | Has the Agent given contradictory or outdated answers in the past month? | |
| 7 | Is there any freshness / working-set checking mechanism? | |

### 4.2 Choose Your Starting Point

| Your Situation | Start Here |
|---------------|------------|
| Mostly blank, no layering or labels | Phase 1 |
| Have memory/ but no layering rules | Phase 1 (reuse existing memory files) |
| Have layering but no multi-agent | Phase 1 confirm → Phase 3 |
| Multi-agent but no write-review separation | Phase 1 confirm → Phase 2 |
| Everything exists but staleness is invisible | Phase 1 confirm → Phase 4 |

### 4.3 Document Status Scan

Use the included script to scan your docs directory:

```bash
python3 skills/openclaw-workspace-governance/scripts/doc-status-scan.py \
  --root docs --include-unlabeled
```

The output tells you:
- How many files already have status labels
- How many are unlabeled (treated as live by default)
- The unlabeled file list → these are what you'll process in Phase 1

**If you have more than 10 unlabeled files, your workspace already needs Phase 1.**

---

## 5. Phase 1 — Source-of-Truth Foundation

> Estimated time: 30-60 minutes
> Prerequisites: None
> Goal: Agent can distinguish "what is true now" from "historical background"

This is the foundation for everything. Without source-of-truth layering, governance, routing, and freshness have nothing to build on.

### 5.1 Create Bridge Files

Bridge files answer "what is true now?" They don't need to be comprehensive, but they must be **updated frequently** (recommend within 3 days).

```bash
mkdir -p memory/health
```

**Create `memory/SYSTEM_OVERVIEW.md`:**

```markdown
# System Overview

Updated: 2026-03-25

## Current System Configuration

- OpenClaw version: (run `openclaw --version`)
- Agent count: (number and roles)
- Memory backend: (qmd / files / none / other)
- Communication channels: (Discord / Telegram / Slack / other)

## Current Operational Status

- Overall: healthy / has issues / degraded
- Active cron jobs: (list key scheduled tasks)
- Known issues: (unfixed issues, or "none")

## Active Work

- (list current tasks, one per line)

## Recent Changes

- (important changes in the last 3-5 days)
```

**Create `memory/health/current-health.json`:**

```json
{
  "updatedAt": "2026-03-25T12:00:00+00:00",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "agentCount": 1,
      "writeReviewSeparation": false,
      "note": "Single agent, no write-review separation"
    },
    "memory": {
      "status": "ok",
      "backend": "qmd",
      "note": "Memory system running"
    },
    "retrieval": {
      "status": "unknown",
      "note": "No retrieval routing policy configured"
    },
    "automation": {
      "status": "ok",
      "cronHealthy": true,
      "note": "Cron/heartbeat normal"
    },
    "maintenance": {
      "status": "unknown",
      "workingSetDefined": false,
      "note": "No working set defined"
    }
  },
  "summary": [
    "Basic operation is healthy",
    "Governance, retrieval, and maintenance mechanisms pending deployment"
  ]
}
```

> **Key**: Fill `overall` honestly. `unknown` is better than pretending `ok`.

### 5.2 Label Existing Documents

This is the most time-consuming but most important step in Phase 1. Every old document needs a decision.

#### Decision Rules

| Situation | Label | Where to Place |
|-----------|-------|---------------|
| Document records past plans/phases/decisions | `Status: historical-reference` | Lines 1-3 of file |
| Document is partially useful but no longer current truth overall | `Status: needs-refresh` | Lines 1-3 of file |
| Safety restrictions, dangerous-action lists (rarely updated but still authoritative) | `Status: special-case active safety restriction` | Lines 1-3 of file |
| Document still represents current truth and needs ongoing maintenance | No label (default live) | — |

#### Steps

1. List all .md files:
   ```bash
   find docs/ -name "*.md" -type f | sort
   ```

2. For each file, ask: **If the Agent uses this file as current truth, would it give a wrong answer?**
   - Yes → label `historical-reference` or `needs-refresh`
   - No → keep live

3. Add the label at the top:
   ```markdown
   Status: historical-reference

   # Original Title
   ...
   ```

#### Worked Example

**Scenario**: Your `docs/` contains:

```
docs/
  setup-guide-v1.md          ← Initial setup guide from 6 months ago
  architecture-plan.md        ← 3-month-old plan, half done, half abandoned
  daily-news-config.md        ← Currently active news config
  debug-log-2025-12.md        ← Debug log from December 2025
  dangerous-commands.md       ← Dangerous commands list, still valid
```

Results:

| File | Decision | Reason |
|------|----------|--------|
| `setup-guide-v1.md` | `historical-reference` | Initial setup is done, doesn't represent current config |
| `architecture-plan.md` | `needs-refresh` | Partially valid but unreliable as a whole |
| `daily-news-config.md` | Keep live | Currently in use |
| `debug-log-2025-12.md` | `historical-reference` | Pure historical record |
| `dangerous-commands.md` | `special-case active safety restriction` | Rarely updated but must remain authoritative |

#### Too Many Files?

If you have 30+ files, don't do them all at once. Batch it:
1. **Round 1**: Label obvious historical files (5-second judgment each)
2. **Round 2**: Label uncertain files as `needs-refresh`
3. **Round 3**: Over the following weeks, upgrade or downgrade `needs-refresh` files based on actual use

### 5.3 Add Source-of-Truth Layering to AGENTS.md

Add the following section to your AGENTS.md (adjust file paths to match your workspace):

```markdown
## Source-of-Truth Layering

When answering questions, search in this priority order. When layers conflict, the higher layer wins:

### Layer 1: Canon (Rules)
- `AGENTS.md` — role definitions, approval rules, operational restrictions
- Safety restriction docs (labeled `special-case active safety restriction`)
- Answers: what are the rules, who is responsible, what is forbidden

### Layer 2: Bridge (Current State)
- `memory/SYSTEM_OVERVIEW.md` — current system state overview
- `memory/health/current-health.json` — health status
- Answers: what is true now, current operational status

### Layer 3: Memory
- `MEMORY.md` — long-term memory
- `memory/topics/` — topic cards (if any)
- `memory/YYYY-MM-DD.md` — daily notes
- Answers: what happened before, how was it done last time

### Layer 4: Historical
- Files labeled `historical-reference`
- Files labeled `needs-refresh` (must cross-verify when used)
- Use: background context only, NOT current-state evidence

### Rules
- Files labeled `historical-reference` must NOT be used to answer "what's happening now" questions
- Files labeled `needs-refresh` may be referenced but must be cross-verified with higher layers
- When unsure if a file represents current truth, check Layer 2 bridge files first
```

### 5.4 Validate Phase 1

| Test | Question | Expected Behavior | ☐ |
|------|---------|-------------------|---|
| T1 | "What's the current system state?" | Reads SYSTEM_OVERVIEW.md first, doesn't piece together from old docs | |
| T2 | "Is this old doc's config still valid?" (point to a historical file) | Identifies it as historical, directs to current-state layer | |
| T3 | "What happened last week?" | Uses memory layer (daily notes / memory_search), not SYSTEM_OVERVIEW | |

All 3 pass → Phase 1 complete ✅

> **Common failure**: Agent ignores layering rules and still reads old docs.
> **Fix**: Move the layering section to an earlier position in AGENTS.md (Agent context is limited, content at the bottom may not be read).

---

## 6. Phase 2 — Multi-Agent Governance

> Estimated time: 30-60 minutes
> Prerequisites: Phase 1 complete
> Goal: Write-review separation; core file modifications require approval
> ⚠️ Single Agent? See [6.5 Single-Agent Degraded Mode](#65-single-agent-degraded-mode)

### 6.1 Choose Your Deployment Shape

| Shape | Best For | Role Assignment |
|-------|---------|----------------|
| **2-agent** | Medium complexity | Agent 1: Coordinator + Implementer / Agent 2: Reviewer |
| **3-agent** | Regular code + docs + review work | Coordinator / Implementer / Reviewer |
| **4-agent** | Frequent rule/process changes | + Policy Auditor |
| **5-agent** | Large systems, frequent structural changes | + Consensus Peer |

**Principle**: Start with the minimum viable shape. 2-agent covers most scenarios.

### 6.2 Define Role Mapping

Add role definitions to AGENTS.md. **Map to your actual Agents**:

```markdown
## Role Definitions

| Role | Responsibility | Agent | Model |
|------|---------------|-------|-------|
| Coordinator | Task decomposition, path selection, closure | (your main agent) | (model) |
| Implementer | File/code changes | (your code agent) | (model) |
| Reviewer | Quality checks | (your review agent) | (model) |
| Policy Auditor | Governance/rule review | (your audit agent) | (model) |

> Same model can serve different roles, but the same Agent must not both write and review the same piece of work.
```

### 6.3 Define Change Routing

```markdown
## Change Routing

### Risk Levels

| Level | Criteria | Examples |
|-------|---------|----------|
| **Low** | No behavior change, no permission change, no rule change | Formatting, comments, logging |
| **Medium** | Changes operational guidance or defaults | Doc rewrite affecting behavior, retrieval policy tuning |
| **High** | Changes rules, role boundaries, or approval requirements | AGENTS.md changes, permission boundaries, automation scope |

### Routing Matrix

| Change Type | Risk | Route | Human Approval? |
|------------|------|-------|----------------|
| Implementation | Low | Implementer → Reviewer | No |
| Implementation | Medium | Implementer → Reviewer | Yes if default behavior changes |
| Implementation | High | Implementer → Reviewer → Policy Auditor | Yes |
| Documentation | Low | Implementer → Reviewer | No |
| Documentation | Medium | Implementer → Reviewer → Policy Auditor | Yes if operational meaning changes |
| Governance/Rules | Medium | Policy Auditor → Human approval → Execute | Yes |
| Governance/Rules | High | Consensus Peer → Policy Auditor → Human approval → Execute | Yes |
| Core Files | Any | Policy Auditor → Explicit human authorization → Execute | Always |

### Core File List

Modifications to these files MUST go through Policy Auditor review and human authorization:
- `AGENTS.md`
- `USER.md`
- `MEMORY.md`
- `HEARTBEAT.md`
- `IDENTITY.md`
- (add your workspace-specific files)
```

### 6.4 Degraded Operation Rules

```markdown
## Degraded Operation

When fewer Agents are available than the deployment shape requires:

### Declaration (must include in execution records)
- Current available Agent count
- Which roles are merged
- Independence loss description
- Impact scope

### Rules
- Low-risk work: May continue, log degraded status
- Medium-risk work: Requires human confirmation to proceed
- High-risk / governance work: Must wait for Agent recovery or get human approval
- Never pretend self-review equals independent review
```

### 6.5 Single-Agent Degraded Mode

With only 1 Agent, Phase 2's core is **human substitution**:

```markdown
## Single-Agent Governance

### Can Complete Independently
- Low-risk implementation work
- Logging, formatting, information queries

### Requires Human Approval
- Any AGENTS.md modification
- Any change to default behavior
- Any change to automation scope

### Execution Declaration
When executing medium-risk or higher work, Agent should proactively state:
"⚠️ Single-agent operation. This change lacks independent review. Please confirm before I proceed."
```

### 6.6 Validate Phase 2

| Test | Action | Expected Behavior | ☐ |
|------|--------|-------------------|---|
| T4 | Ask Agent to modify AGENTS.md | Goes through review process (or requests human authorization) | |
| T5 | Ask Agent to fix a simple bug | Follows Implementer → Reviewer flow | |
| T6 | Request governance change with only 1 Agent | Declares degraded operation, requests human approval | |

---

## 7. Phase 3 — Retrieval Routing

> Estimated time: 20-40 minutes
> Prerequisites: Phase 1 complete
> Goal: Agent selects the right retrieval method per question type instead of blindly using memory_search

### 7.1 Understand the Core Problem

**Symptoms without retrieval routing**:
- Ask "what are the rules?" → Agent searches memory, finds old version of rules
- Ask "current system state" → Agent finds a month-old topic card
- Ask "user preferences" → Agent finds noisy chat log fragments

**Retrieval routing in a nutshell**: Different questions ask different sources first.

### 7.2 Create Retrieval Policy File

**Create `docs/retrieval-policy.md`:**

```markdown
# Retrieval Policy

Updated: 2026-03-25

## Query Classification & Routing

Agent classifies the question type first, then follows the corresponding retrieval path.

### Type 1: Governance / Rules
> "Who is responsible for what" "Does this need approval" "What are the core files"

Path:
1. AGENTS.md
2. Safety restriction docs
3. Do NOT use memory_search

### Type 2: System State
> "Current system state" "Health status" "Is everything running"

Path:
1. memory/SYSTEM_OVERVIEW.md
2. memory/health/current-health.json
3. memory_search only as supplementary

### Type 3: Runtime
> "Which model is active" "Which crons are running"

Path:
1. Runtime snapshot (if exists)
2. Bridge files
3. memory_search only for discovery

### Type 4: User Preferences
> "What style does the user prefer" "Any special requirements"

Path:
1. USER.md
2. Preference sections in MEMORY.md
3. memory_search only as secondary
4. ⚠️ Chat log search results are noisy — do not trust directly

### Type 5: Historical Trace
> "How was this fixed last time" "When was this feature added"

Path:
1. memory_search to discover candidates
2. Open and **verify the source file**
3. Never conclude from search snippets alone

### Type 6: Maintenance
> "Which files are stale" "What should be maintained"

Path:
1. Run freshness-check.py
2. Check working-set config
3. Usually no need for memory_search

## Semantic Search Trust Levels

| Query Type | memory_search Role | Trust Level |
|-----------|-------------------|-------------|
| Governance / Rules | Do not use | — |
| System State | Supporting recall | Medium (cross-verify with bridge) |
| Runtime | Discovery | Low |
| User Preferences | Secondary | Low (chat log noise) |
| Historical Trace | Discovery | Medium (must verify with files) |
| Maintenance | Usually unnecessary | — |
```

### 7.3 Activate in AGENTS.md

Add a short reference (don't duplicate the entire policy file):

```markdown
## Retrieval Policy

Before answering, classify the question type (governance / state / runtime / preference / historical / maintenance), then follow the routing table in `docs/retrieval-policy.md`.

Key rules:
- Governance questions: read AGENTS.md directly, do not use memory_search
- State questions: read bridge files first, do not trust old topic cards
- Historical questions: after memory_search discovery, always open and verify the source file
```

### 7.4 Dealing With Unstable Semantic Search

If your semantic search backend (e.g. qmd) frequently returns irrelevant results:

| Symptom | Likely Cause | Diagnosis |
|---------|-------------|-----------|
| Same files returned for every query | Stale index cache | `openclaw memory status --json` |
| Search results inconsistent with file contents | Path misalignment (split-brain) | Run `scripts/semantic-runtime-check.sh` |
| New files not found | Embedding queue backlog | Check pending embedding count |

See `references/semantic-search-diagnostics.md` for the full diagnostic workflow.

### 7.5 Validate Phase 3

| Test | Question | Expected Behavior | ☐ |
|------|---------|-------------------|---|
| T7 | "What's the core file list in AGENTS.md?" | Reads AGENTS.md directly | |
| T8 | "Current system state?" | Reads SYSTEM_OVERVIEW.md first | |
| T9 | "What did we do last Friday?" | memory_search → verify source file | |
| T10 | "What tone does the user prefer?" | Reads USER.md, not chat logs | |

---

## 8. Phase 4 — Freshness Automation

> Estimated time: 30-45 minutes
> Prerequisites: Phase 1 complete
> Goal: Stale key files become visible automatically, not discovered only after the Agent gives wrong answers

### 8.1 Design Your Working Set

Working set = the minimum set of files that must stay fresh. Not "all important files" but "files whose staleness directly causes the Agent to make mistakes."

#### Evaluation Criteria

| File | Include? | Reason |
|------|---------|--------|
| memory/health/current-health.json | ✅ | Stale → Agent reports wrong health |
| memory/SYSTEM_OVERVIEW.md | ✅ | Stale → Agent answers with outdated state |
| AGENTS.md | ✅ | After changes, outdated = Agent behavior mismatches rules |
| MEMORY.md | ⚠️ Depends | Long-term memory updating slowly is normal; 7-day threshold |
| docs/old-plan.md | ❌ | Historical doc, doesn't need freshness tracking |

#### Threshold Design

Different file types get different staleness thresholds:

| Group | Threshold | File Types | Reason |
|-------|-----------|-----------|--------|
| `health` | 3 days | Health status, consistency checks | Ages fastest |
| `bridge` | 3 days | SYSTEM_OVERVIEW and current-state files | Directly affects "how are things now" answers |
| `runtime` | 3 days | Runtime snapshots | Runtime state changes quickly |
| `entry` | 5 days | AGENTS.md, entry indexes | Changes less often but high impact when stale |
| `structured` | 7 days | MEMORY.md, TOPICS_INDEX | Long-term info updating slowly is normal |

Adjust based on your workspace's update frequency. Core principle: **better to alert too often than miss a critical stale file.**

### 8.2 Create Configuration

```bash
mkdir -p docs
```

**Create `docs/working-set.json` (modify paths to match your workspace):**

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

**Adding more files**: Each entry needs a `group` (maps to threshold) and `path` (relative to workspace root).

### 8.3 Run Initial Check

```bash
python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
  --root ~/.openclaw/workspace \
  --config docs/working-set.json
```

#### Understanding Output

```
Freshness check @ 2026-03-25 18:00 +0000
Thresholds:
- health: 3d
- bridge: 3d
- entry: 5d
- structured: 7d

[OK] memory/health/current-health.json
  group=health threshold=3d age=0.25d source=updatedAt via json seen_at=2026-03-25T12:00:00+00:00
[WARN] memory/SYSTEM_OVERVIEW.md
  group=bridge threshold=3d age=4.50d source=header-date via header seen_at=2026-03-21T00:00:00+00:00
[OK] AGENTS.md
  group=entry threshold=5d age=1.20d source=mtime via filesystem seen_at=2026-03-24T06:00:00+00:00
[OK] MEMORY.md
  group=structured threshold=7d age=3.00d source=header-date via header seen_at=2026-03-22T00:00:00+00:00

Summary: 3 OK, 1 WARN, total 4
Warned files:
- memory/SYSTEM_OVERVIEW.md (4.50d > 3d)
```

- `[OK]` = within threshold
- `[WARN]` = exceeded threshold, needs update
- `source` = where the timestamp came from (json field / markdown header / filesystem mtime)

#### Timestamp Detection Rules

The script extracts "last updated time" in this priority:

1. JSON files: reads `updatedAt` field
2. Markdown files: reads `Updated:`, `Last updated:`, `Last verified:` headers in first 30 lines
3. Fallback: filesystem mtime

**Recommendation**: Manually maintain `Updated: YYYY-MM-DD` headers in files. More reliable than mtime.

### 8.4 Set Up Automation

#### Option A: HEARTBEAT.md (Recommended)

Add to your HEARTBEAT.md:

```markdown
# Freshness check
#
# Every heartbeat, run:
#   python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
#     --root ~/.openclaw/workspace --config docs/working-set.json --json
#
# Rules:
# - If WARN items exist: tell the user which files are stale, suggest updating
# - If all OK: no report needed (pass silently)
# - If script fails: report the execution error
```

#### Option B: Cron Job

```bash
# Check once daily at 9 AM
openclaw cron add --schedule "0 9 * * *" --command "python3 skills/openclaw-workspace-governance/scripts/freshness-check.py --root ~/.openclaw/workspace --config docs/working-set.json"
```

### 8.5 Handling WARN Alerts

1. **Open the warned file**
2. **Check if the content is still accurate**
3. If accurate: just update the `Updated:` date
4. If inaccurate: update content + date
5. If the file no longer needs freshness tracking: remove from `working-set.json`

### 8.6 Validate Phase 4

| Test | Action | Expected Result | ☐ |
|------|--------|----------------|---|
| T11 | Run freshness-check.py | Normal output, identifies stale files | |
| T12 | Intentionally don't update SYSTEM_OVERVIEW for 4 days | Checker reports WARN | |
| T13 | Update the file, run again | WARN disappears, becomes OK | |

---

## 9. Script Reference

### freshness-check.py

| Item | Details |
|------|---------|
| Purpose | Check freshness of working-set files |
| Dependencies | Python 3.6+, no third-party packages |
| Input | `--root` (workspace root) + `--config` (JSON config) |
| Output | Human-readable report (default) or JSON (`--json`) |
| Exit codes | 0 = all OK, 2 = has WARN |

### semantic-runtime-check.sh

| Item | Details |
|------|---------|
| Purpose | Semantic search runtime diagnostics |
| Dependencies | Bash; your semantic CLI must support `<cmd> status` and `<cmd> query "<text>"` |
| Input | `--cmd` (CLI path) + `--config-home` + optional `--cache-home` + `--query` (repeatable) |
| Exit codes | 0 = all passed, 2 = check(s) failed |

### doc-status-scan.py

| Item | Details |
|------|---------|
| Purpose | Scan docs directory for status label distribution |
| Dependencies | Python 3.6+, no third-party packages |
| Limitation | Only scans first 12 lines of each file |
| Input | `--root` (scan directory) + optional `--include-unlabeled` |

---

## 10. Reference Document Index

For deep reading. Not required during deployment — consult when you hit specific issues.

| Document | When to Read |
|----------|-------------|
| `references/multi-agent-governance.md` | Designing multi-agent role separation |
| `references/source-of-truth-layering.md` | Unsure which layer should win in a conflict |
| `references/query-class-routing.md` | Tuning retrieval strategy |
| `references/live-vs-historical-docs.md` | Batch-labeling documents |
| `references/freshness-discipline.md` | Designing working set and thresholds |
| `references/semantic-search-diagnostics.md` | Semantic search returning strange results |
| `references/completion-criteria.md` | Deciding whether an improvement phase can be closed |

---

## 11. Post-Deployment Validation

### Comprehensive Checklist

After completing all deployed Phases:

| # | Test | Expected | Result |
|---|------|---------|--------|
| 1 | Ask current state | Reads bridge files | ☐ |
| 2 | Ask about rules | Reads AGENTS.md | ☐ |
| 3 | Ask about history | memory_search → file verification | ☐ |
| 4 | Ask about freshness | Runs freshness-check or reads working set | ☐ |
| 5 | Request AGENTS.md modification | Goes through approval flow | ☐ |
| 6 | Reference a historical-labeled file | Not treated as current truth | ☐ |

**Criteria**:
- All PASS = deployment successful ✅
- 1-2 FAIL = check configuration of the corresponding Phase
- Majority FAIL = verify the layering rules in AGENTS.md are being read

### Update Health File

After deployment, update health status:

```json
{
  "updatedAt": "(current time)",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "note": "Source-of-truth layering established, change routing defined"
    },
    "retrieval": {
      "status": "ok",
      "note": "Retrieval routing policy activated"
    },
    "maintenance": {
      "status": "ok",
      "workingSetDefined": true,
      "note": "freshness-check configured"
    }
  }
}
```

---

## 12. FAQ

**Q: Can I change the Phase order?**
A: Phase 1 must come first. Phases 2/3/4 can be selected and ordered based on your needs.

**Q: Deploy all Phases at once or incrementally?**
A: Incremental is recommended. Do Phase 1 first, observe for a few days, then proceed.

**Q: Agent behavior didn't change after deployment?**
A: Most common cause: the new rules were added too far down in AGENTS.md. Agent context is limited — content near the bottom may not be read. Move key rules higher.

**Q: Can I use only some features?**
A: Absolutely. The minimum effective unit is Phase 1 (source-of-truth layering). Everything else is incremental.

**Q: How to roll back?**
A: Delete the bridge files and configs you added, remove the new sections from AGENTS.md. If you have the backup, just restore it.

**Q: How big should the working set be?**
A: As small as possible. 4-8 files is typical. If you have more than 15, ask yourself whether some files don't actually need freshness tracking.
