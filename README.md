# skill-portfolio-audit

[![GitHub release](https://img.shields.io/github/v/release/wan-huiyan/skill-portfolio-audit)](https://github.com/wan-huiyan/skill-portfolio-audit/releases) [![Claude Code](https://img.shields.io/badge/Claude_Code-skill-orange)](https://claude.com/claude-code) [![license](https://img.shields.io/github/license/wan-huiyan/skill-portfolio-audit)](LICENSE) [![last commit](https://img.shields.io/github/last-commit/wan-huiyan/skill-portfolio-audit)](https://github.com/wan-huiyan/skill-portfolio-audit/commits)

Maintaining 10–20 Claude Code skills accumulates drift — inconsistent badges, broken links, stale cost tables. This skill orchestrates a 4-reviewer adversarial panel, validates findings with plan-review-integrator, and implements all fixes in parallel.

## Installation

**Claude Code (plugin install — recommended):**
```bash
# Add the marketplace, then install the plugin
claude plugin marketplace add wan-huiyan/skill-portfolio-audit
claude plugin install skill-portfolio-audit@wan-huiyan-skill-portfolio-audit
```

**Claude Code (git clone):**
```bash
git clone https://github.com/wan-huiyan/skill-portfolio-audit.git ~/.claude/skills/skill-portfolio-audit
```

**Cursor** (2.4+):
```bash
# Per-project rule (most reliable)
mkdir -p .cursor/rules
# Copy plugins/skill-portfolio-audit/SKILL.md content into .cursor/rules/skill-portfolio-audit.mdc with alwaysApply: true

# Or via npx skills CLI
npx skills add wan-huiyan/skill-portfolio-audit --global
```

## Quick Start

In a Claude Code session:

```
/skill-portfolio-audit
```

Claude will refresh the skill-sync registry, launch 4 parallel reviewers across all your skill READMEs, validate findings through plan-review-integrator, and execute prioritized fixes via parallel subagents.

## Pipeline

```
1. skill-sync init          → refresh registry (merges local + GitHub file lists)
2. agent-review-panel       → 4 parallel reviewers on all READMEs (Documentation mode)
3. plan-review-integrator   → validate findings, catch reviewer errors, prioritize
4. Parallel subagents       → implement P0/P1/P2/P3 by priority tier
```

**Reviewer panel (Documentation mode):**

| Reviewer | Focus | Weight |
|---|---|---|
| Clarity Editor | Hooks, quick-start, scannable structure | 60% |
| Technical Accuracy | Badge vs release match, cost tables, links | 30% |
| Completeness Checker | Missing sections checklist | 40% |
| Devil's Advocate | Would a skeptical dev install this? | 20% |

**Implementation tiers:**
- P0 (today, ~30 min): factual errors, broken links, version mismatches
- P1 (this week): hook rewrites, install commands, link fixes
- P2 (this month): Requirements/Limitations template rollout
- P3 (ongoing): "Why a skill?" sections, citation trimming

See [SKILL.md](SKILL.md) for full implementation details including cross-checks, override patterns, and the standard badge row template.

## Requirements

- Claude Code v1.x+
- `/usr/local/bin/gh` CLI (GitHub CLI) authenticated with your account
- All skill repos must be accessible on GitHub (authored or public)
- Companion skills: agent-review-panel, plan-review-integrator, skill-sync

## Limitations

- Requires authored repos — forks are read-only and excluded from write operations
- READMEs must exist on GitHub (not just locally) for the panel to review them
- Takes 30–90 min for a portfolio of ~18 skills depending on panel depth
- Background subagents cannot run Bash/Write; implementation agents must run foreground

## Related Skills

- [agent-review-panel](https://github.com/wan-huiyan/agent-review-panel) — the 4-reviewer adversarial panel used in Phase 2
- [plan-review-integrator](https://github.com/wan-huiyan/plan-review-integrator) — validates and prioritizes reviewer findings in Phase 3
- [skill-sync](https://github.com/wan-huiyan/skill-sync) — keeps local and GitHub skill files in sync
- [publish-skill](https://github.com/wan-huiyan/publish-skill) — publishes individual skills to GitHub with release creation

## Version History

| Version | Date | Notes |
|---|---|---|
| v1.0.0 | 2026-04-03 | Initial release: full portfolio audit pipeline with 4-reviewer panel, plan-review-integrator validation, and parallel implementation |
