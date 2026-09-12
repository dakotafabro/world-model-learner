# world-model-learner

Background daemon that trains any world model from real agent threads. You do
your work; the learner watches, extracts knowledge, and opens draft PRs.

## Why

Manually maintaining a world model doesn't scale. The learner automates the
write side: it reads your finished agent threads, identifies what the thread
taught about the system, and proposes updates to the world model as draft PRs.

## Quick Start

```bash
bpm install world-model-learner

# Install the background daemon
wm-learn install

# Check status
wm-learn status

# Uninstall
wm-learn install --uninstall
```

## How It Works

7-stage pipeline: scan threads, route to candidate files (LLM), extract
learnings (LLM), filter, patch (LLM), filter, open PR. One draft PR per
thread. Human reviews and merges.

## Requirements

- A world model (install the `world-model` package first)
- `WM_REPO_DIR` set to your world model path
- `WM_REPO_SLUG` set to your world model's GitHub repo

## Files

- `SKILL.md` - Full convention and pipeline documentation
- `scripts/` - Pipeline implementation
- `prompts/` - LLM prompts for route and extract stages
