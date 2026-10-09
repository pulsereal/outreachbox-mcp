---
name: outreachbox-deliverability
description: Check sender health and diagnose email deliverability problems in OutreachBox. Use when mail is landing in spam, bounce rates are climbing, a campaign is underperforming, a sender account looks broken, or before the first campaign from a new domain.
---

# Sender health and deliverability

Deliverability is the constraint on everything else: a burned domain makes the best campaign worthless, and recovery takes weeks. Check before launching, not after.

## Inspect the senders

`email_accounts_list` — connected accounts and their status. `status` filters.

`email_accounts_stats` — the important one. Aggregate reputation, usage against daily limits, and send volume across senders.

`email_accounts_get` returns a single account's configuration with secrets stripped. `email_accounts_verify` tests IMAP/SMTP connectivity live — run it whenever an account looks stuck, since a silently broken connection looks exactly like a campaign that is not sending.

## Before the first send from a new domain

Cold domains have no sending reputation, and mailbox providers treat sudden volume from one as spam. Walk the user through this before they launch:

1. **Authentication must be in place** — SPF, DKIM and DMARC on the sending domain. Configured at the DNS provider, not here. Without them, a meaningful share of mail will not reach the inbox at all.
2. **Warm up.** Start at 10–20 sends a day and increase gradually over two to four weeks. The warmup feature in the web app automates this.
3. **Verify the list.** Unverified lists bounce, and bounces are the fastest way to lose a domain.
4. **Use a subdomain.** Send outreach from something like `outreach.example.com` so a problem never touches the primary domain's reputation.

If the user is about to send a few thousand cold emails from a domain registered last week, say so plainly. That is the most valuable thing you can tell them.

## Diagnosing a problem

**Bounce rate climbing.** Pause the campaign with `campaigns_email_pause` first — every further bounce compounds the damage. Hard bounces mean a bad list; verify it before resuming. Soft bounces usually mean volume is too high, so lower `dailyLimit`.

**Landing in spam.** Usually authentication, reputation, or content. Check SPF/DKIM/DMARC first. Then reputation in `email_accounts_stats`. Then the copy: link-heavy mail, spam-trigger words, image-only bodies, and no plain-text alternative all hurt. A plain, personal-looking email outperforms a designed one for cold outreach.

**Nothing sending.** `email_accounts_verify` the sender. Then confirm the campaign actually launched (`campaigns_email_get`), that it has contacts, and that the daily limit has not already been consumed.

**Low open rates with clean delivery.** Subject lines and sender name, not infrastructure. Deliverability is fine; the copy is the problem.

## Sustainable settings

Spread load across senders rather than pushing one hard — set `accountRotation: true` in campaign settings with multiple accounts connected. Keep `businessHoursOnly` and `respectTimezones` on: mail arriving at 3am reads as automated.

Always set `targeting.excludeUnsubscribed` and `excludeBounced`. Re-mailing a bounced address repeatedly is a direct signal to providers that the sender does not maintain its list.

Keep daily limits boring. A campaign that takes three weeks and lands in inboxes beats one that takes three days and lands in spam.
