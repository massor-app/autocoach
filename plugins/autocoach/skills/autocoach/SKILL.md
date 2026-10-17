---
name: autocoach
description: Answer a coach's questions about their clients' session packs and sessions left, sync completed sessions from Google Calendar after the coach confirms them, record pack sales, and fix mistakes. Use for "give me an overview of my clients", "how many sessions does X have left", "which pack did I sell X", "sync with my calendar", "X bought another pack", "X no-showed", or any question about client credits.
---

# autocoach

All numbers come from the autocoach tools. Never calculate or estimate a balance, pace, or run-out date yourself. Every write tool takes `confirmed: true`, and you only pass it after the coach has said yes to that exact change in this conversation. Propose first, then write.

Start by calling `get_setup_status`. If setup is incomplete, say what's missing and offer to run the `setup-autocoach` skill.

## Questions

- **Overview** ("how are my clients doing", "who's running low"): call `get_overview`. Show a compact table: client, sessions left, pack and expiry, last session. Below it, list every flag in plain language:
  - `running_low`: projected run-out date
  - `out_of_sessions`
  - `expiring_unused`: how many credits will be lost, and when

  Flagged clients are the ones the coach probably wants to contact.
- **One client** ("how many does Sam have left", "what did I sell Sam"): call `get_client` with their name. If it returns several matches, ask which one. Answer the question first, then add pack and expiry details, and any flags.
- **History** ("when did Sam last come in"): `get_client` includes recent ledger history.

The overview reflects only sessions recorded so far. If `last_synced_at` is more than a few days old, mention it and offer a sync.

## Sync with the calendar

Only sync when the coach asks. Nothing is saved until they confirm.

1. Call `get_sync_window`. It gives you `since`, `until`, `calendar_ids`, `timezone`, the client match data, and `already_decided`.
2. List events from `since` to `until` on each calendar in `calendar_ids`, using the Google Calendar connector. Follow page tokens until you have every event. Expand recurring events into their individual occurrences, and use each occurrence's own event ID (never the recurring series ID).
3. Drop:
   - all-day events
   - cancelled events
   - events that haven't ended yet
   - every (event, client) pair in `already_decided`; an entry with an empty `client_id` means the whole event was dismissed

   The window overlaps the previous sync by a day on purpose, so expect to drop some already-decided events here.
4. Match each remaining event to clients, in this order:
   1. an attendee email equal to a client email
   2. a client phone number appearing in the description or location (compare digits only)
   3. a client name or alias in the title (case-insensitive, whole word)

   An event can match several clients (e.g. a partner session). Ignore a client on an event they declined. Treat a match as uncertain when it came only from a name that several clients share, or when the title suggests it wasn't a session (e.g. "coffee", "call", "consult", "cancelled", "no-show").
5. Show one list, grouped:
   - **Sessions to record**: date, client, title
   - **Not sure**: what's unclear about each
   - **Not sessions**: unmatched events, briefly
   - **Unknown attendees** who appear more than once: offer to add them as clients

   Ask: "Record these? Tell me anything to change."
6. After the coach confirms or edits the list:
   - Call `record_sessions` with every confirmed (event_id, client_id) pair, using the occurrence's event ID and its start time with the UTC offset. If a result says `conflict`, you used a recurring series ID: find the occurrence's own ID and retry that item.
   - Call `dismiss_events` only for the events the coach agreed are not sessions, so they aren't proposed again. Never dismiss events you dropped in step 3 (still running, all-day, cancelled) or "not sure" items the coach didn't answer.
   - Call `complete_sync` with the window's `until`, but only if no "not sure" items are left open. Otherwise the next sync will show them again.
7. Report the new balances returned by `record_sessions`, and mention anyone who's now flagged (call `get_overview` if needed).

If the Google Calendar connector isn't available, say so: the coach needs to connect Google Calendar under Settings → Connectors on claude.ai.

## Changes

Confirm each change before calling the tool.

- **Sold a pack** ("Sam bought another 10-pack"): `list_packs` to find the pack, then `sell_pack`. If the sale wasn't today, ask when, and pass `sold_on`, since expiry counts from it. If the client was charged sessions while out of credits, the new pack covers them; say so. If the pack isn't in the catalog, offer to add it with `upsert_pack`.
- **Session shouldn't have counted** (a no-show they don't charge for, a cancellation, a mistake): find it in `get_client` history, then `unrecord_session`. This gives the credit back. If `refunded_to` isn't `original_pack`, that pack had expired; tell the coach where the credit went.
- **Charged no-show or late cancel that wasn't on the calendar, a goodwill credit, or a correction**: `adjust_credits` with a clear note.
- **New client, new contact details, nickname, or a client who left**: `upsert_client`. To remove a client, set `archived: true`.
- **Change the catalog**: `upsert_pack`.

After any change, show the client's new sessions left.
