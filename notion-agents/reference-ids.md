# Reference IDs

Every UUID, page ID, draft ID, and external system ID the agents (or the tool that builds them) will read or write. Single source of truth — when a value changes, change it here and only here.

## Workspace + top-level

| Thing | ID |
|---|---|
| Workspace name | Liam Glennie's Workspace HQ |
| Workspace ID | `2e11acdf-10d2-812c-b35a-00420a9f042a` |
| User (sole) | liam@deskmonkeyai.com |
| 🦧 Desk Monkey Sales Hub (root page) | `5213634e-4f45-4e32-8cc7-bd4ca837001b` |
| 🚧 Master Notion Agent (skill templates) | `2831acdf-10d2-80ca-9914-c92cb5fd89b5` |

## CRM databases

Pass the **data source ID** (collection://) to agents and DDL. Pass the **page ID** when navigating in the UI.

| DB | Page ID | Data source ID |
|---|---|---|
| 📈 Deals | `fba03253-8130-41e5-a98e-1bb90f7296f3` | `5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4` |
| 👤 Contacts | `4398061e-5070-48ec-a39e-754c547f0690` | `474cee31-b5fe-45e6-906a-b8463eada553` |
| 🧾 Activity Log | `8b15d6dd-962d-4e55-98a9-2aca108abee1` | `b4edca94-b39f-4f9d-9b29-e53cde7b71c6` |
| 🛠️ Projects | `e5c0fab3-5c99-4d50-9d9f-20b580ba6edd` | `d02e88ab-1d9c-4ee6-a551-23c5b3b1bd2b` |
| 🚨 DEFCON Tasks (revivable) | `f8f4d6d6-df40-4091-bec7-bcaaa5c12be3` | `e16abae8-b4c2-4cb1-a41c-0b53a583a44e` |

## Active Deals

| Deal | Page ID | Stage | Champion / Primary Contact |
|---|---|---|---|
| Houston Foam (HFP USA) | `28d39ba2-6ba9-450d-93be-201a481f130a` | POC | Miles Kurtz / Craig Walicek |
| Anthropic Identity | search via Master Agent | POC | TBD |
| Blue Ally / Andrew Brink | NOT YET CREATED | Discovery | Andrew Brink |
| MyVP / Anthony Orlovsky | NOT YET CREATED | New | Anthony Orlovsky |
| CIO Tech (Craig's own program) | search; created May 6 | New | Craig Walicek |

## Active Contacts

| Contact | Page ID | Email | Role |
|---|---|---|---|
| Craig Walicek | `c98ca6b9-2540-4f65-8624-5d83ac4bb3e6` | cwalicek@ciotech.us | Champion + EB on CIO Tech; Referrer on HFP |
| Andrew Brink | NOT YET CREATED | abrink@blueally.com | Lead, BlueAlly |
| Anthony Orlovsky | NOT YET CREATED | anthony@thevirtualvp.net (also aorlofski@gmail.com) | Lead, MyVP |
| Miles Kurtz | search | mileskurtz@hfpusa.com | Champion-in-training, HFP |
| Chris St. Thomas | search | TBD | Referrer (intro source for HFP, MyVP, BlueAlly) |

Spelling note: Attio had "Anthony Orlovsky" but LinkedIn + email show "Orlofski". Verify with Liam.

## Important sub-pages

| Page | ID |
|---|---|
| 📥 Transcript Inbox | `3511acdf-10d2-8129-851f-ef27eee85a91` |
| 🚀 Apollo Outbound Campaign — Dormant Customer Reactivation | `3571acdf-10d2-81c9-a8bf-d229fe391091` |
| Houston Foam — Desk Monkey Audit (project space) | `7ae73e41-d87f-482b-a8a5-5bcea385ffc2` |
| Mutual Action Plan + Next Steps (HFP) | `23e4c4b9-aada-4e9f-a9d4-8b828b73785b` |
| HFP Outreach Sequence v2 — Apollo (May 6) | `cbc6fa29-b0da-4cc6-bb66-ed9cab65c001` |
| Skills (Master Agent skill list) | `48896ec1-2275-4527-b0d7-8fd07c5d6d79` |
| Writing standard (humanizer) | `d1964669-64b9-43b9-84f2-5bbca3b8b81d` |

## Notion native meeting notes (recently relevant)

| Title | ID | Likely Deal |
|---|---|---|
| CRM and Sales Workflow Discussion (May 6 17:32 UTC) | `3581acdf-10d2-80a0-947e-fde39dcc209d` | HFP USA (Sales POC kickoff) |
| Meeting (May 6 16:03 UTC) | `3581acdf-10d2-80d4-be93-f1652c5490cb` | possibly HFP |
| AI Kickoff (May 4) | `3561acdf-10d2-8043-a012-f67202910ae8` | HFP USA |
| Notion Task Pipeline and Automation Improvements (May 4) | `3561acdf-10d2-80cd-871c-dd679e413e63` | internal |
| Customer Outreach Automation Discussion (May 4) | `3561acdf-10d2-8068-ab84-dce77251559c` | likely HFP |
| Sales Process Review (May 1) | `3531acdf-10d2-80e5-86d4-d660487e3769` | internal |
| Automation review (May 1) | `3531acdf-10d2-80b5-a56b-f2bd28090672` | internal |
| Sales & Client Strategy Discussion (Apr 30) | `3521acdf-10d2-80b8-a0f2-f2cedfe5b349` | HFP intro (Chris+Liam Craig intro) |
| Sales Pipeline and Notion Architecture Planning (Apr 28) | `3501acdf-10d2-807b-b4b7-e22a7f748c3f` | already embedded in HFP Deal page |

## Calendar events (next 7 days, relevant)

| Event | ID | When (MT) | Notes |
|---|---|---|---|
| CIO Tech + Apollo (desk monkey) | `ei97ak4krvoui8bt353io0kkio` | Fri May 8 1pm-1:30pm | Craig only; HFP working session |
| Notion walkthrough: Liam + Andrew | `lrjjbo8daeat9p98b0jsu3jk2k` | Fri May 8 4pm-5pm | Andrew RSVP'd Tentative |
| HFP cold call blitz (tentative) | `ueu1671393huf1f4fg50u9n93o` | Thu May 14 12:15pm-1:15pm | Miles + Craig invited, both needsAction |

## Gmail draft IDs (created today, awaiting Liam send)

| Draft | Recipient | Purpose | Status |
|---|---|---|---|
| `r-590308765881101588` | Miles + Craig | Wed POC recap + MAP for Fri AM | ready |
| `r-347385264924036069` | Miles + Craig | Tentative May 14 invite check | ready |
| `r4705530137615562383` | Craig only | POC format question + fly-out offer | ready |
| `r-5440387312552631650` | Andrew Brink | Apology + alt-time windows | ready |
| `r9078906750335126332` | Anthony Orlovsky | Polite re-up on May 6 thread | ready |
| `r3953375124665551055` | Miles only | Friday May 15 dialer windows (OBSOLETE; delete manually) | obsolete |

## Slack

| Thing | ID |
|---|---|
| Workspace | deskmonkey.slack.com |
| Channel `#all-desk-monkey` | `C0B28H75D4N` |
| Liam user | `U0B2PFNV2HF` |
| Last digest message ts | `1778187656.444339` |

## Attio (RETIRED 2026-05-07 — for migration reference only)

These are read-once for the Phase 4 migration, then ignored forever.

| Record | Attio ID |
|---|---|
| HFPUSA Deal | `d46eba25-c6dd-463c-925c-f4c873ea581b` |
| Andrew Brink Person | `bb67f9df-e12f-4936-83b6-8b9fdf622a5c` |
| Anthony Orlovsky Person | `781df056-f2bb-4f32-b315-a0ae73635d87` |
| Craig Walicek Person | `da5f4a8e-19c8-456c-bc90-3895611bc099` |
| Miles Kurtz Person | `8e5a1a8d-8dcc-4deb-aac5-79faa62f0d59` |
| BlueAlly Company | `1149f0c9-cadf-4ca0-891a-2e883d6bd8c4` |
| BRIEF Note CIO Tech (Fri May 8) | `665433a8-97ca-475d-a4e1-c8e2806bf930` |
| BRIEF Note Andrew walkthrough | `4d445d8e-6f2f-4b99-a5bf-f94b78ae5041` |
| MTG Note Sales POC May 6 | `e8f02091-799c-42fd-b066-60eca04f649e` |
| Open Liam Task: Friday May 15 dialer windows | `93fa365e-a347-4cdf-ac95-fe3d8f6da51d` |
| Open Liam Task: POC demo deliver | `242f9fe9-9550-4632-9691-73c0053298f3` |
| Open Liam Task: Apollo to Excel dashboard | `587fc0d6-04eb-46cf-93d9-09f868867c85` |
| Closed Liam Task: HFP USA email sequence first cut | `f8f558e7-83f9-4735-8439-4e0d39855750` |

## Deskmonkey identity

| | |
|---|---|
| Name | Liam Glennie |
| Email | liam@deskmonkeyai.com |
| Phone | 720-431-2310 |
| Domain | deskmonkeyai.com |
| Backup email (legacy) | liamglennie@gmail.com |

## GitHub

| | |
|---|---|
| Repo | `lg2k2000/sales-automation` |
| Current dev branch | `claude/optimistic-keller-UkbdJ` |
| Open PR | #8 (assistant run 2026-05-07T20:00Z) |
