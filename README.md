# autocoach

Track your clients' session packs and sessions left from Claude. autocoach reads your Google Calendar, finds the sessions you've completed, and records them once you confirm. You can ask things like "who's running low?" or "how many sessions does Sam have left?" at any time.

## For the coach

You need the Claude desktop app on a paid plan, with **Google Calendar** connected (claude.ai → Settings → Connectors).

1. Install the plugin:
   - **Desktop app:** open Settings → **Plugins**, click **+ Add**, add the marketplace `massor-app/autocoach`, then install **autocoach**.
   - **Terminal:** run `claude plugin marketplace add massor-app/autocoach`, then `claude plugin install autocoach@autocoach`.
2. Start a new chat. Type `/mcp` in it, pick **autocoach**, choose **Sign in**, and enter the invite code you were given on the page that opens. (`/setup-autocoach` will walk you through this too.)
3. Run `/setup-autocoach`. It walks you through choosing your session calendar, your clients, the packs you sell, and everyone's current sessions left.

Then just ask:

- "Update my sessions": checks your calendar and asks you to confirm what happened. Nothing counts until you say yes, and anything you don't answer waits for next time.
- "Give me an overview of my clients" / "Who's running low?"
- "How many sessions does Sam have left?" / "Which pack did I sell Priya?"
- "Sam came in today" (a session that isn't on your calendar)
- "Sam bought another 10-pack"
- "Sam no-showed Tuesday, don't charge it"
- "Give Sam one extra session" / "Extend Priya's pack two weeks" / "That pack was for Samantha, not Sam"

To get updates later, run `/plugin marketplace update autocoach` in the Code tab.

**Using regular Claude chat or the phone app?** Add a custom connector with the URL `https://autocoach.justinfan827.workers.dev/mcp` and the same invite code. The tools work there too; only the `/setup-autocoach` and `/autocoach` shortcuts are specific to Claude Code.

## For maintainers

See [AGENTS.md](AGENTS.md). The server lives in a separate private repo, `massor-app/autocoach-server`.
