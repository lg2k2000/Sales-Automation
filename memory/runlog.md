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
