<p align="center">
  <img src="https://api.iconify.design/fluent-emoji:brain.svg" width="120" height="120" alt="Brain Icon" />
</p>

<h1 align="center">OpenClaw Workspace Governance</h1>

<p align="center">
  <strong>A System-Level Governance Framework for Complex OpenClaw Workspaces</strong><br/>
  <em>面向复杂 OpenClaw 工作区的系统级治理 Skill</em>
</p>

---

> 🚨 **This is NOT a standard tool skill — it is a system-level architecture upgrade.**
> Back up your workspace before deployment.

## What It Does

- **Multi-Agent Governance** — role separation, write-review split, routing by change type & risk tier
- **Source-of-Truth Layering** — canon → bridge → health → snapshot → memory → historical
- **Query-Class Routing** — different question types hit different layers first
- **Freshness Discipline** — working set + grouped thresholds + automated checks
- **Semantic Search Diagnostics** — path alignment, split-brain detection, per-class trust
- **Completion Criteria** — landed / improving / deferred / close the phase

## When to Use

When your workspace already has real complexity:
- Multiple agents or role-based execution paths
- Growing docs / memory / state layers
- "What is true now?" is getting hard to answer
- Semantic search is useful but not uniformly trustworthy
- Too many live docs, no clear working set

⚠️ This is an **advanced** skill, not a beginner starter pack.

## Package Structure

```
SKILL.md                          # Main skill instructions (中文)
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

## Install

```bash
# Via ClawHub (if published)
clawhub install openclaw-workspace-governance

# Or clone directly
git clone https://github.com/heishiqing/openclaw-workspace-governance.git \
  ~/.openclaw/workspace/skills/openclaw-workspace-governance
```

## License

MIT
