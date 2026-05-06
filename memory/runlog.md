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
