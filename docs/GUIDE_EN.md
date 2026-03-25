# OpenClaw Workspace Governance — Detailed Guide

> 🚨 **WARNING** — This is NOT a standard tool skill. It is a system-level architecture upgrade. Back up your workspace before deployment!

---

## Table of Contents

- [What Is This](#what-is-this)
- [System Capabilities](#system-capabilities)
- [When to Use](#when-to-use)
- [Non-goals](#non-goals)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Core Workflow](#core-workflow)
- [Scripts](#scripts)
- [Reference Index](#reference-index)
- [Example Templates Index](#example-templates-index)
- [FAQ](#faq)

---

## What Is This

A governance skill for **complex OpenClaw workspaces**. It's not about adding more process — it's about taming complexity that already exists:

- Reduce drift (governance / doc / retrieval / maintenance / phase drift)
- Clarify current truth (which layer wins when sources disagree)
- Shrink live-doc surface (not every document needs to stay active)
- Make semantic retrieval controllable (assign trust by query class)
- Make maintenance sustainable (working set + freshness checks)

---

## System Capabilities

### Multi-Agent Architecture

Supervisor-centric dispatch with separation of writing and reviewing:

- **Coordinator** — Frames the work, chooses path, closes the loop
- **Implementer** — Makes operational/file/script changes
- **Reviewer** — Checks implementation quality, catches mistakes
- **Policy Auditor** — Reviews governance, process, and rule changes
- **Consensus Peer** — Optional peer review before risky structural changes

Supports flexible deployment from 2-agent to 5-agent shapes. Core rule: never let the same role both implement and perform final review on risky work.

### Layered Memory System

Progressive distillation from chaos to clarity:

| Layer | Purpose | Typical Files |
|-------|---------|--------------|
| Canon | Rules, approvals, hard boundaries | AGENTS.md, governance docs |
| Bridge | Answers "what is true now?" | SYSTEM_OVERVIEW.md |
| Health | Operational health & consistency | current-health.json |
| Runtime Snapshot | Point-in-time runtime truth | runtime-snapshot.md |
| Structured Memory | Topic cards with medium durability | TOPICS_INDEX.md |
| Long-term Memory | Durable facts only | MEMORY.md |
| Historical | Background, prior phases, audits | Archived plans & docs |

When two layers disagree, don't average them — decide which layer wins.

### Semantic Retrieval Routing

Different question types hit different layers first:

| Query Class | Primary Layer | Semantic Search Role |
|------------|--------------|---------------------|
| Governance / Rules | Canon | Background only |
| System State | Bridge + Health | Supporting recall |
| Runtime | Runtime Snapshot | Discovery only |
| User Preference | Curated Profile | Secondary only |
| Historical Trace | Evidence + Verification | Discovery layer |
| Maintenance | Working Set | Usually unnecessary |

### Approval & Governance

- Core file modifications require independent review and human authorization
- Built-in anti-laziness protocol (code delivery contract): requires complete delivery, no TODO placeholders
- Governance/policy/process changes must go through Policy Auditor
- Degraded operation must declare independence loss explicitly

### Freshness Discipline

- Define a minimal working set (files that must stay fresh)
- Group thresholds by file type (shorter for health/bridge, longer for topic cards)
- Automated checker turns "someone should update this" into a repeatable check

### Completion Criteria

Every phase explicitly classifies work into three states:
- **Landed** — Capability exists, is validated, and is wired into real operation
- **Improving** — Works but needs longer observation
- **Deferred** — Intentionally postponed, not forgotten

Close the mainline when it's complete enough. Don't let phases sprawl indefinitely.

---

## When to Use

Use when your workspace shows these signals:

- ✅ Multiple agents or role-based execution paths exist
- ✅ Growing docs / memory / state layers
- ✅ "What is true now?" is getting hard to answer
- ✅ Semantic search is useful but not uniformly trustworthy
- ✅ Too many live docs, no clear working set, maintenance out of control

⚠️ This is an advanced skill. If your workspace is still simple (1 agent, a few files), you don't need this.

---

## Non-goals

This skill does **NOT**:

- Automate approval decisions or policy changes
- Publish private role lore or internal organization culture
- Ship personal memory, daily logs, or private operational history
- Provide full automated multi-agent orchestration in v1
- Keep every old document live "just in case"

It provides a governance playbook, not an infinite automation engine.

---

## Installation

### Option 1: Via ClawHub

```bash
clawhub install openclaw-workspace-governance
```

### Option 2: Direct Clone

```bash
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

### After Installation

1. **Back up your workspace** (important!)
2. Read `SKILL.md` — this is the main instruction file your Agent will read
3. Start with the 6-step Quick Start, don't jump into editing files
4. Read `references/` docs as needed for deeper guidance

### Package Structure

```
SKILL.md                          # Main skill instructions (Agent entry point)
docs/
  GUIDE_ZH.md                     # Chinese detailed guide
  GUIDE_EN.md                     # English detailed guide (this file)
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

---

## Quick Start

Don't rush into editing files. The mainline is just 6 steps:

1. **Classify the problem** — Identify which drift type you're facing (governance / current-state / retrieval / doc / maintenance / phase)
2. **Choose the governance path** — Separate implementation changes, documentation changes, and governance changes
3. **Find the authoritative layer** — Determine which source-of-truth layer should answer your question
4. **Tighten doc boundaries** — Label docs (live / historical / needs-refresh), define a minimal live subset
5. **Validate with real questions** — Test at least one system-state, governance, maintenance, and historical question
6. **Close on purpose** — When the mainline is complete enough, move to maintenance observation

If the workspace is messy, **reduce ambiguity first, don't add more process**.

---

## Core Workflow

The full 9-step workflow is in `SKILL.md`. Key points:

1. Classify the drift type (6 types)
2. Route changes by type (implementation / documentation / governance)
3. Build source-of-truth layering (7 layers with explicit precedence)
4. Classify queries before choosing retrieval order (7 query classes)
5. Mark docs with role boundaries (live / historical / needs-refresh / special-case)
6. Establish working set and freshness checks
7. Diagnose semantic-search runtime (path alignment, split-brain, per-class trust)
8. Validate with representative questions (PASS / PASS WITH ESCALATION / FAIL)
9. Close phases on purpose (landed / improving / deferred)

---

## Scripts

### freshness-check.py

Check freshness of files in your working set.

```bash
python3 scripts/freshness-check.py \
  --root . \
  --config assets/examples/working-set.example.json

# JSON output
python3 scripts/freshness-check.py \
  --root . \
  --config assets/examples/working-set.example.json \
  --json
```

Config file defines grouped thresholds and file list. See `assets/examples/working-set.example.json`.

### semantic-runtime-check.sh

Semantic search runtime diagnostic. Assumes your CLI supports `<cmd> status` and `<cmd> query "<text>"` subcommands.

```bash
bash scripts/semantic-runtime-check.sh \
  --cmd /path/to/your-semantic-cli \
  --config-home ~/.config/your-backend \
  --cache-home ~/.cache/your-backend \
  --query "system state" \
  --query "weekly review status"
```

### doc-status-scan.py

Scan a docs directory for status labels. Note: only scans the first 12 lines of each file.

```bash
python3 scripts/doc-status-scan.py --root docs
python3 scripts/doc-status-scan.py --root docs --json
python3 scripts/doc-status-scan.py --root docs --include-unlabeled
```

---

## Reference Index

| Document | Content |
|----------|---------|
| `references/multi-agent-governance.md` | Role separation, change routing, degraded operation, approval boundaries |
| `references/source-of-truth-layering.md` | 7-layer fact model, conflict resolution rules |
| `references/query-class-routing.md` | 7 query classes, retrieval priorities, semantic search roles |
| `references/live-vs-historical-docs.md` | Doc status labeling, minimal live subset, special-case safety docs |
| `references/freshness-discipline.md` | Working set, grouped thresholds, refresh order |
| `references/semantic-search-diagnostics.md` | Path alignment, split-brain, per-class trust judgment |
| `references/completion-criteria.md` | Landed/Improving/Deferred, closure criteria, maintenance observation |

---

## Example Templates Index

| Template | Purpose |
|----------|---------|
| `assets/examples/working-set.example.json` | Config file template for freshness-check.py |
| `assets/examples/working-set.example.md` | Working set documentation template |
| `assets/examples/current-health.example.json` | Health state JSON template |
| `assets/examples/system-state-index.example.md` | System state unified index template |

---

## FAQ

**Q: I only have 1 Agent. Can I still use this?**
A: Yes. But explicitly state where a second review would normally be needed — don't pretend self-review is equivalent. High-risk changes require human approval.

**Q: How does this relate to AGENTS.md?**
A: This skill provides a generic governance framework. Your AGENTS.md is your workspace's own governance rules (canon layer). They complement each other.

**Q: What if semantic search isn't reliable?**
A: Don't give a blanket trust label. Evaluate trust per query class, and downgrade unreliable classes to "supporting recall only" or "discovery only". See `references/semantic-search-diagnostics.md`.

**Q: Can I adopt this partially?**
A: Absolutely. Read references as needed. You don't have to deploy everything at once. Starting with "classify the problem + build source-of-truth layering" is the most effective first step.
