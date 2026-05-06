# coworker

The tactical loop. Runs 3x weekdays. Past-24h sweep across Fireflies, Gmail, Calendar. Logs Activity, creates DEFCON Tasks, drafts emails. Never sends.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `0 7,12,18 * * 1-5`)
- **Connectors:** Notion, Gmail, Google Calendar, Fireflies, Zapier (for Gmail per-message label)
- **Routine prompt:** `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Scope (in)
- New Fireflies transcripts → log + DEFCON Tasks + draft (delegates to `skills/parse-call.md`)
- Unanswered Gmail threads from Deal contacts >24h → log + draft
- Tomorrow's external Calendar meetings → brief + prep task

## Scope (out — handled elsewhere)
- Counterpart commitment verification → `daily-review`
- Deal property updates rolled up across multiple activities → `daily-review`
- Left-on-read prospecting (>5d silence) → `daily-review`
- Stale Deal sweep → `self-audit` (weekly)
- iMessage triage and send → deferred (local Mac stdio MCPs not currently wired)

## Idempotency
Every step queries the Activity Log first to check whether the source (Fireflies transcript, Gmail thread, Calendar event) has already been processed. Re-running the routine is safe.

## See also
- `prompt.md` — exact steps
- `../../skills/parse-call.md` — Fireflies transcript handling
- `../daily-review/` — the rollup routine
- `../../CLAUDE.md` — working memory
