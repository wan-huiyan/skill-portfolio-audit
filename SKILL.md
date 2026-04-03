---
name: skill-portfolio-audit
description: |
  Run a full quality audit across a portfolio of Claude Code skills and implement
  all findings. Use when: (1) you want to standardize READMEs across 10+ skills,
  (2) skills have inconsistent badges, hooks, install commands, or missing sections,
  (3) you want to catch factual errors (wrong costs, broken links, version mismatches)
  across many repos at once. Orchestrates: agent-review-panel (4 parallel reviewers)
  → plan-review-integrator (validates findings against domain context) → subagents
  (implement P0/P1/P2/P3 in parallel). Covers badge addition, release creation,
  README hook rewrites, Requirements/Limitations sections, and cost table correctness.
author: wan-huiyan
version: 1.0.0
date: 2026-04-03
---

# Skill Portfolio Audit

## Problem

A portfolio of 10–20 skills accumulates drift: inconsistent badges, mechanism-first hooks,
missing sections, stale cost tables, non-standard install commands, and files that exist in
GitHub but were never pulled locally. Fixing each manually is tedious and easy to get wrong.

## Pipeline

```
1. skill-sync init          → refresh registry (merges local + GitHub file lists)
2. agent-review-panel       → 4 parallel reviewers on all READMEs (Documentation mode)
3. plan-review-integrator   → validate findings, catch reviewer errors, prioritize
4. Parallel subagents       → implement P0/P1/P2/P3 by priority tier
```

## Phase 1: Refresh Registry

Before auditing, ensure skill-sync knows about all files in every repo:

```bash
/skill-sync init
```

This merges local file listings with `gh api repos/{repo}/contents` so GitHub-only
files (READMEs created by publish-skill but never pulled) are included in tracked_files.

## Phase 2: Review Panel (4 reviewers, parallel)

Launch 4 agents simultaneously. Each fetches all READMEs independently via:

```bash
/usr/local/bin/gh api repos/{username}/{skill}/contents/README.md --jq '.content' | base64 -d
```

**Reviewer personas for Documentation mode:**

| Reviewer | Focus | Key questions |
|---|---|---|
| Clarity Editor (60%) | Hooks, quick-start, scannable structure | Does the opener lead with pain or mechanism? Is the quick-start under 2 min? |
| Technical Accuracy (30%) | Badge vs release match, cost tables, links | Does `gh release list` match README version? Are cost estimates correct? |
| Completeness Checker (40%) | Missing sections checklist | Requirements, Limitations, Version History, Install command present? |
| Devil's Advocate (20%) | Would a skeptical dev install this? | What's the #1 bounce reason? Is there any proof it works? |

**Cross-checks to always run:**
- `gh release list --repo {username}/{skill}` vs README version history
- Badge links: does `anthropics/claude-code` appear where `wan-huiyan/{skill}` should?
- Cost tables: verify math against actual pricing (Cloud Run: $0.000024/vCPU-s, $0.0000025/GB-s)
- Deprecation status: cross-reference skills that claim to supersede others

## Phase 3: Plan-Review-Integrator Validation

Feed the Supreme Judge report to plan-review-integrator. It catches ~5 reviewer errors per
18-skill audit:

- Findings already implemented (badge order already standardized → cancel)
- Math ambiguity requiring user input (448 vs 440 → ask user)
- P1 structural additions that are better bundled with P2 template rollout
- Reviewer scope overcounts (8 repos with schliff scores → actually 6)
- New cross-repo issues reviewers missed (cost claim in repo A references wrong value fixed in repo B)

**Key override patterns seen:**
- "Badge order standardization" → verify first, often already done
- "X skills need Y section" → spot-check 3-4 to confirm actual count before committing scope
- Structural additions (Requirements, Limitations) → defer to template rollout to avoid patchwork
- Deprecated skills → check if the deprecating skill's README actually mentions the merger

## Phase 4: Implementation (Parallel Subagents)

Split by priority tier. Use **foreground agents** for any tier that writes files or runs git:

```
P0 (today, ~30 min):   1 foreground agent — factual errors, broken links, version mismatches
P1 (this week):        2-3 foreground agents — hook rewrites, install commands, link fixes
P2 (this month):       1 foreground agent — template, Requirements/Limitations rollout
P3 (ongoing):          1 foreground agent — "Why a skill?" sections, citation trimming
```

> **Critical:** Do NOT use `run_in_background: true` for agents that need Bash or Write.
> Background agents may be silently denied tool permissions. Read-only research agents are
> safe to background; write agents must run foreground.

## Standard Badge Row

Apply to all authored skills that lack it:

```markdown
[![GitHub release](https://img.shields.io/github/v/release/{username}/{skill})](https://github.com/{username}/{skill}/releases) [![Claude Code](https://img.shields.io/badge/Claude_Code-skill-orange)](https://claude.com/claude-code) [![license](https://img.shields.io/github/license/{username}/{skill})](LICENSE) [![last commit](https://img.shields.io/github/last-commit/wan-huiyan/{skill})](https://github.com/{username}/{skill}/commits)
```

Add eval badge if `eval-suite.json` exists:
```markdown
[![eval](https://img.shields.io/badge/eval_assertions-N_passed-brightgreen)](eval-suite.json)
```

After adding badges, create GitHub releases for skills that lack them (badge shows blank without a release):
```bash
/usr/local/bin/gh release create v{X.Y.Z} --repo {username}/{skill} --title "v{X.Y.Z}" --notes "Initial release"
```

Get version from: SKILL.md frontmatter → plugin.json → marketplace.json (in that order).

## Problem-First Hook Pattern

14/18 skills in a typical portfolio use mechanism-first openers. Fix template:

```
# Before (mechanism-first):
A Claude Code skill that [does X].

# After (problem-first):
[One sentence pain point before this skill existed]. This skill [mechanism description].
```

Examples:
- git-squash: "Noisy commit history buries meaningful changes. This skill auto-detects trivial commits..."
- skill-sync: "After improving skills locally, pushing changes means cloning and copying for every skill. This skill automates that for your entire portfolio."

## README Completeness Checklist

Standard sections every skill README should have:

- [ ] Badge row (release, Claude Code, license, last-commit)
- [ ] Problem-first hook (pain → solution)
- [ ] Install command (`git clone https://github.com/{username}/{skill} ~/.claude/skills/{skill}`)
- [ ] Quick Start with realistic example
- [ ] Requirements section (Claude Code version + any deps)
- [ ] Limitations section (what it can't do / when not to use)
- [ ] Related Skills
- [ ] Version History with dates

## Notes

- **skill-sync init bug (fixed in v1.1.0):** Earlier versions only scanned local dirs, missing
  GitHub-only files. Always run init before auditing so tracked_files is accurate.
- **gh CLI path:** Use `/usr/local/bin/gh` explicitly in all agent prompts on macOS — `gh` may
  not be on PATH in subagent environments.
- **Deprecated skills:** Add a prominent banner and archive the repo:
  ```bash
  /usr/local/bin/gh repo edit {username}/{skill} --archived
  ```
- **Cost table verification:** Always check Cloud Run costs independently. A 20x inflation
  error was found in one portfolio (README said $7.50, actual was $0.35).
