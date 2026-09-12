---
name: world-model-learner
description: >
  Background daemon that trains any world model from real agent threads.
  Scans finished threads from supported clients, routes them to candidate
  world model files, extracts learnings, and opens one draft PR per thread.
  No in-thread instructions required - learning happens post-hoc.
author: dakotafabro
version: "0.1.0"
tags:
  - world-model
  - learner
  - auto-training
  - background-daemon
---

# World Model Learner

A background daemon that trains your world model from your real work. Every
thread you run in any supported client is read post-hoc by a two-pass
route-then-extract pipeline that opens one draft PR per thread in the world
model repo.

## Philosophy

The world model should learn from how you actually work. You do your real work;
learning happens in the background. No prompting. No compliance ritual.

Three principles:

**1. MAP, not mirror.** The world model never duplicates information that lives
in an authoritative source. Pointers stay correct; snapshots go stale.

**2. Implicit over explicit.** The high-value signal is what you did, not what
you said about it. Files you opened, claims you contradicted, dead-ends you
hit, patterns that repeated.

**3. Humans do structure; the learner does content.** Content into the world
model is the learner's job, derived from real threads. The human job is to
scale the world model: sharding, refactoring layers, designing routing tables,
tuning prompts.

## When the Agent Should Act

This skill is informational unless one of these is true:

| Situation | Run |
|---|---|
| User asks how the world model auto-updates | Explain the daemon (this file) |
| User says "install the learner" | `wm-learn install` |
| User says "is the learner running?" | `wm-learn install --check` |
| Something looks broken | `wm-learn status` |
| User says "stop the learner" | `wm-learn install --uninstall` |

Do not call `wm-learn finalize` during normal work. The daemon does that on
its own schedule.

## The 7-Stage Pipeline

```
scan -> route -> extract -> pre_filter -> patch -> post_filter -> commit+PR
  |       |         |           |           |           |            |
cheap    LLM       LLM     deterministic   LLM    deterministic  deterministic
fs walk  1x     N x candidate   gate      1x entry     gate     git + PR
```

### 1. scan
Walk agent thread directories for finished threads since the last cursor.
No LLM, no network - just a filesystem walk.

### 2. route (LLM pass 1)
Given the transcript + world model tree + INDEX files, emit at most 8
candidate file paths the thread plausibly added or changed knowledge about.

### 3. extract (LLM pass 2)
For each candidate, read the file's full content alongside the transcript.
Emit learnings only for claims not already in the file.

### 4. pre_patch_filter
Deterministic gate: dedup, budget limits, quality thresholds.

### 5. patch (LLM)
Apply the learning to the world model file content.

### 6. post_patch_filter
Deterministic gate: validate the patch didn't break frontmatter, didn't
duplicate existing content, didn't violate MAP-not-mirror.

### 7. commit + PR
Create branch, commit one change per learning, open draft PR.

## Configuration

```bash
export WM_REPO_DIR=~/path/to/your/world-model
export WM_REPO_SLUG=your-org/your-world-model
```

Or use `~/.config/world-model/active.env` (shared with the world-model package).

## Supported Clients

The scanner reads finished threads from:
- `~/.amp/` (Amp)
- `~/.claude/` (Claude)
- `~/.codex/` (Codex)

Additional clients can be added by implementing the client reader interface
in `scripts/clients/`.
