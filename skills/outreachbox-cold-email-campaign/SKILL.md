---
name: outreachbox-cold-email-campaign
description: Plan, build and launch a cold email or drip sequence in OutreachBox end to end. Use when the user wants to run an outreach campaign, send a cold email sequence, follow up with a contact list, or asks to pause, resume, stop or check on an existing email campaign.
---

# Running a cold email campaign

Five stages: sender, audience, sequence, launch, monitor. Do them in order — a campaign created without contacts or a sender will fail at launch, not at creation.

## 1. Confirm a sender exists

`email_accounts_list`. You need an `accountId` with healthy status. None connected means stop and tell the user to connect a mailbox in the web app; this cannot be done through the API.

For a first-ever campaign from a new domain, read `outreachbox-deliverability` before sending anything. Blasting from a cold domain is the single most common way to burn it.

## 2. Build the audience

For a list the user supplies, `contacts_bulk` upserts up to 1000 at a time, matching on email. Chunk larger lists.

```json
{ "contacts": [
  { "email": "ada@example.com", "firstName": "Ada", "lastName": "Lovelace",
    "company": "Example Co", "jobTitle": "CTO", "tags": ["q4-outbound"] }
] }
```

Tag every import. Tags are how you select the audience later and how you keep campaigns from overlapping.

To find new people rather than import them, use `outreachbox-prospect-research` first.

Verify the set with `contacts_list` (filter by `tag`) and keep the returned ids for `contactIds`.

## 3. Design the sequence

`campaigns_email_create` with `type: "sequence"` for multi-step outreach. Other types: `one-time`, `drip`, `ab-test`.

Steps cap at **5**. Each needs `stepNumber` (1-based), `name`, and `delay`. Step 1 should have `delay: { value: 0, unit: "minutes" }`.

```json
{
  "name": "Q4 CTO outbound",
  "type": "sequence",
  "contactIds": ["..."],
  "emailAccountId": "...",
  "settings": {
    "dailyLimit": 50,
    "businessHoursOnly": true,
    "respectTimezones": true,
    "timezone": "America/New_York"
  },
  "targeting": { "excludeUnsubscribed": true, "excludeBounced": true },
  "steps": [
    {
      "stepNumber": 1,
      "name": "Opener",
      "subject": "{{firstName}}, quick question about {{company}}",
      "content": "<p>Hi {{firstName}},</p><p>...</p>",
      "delay": { "value": 0, "unit": "minutes" },
      "conditions": { "stopOnReply": true }
    },
    {
      "stepNumber": 2,
      "name": "Bump",
      "subject": "Re: {{firstName}}, quick question about {{company}}",
      "content": "<p>Floating this back up.</p>",
      "delay": { "value": 3, "unit": "days" },
      "conditions": { "stopOnReply": true }
    }
  ]
}
```

Set `stopOnReply: true` on every follow-up. Without it the sequence keeps emailing people who already replied, which reads as spam and is the fastest way to lose a prospect.

`{{firstName}}`, `{{lastName}}`, `{{company}}`, `{{jobTitle}}` interpolate from the contact. Only use a variable you actually populated — an empty one renders as a blank and gives the send away.

Reusable copy belongs in a template (`templates_create`), referenced per step by `templateId`.

### Writing the copy

Short. One ask. Specific to the recipient — if the same mail could go to anyone on the list unchanged, it will perform like it. Give the user a draft and let them edit before launching.

## 4. Launch

Campaigns are created in draft. **Confirm with the user before `campaigns_email_launch`** — summarise recipient count, step count, daily limit and sender, then wait for an explicit yes. Launching sends real mail to real people and cannot be unsent.

Start conservative on a new domain: `dailyLimit` of 20–50, raised over a week or two.

## 5. Monitor and adjust

`campaigns_email_get` for status and stats. `inbox_list` with `type` filtered to replies picks up responses — see `outreachbox-inbox-triage`.

- `campaigns_email_pause` — reversible, use at the first sign of bounces or complaints
- `campaigns_email_stop` — permanent, ends the campaign

Check within the first hours of a launch. A bounce rate above a few percent means the list needs verifying; pause rather than let it run.
