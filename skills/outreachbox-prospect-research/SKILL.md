---
name: outreachbox-prospect-research
description: Find new prospects, research target accounts, or discover influencers with OutreachBox research campaigns. Use when the user wants to find leads matching a profile, build a list from scratch, research companies or competitors before outreach, or identify creators and influencers in a niche.
---

# Finding prospects

Three research campaign types, same shape, different intent:

| Tool family | Finds | Use for |
| --- | --- | --- |
| `campaigns_prospects_*` | Individual people matching a profile | Building an outbound list |
| `campaigns_sales_*` | Intelligence on specific accounts | Pre-call research, account planning |
| `campaigns_influencer_*` | Creators and influencers | Partnerships, sponsorships |

All three are **asynchronous**: create starts the work, results arrive later via `get`.

## 1. A project is required

Every research campaign needs a `projectId`. `projects_list` first; `projects_create` if the user has none. The project supplies the brand context the search is interpreted against, so a vague project produces vague results.

## 2. Create the campaign

```json
{
  "name": "Series B fintech CTOs",
  "projectId": "...",
  "query": "CTOs at Series B fintech companies in the UK",
  "searchType": "company_leads",
  "keywords": ["fintech", "payments", "Series B"],
  "contactDiscovery": true,
  "tags": ["q4-outbound", "fintech"]
}
```

`query` is natural language and carries most of the weight. Be specific about role, industry, company stage and geography — "CTOs at Series B fintech companies in the UK" beats "fintech leads" by a wide margin.

`searchType` accepts `web`, `news`, `images`, `maps`, `company_leads`. Use `company_leads` for people, `maps` for local businesses, `news` for recent events and trigger-based outreach.

`contactDiscovery: true` attempts to find email addresses. Leave it off when you only need company-level intelligence — it consumes quota.

Tag consistently. Tags are how the resulting contacts get selected into a campaign later.

## 3. Poll for results

`campaigns_prospects_get` with the id. Discovery takes minutes, not seconds. Poll at a sensible interval and tell the user it is running rather than blocking silently — do not hammer it in a tight loop.

## 4. Review before using

Discovered contacts land in the organization's contact list. Pull them with `contacts_list` filtered by the tag you set.

Check quality before building a campaign on them: look at a sample, confirm the job titles and companies actually match the brief, and discard what does not. Research output is a starting point, not a verified list. Sending to unverified addresses damages the sending domain — `outreachbox-deliverability` covers why that matters.

Then hand off to `outreachbox-cold-email-campaign`.

## Notes

- `campaigns_prospects_list` shows previous runs; check it before starting a near-duplicate search.
- Results depend on plan quota. A run that returns little may have hit a limit rather than found nothing.
- For market and topic intelligence rather than people, use `research_topic_create` (see `outreachbox-content-engine`).
