---
name: autocoach
description: Answer a coach's questions about their clients' session packs and sessions left, sync completed sessions from Google Calendar after the coach confirms them, record pack sales, and fix mistakes. Use for "give me an overview of my clients", "how many sessions does X have left", "which pack did I sell X", "sync my calendar", "update my sessions", "X bought another pack", "X no-showed", "X came in today", or any question about client credits.
---

# autocoach

All numbers come from the autocoach tools. Never calculate or estimate a balance, pace, run-out date, or which pack paid for a session yourself. Every write tool takes `confirmed: true`, and you only pass it after the coach has said yes to that exact change in this conversation. The one exception is `propose_sessions`: it saves sessions as waiting, which changes no balance.

Start by calling `get_overview`:
- **A `setup` block** means onboarding isn't finished. Say what's missing and offer to run the `setup-autocoach` skill.
- **`waiting_sessions` that aren't empty** means sessions from an earlier sync still need the coach's answer. Mention them briefly.

## Questions

- **Overview** ("how are my clients doing", "who's running low"): use `get_overview`.
  - Show a compact table: client, sessions left, active pack and last usable day, last session.
  - Below it, list every flag in plain language:
    - `running_low`: with the projected run-out date
    - `out_of_sessions`
    - `expiring_unused`: how many credits will be lost, and when

    Flagged clients are the ones the coach probably wants to contact.
- **One client** ("how many does Sam have left", "what did I sell Sam"): call `get_client` with their name.
  - If `client` is null, look at `matches`: if there are several, ask which one; if there are none, say so.
  - Answer the question first, then give pack details and any flags.
  - Quote `sessions_left`. `paid_by` on a session can shift as past sessions or sales change.
- **History** ("when did Sam last come in"): `get_client` → `recent_sessions`.

## Update my sessions (calendar sync)

Only sync when the coach asks. Nothing counts until they confirm.

1. **Range.** `get_overview` → `sync` gives `from`, `to`, `google_calendar_ids` and `timezone`. Use another range only if the coach names one ("check all of September"). Sessions before onboarding are already in the opening balances, and the server will say so.
2. **Read.** List events from `from` to `to` on each calendar, using the Google Calendar connector. Follow page tokens until you have every event. Expand recurring events into their occurrences, and use each occurrence's own event ID (never the recurring series ID).
3. **Filter.** Drop all-day events, cancelled events, and events that haven't ended yet.
4. **Match** each event to clients using `get_overview` → `clients[].match`, in this order:
   1. an attendee email equal to a client email
   2. a client phone number appearing in the description or location (compare digits only)
   3. a client name or alias in the title (case-insensitive, whole word)

   An event can match several clients (e.g. a partner session). Ignore a client on an event they declined. Drop events that match nobody: don't list them for the coach.
5. **Propose.** Call `propose_sessions` with every match. Each result says:
   - `new` or `already_waiting`: needs the coach's answer
   - `already_confirmed`, `already_dismissed`, `already_voided`: already answered; skip silently
   - `before_onboarding`: already in the opening balance; skip
   - `possible_duplicate`: the client already has a session that day (maybe added by hand). Point it out.
   - `start_at_changed`: an answered event now has a different time. Tell the coach, since it may have been moved.
6. **Ask.** Show one list of the sessions that need an answer (date, client, title), and flag anything uncertain:
   - a name that several clients share
   - a title that suggests it wasn't a session ("coffee", "call", "consult", "cancelled", "no-show")
   - a possible duplicate

   Ask: "Did these happen? Tell me anything to change."
7. **Save.**
   - `confirm_sessions` with the ones that happened.
   - `void_sessions` with the ones that didn't; a note is optional here.
   - Leave unanswered ones waiting. They'll show up next time.
8. **Report** the new balances returned by `confirm_sessions`, and anyone now flagged.

If the Google Calendar connector isn't available, say so: the coach needs to connect Google Calendar under Settings → Connectors on claude.ai.

## Changes

Confirm each change before calling the tool. After any change, show the client's new sessions left.

- **A session that isn't on the calendar** ("Sam came in today", "Priya late-cancelled, charge her"): use `add_session` with a `note`. If it refuses because the client already has a session that day, ask the coach. Pass `allow_duplicate: true` only if it really was a second session.
- **A session that shouldn't count** (a no-show they don't charge for, or a mistake): find it in `get_client` → `recent_sessions`, then `void_sessions` with a note. This gives the credit back. `confirm_sessions` undoes it.
- **Sold a pack** ("Sam bought another 10-pack"):
  - Find the pack type in `get_overview` → `pack_types`.
  - If the sale wasn't today, ask when, and pass `sold_on`, since expiry counts from it.
  - If the client was out of credits, the new pack covers those sessions; say so.
  - If the pack type isn't in the catalog, offer to add it with `add_pack_type`.
- **Goodwill credit, extension, or a sale entered wrongly:** use `update_pack` with a note.
  - `sessions_total` adds sessions.
  - `last_usable_day` extends or shortens the pack.
  - `voided: true` cancels a sale that went to the wrong client or was the wrong pack (sell the right one afterwards).
- **New client, new contact details, nickname, or a client who left:** use `add_client` or `update_client`. To remove a client, set `archived: true`.
- **Change the catalog:** `update_pack_type`. Packs already sold keep their terms.
