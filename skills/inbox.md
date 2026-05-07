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
Vendor           services I pay for, e.g. Apollo, Notion, Anthropic, Attio (gray)
Personal         non-Desk-Monkey personal mail (gray)

System/Receipts
System/Newsletters
System/Notifications
```

The underscore prefix sorts state labels (`_Action`, `_Waiting`, `_Reference`) to the top of the Gmail sidebar so they're always visible. The `System/` prefix nests utility labels under one parent, out of the way.

11 user labels total (plus the `System` parent that auto-nests the System/* children).

## Mapping Attio Deal Stage → Relationship label

When `coworker` or `daily-review` processes a Gmail thread tied to an Attio Person Record, it applies one relationship label based on the Person's linked Deal Stage:

- **Closed Won** → `Customer`
- **Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement** → `Prospect`
- **Closed Lost / Nurture / Unresponsive** → `Prospect` (still in funnel conceptually; manual move to `Customer` after Closed Won)
- **Person's Relationship attribute = Referrer or Ally** → `Partner` (overrides Deal Stage; surface the Partner role since it's more actionable)

If the Person is a vendor (services Liam pays for), label `Vendor` (manually classified in Attio; vendors don't have Deals).

If no Person Record match: leave un-labeled. It's general inbox noise.

## After-action playbook

| What happened | What to do |
|---|---|
| Accepted calendar invite | Apply relationship label, archive (calendar holds the event) |
| Declined or rescheduled invite | Apply relationship label, archive |
| Tentative invite | Leave in inbox with `_Action` until decided |
| Replied, expecting reply back | Apply `_Waiting` + relationship label, archive |
| Replied, thread closed | Apply relationship label, archive (no `_Waiting`) |
| Got a reply needing action | Apply `_Action` + relationship label, leave in inbox |
| Receipt / newsletter / notification | Auto-classified by coworker Step 2a, skips inbox |
| Sat in `_Waiting` >5 days | `daily-review` creates a soft-nudge Attio Task. Email stays in `_Waiting`. |

## Filters — routine-based, not Gmail-UI-based

**Important:** Gmail filter creation is not exposed by Zapier MCP or the direct Gmail MCP. Liam doesn't manually set up filters. Instead, the `assistant` routine Phase 2b sweeps the inbox on every run, classifies, labels, archives noise, and aggressively deletes + unsubscribes from marketing/promo email.

**Trade-off:** worst-case ~3-hour delay (since assistant runs 5x weekdays / 2x+ weekends) between when a noisy email lands and when it gets cleaned. Compared to in-product filters (which fire instantly), small lag is acceptable given the alternative is manual filter setup.

The noise-classification rules:

| Pattern | Action |
|---|---|
| Subject/body contains `unsubscribe` OR sender on known newsletter list (substack, mailchimp, sendgrid, mailerlite, hubspot, marketo, klaviyo, customer.io) | label `System/Newsletters` + archive (don't delete; some are useful) |
| Billing / receipt senders (stripe, billing@, invoice@, anthropic/notion/apollo/attio/google/zapier/slack billing, no-reply receipt patterns) | label `System/Receipts` + archive (NEVER delete — tax records) |
| `notifications@github.com`, `calendar-notification@google.com`, no-reply / donotreply patterns from non-marketing senders | label `System/Notifications` + archive |

The relationship-type labels (Customer, Prospect, Partner, Vendor) are NOT applied via noise classification. Phase 2 (Update) applies those based on Attio Person Record lookup when processing Deal-related threads.

## Aggressive promo/marketing cleanup (Phase 2b — sanitation)

This goes beyond labeling. The routine actively deletes promotional and marketing email and tries to unsubscribe at the source. Run on every assistant invocation.

### Detection rules (multi-signal, conservative)

An email triggers sanitation only when **at least 2 of these signals match**:

- Sender domain on the known marketing-platform list (`mailchimp.com`, `mcdlv.net`, `sendgrid.net`, `mailerlite.com`, `hubspotemail.net`, `marketo.com`, `klaviyomail.com`, `customeriomail.com`, `mailgun.net`, `sendinblue.com`, `convertkit.com`, `activehosted.com`, `cmail19.com`, etc.)
- `List-Unsubscribe` header present on the message
- Body contains `View in browser` OR `Update your preferences` OR `If you no longer wish to receive`
- Subject contains `% off`, `sale`, `deal`, `discount`, `flash`, `limited time`, `last chance`, `final hours`, `clearance`, `exclusive offer`
- Sender is NOT in the Attio People records (cross-reference by `from` address)
- Sender is NOT on the Receipt allowlist (NEVER delete receipts)
- Sender is NOT on the Personal allowlist (Liam's family / friends — separate file `memory/allowlists/personal.md` if it exists)

### Action sequence per matched email

1. **Extract unsubscribe URL.**
   - First try the `List-Unsubscribe` header (standard email header). If present and starts with `<https://...>`, that's the URL.
   - Otherwise scan body for the literal `unsubscribe` link href.
   - If neither found, skip the unsubscribe step (still proceed to delete).

