# Desk Monkey Architecture

Three-layer system. GitHub repo as the shared persistence layer between cloud routines and local Mac-bound tasks. Notion is canonical state and Liam's dashboard.

## Data flow

```mermaid
flowchart TB
    subgraph Local["LOCAL — Mac-bound (laptop)"]
        TRIAGE_LOCAL["desk-monkey-imessage-triage<br/>cron: hourly 8-18<br/>iMessage + Gmail + Notion + GCal<br/>Reads inbound, drafts, reply detection"]
        SEND_LOCAL["desk-monkey-send-worker<br/>cron: every 15 min weekdays 8-18<br/>iMessage send + Gmail draft<br/>Reads Notion Send Now Queue"]
    end

    subgraph Cloud["CLOUD — Anthropic Claude Code Routines"]
        DEAL_CLOUD["desk-monkey-deal-updater<br/>cron: daily 7am weekdays<br/>Reads Notion Activity Log<br/>Updates Deal page properties"]
        AUDIT_CLOUD["desk-monkey-self-audit<br/>cron: Sundays 7pm<br/>Reads runlog (last 200 lines)<br/>Writes audit.md"]
    end

    subgraph Repo["GitHub: desk-monkey-routines (shared state)"]
        CLAUDE_MD["CLAUDE.md — working memory"]
        MCP_JSON[".mcp.json — connector list"]
        SKILLS_DIR["skills/ — humanizer.md + gotchas.md"]
        ROUTINES_DIR["routines/<name>/prompt.md"]
        MEMORY_DIR["memory/ — runlog.md, audit.md, state JSON"]
        ARCH_MD["architecture.md — this diagram"]
    end

    subgraph Notion["Notion (canonical state + dashboard)"]
        ACTIVITY_DB["Activity Log DB"]
        CONTACTS_DB["Contacts DB"]
        DEALS_DB["Deals DB"]
        DASHBOARD["Desk Monkey Dashboard"]
    end

    Local -->|git pull/push| Repo
    Cloud -->|fresh clone each run| Repo
    Local -->|read/write| Notion
    Cloud -->|read/write| Notion
    Notion --> DASHBOARD

    style TRIAGE_LOCAL fill:#d4ebd7
    style SEND_LOCAL fill:#f7e6cc
    style DEAL_CLOUD fill:#d6e3f5
    style AUDIT_CLOUD fill:#e0d6f5
    style Repo fill:#fce8d5
    style Notion fill:#f0e0fa
```

## Why this split

Local stdio MCPs (iMessage, Apple Notes, computer-use) only run as local processes. Cloud Claude Code routines see only remote HTTP/SSE connectors. So anything iMessage-bound stays local. Notion, Gmail, Google Calendar, Apollo are all remote-capable, so deal-updater and self-audit move to cloud routines. The repo is the shared state layer: cloud routines read/write `memory/runlog.md` directly via git; local tasks pull and push around their runs.

## Source of truth

Reality (iMessage / Gmail / Calendar) → Notion → repo memory files. Notion is the canonical record. Repo memory files are operational scratchpads for the routines themselves (runlog, audit findings, cross-run state).

## Cron schedule

| Job | Where | Cron | Purpose |
|---|---|---|---|
| desk-monkey-imessage-triage | Local Mac | `0 8-18 * * *` | Hourly inbound triage + draft generation |
| desk-monkey-send-worker | Local Mac | `*/15 8-18 * * 1-5` | Every 15 min weekdays — sends queued drafts |
| desk-monkey-deal-updater | Cloud routine | `0 7 * * 1-5` | Daily 7am — updates Deal page properties from yesterday's activity |
| desk-monkey-self-audit | Cloud routine | `0 19 * * 0` | Sundays 7pm — scans runlog, writes audit findings |

Local and cloud schedules don't overlap (cloud runs at 7am and 7pm-Sunday, local runs hourly during business hours). Runlog merge collisions should not happen in practice; if they do, audit logs Drift.

## Repo sync contract

- **Cloud routines**: fresh clone each run, write changes, commit, push. No long-lived state on the cloud side.
- **Local tasks**: `git pull --rebase origin main` before work, `git add memory/runlog.md && git commit && git push origin main` after.
- **Conflicts**: surfaced to runlog as Drift. Liam resolves manually. No auto-merge of conflicting runlog edits.

## Repo structure

```
desk-monkey-routines/
├── CLAUDE.md                          # auto-loaded into every session
├── README.md                          # operator docs
├── .mcp.json                          # MCP servers for cloud routines
├── architecture.md                    # this file
├── skills/
│   ├── humanizer.md                   # voice rules
│   └── gotchas.md                     # canonical rule list
├── routines/
│   ├── deal-updater/
│   │   ├── prompt.md
│   │   └── README.md
│   └── self-audit/
│       ├── prompt.md
│       └── README.md
├── local-tasks/
│   ├── triage/
│   │   └── SKILL.md                   # local — iMessage triage
│   └── send-worker/
│       └── SKILL.md                   # local — iMessage send worker
└── memory/
    ├── runlog.md                      # append-only run history
    ├── audit.md                       # weekly audit findings
    └── state.json                     # cross-run state
```
