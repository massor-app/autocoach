---
name: setup-autocoach
description: Set up autocoach for a coach: connect it, choose the session calendar, build the client list, enter the packs they sell, and record each client's current sessions left. Use when the coach runs /setup-autocoach, says they want to set up or start using autocoach, or when get_overview returns a setup block.
---

# Set up autocoach

You are onboarding a fitness coach. Make it feel like a friendly setup wizard:
- Start with a welcome, before anything else.
- Then go one step at a time. Start each step's message with a bold label like **Step 2 of 6: Your calendar**, so the coach always knows where they are.
- Keep messages short, with one question or decision at a time.

**When to ask first.** Every write tool takes `confirmed: true`.
- **A direct, complete instruction from the coach counts as approval.** For example: "Priya bought a 10-pack today", "Sam no-showed on the 8th, don't charge it", "add Mike Davis, mike.davis@example.com". Do it, then say what you saved.
- **Ask first when you inferred anything:** calendar matches, which of two clients, a date or time the coach didn't give, a nickname you came up with, or any change the coach didn't explicitly ask for.
- During setup, show a summary and get a yes for anything you assembled yourself: a client list from the calendar scan, nicknames, or a balances table. Things the coach just told you, like "I sell a 10-pack good for 90 days", don't need a second confirmation.

## Welcome (always first)

The first thing the coach sees is a welcome message. Before it, don't call any tools, start a task list, or look anything up. The one exception: if the autocoach tools are already connected, you may call `get_overview` once, quietly, to check whether this is a returning coach.

**New coach:** send this, adapting the wording but keeping every point:

> Hi! I'm going to help you set up **autocoach**.
>
> autocoach keeps track of your clients, the session packs they've bought, and how many sessions each one has left. Once it's set up, you can ask me things like "who's running low?" or "how many sessions does Sam have left?". When you say "update my sessions", I'll check your Google Calendar and record the sessions you confirm.
>
> Setup takes about 10 minutes, in 6 steps:
> 1. Connect autocoach
> 2. Pick the calendar your sessions are on
> 3. Add your clients
> 4. Add the packs you sell
> 5. Enter everyone's current sessions left
> 6. Check everything and finish
>
> Nothing is saved without your OK, and you can stop anytime and pick up where you left off by running `/setup-autocoach` again.
>
> First, what should I call you?

Wait for their answer, then use their name from then on. Save it with `update_settings` (`coach_name`) as soon as autocoach is connected.

**Returning coach** (`get_overview` has a `coach_name` and a `setup` block): skip the name question. Welcome them back by name, list which of the 6 steps are already done and which are left (from `setup.missing`), and continue with the first one that's left. If there's no `setup` block, setup is already finished: say so, and offer the daily commands from step 6.

## Step 1 of 6: Connect autocoach

- **autocoach tools** (`get_overview` and the rest). If they aren't available, or you're told autocoach needs authentication, walk the coach through signing in. You can't sign in for them, but you can make it easy. Send these steps and wait:
  1. "Type **/mcp** right here in this chat and press Enter." This opens the list of connected servers, in both the desktop app and the terminal. Never send them to a separate terminal.
  2. "Pick **autocoach** and choose **Sign in** (or **Authenticate**)."
  3. "A browser page opens. Enter the invite code you were given and click **Connect**. When it says you're connected, close the tab."
  4. "Come back here and say **done**."

  When they say done, call `get_overview` to check. If it still fails, ask what they saw:
  - an "invite code didn't work" message means the code was mistyped
  - no browser page means they should try `/mcp` → autocoach → Sign in again

  While they're signing in, you can already list their calendars for step 2, so no time is lost. Just don't save anything until autocoach works.
- **Google Calendar connector** (tools that list calendars and events). If it's missing, tell the coach to connect Google Calendar under Settings → Connectors on claude.ai, then start a new chat. They can finish setup without it, but calendar syncs won't work until it's connected.

Once connected, save their name (`update_settings` with `coach_name`), then call `get_overview`. Its `setup` block lists what's `missing` and the `next_step`. Skip anything that's already done and say so. Re-running setup must never duplicate anything: `add_client` and `add_pack_type` refuse duplicates, and if one does, use the existing record.

## Step 2 of 6: Your calendar

List the coach's calendars and ask which one(s) hold client sessions. Suggest a calendar whose name looks like training if there is one; otherwise suggest their primary calendar. Save the chosen calendar IDs **and that calendar's time zone** with `update_settings` (`google_calendar_ids`, `timezone`). Ask about the time zone only if it looks wrong: dates, expiry and "today" all use it.

## Step 3 of 6: Your clients

Offer both options, and let the coach do either or both:

- **Scan the calendar.** Ask first: "Can I look through the last 90 days of that calendar to find your regular clients?" If they say yes, list events from 90 days ago until now. Group them by attendee email (ignore the coach's own address and resource rooms), and by names in event titles for events with no attendees. Propose the people who show up at least twice: name, email, how many sessions you saw, and the most recent one.
- **Paste a list.** Accept any format: names, emails, phone numbers.

For each client, collect a name and every way they appear on the calendar:
- **emails:** attendee addresses
- **phones:** if the coach puts numbers in event descriptions
- **aliases:** nicknames or short names used in titles, e.g. "Sam" for "Samantha Patel"

Show the final list, and save each client with `add_client` once the coach confirms.

## Step 4 of 6: Your packs

Ask what session packs they sell, e.g. "10 sessions, good for 90 days" or "5-pack, no expiry". Capture a name, the number of sessions, and `expires_after_days` (the sale day counts as day 1), or none. autocoach doesn't track prices; if the coach mentions money, don't store it. Confirm, then `add_pack_type` each one.

## Step 5 of 6: Current balances

For every client in `setup.clients_without_pack`, ask:
- which pack they're on
- roughly when they bought it (for the expiry)
- how many sessions they have left

Batch it: show a table the coach can correct in one reply, pre-filled with anything you learned from the calendar scan.

Record each one with `sell_pack`, passing `sold_on` (if known) and `sessions_left`. If a client has more left than the pack holds (rolled-over credits), also pass `sessions_total` with the real number. If a client has no active pack, skip them; they'll show 0 sessions left.

## Step 6 of 6: Check and finish

Show a summary from `get_overview`: the calendar(s), the clients, the packs, and each client's sessions left. When the coach confirms, call `update_settings` with `onboarded: true`.

From that moment, calendar syncs only look at sessions after it, because everything before is already in the opening balances. Don't sync the calendar before this step.

Finally, congratulate them by name, and tell them how to use it day to day:
- **"Update my sessions":** I'll check your calendar and ask you to confirm what happened
- **"How many sessions does Sam have left?"**, **"Give me an overview of my clients"**
- **"Sam bought another 10-pack"**, **"Sam no-showed Tuesday, don't charge it"**, **"Sam came in today"**
