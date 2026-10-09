---
name: outreachbox-getting-started
description: Orient yourself in an OutreachBox workspace before doing anything else. Use when the user first mentions OutreachBox, when a tool returns a 401/403 or "no active API token", when you need to find a projectId or an email account id, or when the user asks what they can do with OutreachBox.
---

# Getting started with OutreachBox

OutreachBox is an email outreach and marketing automation platform. The tools here act on a single **organization** (workspace), determined by the API key — you never pass an organization id.

## Orient before acting

Two ids come up constantly and are worth fetching once at the start of a session:

1. **`projectId`** — most research and content tools require one. Call `projects_list`. If the organization has no projects, create one with `projects_create` (needs `name`, `description`, `url`); the URL is crawled to build brand context, so use the user's real site.
2. **`accountId`** — any tool that sends mail needs a connected sender. Call `email_accounts_list`. If it returns nothing, the user must connect a mailbox in the web app first; you cannot do it through the API.

Do not guess ids. If a required id is missing, ask.

## What is available

| Area | Tools |
| --- | --- |
| Contacts | `contacts_create`, `contacts_bulk`, `contacts_list`, `contacts_get`, `contacts_update`, `contacts_delete` |
| Email campaigns | `campaigns_email_create`, `..._list`, `..._get`, `..._launch`, `..._pause`, `..._stop` |
| Research campaigns | `campaigns_prospects_*`, `campaigns_sales_*`, `campaigns_influencer_*` |
| Topic research | `research_topic_create`, `..._list`, `..._get` |
| Content | `content_generate`, `content_list`, `content_get`, `content_calendar_*`, `designs_generate` |
| Social | `social_accounts_list`, `social_publish` |
| Email infrastructure | `email_accounts_list`, `..._stats`, `..._get`, `..._verify` |
| Inbox | `inbox_list`, `inbox_stats`, `inbox_check`, `inbox_get`, `inbox_read`, `email_send`, `inbox_reply` |
| Templates | `templates_list`, `templates_get`, `templates_create`, `templates_update`, `templates_delete` |
| Analytics | `analytics_members`, `analytics_user` |
| Projects | `projects_list`, `projects_create`, `projects_get`, `projects_update`, `projects_delete` |

Related skills go deeper: `outreachbox-cold-email-campaign`, `outreachbox-prospect-research`, `outreachbox-inbox-triage`, `outreachbox-content-engine`, `outreachbox-deliverability`, `outreachbox-analytics-report`.

## Always confirm before these

These either contact real people or destroy data. State exactly what will happen and get an explicit yes first — never chain them automatically off an earlier instruction.

- `campaigns_email_launch` — begins sending to every contact in the campaign
- `email_send`, `inbox_reply` — sends mail from the user's real mailbox
- `social_publish` — posts publicly
- `contacts_delete`, `templates_delete`, `projects_delete` — permanent

## Reading results

Write tools return an `appUrl`. Surface it as a link so the user can see the result in the product.

A tool you expected may be missing from your tool list: the server only advertises what the API key's scopes permit. A read-only key genuinely cannot send mail, and the fix is a new key, not a retry.

## When something fails

| Symptom | Cause and fix |
| --- | --- |
| 401 or 403 | Key is wrong, expired, or revoked. The user creates a new one at `/apiEndpoints` in the web app. |
| "No active API token found" | In-app agent chat has no key for the organization; create one at `/apiEndpoints`. |
| 429 | Organization rate limit. Back off and retry; do not loop. |
| Connection refused / no response | `OUTREACHBOX_BASE_URL` points somewhere unreachable. |
| Tool missing from your list | The key's scopes exclude it. |

`npx -y @outreachbox/mcp doctor` from a terminal validates the key, names the organization, and reports how many tools the scopes allow.
