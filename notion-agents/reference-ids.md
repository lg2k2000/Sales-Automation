# Reference IDs

Every Notion UUID an agent will read or write. Plus Liam's identity for email signatures.

## Workspace + top-level

| Thing | ID |
|---|---|
| Workspace | Liam Glennie's Workspace HQ (`2e11acdf-10d2-812c-b35a-00420a9f042a`) |
| User (sole) | liam@deskmonkeyai.com |
| 🦧 Desk Monkey Sales Hub (root) | `5213634e-4f45-4e32-8cc7-bd4ca837001b` |
| 🚧 Master Notion Agent (skill templates) | `2831acdf-10d2-80ca-9914-c92cb5fd89b5` |

## CRM databases (agents read+write these)

Pass the **data source ID** when you scope an agent. Pass the **page ID** when navigating in the UI.

| DB | Page ID | Data source ID |
|---|---|---|
| 📈 Deals | `fba03253-8130-41e5-a98e-1bb90f7296f3` | `5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4` |
| 👤 Contacts | `4398061e-5070-48ec-a39e-754c547f0690` | `474cee31-b5fe-45e6-906a-b8463eada553` |
| 🧾 Activity Log | `8b15d6dd-962d-4e55-98a9-2aca108abee1` | `b4edca94-b39f-4f9d-9b29-e53cde7b71c6` |
| 🛠️ Projects | `e5c0fab3-5c99-4d50-9d9f-20b580ba6edd` | `d02e88ab-1d9c-4ee6-a551-23c5b3b1bd2b` |
| 🚨 DEFCON Tasks | `f8f4d6d6-df40-4091-bec7-bcaaa5c12be3` | `e16abae8-b4c2-4cb1-a41c-0b53a583a44e` |

## Active Deals (referenced by agent prompts for testing)

| Deal | Page ID | Stage |
|---|---|---|
| Houston Foam (HFP USA) | `28d39ba2-6ba9-450d-93be-201a481f130a` | POC |
| Anthropic Identity | search by name | POC |
| Blue Ally / Andrew Brink | search by name | Discovery |
| MyVP / Anthony Orlovsky | search by name | New |
| CIO Tech | search by name | New |

## Active Contacts (Champion / EB / Referrer roles)

| Contact | Page ID | Email |
|---|---|---|
| Craig Walicek | `c98ca6b9-2540-4f65-8624-5d83ac4bb3e6` | cwalicek@ciotech.us |
| Miles Kurtz | search by name | mileskurtz@hfpusa.com |
| Andrew Brink | search by name | abrink@blueally.com |
| Anthony Orlovsky | search by name | anthony@thevirtualvp.net |

## Native meeting notes data source

Notion's built-in `meeting_notes` data source. Agents query this for new transcripts. No fixed ID — use Notion's `query_meeting_notes` tool with date filters.

Test meeting note for Phase 2 verification:
- May 6 17:32 UTC — "CRM and Sales Workflow Discussion" — `3581acdf-10d2-80a0-947e-fde39dcc209d` — should link to Houston Foam Deal

## Pages agents create / write

| Page | ID | Created by |
|---|---|---|
| 🦧 Daily Brief | TBD on first Triage run | Triage agent (Phase 2.4) |
| 📋 Run Log (optional, if tracking) | TBD | first agent that needs it |

## Slack (for Daily Brief fallback)

| Thing | ID |
|---|---|
| Workspace | deskmonkey.slack.com |
| `#all-desk-monkey` channel | `C0B28H75D4N` |
| Liam user | `U0B2PFNV2HF` |

## Liam's identity (for email signature)

```
Best
--
Liam Glennie
720-431-2310
deskmonkeyai.com
```

Backup email: liamglennie@gmail.com (legacy, still receives some inbound).
