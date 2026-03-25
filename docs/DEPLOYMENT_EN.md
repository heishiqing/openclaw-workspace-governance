# Deployment Guide (English)

> This guide walks you through deploying workspace governance **from scratch**. Follow the steps in order — each step includes exact file contents to create.
>
> ⚠️ Back up your workspace before starting: `cp -r ~/.openclaw/workspace ~/.openclaw/workspace-backup-$(date +%Y%m%d)`

---

## Table of Contents

- [Phase 1: Minimal Deployment (30 Minutes)](#phase-1-minimal-deployment-30-minutes)
- [Phase 2: Multi-Agent Governance](#phase-2-multi-agent-governance)
- [Phase 3: Semantic Search & Retrieval Routing](#phase-3-semantic-search--retrieval-routing)
- [Phase 4: Freshness Automation](#phase-4-freshness-automation)
- [Post-Deployment Validation](#post-deployment-validation)
- [Troubleshooting](#troubleshooting)

---

## Phase 1: Minimal Deployment (30 Minutes)

Goal: Let your Agent distinguish "what is true now" from "historical background."

### Step 1: Create Bridge Files

Bridge files answer "what is true now?" Create these in your workspace:

```bash
mkdir -p memory/health
```

**File: `memory/SYSTEM_OVERVIEW.md`**

```markdown
# System Overview

Updated: 2026-03-25

## Current State

- Agent count: 1 (main Agent)
- Memory backend: (fill yours, e.g. qmd / files / none)
- Primary channel: (fill yours, e.g. Discord / Telegram / Slack)
- OpenClaw version: (fill yours, run `openclaw --version`)

## Active Work

- (list current tasks in progress)

## Known Issues

- (list known unfixed issues, or "none")
```

**File: `memory/health/current-health.json`**

```json
{
  "updatedAt": "2026-03-25T12:00:00+00:00",
  "overall": "healthy",
  "domains": {
    "governance": {
      "status": "ok",
      "note": "Single agent, no multi-role governance needed"
    },
    "memory": {
      "status": "ok",
      "note": "Basic memory system running"
    },
    "automation": {
      "status": "ok",
      "note": "Cron/heartbeat normal"
    }
  }
}
```

### Step 2: Label Old Documents

Find old documents in your workspace that **no longer represent current truth**. Add a status label at the top:

```markdown
Status: historical-reference

# Old Deployment Plan
...
```

Available labels:
- `historical-reference` — Valuable for context but doesn't represent current truth
- `needs-refresh` — Partially outdated, read with caution
- No label = treated as active/live by default

**Action**: Go through your `docs/` directory and add labels to old files. When in doubt, use `needs-refresh`.

### Step 3: Add Source-of-Truth Layering to AGENTS.md

Add this section to your `AGENTS.md` (or equivalent Agent instruction file):

```markdown
## Source-of-Truth Layering

When answering questions, search in this order:
1. **Canon layer**: AGENTS.md, safety restriction docs → answers "what are the rules?"
2. **Current-state layer**: memory/SYSTEM_OVERVIEW.md, memory/health/ → answers "what is true now?"
3. **Memory layer**: MEMORY.md, memory/topics/ → answers "what happened before?"
4. **Historical layer**: Files marked `historical-reference` → background context only

When two layers conflict, the higher layer wins. Files marked `historical-reference` must not be used as current-state evidence.
```

### Step 4: Validate

Ask your Agent these questions and check the behavior:

| Question | Expected Behavior |
|----------|------------------|
| "What's the current system state?" | Reads `memory/SYSTEM_OVERVIEW.md` first, not pieced from old docs |
| "Is this old plan still valid?" (pointing to a historical-labeled file) | Answers "this is a historical document, see SYSTEM_OVERVIEW for current state" |

**Phase 1 done when**: Agent can distinguish current state from historical context and stops using old docs to answer "what's happening now."

---

## Phase 2: Multi-Agent Governance

> If you only have 1 Agent, you can skip this Phase, but read the principles at least once.

Goal: Establish write-review separation. Prevent Agent from modifying its own rules and approving them.

### Step 1: Define Roles

Add to your `AGENTS.md`:

```markdown
## Role Definitions

| Role | Responsibility | Assigned To |
|------|---------------|-------------|
| Coordinator | Task decomposition, path selection, closure | (your main Agent) |
| Implementer | File/code/script changes | (your code Agent) |
| Reviewer | Quality checks | (your review Agent) |
| Policy Auditor | Governance/rule/process change review | (your audit Agent) |

### Core Rules
- High-risk work must not have the same role both implement and perform final review
- Governance/rule changes must go through Policy Auditor
- Core file modifications (AGENTS.md, MEMORY.md, etc.) require explicit human authorization
```

### Step 2: Define Change Routing

Add to your `AGENTS.md`:

```markdown
## Change Routing

| Change Type | Route |
|------------|-------|
| Normal implementation (bug fix, feature) | Coordinator → Implementer → Reviewer |
| Documentation change | Coordinator → Implementer → Reviewer |
| Governance/rule/process change | Coordinator → Policy Auditor → Human approval → Execute |
| Core file modification | Coordinator → Policy Auditor → Explicit human authorization → Execute |
```

### Step 3: Degraded Operation Declaration

If you only have 2 Agents or temporarily only 1 available, include in execution records:

```markdown
## Degraded Operation Declaration
- Available Agents: 1 (main only)
- Independence loss: No independent review
- Impact: High-risk changes require human approval
```

**Phase 2 done when**: Agent no longer self-modifies rules and self-approves. Governance changes have a clear review path.

---

## Phase 3: Semantic Search & Retrieval Routing

Goal: Make the Agent choose the right retrieval method based on question type.

### Step 1: Create Retrieval Policy File

**File: `docs/retrieval-policy.md`**

```markdown
# Retrieval Policy

Updated: 2026-03-25

## Query-Class Routing

| Query Class | Search First | Semantic Search Role |
|------------|-------------|---------------------|
| Governance / Rules | AGENTS.md, safety docs | Do not use |
| System State | memory/SYSTEM_OVERVIEW.md, memory/health/ | Supporting recall |
| Runtime Info | docs/runtime-snapshot.md (if exists) | Discovery only |
| User Preferences | USER.md, preference sections in MEMORY.md | Secondary only |
| Historical Trace | memory/ logs → file verification | Discovery layer |
| Maintenance | Working set file list | Usually unnecessary |

## Rules
- Semantic search (memory_search) is NOT the primary source for governance and rule questions
- Current-state questions read bridge files first, not old topic cards
- Historical questions use semantic search for discovery, but verify against source files
```

### Step 2: Reference in AGENTS.md

```markdown
## Retrieval Policy

Before answering a question, classify the question type first, then follow the routing table in `docs/retrieval-policy.md`.
Do not blindly use memory_search for every question.
```

**Phase 3 done when**: Agent reads AGENTS.md for governance questions (not memory search); reads bridge files for state questions.

---

## Phase 4: Freshness Automation

Goal: Make key file staleness automatically visible.

### Step 1: Create Your Working Set Config

Copy the example and customize for your workspace:

```bash
cp skills/openclaw-workspace-governance/assets/examples/working-set.example.json \
   docs/working-set.json
```

Edit `docs/working-set.json` with your actual file paths:

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

Explanation:
- Numbers in `groups` are day thresholds (alerts when exceeded)
- `health` / `bridge`: recommend 3 days
- `entry`: recommend 5 days
- `structured`: recommend 7 days

### Step 2: Run a Check

```bash
python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
  --root ~/.openclaw/workspace \
  --config docs/working-set.json
```

Expected output:

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

Summary: 3 OK, 1 WARN, total 4
Warned files:
- memory/SYSTEM_OVERVIEW.md (4.50d > 3d)
```

### Step 3: (Optional) Add to Heartbeat

Add to your `HEARTBEAT.md`:

```markdown
# Freshness check
# Every heartbeat, run:
#   python3 skills/openclaw-workspace-governance/scripts/freshness-check.py \
#     --root ~/.openclaw/workspace --config docs/working-set.json
# If WARN files exist, notify the user which files are stale
```

**Phase 4 done when**: `freshness-check.py` runs successfully and stale files are automatically visible.

---

## Post-Deployment Validation

After completing the Phases above, test your Agent with these questions:

| # | Question | Expected Behavior | Result |
|---|---------|-------------------|--------|
| 1 | "What's the current system state?" | Reads SYSTEM_OVERVIEW.md first | ☐ PASS / ☐ FAIL |
| 2 | "What are the review rules in AGENTS.md?" | Reads AGENTS.md directly, not via semantic search | ☐ PASS / ☐ FAIL |
| 3 | "What did we do last week?" | Uses memory_search to discover, then verifies with files | ☐ PASS / ☐ FAIL |
| 4 | "Which files need updating?" | Runs freshness-check.py or reads working set | ☐ PASS / ☐ FAIL |

All 4 PASS = deployment successful ✅

---

## Troubleshooting

**Agent still uses old docs for current-state answers**
→ Check that old docs are labeled `historical-reference`
→ Check that source-of-truth layering rules are in AGENTS.md

**freshness-check.py reports "file missing"**
→ Check paths in `working-set.json` are correct (relative to `--root`)

**Agent modified AGENTS.md without review**
→ Check that AGENTS.md includes "core file modifications require human authorization"
→ With only 1 Agent, this rule requires human oversight to enforce

**Don't know which Phase to start with**
→ Start with Phase 1. It's the foundation for everything and takes 30 minutes.
→ Phases 2-4 are optional and can be deployed incrementally.
