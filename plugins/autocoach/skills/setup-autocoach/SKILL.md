---
name: setup-autocoach
description: Set up autocoach for a coach: connect it, choose the session calendar, build the client list, enter the packs they sell, and record each client's current sessions left. Use when the coach runs /setup-autocoach, says they want to set up or start using autocoach, or when get_overview returns a setup block.
---

# Set up autocoach

You are onboarding a fitness coach. Keep it conversational: one topic at a time, and short questions.

**When to ask first.** Every write tool takes `confirmed: true`.
- **A direct, complete instruction from the coach counts as approval.** For example: "Priya bought a 10-pack today", "Sam no-showed on the 8th, don't charge it", "add Mike Davis, mike.davis@example.com". Do it, then say what you saved.
- **Ask first when you inferred anything:** calendar matches, which of two clients, a date or time the coach didn't give, a nickname you came up with, or any change the coach didn't explicitly ask for.
- During setup, show a summary and get a yes for anything you assembled yourself: a client list from the calendar scan, nicknames, or a balances table. Things the coach just told you, like "I sell a 10-pack good for 90 days", don't need a second confirmation.

## 1. Check the connections

- **autocoach tools** (`get_overview` and the rest). If they're missing or ask for sign-in, tell the coach to run `/mcp`, choose **autocoach**, and enter the invite code they were given. Then continue.
- **Google Calendar connector** (tools that list calendars and events). If it's missing, tell the coach to connect Google Calendar under Settings → Connectors on claude.ai, then restart Claude. They can finish setup without it, but calendar syncs won't work until it's connected.

Call `get_overview`. Its `setup` block lists what's `missing` and the `next_step`. Skip anything that's already done and say so. Re-running setup must never duplicate anything: `add_client` and `add_pack_type` refuse duplicates, and if one does, use the existing record.

## 2. Pick the session calendar

List the coach's calendars and ask which one(s) hold client sessions. Suggest a calendar whose name looks like training if there is one; otherwise suggest their primary calendar. Save the chosen calendar IDs **and that calendar's time zone** with `update_settings` (`google_calendar_ids`, `timezone`). Ask about the time zone only if it looks wrong: dates, expiry and "today" all use it.

## 3. Build the client list

Offer both options, and let the coach do either or both:

- **Scan the calendar.** Ask first: "Can I look through the last 90 days of that calendar to find your regular clients?" If they say yes, list events from 90 days ago until now. Group them by attendee email (ignore the coach's own address and resource rooms), and by names in event titles for events with no attendees. Propose the people who show up at least twice: name, email, how many sessions you saw, and the most recent one.
- **Paste a list.** Accept any format: names, emails, phone numbers.

For each client, collect a name and every way they appear on the calendar:
- **emails:** attendee addresses
- **phones:** if the coach puts numbers in event descriptions
- **aliases:** nicknames or short names used in titles, e.g. "Sam" for "Samantha Patel"

Show the final list, and save each client with `add_client` once the coach confirms.

## 4. The pack catalog

Ask what session packs they sell, e.g. "10 sessions, good for 90 days" or "5-pack, no expiry". Capture a name, the number of sessions, and `expires_after_days` (the sale day counts as day 1), or none. autocoach doesn't track prices; if the coach mentions money, don't store it. Confirm, then `add_pack_type` each one.

## 5. Current balances

For every client in `setup.clients_without_pack`, ask:
- which pack they're on
- roughly when they bought it (for the expiry)
- how many sessions they have left

Batch it: show a table the coach can correct in one reply, pre-filled with anything you learned from the calendar scan.

Record each one with `sell_pack`, passing `sold_on` (if known) and `sessions_left`. If a client has more left than the pack holds (rolled-over credits), also pass `sessions_total` with the real number. If a client has no active pack, skip them; they'll show 0 sessions left.

## 6. Finish

Show a summary from `get_overview`: the calendar(s), the clients, the packs, and each client's sessions left. When the coach confirms, call `update_settings` with `onboarded: true`.

From that moment, calendar syncs only look at sessions after it, because everything before is already in the opening balances. Don't sync the calendar before this step.

Finally, tell them how to use it day to day:
- **"Update my sessions":** I'll check your calendar and ask you to confirm what happened
- **"How many sessions does Sam have left?"**, **"Give me an overview of my clients"**
- **"Sam bought another 10-pack"**, **"Sam no-showed Tuesday, don't charge it"**, **"Sam came in today"**
