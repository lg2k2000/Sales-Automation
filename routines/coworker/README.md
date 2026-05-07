# coworker

> **⚠️ SUPERSEDED 2026-05-07** — replaced by `routines/assistant/`. Phases 1-4 of the new `assistant` routine cover everything coworker did, plus Slack digest surfacing in phases 5-6. This README kept for trace; do not schedule this routine.

The tactical loop. Runs 2x weekdays. Past-24h sweep across Fireflies, Gmail, Calendar. Writes to Attio (Notes for activity, Tasks for action items, Deal Record updates for high-confidence evidence). Drafts emails into Gmail Drafts. Never sends.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `0 8,16 * * 1-5`)
- **Connectors:** Notion (Projects + knowledge only), Attio, Gmail, Google Calendar, Fireflies, Zapier (for Gmail per-message label + archive)
- **Routine prompt:** `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Scope (in)
- New Fireflies transcripts → Attio Note + Tasks + Deal updates + draft (delegates to `skills/parse-call.md`)
- Inbox noise classification (newsletters, receipts, notifications → label + archive via Zapier)
- Unanswered Gmail threads from Attio People >24h → Attio Note + draft
- Tomorrow's external Calendar meetings → Attio Brief Note + prep Task

## Scope (out — handled elsewhere)
- Counterpart commitment verification → `daily-review`
- Deal property rollups across multiple activities → `daily-review`
- Left-on-read prospecting (>5d silence) → `daily-review`
- Stale Deal sweep → `self-audit` (weekly)
- iMessage triage and send → deferred (local Mac stdio MCPs not currently wired)
- Notion CRM writes → never. Attio is canonical for CRM.

## Idempotency
Every step queries Attio Notes (title-prefix source ID match) before processing to check whether the source has already been logged. Re-running the routine is safe.

## See also
- `prompt.md` — exact steps
- `../../skills/parse-call.md` — Fireflies transcript handling
- `../daily-review/` — the rollup routine
- `../../CLAUDE.md` — working memory
