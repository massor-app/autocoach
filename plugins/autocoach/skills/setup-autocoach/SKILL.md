---
name: setup-autocoach
description: Set up autocoach for a coach: connect it, choose the session calendar, build the client list, enter the packs they sell, and record each client's current sessions left. Use when the coach runs /setup-autocoach, says they want to set up or start using autocoach, or when get_setup_status shows setup is incomplete.
---

# Set up autocoach

You are onboarding a fitness coach. Keep it conversational: one topic at a time, short questions, and a summary they can confirm before anything is saved. Every write tool takes `confirmed: true`. Only pass it after the coach has said yes to that exact change.

## 1. Check the connections

- **autocoach tools** (`get_setup_status` and the rest). If they're missing or ask for sign-in, tell the coach to run `/mcp`, choose **autocoach**, and enter the invite code they were given. Then continue.
- **Google Calendar connector** (tools that list calendars and events). If it's missing, tell the coach to connect Google Calendar under Settings → Connectors on claude.ai, then restart Claude. They can finish the rest of setup without it, but calendar syncs won't work until it's connected.

Call `get_setup_status`. If something is already done, skip that step and say so. Re-running setup must never duplicate clients or packs: check `list_clients` / `list_packs` first.

## 2. Pick the session calendar

List the coach's calendars and ask which one(s) hold client sessions. Suggest a calendar whose name looks like training if there is one; otherwise suggest their primary calendar. Always save the chosen calendar's time zone too (ask only if it looks wrong). Dates, expiry, and "today" all use it. Save with `update_settings` (`calendar_ids`, `timezone`).

## 3. Build the client list

Offer both options, and let the coach do either or both:

- **Scan the calendar.** Ask first: "Can I look through the last 90 days of that calendar to find your regular clients?" If they say yes, list events from 90 days ago until now. Group them by attendee email (ignore the coach's own address and resource rooms), and by names in event titles for events with no attendees. Propose the people who show up at least twice: name, email, how many sessions you saw, and the most recent one.
- **Paste a list.** Accept any format: names, emails, phone numbers.

For each client, collect a name and every way they appear on the calendar. That means emails (attendee addresses), phone numbers (if the coach puts them in event descriptions), and aliases (nicknames or short names used in titles, e.g. "Sam" for "Samantha Patel"). Show the final list and save each client with `upsert_client` once the coach confirms.

## 4. The pack catalog

Ask what session packs they sell, e.g. "10 sessions, expires after 90 days" or "5-pack, no expiry". Capture a name, the number of sessions, and the expiry in days (or none). autocoach doesn't track prices; if the coach mentions money, don't store it. Confirm, then `upsert_pack` each one.

## 5. Current balances

For every client without a pack (`get_setup_status` → `clients_without_pack`), ask which pack they're on, roughly when they bought it (to work out expiry), and how many sessions they have left. Batch it: show a table the coach can correct in one reply, pre-filled with anything you learned from the calendar scan.

Record each one with `sell_pack`, passing `sold_on` (if known) and `sessions_used` = pack sessions − sessions left. If a client has more left than the pack holds (e.g. rolled-over credits), sell with `sessions_used: 0` and add the extra with `adjust_credits`. If a client has no active pack, skip them, and they'll show 0 sessions left.

## 6. Finish

Show a summary: the calendar(s), the number of clients, the packs, and each client's sessions left (from `get_overview`). When the coach confirms, call `update_settings` with `onboarding_complete: true`. This starts calendar syncing from now, so sessions already reflected in the opening balances won't be counted twice. Don't sync the calendar before this step.

Finally, tell them how to use it day to day:
- **"Sync with my calendar":** I'll find completed sessions and ask you to confirm them.
- **"How many sessions does Sam have left?"**, **"Give me an overview of my clients"**
- **"Sam bought another 10-pack"**, **"Sam no-showed Tuesday, don't charge it"**
