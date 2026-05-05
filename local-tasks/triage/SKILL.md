---
name: desk-monkey-imessage-triage
description: Hourly iMessage triage. Pulls inbound messages, drafts responses, logs to Notion Activity Log.
---

# desk-monkey-imessage-triage

> PLACEHOLDER. Liam: paste your existing SKILL.md from `~/Documents/Claude/Scheduled/desk-monkey-imessage-triage/SKILL.md` over this file. Add the repo-sync wrapper below to the top of the prompt section.

## Repo sync (FIRST ACTION on every run)

Before anything else:

```bash
cd ~/Documents/Code/desk-monkey-routines  # or wherever this repo lives locally
git pull --rebase origin main
```

If the pull conflicts, abort the run, write a 🔴 Failed entry to `memory/runlog.md` flagging Drift, and let Liam resolve manually. Do NOT auto-resolve runlog conflicts.

After the work is done and you've written your runlog entry:

```bash
git add memory/
git commit -m "triage run <ISO>"
git push origin main
```

If push fails on network: retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s).

## Why this lives local

iMessage MCP is stdio-only. Cloud Claude Code routines can only see remote HTTP/SSE connectors. This task has to run on the Mac.

## Original prompt

> [paste original triage SKILL.md content here]
