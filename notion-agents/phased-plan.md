# Phased plan

The build sequence for transitioning Desk Monkey from Attio + Claude Code routines to Notion-canonical + Notion AI Agents. Six phases, each independently resumable. Liam approves at every phase boundary before crossing to the next.

Pair with: `voice-rules.md`, `banned-list.md`, `reference-ids.md`, `agent-specs.md`, `operating-constraints.md`.

## Phase 0 — Confirm decisions (5 min)

Before touching anything:

1. **Attio retirement confirmed.** Done as of 2026-05-07. Notion is canonical going forward.
2. **Notion 3.0 AI Agents access verified.** Open Notion sidebar, look for the Agents entry. If missing, ask Liam (plan upgrade may be needed).
3. **Email + Calendar feed cadence.** Default: scheduled hourly sweep. Confirm or swap to webhook-based real-time.
4. **Daily Brief shape.** Default: single overwriting page rebuilt twice daily. Confirm or swap to a Briefs DB (one row per brief, history retained).
5. **Repo fate.** Default: demote to docs (skill files + architecture stay; runlog moves to Notion Run Log DB; routines deleted).

**"Go" criteria:** Liam's answers written down before starting Phase 1.

## Phase 1 — Schema fixes (additive, safe)

Modify the four CRM databases to support the activity feed pattern and the new agent topology.

**1a. Two-way Contacts↔Deals multi-relation**
- Add property on Deals: `Associated Contacts` (Relation → Contacts, multi, dual-synced as `Associated Deals` on Contacts).
- Keep `Primary Contact` and `Economic Buyer` (single-link) for Selling With usage.

**1b. Two-way Contacts↔Activity Log multi-relation**
- On Activity Log: rename current `Contact` (single) to `Associated Contacts` (multi, dual-synced as `Activity Log entries` on Contacts).
- Keep `Deal` as single (one Activity Log row → at most one Deal).

**1c. Activity Feed view per Deal**
- On Deal page template: add inline linked DB view of Activity Log.
- Filter: `Deal = current page` OR `Associated Contacts contains any of (current page's Associated Contacts)`.
- Sort: Timestamp desc.
- Columns: Timestamp, Channel, Direction, Status, Summary, Action Items.

**1d. Sales Hub callout flip**
- Replace top callout on Sales Hub root from "Attio is canonical" to:
  ```
  ✅ Notion is canonical. Attio retired 2026-05-07. Pipeline lives in 📈 Deals; activity in 🧾 Activity Log; people in 👤 Contacts; post-Closed-Won in 🛠️ Projects. Six Notion AI Agents handle sweeping + drafting + the Daily Brief — see Master Notion Agent for what each one does.
  ```
- Remove "do not write here" headers above DEFCON Tasks, Activity Log, and Contacts.

**"Go" criteria:**
- Test on Houston Foam Deal: open the page, confirm `Associated Contacts` exists and accepts multi-link.
- Sales Hub callout reads correctly.
- Activity Feed view loads on the Deal page (empty until Phase 3 populates).

## Phase 2 — Connections

In Notion Settings → Connections:

| Connection | Required scopes | Verify |
|---|---|---|
| Gmail (liam@deskmonkeyai.com) | read mail, manage drafts, manage labels, send drafts | yes |
| Google Calendar (primary) | read events, create events | yes |
| Google Drive | read files, read metadata | yes (for SOWs / attachments) |
| Slack (deskmonkey.slack.com) | read channel, post message, read user profile | optional, only if keeping Slack as sidecar |
| Fireflies | none | DISCONNECT (transcripts come natively from Notion now) |

