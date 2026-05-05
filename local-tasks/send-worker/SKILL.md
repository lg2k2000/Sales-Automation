---
name: desk-monkey-send-worker
description: Every 15 min weekdays 8-18. Reads the Notion Send Now Queue, sends iMessage drafts, flips status to Logged.
---

# desk-monkey-send-worker

> PLACEHOLDER. Liam: paste your existing SKILL.md from `~/Documents/Claude/Scheduled/desk-monkey-send-worker/SKILL.md` over this file. Add the repo-sync wrapper below to the top of the prompt section.

## Repo sync (FIRST ACTION on every run)

```bash
cd ~/Documents/Code/desk-monkey-routines
git pull --rebase origin main
```

If pull conflicts, abort and write a 🔴 Failed entry. Do not auto-resolve.

After the work, runlog entry, and any state.json updates:

```bash
git add memory/
git commit -m "send-worker run <ISO>"
git push origin main
```

Retry policy: 4 attempts, exponential backoff.

## Why this lives local

iMessage send is stdio-only via the local iMessage MCP. Cloud routines cannot send iMessage.

## Scope

- Reads the Notion ⚡ Send Now Queue view
- Channel must be `Text` and Direction must be `Out` to send
- Email drafts are handled by the cloud `coworker` routine (Gmail Drafts directly, no Send Now queue)
- After successful send: flip Status to Logged, append delivery timestamp

## Original prompt

> [paste original send-worker SKILL.md content here]
