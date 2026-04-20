# Skill 1: Parse Call

## Purpose
Turn a completed Fireflies transcript into clean CRM data, follow-up tasks, and a reviewable draft email.

## Trigger
- Fireflies transcript complete → Make.com webhook → Claude Routine `parse-call`
- Manual fallback: "parse my last call"

## Connected tools
- Fireflies
- HubSpot
- Gmail
- Apollo
- Notion

## Inputs
- Completed Fireflies transcript
- Meeting metadata
- HubSpot records
- Gmail evidence if participant identity is unclear
- Apollo enrichment only if needed

## Workflow
1. Pull the completed transcript and metadata.
2. Identify external participants.
3. Resolve name, title, company, and email.
   - Check HubSpot first for exact email matches, then name + company.
   - Use Gmail and Apollo only when HubSpot + transcript are insufficient.
4. Update or create the right contact and company records.
5. Write the meeting summary as a structured HubSpot activity note, not as a raw transcript dump.
6. Only map to fields that currently exist in HubSpot. For the current live schema, update only when supported by the transcript:
   - **contact:** Email, Phone Number, LinkedIn account, Company Name, `project_objective`, Budget, Lead Status
   - **company:** City, Lifecycle Stage, Lead Status, Industry, Last Contacted
7. Create or update a HubSpot deal when the call involves an external prospect. Skip deal creation for internal coordination calls or partner-only calls where no external prospect is present. When creating a deal, set: deal name, associated contact, associated company, and deal stage based on transcript evidence.
8. Create follow-up tasks in HubSpot for each clear action item from the transcript. Link tasks to the deal if one exists, otherwise link to the contact. Set realistic due dates based on urgency signals in the transcript.
9. Draft a follow-up email for Chris to review.
10. If identity confidence is low, write a Notion review item instead of guessing.
11. If future custom fields do not exist yet, skip them without failing the run.

## Output format
- Participants resolved
- CRM actions taken (contacts, companies, deals, tasks)
- Call summary
- Pain / blockers
- Next steps
- Draft follow-up
- Ambiguities / open questions

## Out of scope
- Sending texts
- Booking meetings
- Enrolling in Apollo
- Auto-replying

## Guardrails
- Prefer exact email match first
- Do not guess identity if confidence is low
- Do not overwrite clearly better HubSpot data with weaker enrichment
- Do not dump raw transcript into HubSpot
- Do not write to contact or company properties unless the field already exists and the transcript clearly supports it
- Do not force values into Budget, Industry, City, Lifecycle Stage, or Lead Status when the evidence is weak
- Do not create a deal for internal coordination or partner-only calls (no external prospect present)
- Do not create tasks without a clear action item supported by the transcript