2. **Hit the unsubscribe URL.**
   - Use `WebFetch` (GET) to the URL. Most legit unsubscribe links work as a single GET.
   - If the URL response includes a confirmation page or "you've been unsubscribed" message, log success.
   - If it requires a button-click POST, log `unsubscribe-needs-manual` and surface in the digest later.

3. **Delete the email** via Zapier `delete_email` (action key: `delete_email` on the Gmail Zapier app). Trash, not archive — these don't deserve to keep your storage.

4. **Log to runlog:**
   ```
   PROMO_DELETED: <sender_domain>, <subject_truncated>, unsubscribe=<success|needs-manual|none>
   ```

5. **Aggregate in digest** (Phase 5): count of cleanups in the run, list of domains where unsubscribe needed manual click. Format:
   ```
   📨 Inbox sanitation: 14 promos deleted (domains: mailchimp, klaviyo, sendgrid).
       3 unsubscribes need manual click — see thread.
   ```

### Allowlists (don't touch even if they look promotional)

Maintained in the routine prompt or a future `memory/allowlists/` file:

- **Receipts (never delete):** stripe.com, anthropic.com, notion.so, attio.com, apollo.io, slack.com, zapier.com, google.com (billing), microsoft.com (billing), aws.amazon.com (billing), linkedin.com (billing only)
- **Liam-personal (never touch):** any sender Liam has flagged manually. Default empty until populated.
- **Active deals (never touch):** any sender on an Attio Person Record linked to a non-Closed Deal. Re-checked every run.

### Conservative defaults

- If only 1 signal matches → archive only, don't delete. (e.g., subject has "sale" but sender is a known person → just archive.)
- If sender has ever replied to Liam's outbound → never delete. Build this from Gmail's `from:` history.
- If unsubscribe attempt fails (HTTP error, 4xx/5xx) → still delete the email, but flag for manual unsubscribe in the digest.
- If the routine deletes more than 50 emails in one run → cap at 50, surface remainder in digest, wait for next run. Prevents runaway accidental deletion if rules drift.

### Risk + recovery

False positives are inevitable. Mitigations:

- Gmail's Trash retains deleted emails for 30 days. Recoverable.
- The runlog records every deletion with sender + subject snippet, so you can ctrl-F and recover specific ones.
- Liam can add senders to a "never delete" allowlist by replying to a digest with `allowlist <sender>` (parsed in Phase 6 per `skills/slack.md`).

## Daily Liam flow

1. Open Gmail. Inbox has 0-15 items (auto-filters caught the noise).
2. Triage each in 30 seconds: action / waiting / reference / archive. Apply state label, archive what you can.
3. End of day: inbox empty.
4. Once a week: scan `_Waiting` view. Anything sitting there >5 days will also have an Attio nudge task by then.

## What `assistant` does with labels

When processing a Gmail thread from an Attio Person:
1. Apply relationship label per the mapping above (Customer / Prospect / Partner / Vendor).
2. Apply `_Action` if the thread needs Liam's reply.
3. Apply `_Waiting` if Liam already replied and is waiting on them.
4. Never apply two state labels at once. `_Action` and `_Waiting` are mutually exclusive.

When sweeping for left-on-read prospects (Phase 3 — Triage):
1. Find threads with relationship label = `Prospect` AND last reply from counterpart >5d ago.
2. Don't change Gmail labels. Surface as a Liam-owned Attio nudge Task in Phase 4 + a digest item in Phase 5.

## Hard rules

- NEVER label individual emails by company or deal name. Use Gmail search.
- NEVER apply both `_Action` and `_Waiting`. Mutually exclusive.
- NEVER auto-archive an email tagged `_Action`. Liam needs to act on it.
- NEVER apply `_Action` to mail in `System/*`. System mail is noise; if something there needs action, it's wrongly auto-classified, fix the rules.
