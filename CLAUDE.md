# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `bun run dev` — start Next.js dev server (hot reload)
- `bun run build` — production build
- `bun run test` — run all tests with coverage (Vitest + jsdom)
- `bun run test:junit` — run tests with JUnit XML output (for CI)
- `bun run check` — Biome lint + format check on `./app`
- `bun run check:fix` — auto-fix Biome issues in `./app`
- Single test: `bunx vitest run __tests__/home.test.tsx`

## Architecture

Next.js 15 App Router application. This is the **Illustrious Online** cloud platform — the hosted SaaS layer (multi-tenant, Steam integration, billing). Distinct from `illustrious.dashboard` (the open-source CE).

```
app/                  # Next.js App Router — pages, layouts, route handlers
  components/         # Chakra UI v3 component wrappers (button, checkbox, field, etc.)
  profile/            # User profile pages
  global-error.tsx    # Next.js global error boundary
__tests__             # Integration and page-level tests (co-located by feature name)
  home.test.tsx
  layout.test.tsx
  steam-callback.test.tsx
  steam-link.test.tsx
k8s/                  # Kubernetes manifests
public/               # Static assets
setupTests.ts         # Vitest global setup
vitest.config.mts     # Vitest config — jsdom environment, no globals (use explicit imports)
```

### Components

`app/components/` contains thin Chakra UI v3 wrappers — `button.tsx`, `checkbox.tsx`, `close-button.tsx`, `field.tsx`, `input-group.tsx`, `provider.tsx`, `wrapper.tsx`, and `color-mode.tsx`. These wrap Chakra primitives to expose a consistent API across the platform. Each has a corresponding `.test.tsx`.

### Auth & Steam

- Supabase handles auth (no better-auth here — unlike the dashboard)
- Steam OAuth integration: Steam link + callback routes in `app/`
- Never use `console.log` — Sentry (`@sentry/nextjs`) handles error capture

### Testing

- **Vitest** with `jsdom` environment — not `bun test`
- `@testing-library/react` for component and page tests
- Tests live in `__tests__/` at the repo root (not co-located)
- Note: `vitest.config.mts` does NOT set `globals: true` — import `describe`, `it`, `expect` explicitly
- Coverage excludes `.next/`, Sentry configs, `__tests__/`, and `global-error.tsx`

## Conventions

- Biome for lint + format — only runs on `./app` (not `__tests__/`)
- Husky pre-commit hook via `bun run prepare`
- Releases managed by semantic-release on `main` and `next` branches
- Never commit secrets or `.env` files — use Sealed Secrets for K8s deployments
