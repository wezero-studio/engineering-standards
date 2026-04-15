# CI/CD & Deployment

**Standard: All projects deploy via GitHub Actions to Cloudflare Workers using `@opennextjs/cloudflare` and org-level secrets.**

## Why Cloudflare (Not Vercel)

Vercel's Git integration requires the commit author's email to match a Vercel team member. This caused:

- Deployments failing when a contributor isn't on the Vercel account
- Projects scattered across personal Vercel accounts
- No consistent deployment pipeline
- Poor DX when multiple people contribute

Cloudflare has none of these issues - deploy with an API token, no account matching required.

## How It Works

Next.js apps are built with `@opennextjs/cloudflare`, which adapts the Next.js output (including SSR, API routes, middleware) to run on Cloudflare Workers. The build produces a `.open-next/` directory containing the worker and static assets.

### Required Files

| File | Purpose |
|------|---------|
| `wrangler.jsonc` | Cloudflare Workers config (name, compatibility, assets) |
| `open-next.config.ts` | OpenNext adapter config |
| `.dev.vars` | Local dev environment variables |
| `public/_headers` | Static asset cache headers |

### Required Packages

```bash
bun add -D @opennextjs/cloudflare wrangler
```

### Required Scripts

```json
{
  "build": "next build",
  "deploy:build": "opennextjs-cloudflare build",
  "preview": "opennextjs-cloudflare build && opennextjs-cloudflare preview",
  "deploy": "opennextjs-cloudflare build && opennextjs-cloudflare deploy"
}
```

### next.config.ts

Must include `output: "standalone"` (required by opennext):

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "standalone",
};

export default nextConfig;

if (process.env.NODE_ENV === "development") {
  import("@opennextjs/cloudflare").then(({ initOpenNextCloudflareForDev }) =>
    initOpenNextCloudflareForDev(),
  );
}
```

## Required Secrets (set at org level)

| Secret/Variable | Type | Description |
|----------------|------|-------------|
| `CLOUDFLARE_API_TOKEN` | Secret | API token with Workers edit permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Secret | Wezero Studio Cloudflare account ID |

### Creating the API Token

1. Go to [Cloudflare Dashboard > API Tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Create a token with **Workers Scripts: Edit** permission
3. Add it as an org-level secret: `CLOUDFLARE_API_TOKEN`

## Standard Workflow

Every repo should have `.github/workflows/ci.yml`:

```yaml
name: CI & Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  ci:
    name: Lint, Type-check & Build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - run: bun install --frozen-lockfile
      - run: bun run lint
      - run: bun run type-check
      - run: bun run build

  deploy:
    name: Deploy to Cloudflare
    needs: ci
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - run: bun install --frozen-lockfile
      - run: bun run deploy:build

      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: deploy --commit-dirty=true

  deploy-preview:
    name: Deploy Preview
    needs: ci
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      deployments: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4

      - uses: oven-sh/setup-bun@v2
        with:
          bun-version: latest

      - run: bun install --frozen-lockfile
      - run: bun run deploy:build

      - uses: cloudflare/wrangler-action@v3
        id: deploy
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: versions upload --commit-dirty=true

      - name: Comment preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `Preview version uploaded. Use \`wrangler versions deploy\` to promote.`
            })
```

### Key Points

- **PR builds run CI + upload a preview version**
- **Pushes to `main` run CI then deploy to production** - only after CI passes
- `deploy:build` runs `opennextjs-cloudflare build` which calls `next build` internally then adapts the output
- `wrangler deploy` reads `wrangler.jsonc` for the worker config and deploys the `.open-next/` output
- The worker name in `wrangler.jsonc` determines the `*.workers.dev` subdomain

### Setting Up a New Project

1. Clone the `template-nextjs` repo (already has all config)
2. Update `name` and `services[0].service` in `wrangler.jsonc` to your project name
3. Ensure org-level secrets (`CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`) are accessible to the repo
4. Push to `main` - the worker is created and deployed automatically

### Custom Domains

After the first deploy:

1. Go to Cloudflare Dashboard > Workers & Pages > your worker > Settings > Domains & Routes
2. Add your domain (must be on Cloudflare DNS)
3. SSL is automatic

### Migrating from Vercel

1. Install `@opennextjs/cloudflare` and `wrangler`
2. Add `wrangler.jsonc`, `open-next.config.ts`, `.dev.vars`, `public/_headers`
3. Set `output: "standalone"` in `next.config.ts`
4. Add `initOpenNextCloudflareForDev()` for local dev
5. Update package.json scripts (`deploy:build`, `preview`, `deploy`)
6. Add the GitHub Actions workflow
7. Push to `main`
8. Update DNS to point to Cloudflare
9. Remove the Vercel project, `.vercel/`, and `vercel.json`
10. Update `.gitignore` (add `.open-next/`, `.wrangler/`, remove `.vercel`)
