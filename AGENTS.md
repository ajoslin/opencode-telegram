# AGENTS.md

Guidance for agentic coding assistants working in this repo.
Scope: repository root (applies to all files).

## Overview
- Project: Telegram mirror for OpenCode sessions.
- Primary runtime: Bun (root) 
- Language: TypeScript with ESM modules and strict type checking.
- Keep changes small and aligned with existing style.

## Repo Layout
- `src/`: Bun-based Telegram mirror bot.
- `tsconfig.json`: root TS config (strict, ESM).

## Commands (Root - Telegram Mirror)
Install dependencies:
  - `bun install`
Run the bot:
  - `bun run start`
Run from source (explicit):
  - `bun run src/main.ts`
Typecheck:
  - `bun run typecheck`

## Tests

The test harness enables **autonomous testing feedback loops** by mocking external dependencies. Instead of hitting real Telegram servers, the bot talks to a local mock server that:
1. Serves pre-recorded updates from fixture files
2. Captures all outgoing Telegram API calls for inspection

This allows fully automated test runs without any external dependencies.

### Test Files
- `test/fixtures/sample-updates.json` - Mock Telegram updates
- `test/mock-server.ts` - Mock server (serves updates + captures API calls)
- `test/run-test.ts` - Test runner (starts mock server + bot together)

### Running Tests

**Quick test run** (30 seconds, uses mock OpenCode if no OPENCODE_URL):
```bash
bun run test:run
```

**With custom timeout**:
```bash
bun run test/run-test.ts test/fixtures/sample-updates.json 60
```

**Manual testing** (two terminals):
```bash
# Terminal 1: Start mock server
bun run test:mock-server

# Terminal 2: Start bot with mock endpoints
TELEGRAM_UPDATES_URL=http://localhost:3456/updates \
TELEGRAM_SEND_URL=http://localhost:3456 \
TELEGRAM_BOT_TOKEN=test:token \
TELEGRAM_CHAT_ID=-1003546563617 \
bun run start
```

### Mock Server Control Endpoints

The mock server at `http://localhost:3456` provides:
- `GET /_control?action=status` - Server stats (updates served, requests captured)
- `GET /_control?action=captured` - All captured Telegram API requests
- `POST /_control?action=inject` - Inject new updates dynamically
- `GET /_control?action=reset` - Reset update pointer and clear captured requests

### Adding Test Fixtures

Add new fixture files to `test/fixtures/` as JSON arrays of updates in the DO format:
```json
[
  {
    "update_id": 123,
    "chat_id": "-1003546563617",
    "payload": { "update_id": 123, "message": { ... } }
  }
]
```

## Single-File Checks
- Root: `bun run typecheck` validates `src/**/*`.
- Lint single file: `npx next lint --file app/page.tsx`.
- Use `npm run lint -- --file <path>` if you prefer npm.

## TypeScript & Module Conventions
- TypeScript `strict` is enabled in both packages.
- ESM modules are used; prefer `import`/`export` syntax.
- Use `import type` for type-only imports (see `src/telegram.ts`).
- Prefer explicit return types for exported/public functions.
- Avoid `any`; use `unknown` + narrowing when needed.
- Prefer `interface` for object shapes and `type` for unions.
- Keep JSON parsing typed (cast to known interfaces).
- Prefer `const` over `let`; keep functions small and focused.

## Formatting
- Indentation: 2 spaces.
- Semicolons are omitted in existing files.
- Root `src/` uses double quotes.
- Keep line length readable (roughly 100–120 chars).
- Use trailing commas in multi-line objects/arrays.
- Separate logical blocks with blank lines.

## Imports
- Order: Node built-ins (`node:`), external deps, internal modules.
- Keep internal imports relative (no absolute paths in root).
- Avoid unused imports; remove when refactoring.

## Naming
- Files: kebab-case or lower-case with hyphens (existing pattern).
- Functions: `camelCase`.
- Classes: `PascalCase`.
- Types/interfaces: `PascalCase`.
- Constants: `UPPER_SNAKE_CASE` when truly constant.
- Prefer descriptive names (e.g., `sessionId`, `updatesUrl`).

## Error Handling
- Use `try/catch` around network and IO calls.
- Log errors with context and return safe defaults.
- Throw `Error` for fatal conditions (e.g., invalid bot token).
- Use `String(error)` when logging unknown errors.
- Prefer early returns for invalid state.
- Use `console.error` + `process.exit(1)` only for fatal startup errors.

## Logging
- Root uses `createLogger()` and `log(level, message, extra)`.
- Use `log("info" | "warn" | "error" | "debug", ...)` in bot code.
- Avoid `console.log` except for fatal startup errors.

## Async & Concurrency
- Use `async/await` for clarity.
- Avoid blocking loops; use `await Bun.sleep(...)` for backoff.
- If retrying, log retry context and keep delays reasonable.

## Data & API Handling
- Validate request payloads before use.
- When reading env vars, allow file config to be overridden.
- Normalize ids to strings for comparisons.
- Keep Telegram API interactions resilient (retry without Markdown).

## Config & Environment
- Root config loads from:
  - `~/.config/opencode/telegram.json`
  - `<repo>/.opencode/telegram.json`
- Environment variables override config:
  - `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, `TELEGRAM_UPDATES_URL`, `TELEGRAM_SEND_URL`
- Diff viewer base URL uses `VERCEL_URL` or `NEXT_PUBLIC_BASE_URL`.

## Dependency Management
- Root uses Bun; avoid adding npm scripts unless required.
- Keep dependency upgrades minimal and justified.

## Generated Files
- Do not commit `dist/` or `node_modules/`.
- Root build output goes to `dist/` (see `tsconfig.json`).

## Code Organization
- Prefer small helpers over large monolithic functions.
- Keep side effects near the edges (IO, network).
- Keep types co-located with their usage when small.

## Updating This File
- Update commands if scripts change.
- Add new tool or lint rules as they are introduced.
- Keep this file around ~150 lines.

## Cursor/Copilot Rules
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` found.
- If added later, summarize them here.

<!-- opensrc:start -->

## Source Code Reference

Source code for dependencies is available in `opensrc/` for deeper understanding of implementation details.

See `opensrc/sources.json` for the list of available packages and their versions.

Use this source code when you need to understand how a package works internally, not just its types/interface.

### Fetching Additional Source Code

To fetch source code for a package or repository you need to understand, run:

```bash
npx opensrc <package>           # npm package (e.g., npx opensrc zod)
npx opensrc pypi:<package>      # Python package (e.g., npx opensrc pypi:requests)
npx opensrc crates:<package>    # Rust crate (e.g., npx opensrc crates:serde)
npx opensrc <owner>/<repo>      # GitHub repo (e.g., npx opensrc vercel/ai)
```

<!-- opensrc:end -->
