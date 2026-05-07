# Desk Monkey Run Log

Append-only log of every routine + local-task run. Each entry uses expected-vs-actual format and lands ✅ Healthy / ⚠️ Drift / 🔴 Failed.

Routines append a `[IN_PROGRESS]` stub at start and replace it with the final report at end. If you see a lingering `[IN_PROGRESS]` entry, that run died mid-execution and the routine needs investigation.

---

## 2026-05-06 — manual one-shot — HFP USA POC follow-up + CRM update — ✅ Healthy

**Trigger:** Liam asked for follow-up drafts (joint Miles+Craig, individual Craig), Attio next-steps update, Notion update. Ref Sales POC meeting earlier today (Fireflies 01KQWQSPE7PEC4V30XZK2JN0QS).

**TOOL_CONTRACT Attio:** search=yes list=yes create=yes upsert=yes update=yes attrs=yes notes=yes tasks=yes lists=yes

**Context resolved:**
- Deal: HFPUSA, record d46eba25-c6dd-463c-925c-f4c873ea581b, Stage POC, value $1,250
- Craig Walicek (cwalicek@ciotech.us) + Miles Kurtz (mileskurtz@hfpusa.com) both linked
- Fireflies meeting "Sales POC" at 19:30 UTC on 2026-05-06 (98 min)
- First-pass recap email already sent at 22:07 UTC May 6 to both (joint thread `Sales POC`, message 19dff5489975719f)
- Zero existing Notes on the Deal before this run

**Writes:**
- Gmail draft: joint reply on `Sales POC` thread to Miles+Craig (id r3916374991934218578) — Friday review check vs async
- Gmail draft: new thread to Craig only `Couple questions on the POC` (id r4705530137615562383) — POC format question, in-person offer, 5h budget
- Attio Note: `MTG-01KQWQSPE7PEC4V30XZK2JN0QS — Sales POC` on Deal (id e8f02091-799c-42fd-b066-60eca04f649e)
- Attio Tasks (5):
  - 242f9fe9 Liam DEFCON 2 POC demo delivery (deadline 2026-05-29)
  - f8f558e7 Liam DEFCON 2 Email sequence first cut (deadline 2026-05-15)
  - 023a54cf Liam DEFCON 2 Sending domain stand-up + warm (deadline 2026-05-12)
  - 3d51dead Craig counterpart DEFCON 2 Domain purchase (deadline 2026-05-09, assignees=[])
  - ad22930c Miles counterpart DEFCON 3 Dial blitz coordination (deadline 2026-05-15, assignees=[])
- Notion: updated `🗺️ Mutual Action Plan + Next Steps` (23e4c4b9-aada-4e9f-a9d4-8b828b73785b). Added `## POC Track (current, May 2026)` section above the parked audit-first plan.

**FIELD_OPTION_GAP:** Attio Deal object missing `Deal Evolution`, `Identified Pain`, `Champion`, `Champion Verdict`, `Champion Tests Passed`, `Champion Test Evidence`, `Compelling Event`, `Cost of Doing Nothing`, `Open Questions`, `Future State`, `Why Change`, `Why Now`, `Why Desk Monkey`, `Required Capabilities`, `Buyer-Owned Action Ratio`, `Forecast Category`, `Service Line`, `Source`, `Deal Type`, `Personal Stakes`, `Internal Priority`, `Show-Stoppers`. Workspace also has duplicate `Decision Process` and `Emotional State` attributes (5+ variants each). Selling With attribute schema needs to be set up before parse-call can write to it — surface for Liam.

**Drift watched:** Deal had no Notes prior; meeting was logged for the first time (so the dedupe gate would have allowed reprocessing — but only ran once here). Also: Buyer Behavior Stage left untouched (POC ongoing, no new behavioral evidence beyond what the recap captured).

**Next:** Liam reviews + sends the two drafts. Daily-review will prune the first-pass recap from earlier today.

---

## 2026-05-06 — manual one-shot — HFP USA POC follow-up CORRECTION — ⚠️ Drift fixed

**Trigger:** Liam called out two errors in the prior run.
1. Created a parallel Liam-owned domain task when Craig owns the domain end-to-end.
2. Wrote a Miles counterpart task asking him to coordinate dial blitz availability instead of having Liam offer specific windows from his calendar.

**Drift root cause:** No directive in `skills/humanizer.md` about scheduling. Default behavior fell back to "ask the counterpart" / "split ownership across both parties" which reads as commanding-by-omission.