For each missing connection, walk Liam through OAuth (he must click Allow on Google's consent screen).

**"Go" criteria:** every required Tool in each Connection's enabled list shows green.

## Phase 3 — Deploy six AI Agents

Build in dependency order. After each, manually trigger once with a test input, verify output, then move on. See `agent-specs.md` for full per-agent specs.

| # | Agent | Trigger | Why this order |
|---|---|---|---|
| 1 | 🔗 Meeting Linker | scheduled hourly | nothing depends on it; safe to test first |
| 2 | 📧 Email Sweeper | scheduled hourly | Activity Feed view needs Activity Log rows |
| 3 | 📅 Calendar Sweeper | scheduled 4h | adds BRIEF rows to Activity Log |
| 4 | 🦧 Triage / Daily Brief | scheduled 7am + 8pm MT | needs all Activity Log rows from 1-3 |
| 5 | ✉️ Reply Executor | on-comment on Daily Brief, fallback scheduled 15m | needs Daily Brief to exist |
| 6 | 🧹 Inbox Sanitation | scheduled 4h | independent; can run last |

**"Go" criteria per agent:**
1. Meeting Linker: trigger on May 6 "CRM and Sales Workflow Discussion" note → links to HFP Deal, renames title, creates Activity Log row, drafts follow-up email.
2. Email Sweeper: trigger → Activity Log rows for any threads from a Contact since the last `#all-desk-monkey` digest.
3. Calendar Sweeper: trigger → BRIEF rows for tomorrow's CIO Tech + Apollo and Andrew Brink walkthrough.
4. Triage: trigger → 🦧 Daily Brief page renders with sections.
5. Reply Executor: post `skip 1` comment on Daily Brief → confirmation reply within 15m.
6. Inbox Sanitation: trigger → promo deletion log entries with allowlist respected.

## Phase 4 — Migrate Attio state back to Notion

Pull every active record from Attio and recreate in Notion. After this phase, Attio is read-only forever.

**4a. HFPUSA Deal (`28d39ba2...`)** — already in Notion, just append Deal Evolution entry for today's Attio activity (drafts staged, Calendar invite sent for May 14, etc.). Don't recreate.

**4b. Andrew Brink Person Record** — create new in Contacts DB. abrink@blueally.com, BlueAlly, Source: Chris Introduction, Relationship: Lead. Notes block carries the BRIEF content.

**4c. Anthony Orlovsky Person Record** — create new in Contacts DB. Both emails (anthony@thevirtualvp.net, aorlofski@gmail.com), MyVP / The Virtual VP, Source: Chris Introduction, Relationship: Lead. Spelling note (Orlovsky vs Orlofski) in Notes.

**4d. Open Liam Tasks (3)** — create rows in DEFCON Tasks DB:
- `[DEFCON 2] [Internal] [Liam] Send Miles Friday May 15 dialer-kickoff windows` — deadline 2026-05-12
- `[DEFCON 2] [Client Deliverable] [Liam] Deliver POC demo to HFP sales leadership` — deadline 2026-05-29
- `[DEFCON 3] [Client Deliverable] [Liam] Build weekly Apollo→Excel dashboard for HFP exec visibility` — deadline 2026-05-29

(Skip the email sequence task — Liam said it's done.)

**4e. Today's BRIEF Notes (2)** — recreate as Activity Log rows:
- `BRIEF-ei97ak4krvoui8bt353io0kkio — CIO Tech + Apollo (desk monkey)` linked to HFP Deal
- `BRIEF-lrjjbo8daeat9p98b0jsu3jk2k — Notion walkthrough: Liam + Andrew` linked to Andrew Brink Person (created in 4b)

**4f. Sales Hub Attio memorial** — one-line addendum: "Attio retired 2026-05-07 after 9 days as canonical CRM. Selling With schema and the cron-routine architecture lessons live on in Notion."

**"Go" criteria:**
- All 5 net-new records visible in their target DBs.
- HFP Deal Evolution shows a 2026-05-07 Attio-retirement entry at the top.
- Activity Log Activity Feed view on HFP Deal shows the BRIEF row.

## Phase 5 — End-to-end verification

Run the full pipeline on real data, watching for failures.

1. Trigger Meeting Linker → verify May 6 meeting linked to HFP, Activity Log row created, draft staged.
2. Trigger Email Sweeper → verify rows for any new emails from active Contacts.
3. Trigger Calendar Sweeper → verify BRIEFs for tomorrow's two meetings (idempotent — should match the ones from 4e, not create duplicates).
4. Trigger Triage → verify 🦧 Daily Brief renders.
5. Post `skip 1` on Daily Brief → verify Reply Executor logs and replies.
6. Trigger Inbox Sanitation → verify promo deletion log + allowlist respect.
7. Wait 1 hour → verify scheduled triggers fire on their own.

**"Go" criteria:** all 7 checks pass; 24-hour stability before Phase 6.

## Phase 6 — Decommission the old system

Once Phase 5 is green and the new system runs unattended for 24 hours:

1. Liam deletes the `assistant` Claude routine from the Anthropic routine UI (must be done by Liam — outside Notion / Chrome scope).
2. Repo handling per Phase 0 question 5:
   - **Demote to docs (default):** add `DECOMMISSIONED.md` at repo root summarizing the migration; routines/ dir deleted; skills/ dir kept; memory/runlog.md frozen with a final "moved to Notion Run Log DB" note.
   - **Kill repo:** archive the GitHub repo.
3. Open PR #8 (today's last `assistant` run): close without merge, or merge for trace.
4. Final commit on `main` includes: `DECOMMISSIONED.md` (if demoting), the six new files in `notion-agents/`, and a closing line in CLAUDE.md.

**"Go" criteria:** Liam confirms in chat or in `#all-desk-monkey` that the new system is reliable for 24 hours.
