# autocoach

Track your clients' session packs and sessions left from Claude. autocoach reads your Google Calendar, finds the sessions you've completed, and records them once you confirm. You can ask things like "who's running low?" or "how many sessions does Sam have left?" at any time.

## For the coach

You need the Claude desktop app on a paid plan, with **Google Calendar** connected (claude.ai → Settings → Connectors).

1. In the desktop app's **Code** tab, run:
   ```
   /plugin marketplace add massor-app/autocoach
   /plugin install autocoach@autocoach
   ```
2. Restart Claude Code. When autocoach asks you to sign in (or run `/mcp` → **autocoach**), enter the invite code you were given.
3. Run `/setup-autocoach`. It walks you through choosing your session calendar, your clients, the packs you sell, and everyone's current sessions left.

Then just ask:

- "Sync with my calendar"
- "Give me an overview of my clients"
- "How many sessions does Sam have left?"
- "Which pack did I sell Priya?"
- "Sam bought another 10-pack"
- "Sam no-showed Tuesday, don't charge it"

**Using regular Claude chat or the phone app?** Add a custom connector with the URL `https://autocoach.justinfan827.workers.dev/mcp` and the same invite code. The tools work there too; only the `/setup-autocoach` and `/autocoach` shortcuts are specific to Claude Code.

## For maintainers

See [AGENTS.md](AGENTS.md). The server lives in a separate private repo, `massor-app/autocoach-server`.