**Calendar pull (Liam primary, MT):** Wide open Friday May 15 09:00-17:00 MT and all weekdays May 18-22 09:00-17:00 MT. Picked these to offer:
- Joint Friday review: 10am CT or 2pm CT on May 15.
- Miles dialer support: Tue May 19 10am-12pm CT, Wed May 20 1pm-3pm CT, Thu May 21 10am-12pm CT.

**Fixes:**
- Attio Task 023a54cf (Liam stand up domain) → marked complete (Craig owns end-to-end, no Liam duplicate).
- Attio Task ad22930c (Miles coordinate availability) → marked complete (wrong framing).
- Attio Task 1ad724f1 created — Liam-owned, lists the three offered windows, replaces ad22930c. (Note: `update-task` doesn't accept a content parameter, so complete-and-recreate was the only path.)
- Gmail draft r5430521722782864968 — corrected joint reply to Miles + Craig with Friday May 15 10am or 2pm CT offered.
- Gmail draft r4281166858187808495 — new Miles-only draft with the three dialer-support windows offered.
- Notion `🗺️ Mutual Action Plan + Next Steps` — POC Track section updated: Craig owns domain end-to-end, Friday review carries the offered times, Week of May 18 lists the three dialer-support windows.
- `skills/humanizer.md` — added `### Scheduling: offer specific times, never ask` subsection under Liam-specific voice rules. Covers email drafts, Attio Tasks, and Notion entries. Bad/good examples included. Reinforces the CLAUDE.md hard NEVER on autonomous Calendar invite creation.

**Old drafts to delete manually (Gmail MCP surface has no draft-delete; Zapier `delete_email` is for sent mail):**
- r3916374991934218578 (old joint draft, asked instead of offered)

**Drafts that stay as-is:**
- r4705530137615562383 (Craig-only POC format question + fly-out offer). The fly-out is a yes/no, the format question is genuine substance discovery, not scheduling. Reads clean.

**Skill change effective immediately for `coworker`, `daily-review`, `parse-call`, and any drafting routine.**

**TOOL_CONTRACT Attio:** search=yes list=yes create=yes upsert=yes update=yes (partial — task content not updateable) attrs=yes notes=yes tasks=yes lists=yes

---

## 2026-05-06 — manual one-shot — HFP USA POC follow-up SECOND CORRECTION — ⚠️ Drift fixed

**Trigger:** Liam called out a second error: I missed that Friday is the dial blitz day, not a review day. Transcript said "sessions starting the week after current Friday out-of-office" — Miles is OOO this Friday May 8, so blitz starts week of May 11, with Friday May 15 as the actual blitz day. My previous fixes had:
1. Joint review on Friday May 15 (wrong — that's blitz day)
2. Miles dialer-support windows on Tue/Wed/Thu of week of May 18 (wrong — should be Friday May 15 kickoff for the blitz)

**Drift root cause:** Skipped close reading of the Fireflies action items. "Dial blitz starting the week after current Friday out-of-office" got read loosely. Should have flagged "what specific day is the blitz?" before drafting times.

**Fixes:**
- Attio Task 1ad724f1 → marked complete (wrong dates).
- Attio Task f8f558e7 (sequence first cut) → deadline pulled in to 2026-05-14 (Wed May 13 EOD), so review can happen Thursday before Friday's blitz.
- Attio Task 93fa365e created — Liam-owned, lists Friday May 15 kickoff windows (9am, 10am, 11am CT) for the dial blitz. Replaces 1ad724f1.
- Gmail draft r3953375124665551055 — new Miles-only draft for Friday May 15 dialer kickoff (replaces r4281166858187808495).
- Gmail draft r-897487427422999950 — new joint draft for Thursday May 14 review (replaces r5430521722782864968).
- Notion `🗺️ Mutual Action Plan + Next Steps` — POC Track restructured: Week of May 11 review now Thursday May 14, new step 3 "Friday May 15 dial blitz day" with kickoff windows + standby through the day, Week of May 18+ now ongoing Apollo ramp + subsequent blitz days.

**Old drafts to delete manually (no draft-delete capability):**
- r3916374991934218578 (oldest joint, asked instead of offered)
- r4281166858187808495 (Miles draft with Tue/Wed/Thu May 18-21 windows — wrong dates)
- r5430521722782864968 (joint draft with Friday May 15 review — wrong day)

**Drafts that stay as-is:**
- r4705530137615562383 (Craig-only POC format question + fly-out offer).

**Skill follow-up to consider:** `parse-call.md` Step 4 (Note creation) and Step 5 (Tasks) currently transcribe action items as-stated. When the meeting only loosely scopes timing (e.g. "starting the week after"), the routine should call out "needs explicit blitz/launch date" rather than guessing a default. Will surface this for Liam's review of `parse-call.md` if drift recurs.

---

## 2026-05-06 — manual one-shot — HFP USA POC follow-up THIRD CORRECTION — ⚠️ Drift fixed

**Trigger:** Liam called out three more issues:
1. Three branches existed on GitHub (an orphan merged-but-not-deleted from PR #2 was clutter).
2. Two Notion DEFCON tasks for the dormant customer list cleanup + Apollo enrichment were still open even though Liam already did the work pre-meeting (transcript captured "list trimmed from ~3,500 to ~1,500 active").
3. Notion's pipeline situation is broken: two legacy Deal DBs (`Deals` on Sales Hub + standalone `💰 Deals Pipeline`) with stale stages, no clear flag that Attio is canonical now.

**Fixes:**
- Notion task `b2e6a831` (Clean 3,800+ dormant customer CSV) → Status set to Done, Notes updated to "DONE as of pre-2026-05-06 meeting. Liam scrubbed list from ~3,500 to ~1,500 active."
- Notion task `eaac557b` (Enrich dormant list in Apollo) → Status set to Done, Notes updated similarly.
- Notion `🗺️ Mutual Action Plan + Next Steps` POC Track step 1 updated: list enrichment now flagged Done up front.
- Notion `🦧 Desk Monkey Sales Hub` (root page `5213634e...`) → red-background callout inserted at top: **Attio is canonical for pipeline now**, listing the current Attio stages (New / Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement / Closed Won / Closed Lost / Nurture / On Hold / Unresponsive) and flagging legacy Notion DBs (Deals / DEFCON Tasks / Activity Log / Contacts) as do-not-write.

**Branch cleanup attempted but blocked:**
- `git push origin --delete claude/sweet-heisenberg-0Vsdz` → HTTP 403 (harness restricts pushes to designated session branch only).
- GitHub MCP surface has no `delete_branch` capability either.
- Surfaced to Liam: nuke the orphan from GitHub UI manually.

**Drift root cause:**
- The `parse-call` skill (and the prior runs of this manual one-shot) didn't reconcile new meeting state with existing Notion DEFCON Tasks. When a transcript indicates a task is already done, the routine should sweep for matching open Notion tasks and close them.
- Sales Hub root page hadn't been updated when the Notion → Attio CRM migration happened. Stale entry point for anyone (or any routine) opening Notion to find pipeline.

**Skill follow-ups:**
- Add to `parse-call.md`: after Step 5 (Attio Tasks), add a "legacy Notion DEFCON Tasks reconciliation" check — if transcript reveals a task is done that has a matching open Notion DEFCON task, close the Notion task too. This is transitional until Notion DEFCON Tasks DB is fully retired.
- Long-term: `contact-migration` should also archive/delete the legacy Notion CRM DBs (Deals / `💰 Deals Pipeline` / Activity Log / Contacts). Today they linger and confuse.
- Branch cleanup: surface to Liam that the harness creates a fresh session branch each run despite CLAUDE.md saying "no session branches." Either CLAUDE.md needs updating or the harness setting needs changing. Will not write code to fix without explicit guidance.

---

## 2026-05-07 — manual one-shot — HFP USA verification sweep + tomorrow's invite + skill rules update — ✅ Healthy

**Trigger:** Liam asked for (a) verification that current Attio tasks are truly outstanding (haven't already been emailed/invited), (b) a tentative Calendar invite for May 8 2pm MT cold-call blitz with Miles + Craig, (c) a polished recap email + MAP for tomorrow's send, (d) a directive that counterpart-owned tasks should NOT be in Attio (they live in counterparts' own systems — Miles → Apollo).

**Calendar invite SENT** (per Liam's explicit per-invite authorization):
- Event id `ueu1671393huf1f4fg50u9n93o`
- "HFP cold call blitz (tentative)"
- May 8 14:00-15:00 MT
- Attendees: mileskurtz@hfpusa.com, cwalicek@ciotech.us
- Google Meet generated, notificationLevel=ALL (counterparts received the invite email)

**Gmail draft created (NOT sent):**
- `r-590308765881101588` — "Wednesday recap + path forward" to Miles + Craig — recap of May 6 Sales POC + full Mutual Action Plan table. Written to send Friday May 8 morning.

**Attio Tasks marked complete (counterpart-owned, moved out of Liam's todo per new directive):**
- 745ea5e2 (Miles deliver sample to Basil)
- a0a83b54 (Miles save Basil contact in Apollo)
- eb6be79d (Miles share Apollo with Laura/Jesse)
- 3d51dead (Craig domain purchase)

**Notion updates:**
- Houston Foam Deal `28d39ba2` — `MAP Draft` field replaced with v3 (post-May-6 path), v2 (pre-meeting plan) preserved below as superseded.

**Skill changes (durable):**
- `skills/gotchas.md` — `Counterpart-owned task pattern` rewritten: counterpart tasks do NOT live in Attio, they live in the counterpart's system; the Notion MAP captures cross-owner commitments. Old pattern (empty-assignee Attio Tasks) deprecated. Also added `Calendar invite authorization` section: per-invite explicit auth from Liam allows `create_event` with notificationLevel=ALL; Gmail send still NEVER.
- `skills/parse-call.md` — Step 5 rewritten: counterpart action items go in meeting Note + Notion MAP Draft, NOT Attio Tasks. Hard NEVERs updated.

**Verification evidence (truly-outstanding sweep results):**
- Original Sales POC recap email (22:07 UTC May 6) NOT sent (was a draft, since deleted by Liam via Zapier).
- All other Liam-owned outbound either sent (Anthony follow-up, Andrew Brink invite switch, Craig POC format question, Top-10-titles thread) or in current drafts (Friday review + Friday dialer windows).
- Calendar invites already in place: Thu May 8 1pm MT (CIO Tech + Apollo, Craig only), Fri May 8 4pm MT (Notion walkthrough with Andrew Brink), Thu May 7 9am BST (Attio onboarding with Victor).

**Open issues surfaced:**
- Spelling: Apollo + LinkedIn + email all show **Walicek** (cwalicek@ciotech.us, linkedin.com/in/craigwalicek). Liam asked to rename to **Walachek** in Google Contacts. Did not change Attio name pending Liam confirmation; flagged the evidence.
- Google Contacts MCP not wired in current surface; will check Zapier discover for Google Contacts actions.
- Apollo as Miles's task store: surfaces a product idea — "build a Desk-Monkey-equivalent for Miles that parses transcripts and writes tasks into Apollo." Logged for future.

---

## 2026-05-07 — Notion redesign + Google Contacts wiring + branch cleanup — ✅ Healthy

**Trigger:** Liam authorized the Notion cleanup pass + Zapier Google Contacts.

**PR #4 merged into main** (squash, commit `704adb76`). Outstanding orphan branch `claude/sweet-heisenberg-DGLg4` still on remote — Liam to delete in GitHub UI.

**Notion `📈 Deals` (canonical) — 4 saved views added:**
- 🔥 Hot This Week — Buyer Behavior ≥ 3 AND Champion Verdict in [Coach, True Champion] AND Stage NOT Closed
- ⏳ Stalled (oldest first) — Stage NOT IN [Closed Won/Lost, Nurture, On Hold, Unresponsive], sorted Last Touch ASC
- 🌳 By Source — board grouped by Source (Chris Introduction / Direct / Referral / Cold)
- 🤝 Resale / Referral Track — Source = Chris Introduction OR Service Line contains Custom

**Notion `💰 Deals Pipeline` (legacy) — renamed to `📦 ARCHIVED — Deals Pipeline (legacy, do not use — Attio canonical)`** so it visibly clears out of the working set. Single orphan stub row (Anthropic, $10K Lead) retained for trace; not actively referenced.

**Notion Sales Hub root — pruned:**
- DEFCON Tasks header now reads: "Migrated to Attio Tasks. The legacy DB below is read-only reference; do not write here."
- Activity Log header now: "Migrated to Attio Notes (per Deal Record). The legacy DB below is read-only reference; do not write here."
- Contacts header now: "Migrated to Attio People. The legacy DB below is read-only reference; do not write here. Gmail integration in Attio handles contact-relationship visibility natively."
- Inline DBs left in place (Notion MCP doesn't expose delete-block reliably) but flagged.

**Zapier Google Contacts:** already enabled (7 actions: read contact, create contact, update contact, add to group, group, photo, raw request). No auth needed. Ready to add Craig once Liam confirms spelling (Walicek per LinkedIn/email vs Walachek per Liam's note).

**Outstanding skill follow-ups for next round (Liam's strategic asks):**
- Routine consolidation: collapse coworker (2x/day) + daily-review (1x/day) into one `assistant` routine running 4-5x/day in proper order (sweep → process → digest → surface).
- Surfacing channel: Slack as primary push (native official Anthropic MCP), email digest as fallback. Common Room evaluated and skipped (community-engagement tool, not aligned with Desk Monkey buyers).
- Memory architecture: per-deal context files (`memory/deals/<slug>.md`) populated by the consolidated routine.
- Per-deal MAP doctrine: Notion `MAP Draft` field is the canonical mutual action plan. Routines write the cross-owner MAP there, not Attio Tasks.

---

## 2026-05-07 — architecture rewrite — `assistant` routine + Slack surfacing — ✅ Healthy

**Trigger:** Liam's "secretary vision" conversation — collapse multi-routine architecture into one routine running 5x/day in fixed phase order, surface via Slack `#desk-monkey` with free-form text reply DSL.

**Files written:**
- `routines/assistant/README.md` — new
- `routines/assistant/prompt.md` — new (replaces coworker + daily-review prompts)
- `skills/digest.md` — new (Slack digest composition)
- `skills/slack.md` — new (channel mechanics + reply DSL parsing)

**Files updated:**
- `CLAUDE.md` — date stamp added; tool routing table now lists Google Contacts WIRED (Zapier) and Slack added; counterpart-task rule rewritten; routines section collapsed to single `assistant` row + `self-audit` weekly + `contact-migration` one-shot; old `coworker` and `daily-review` flagged superseded; skills list expanded with `digest.md` + `slack.md`; Hard NEVERs updated for per-item Slack-authorized Gmail send and per-invite Calendar auth, plus "ALWAYS post the digest even when empty" rule.
- `routines/coworker/README.md` — SUPERSEDED 2026-05-07 banner added
- `routines/daily-review/README.md` — SUPERSEDED 2026-05-07 banner added

**Architecture summary:**

| Layer | Tool | Role |
|---|---|---|
| Reality | Fireflies / Gmail / Calendar / Apollo | events |
| Canonical CRM | Attio | People / Companies / Deals / Notes / Tasks |
| Canonical mutual action plan | Notion `📈 Deals` `MAP Draft` field | who owes what by when |
| Surfacing | Slack `#desk-monkey` | digest in, free-form text out |
| Memory | repo `memory/runlog.md` + skills/* + CLAUDE.md | durable across runs |

**Phase order in `assistant`:**
1. Sweep — Fireflies / Gmail / Calendar / Apollo / Slack replies since last_run_at
2. Update — `parse-call` for transcripts; label + archive email; refresh MAP Drafts
3. Triage — score deals against drift criteria; build surface-list ranked DEFCON
4. Draft — Gmail drafts, Calendar invite proposals, Liam-owned nudge tasks
5. Digest — post to Slack per `skills/digest.md`
6. Execute replies — read Slack since last digest, parse free-form intent, send / send-invite / rewrite / skip / defer per `skills/slack.md`

**Cadence:** 07:00, 11:00, 14:00, 17:00, 20:00 MT weekdays. 5 routine runs/day = full Pro plan budget.

**Slack reply latency:** ~3hr worst case at 5x/day. Open issue documented in `routines/assistant/README.md`. Real-time options (Zapier sentinel writing trigger files, or Agent SDK migration) deferred until latency becomes painful.

**Tactical follow-ups today (separate from architecture):**
- Craig added to Google Contacts via Zapier (Walicek per LinkedIn/email evidence). Contact ID `people/c3934690368546719101_2026-05-07T08:02:23.747421Z`. Includes job title, company, work email, LinkedIn, Tampa address, source-of-relationship note.
- All 4 saved Notion `📈 Deals` views live: 🔥 Hot This Week, ⏳ Stalled (oldest first), 🌳 By Source, 🤝 Resale / Referral Track.
- Legacy `💰 Deals Pipeline` Notion DB renamed to `📦 ARCHIVED — Deals Pipeline`.
- Sales Hub root section headers updated to flag legacy DBs as do-not-write.

**Open architectural questions still on Liam's plate:**
- Slack MCP: Liam to enable on his end. Once enabled, the new routine is unblocked.
- Real-time Slack→routine triggering: defer until latency is painful.
- Common Room: skipped (community-engagement tool, not aligned with current buyers).
- Per-deal context files (`memory/deals/<slug>.md`): not built yet. Optional improvement when Attio Notes alone aren't enough context.
- Apollo for Miles ("build a Desk-Monkey-equivalent for Miles that parses transcripts into Apollo"): logged as a product idea for later.
