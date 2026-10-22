# world-model-learner - what changes for you

world-model-learner is meant to keep your world model current from the work you already do. It reads your finished agent threads and opens one draft PR per thread with what it learned.

## What it does

A background daemon runs a seven-stage pipeline: scan finished threads, route each to candidate world model files, extract learnings, filter, patch, filter again, and open a draft PR. Three of the stages call an LLM. It doesn't change how your agent behaves in a session.

## When to use it

When you have a world model (install `world-model` first) and keeping it current by hand isn't keeping up.

## Reserved for human judgment

Every change arrives as a draft PR. You review and merge. The skill's own split: the learner writes content, and humans own structure (sharding, layers, routing, prompts). Read each PR as the learner's reading of a thread, not as verified fact.

## What it reads, writes and sends

- Reads finished thread transcripts from `~/.amp/`, `~/.claude/` and `~/.codex/`. Those can hold anything you said or pasted in a session.
- Sends thread content to an LLM for routing, extraction and patching. Which LLM, and where it runs: not documented.
- Creates branches, commits and draft PRs in the repo at `WM_REPO_SLUG`.
- How the daemon is scheduled and where it logs: not documented.

## Cost

About 900 tokens of context (`token_cost` in bpm.json). LLM spend per thread: not documented.

## Check it's working

`wm-learn status`, then look for draft PRs in your world model repo.

## Turn it off

`wm-learn install --uninstall`, then `bpm remove world-model-learner`.

Note: at commit `5ccb8cb`, the repo's `scripts/` and `prompts/` folders are empty, so `wm-learn` and the pipeline described here weren't found in the package.
