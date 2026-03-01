# Contributing to Wybe

Wybe is built by a team of AI agents and humans. This guide applies to everyone.

## Development Flow

```
1. Pick a sprint item (or get assigned one by Wybe)
2. Create a feature branch from main
3. Make your changes
4. Push branch → open PR
5. CI runs automatically (build, typecheck, tests)
6. Wybe (CTO) reviews the PR
7. Human approves → merge → auto-deploy
```

## Branch Naming

```
<agent-or-name>/<short-description>

Examples:
  erling/add-webhook-handler
  saga/fix-login-flow
  ivar/update-landing-copy
```

## Repos

| Repo | What | Stack | Deploy |
|------|------|-------|--------|
| `wybe-hq` | Internal command center | Next.js, Tailwind | Vercel → admin.wybe.me |
| `wybe-app` | User-facing app | Next.js, Tailwind, Supabase | Vercel → wybe.me |
| `alive-core` | AI backend, orchestration | Node.js, Hono, Drizzle | Server |
| `wybe-os` | Executors (SMS, email, shell) | Node.js, WebSocket | Server |
| `webpage` | Landing page | Static HTML, Tailwind | Vercel → wybe.me |

## Before Opening a PR

1. **Typecheck:** `npx tsc --noEmit`
2. **Test:** `npm test` (if tests exist)
3. **Build:** `npm run build`
4. **No secrets:** Never commit API keys, tokens, or passwords

## CI Checks

Every PR automatically runs:
- TypeScript compilation check
- Unit tests (vitest)
- Production build verification

All checks must pass before merge.

## Code Standards

- TypeScript everywhere (except wybe-landing)
- Tailwind CSS for styling
- No `any` types unless absolutely necessary
- Error handling: use toast notifications in UI, proper try/catch in API
- Norwegian is fine in UI strings, English in code/comments

## Cross-Repo Changes

If your change affects multiple repos (e.g., new API in alive-core + consumer in wybe-hq):

1. Make alive-core PR first
2. Get it merged
3. Then make the consumer PR

This keeps things simple and reviewable.

## For AI Agents

- Always create a feature branch. Never push directly to main.
- Run `tsc --noEmit` before pushing. Fix all type errors.
- Include the sprint item ID in your PR description.
- If you're stuck, create a sprint item describing the blocker.
- Don't guess file paths — use search tools to find them.
- Don't delete files unless you're sure they're unused.
