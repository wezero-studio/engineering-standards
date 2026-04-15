# CI/CD & Deployment

**Standard: All projects deploy via GitHub Actions to Cloudflare Pages using org-level secrets.**

## Why Cloudflare Pages (Not Vercel)

Vercel's Git integration requires the commit author's email to match a Vercel team member. This caused:

- Deployments failing when a contributor isn't on the Vercel account
- Projects scattered across personal Vercel accounts
- No consistent deployment pipeline
- Poor DX when multiple people contribute

Cloudflare Pages has none of these issues - deploy with an API token, no account matching required, and preview deployments are free and automatic.

## Required Secrets (set at org level)

| Secret/Variable | Type | Description |
|----------------|------|-------------|
| `CLOUDFLARE_API_TOKEN` | Secret | API token with Cloudflare Pages edit permissions |
| `CLOUDFLARE_ACCOUNT_ID` | Secret | Wezero Studio Cloudflare account ID |
| `CLOUDFLARE_PROJECT_NAME` | Variable | Per-repo Cloudflare Pages project name |

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
          command: pages deploy .next --project-name=${{ vars.CLOUDFLARE_PROJECT_NAME }} --commit-dirty=true

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
          command: pages deploy .next --project-name=${{ vars.CLOUDFLARE_PROJECT_NAME }} --commit-dirty=true

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
- The build happens in CI (not on Cloudflare's build system), giving us full control over the build environment
- Cloudflare's Git integration can optionally be enabled alongside this, but the GitHub Actions workflow is the canonical deploy path

### Setting Up a New Project

The Cloudflare Pages project is created automatically on first deploy - `wrangler pages deploy` with `--project-name` creates it if it doesn't exist. No manual setup needed on the Cloudflare side.

1. Add `CLOUDFLARE_PROJECT_NAME` as a repo-level variable on GitHub (Settings > Variables)
2. Ensure org-level secrets (`CLOUDFLARE_API_TOKEN`, `CLOUDFLARE_ACCOUNT_ID`) are accessible
3. Add the workflow file
4. Push to `main` - the project is created and deployed in one step

### Custom Domains

After the first deploy:

1. Go to Cloudflare Dashboard > Pages > your project > Custom domains
2. Add your domain (must be on Cloudflare DNS or you add a CNAME)
3. SSL is automatic

### Migrating from Vercel

1. Add the GitHub Actions workflow
2. Set up secrets/variables on GitHub
3. Push to `main` (Cloudflare Pages project is auto-created)
4. Update DNS to point to Cloudflare Pages
5. Remove the Vercel project and any `.vercel/` directories
6. Remove `vercel.json` if present
7. Update `.gitignore` (replace `.vercel` with `.wrangler/`)
