---
name: outreachbox-inbox-triage
description: Triage the OutreachBox unified inbox, classify campaign replies, and draft responses. Use when the user asks what replies came in, wants help answering prospects, asks to clear or summarise the inbox, or wants to find interested leads among campaign responses.
---

# Triaging the inbox

The unified inbox holds mail across every connected sender. Replies to outreach are the point of the whole system, so handle them promptly and never automatically.

## 1. Sync, then read

`inbox_check` with an `emailAccountId` pulls new mail from the mail server. Without it you may be reading a stale view. Run it per account from `email_accounts_list`.

Then `inbox_list`:

```json
{ "type": "received", "isRead": false, "limit": 50 }
```

Filter by `emailAccountId`, `fromDate`/`toDate`, or `search` to narrow. `inbox_stats` gives read/unread counts for a quick summary.

`inbox_list` returns metadata. Use `inbox_get` for the full body before judging intent — subject lines mislead.

## 2. Classify

Sort replies into:

- **Interested** — wants a call, asks a question, requests pricing. Highest value; surface these first with the full text.
- **Not now** — timing objection, "circle back in Q3". Worth a short acknowledgement and a note to follow up.
- **Not interested** — a clear no. Acknowledge briefly and stop. Do not argue.
- **Unsubscribe** — treat as binding immediately, whatever the wording. Remove them from active campaigns; never email again.
- **Out of office** — no action, but a later follow-up date may be extractable.
- **Bounce / auto-reply** — not a human. If bounces are clustering, pause the campaign and read `outreachbox-deliverability`.
- **Not campaign-related** — leave it alone.

Lead with interested replies and unsubscribes. The rest can be a summary.

## 3. Draft, do not send

Draft the reply, show it to the user, and **wait for explicit approval before calling `inbox_reply` or `email_send`.** These send from the user's real mailbox to a real person. Never send on your own initiative, and never batch-send approved and unapproved drafts together.

`inbox_reply` needs `id` and `body`; recipient and `Re:` subject default from the original message. Set `isHtml` to match the body you wrote.

```json
{ "id": "...", "body": "<p>Thanks Ada — Thursday at 10 works. Sending an invite now.</p>", "isHtml": true }
```

`email_send` is for new threads and needs `accountId`, `to`, `subject`, `body`.

Reply from the account that sent the original. Switching senders mid-thread looks like a hand-off and hurts reply rates.

### Writing replies

Match the prospect's register and length. Answer the question actually asked. One clear next step — usually a specific time, not "let me know what works". Drop the marketing voice; by this point it is a conversation.

## 4. Close the loop

`inbox_read` marks a message handled so it stops reappearing in unread triage. Mark messages as you finish them, not in a batch at the end.

For anyone who asked to stop, confirm they are excluded from running campaigns. `targeting.excludeUnsubscribed` covers future campaigns but check nothing in flight will reach them.
