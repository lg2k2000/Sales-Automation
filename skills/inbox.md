# Inbox Strategy

Called by `coworker` and `daily-review` when applying Gmail labels. Read once at the top of any session that touches Gmail labels.

## Principle

Organize by **role**, not by **named entity**. Roles persist for years; entities (deals, companies, individuals) come and go. The label scheme is built around what kind of relationship someone has with Liam, not which specific deal they're attached to.

For finding company-specific or person-specific email, use Gmail search (`from:cwalicek@ciotech.us` or `from:@hfpusa.com`). Faster than scrolling a label tree, zero maintenance.

## Label set (timeless)

```
_Action          needs my response (red)
_Waiting         I sent something, waiting on them (yellow)
_Reference       keep for context, no action needed (gray)

Customer         pays or has paid (green)
Prospect         active sales conversation (blue)
Partner          referrers, co-sellers, design partners (purple)
Vendor           services I pay for, e.g. Apollo, Notion, Anthropic (gray)
Personal         non-Desk-Monkey personal mail (gray)

System/Receipts
System/Newsletters
System/Notifications
```

The underscore prefix sorts state labels (`_Action`, `_Waiting`, `_Reference`) to the top of the Gmail sidebar so they're always visible. The `System/` prefix nests utility labels under one parent, out of the way.

11 user labels total (plus the `System` parent that auto-nests the System/* children).

## Mapping Deal Stage → Relationship label

When `coworker` or `daily-review` processes a Gmail thread tied to a Notion Contact, it applies one relationship label based on the Contact's linked Deal Stage:

- **Closed Won** → `Customer`
- **Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement** → `Prospect`
- **Closed Lost / Nurture / Unresponsive** → `Prospect` (still in the funnel conceptually; if revived later, manual move to `Customer` after Closed Won)
- **Contact Relationship = Referrer or Ally** → `Partner` (overrides Deal Stage; surface the Partner role since it's more actionable)

If the Contact is a vendor (services Liam pays for), label `Vendor` (manually classified in Notion; vendors don't have Deals).

If no Contact match: leave un-labeled. It's general inbox noise.

## After-action playbook

| What happened | What to do |
|---|---|
| Accepted calendar invite | Apply relationship label, archive (calendar holds the event) |
| Declined or rescheduled invite | Apply relationship label, archive |
| Tentative invite | Leave in inbox with `_Action` until decided |
| Replied, expecting reply back | Apply `_Waiting` + relationship label, archive |
| Replied, thread closed | Apply relationship label, archive (no `_Waiting`) |
| Got a reply needing action | Apply `_Action` + relationship label, leave in inbox |
| Receipt / newsletter / notification | Auto-filtered, skips inbox |
| Sat in `_Waiting` >5 days | `daily-review` creates a soft-nudge DEFCON Task. Email stays in `_Waiting`. |

## Filters to set up manually in Gmail UI

The Gmail API doesn't expose filter creation. These are one-time manual setup at Settings → Filters and Blocked Addresses → Create new filter:

1. **Calendar invitation auto-replies** — match `from:calendar-notification@google.com` → Skip Inbox + apply `System/Notifications`
2. **GitHub notifications** — match `from:notifications@github.com` → Skip Inbox + apply `System/Notifications`
3. **Newsletters** — match each newsletter sender or use `unsubscribe` keyword in body → Skip Inbox + apply `System/Newsletters`
4. **Receipts** — match billing senders (Stripe, Apollo, Notion, Anthropic billing, Google billing) → Skip Inbox + apply `System/Receipts`
5. **Vendor support replies** — vendor support emails → apply `Vendor`, leave in inbox

The relationship-type labels (Customer, Prospect, Partner) are NOT applied via filter. `coworker` does that automatically based on Notion Contact lookup. Filters only handle the System/* noise categories.

## Daily Liam flow

1. Open Gmail. Inbox has 0-15 items (auto-filters caught the noise).
2. Triage each in 30 seconds: action / waiting / reference / archive. Apply state label, archive what you can.
3. End of day: inbox empty.
4. Once a week: scan `_Waiting` view. Anything sitting there >5 days will also have a DEFCON nudge task by then.

## What `coworker` does with labels

When processing a Gmail thread from a Notion Contact:
1. Apply relationship label per the mapping above (Customer / Prospect / Partner / Vendor).
2. Apply `_Action` if the thread needs Liam's reply.
3. Apply `_Waiting` if Liam already replied and is waiting on them.
4. Never apply two state labels at once. `_Action` and `_Waiting` are mutually exclusive.

## What `daily-review` does with labels

When sweeping for left-on-read prospects:
1. Find threads with relationship label = `Prospect` AND last reply from counterpart >5d ago.
2. Don't change Gmail labels. Surface as a DEFCON soft-nudge Task instead.

## Hard rules

- NEVER label individual emails by company or deal name. Use Gmail search.
- NEVER apply both `_Action` and `_Waiting`. Mutually exclusive.
- NEVER auto-archive an email tagged `_Action`. Liam needs to act on it.
- NEVER apply `_Action` to mail in `System/*`. System mail is noise; if something there needs action, it's wrongly auto-filtered, fix the filter.
