# CI/CD & Deployment

**Standard: All projects deploy via GitHub Actions to Cloudflare Pages using org-level secrets.**

## Why Cloudflare Pages (Not Vercel)

Vercel's Git integration requires the commit author's email to match a Vercel team member. This caused:

- Deployments failing when a contributor isn't on the Vercel account
- Projects scattered across personal Vercel accounts
- No consistent deployment pipeline
- Poor DX when multiple people contribute

Cloudflare Pages has none of these issues - deploy with an API token, no account matching required, and preview deployments are free and automatic.

## How It Works

Next.js builds with `output: "export"` which produces a static `out/` directory. This gets deployed directly to Cloudflare Pages via `wrangler pages deploy out`.

### next.config.ts

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  output: "export",
};

export default nextConfig;
```

### Static Asset Caching

Add `public/_headers` for immutable cache on hashed assets:

```
/_next/static/*
  Cache-Control: public,max-age=31536000,immutable
```

## Required Secrets (set at org level)

| Secret/Variable | Type | Description |
|----------------|------|-------------|
| `CLOUDFLARE_API_TOKEN` | Secret | API token with Cloudflare Pages edit permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Secret | Wezero Studio Cloudflare account ID |
| `CLOUDFLARE_PROJECT_NAME` | Variable (per-repo) | Cloudflare Pages project name |

### Creating the API Token

1. Go to [Cloudflare Dashboard > API Tokens](https://dash.cloudflare.com/profile/api-tokens)
2. Create a token with **Cloudflare Pages: Edit** permission
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
    name: Deploy to Cloudflare Pages
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
      - run: bun run build

      - name: Create Pages project if needed
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages project create ${{ vars.CLOUDFLARE_PROJECT_NAME }} --production-branch=main
        continue-on-error: true

      - uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy out --project-name=${{ vars.CLOUDFLARE_PROJECT_NAME }} --commit-dirty=true

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
      - run: bun run build

      - name: Create Pages project if needed
        uses: cloudflare/wrangler-action@v3
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages project create ${{ vars.CLOUDFLARE_PROJECT_NAME }} --production-branch=main
        continue-on-error: true

      - uses: cloudflare/wrangler-action@v3
        id: deploy
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: ${{ secrets.CLOUDFLARE_ACCOUNT_ID }}
          command: pages deploy out --project-name=${{ vars.CLOUDFLARE_PROJECT_NAME }} --commit-dirty=true

      - name: Comment preview URL
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `Preview deployed to: ${{ steps.deploy.outputs.deployment-url }}`
            })
```

### Key Points

- **PR builds run CI + deploy a preview** with the URL posted as a PR comment
- **Pushes to `main` run CI then deploy to production** - only after CI passes
- `next build` with `output: "export"` produces a static `out/` directory
- `wrangler pages deploy out` uploads that directory to Cloudflare Pages
- The Pages project is auto-created on first deploy (`continue-on-error: true` handles existing projects)
- No Workers, no adapters, no extra dependencies

### Setting Up a New Project

1. Clone the `template-nextjs` repo (already has all config)
2. Add `CLOUDFLARE_PROJECT_NAME` as a repo-level variable on GitHub (Settings > Variables)
3. Ensure org-level secrets (`CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`) are accessible to the repo
4. Push to `main` - the Pages project is created and deployed automatically

### Custom Domains

After the first deploy:

1. Go to Cloudflare Dashboard > Pages > your project > Custom domains
2. Add your domain (must be on Cloudflare DNS or you add a CNAME)
3. SSL is automatic

### Migrating from Vercel

1. Set `output: "export"` in `next.config.ts`
2. Add `public/_headers` for static cache
3. Add the GitHub Actions workflow
4. Set up secrets/variables on GitHub
5. Push to `main` (Pages project is auto-created)
6. Update DNS to point to Cloudflare Pages
7. Remove the Vercel project, `.vercel/`, and `vercel.json`
8. Update `.gitignore` (replace `.vercel` with `.wrangler/`)

### Need SSR?

If a project needs server-side rendering, API routes, or middleware, use `@opennextjs/cloudflare` to deploy to Cloudflare Workers instead. See the [OpenNext Cloudflare docs](https://opennext.js.org/cloudflare) for setup. The static Pages approach is preferred when possible - it's simpler, faster, and cheaper.
