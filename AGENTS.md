# autocoach plugin: agent guide

> **Testing: ZERO unit tests. End-to-end only, against staging. See "Testing policy" below.**

This repo is only the Claude Code plugin coaches install: skills plus an MCP connection. It's public, so never commit secrets, invite codes, or client data.

The server (Cloudflare Worker, Durable Object, MCP tools) lives in the private repo `massor-app/autocoach-server` (locally `~/Developer/autocoach-server`). Its `docs/SPEC.md` is the design and its `AGENTS.md` covers environments and deploys.

## Layout

- `.claude-plugin/marketplace.json`: makes the repo installable with `/plugin marketplace add massor-app/autocoach`.
- `plugins/autocoach/.claude-plugin/plugin.json`: the plugin manifest. Bump `version` on every release.
- `plugins/autocoach/.mcp.json`: points at the production server, `https://autocoach.justinfan827.workers.dev/mcp`.
- `plugins/autocoach/skills/setup-autocoach/`: onboarding.
- `plugins/autocoach/skills/autocoach/`: daily use and calendar sync.

The skills call the server's MCP tools by name (`get_overview`, `propose_sessions`, `confirm_sessions`, and so on). If a tool changes in `autocoach-server`, update the skills to match.

## Testing policy: ZERO unit tests, end-to-end only

This is a hard rule from the project owner, not a preference.

- **This repo has zero unit tests and must stay that way.** Don't add unit tests, integration tests, snapshot tests, mocks, stubs, fakes, or test doubles. Don't add a test runner or framework either: no vitest, jest, mocha, node:test, miniflare, or `@cloudflare/vitest-pool-workers`. No `test` script in package.json, and no `*.test.*` / `*.spec.*` files or `test/` / `__tests__/` folders.
- **Verification is end-to-end only:** real Claude, the real plugin and skills, the real MCP server deployed to **staging**, and the real Google Calendar.
- **The e2e suite still needs to be built.** It's planned as a `claude -p` runner. For each case it will: run the real OAuth flow, load starting state through `/admin/seed`, drive Claude through the plugin with scripted coach replies via `--resume`, assert on `/admin/state`, and reset afterwards. Once it exists, a change is done only when that suite passes.
- **If something isn't covered,** add an e2e case, not a unit test. If you think a unit test is needed, stop and ask the owner instead.

The e2e suite will live in `massor-app/autocoach-server` and run this plugin from a local checkout against staging. Before pushing plugin changes, run `claude plugin validate .`.
