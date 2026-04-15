# Environment Variables

**Standard: Every repo must have a `.env.example` file documenting all required environment variables.**

## Why

Without `.env.example`:

- New developers don't know what variables are needed
- Missing variables cause cryptic runtime errors
- There's no documentation of what each variable does or where to get it

## Rules

1. **`.env.example` must exist** at the repo root with every required variable
2. **Values should be empty or contain format hints** - never real secrets
3. **Add comments** explaining non-obvious variables
4. **Keep it in sync** - when you add a new env var to the code, add it to `.env.example`

## Format

```bash
# Database
DATABASE_URL=libsql://your-db.turso.io
DATABASE_AUTH_TOKEN=

# Authentication
AUTH_SECRET=           # Generate with: openssl rand -base64 32
BETTER_AUTH_URL=http://localhost:3000

# Third-party
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Public (exposed to browser)
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Next.js Specifics

- Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser - never put secrets here
- Server-only variables are only available in Server Components, Route Handlers, and `next.config.ts`
- Use `@t3-oss/env-nextjs` or manual runtime validation to catch missing variables at startup instead of at request time

## .gitignore

Ensure these are in `.gitignore` (they should be by default):

```
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
```

Never commit `.env` files with real values.
