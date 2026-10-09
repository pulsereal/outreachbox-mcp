---
name: outreachbox-analytics-report
description: Report on OutreachBox campaign and team performance. Use when the user asks how campaigns are doing, wants a weekly or monthly outreach summary, asks which campaign performed best, or wants team member activity metrics.
---

# Reporting on performance

## Gathering the numbers

| Source | Tool |
| --- | --- |
| Per-campaign stats | `campaigns_email_get` for each id from `campaigns_email_list` |
| Team member activity | `analytics_members` (optional `userId` for one person) |
| Current user dashboard | `analytics_user` |
| Reply volume | `inbox_stats`, and `inbox_list` filtered by date |
| Sender health | `email_accounts_stats` |

There is no single cross-campaign endpoint. For a portfolio view, list campaigns and fetch each one, then aggregate yourself. Mention how many campaigns you covered so the user knows the scope.

## Interpreting it

Report rates, not just totals — "412 sent, 38% opened, 6% replied" is useful; "412 sent" is not. Always give the denominator.

Rough bands for cold outreach, as orientation rather than targets:

- **Bounce rate** above ~3% is a list problem and needs acting on, not noting.
- **Open rate** is a weak signal now that mail clients prefetch images. Treat a change as directional, not precise, and never optimise for it alone.
- **Reply rate** is the number that matters for outreach. Low single digits is normal for cold; anything above that is working.
- **Positive reply rate** matters more than reply rate. Fifty angry replies is not success. `outreachbox-inbox-triage` covers classifying them.

Small samples lie. Do not draw conclusions about a campaign that has sent to 40 people, and say so rather than reporting a percentage that looks authoritative.

## Making it useful

A good report answers "what should I change?":

- Which campaign has the best **positive** reply rate, and what is different about it — audience, subject, sequence length?
- Where does the sequence lose people? If replies all come from step one, the follow-ups are adding noise rather than value.
- Any sender degrading in `email_accounts_stats`? That is urgent and outranks the rest of the report.
- Any campaign still running that should be stopped?

End with a short list of concrete recommendations, not a wall of numbers. Two or three changes the user can act on this week.

## Comparing over time

These tools return current state, not history. For week-over-week or month-over-month, either the user supplies the earlier figures or you scope the comparison with date filters on `inbox_list`. Do not invent a baseline — if you cannot source the prior period, say so.
