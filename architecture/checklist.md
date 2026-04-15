# New Project Checklist

Use this checklist when starting a new project or auditing an existing one.

## Setup

- [ ] Created from `template-nextjs` or manually set up with equivalent config
- [ ] `bun.lock` is the only lockfile (no `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`)
- [ ] `.env.example` exists with all required variables documented
- [ ] `.gitignore` includes `.env*`, `.next/`, `node_modules/`, `.wrangler/`

## Dev Environment

- [ ] `flake.nix` and `shell.nix` present for Nix users
- [ ] `treefmt.nix` and `treefmt.toml` configured
- [ ] `lefthook.yml` configured with pre-commit hooks
- [ ] `lefthook install` runs (automatic in `nix develop`)

## Code Quality

- [ ] TypeScript `strict: true` in `tsconfig.json`
- [ ] ESLint configured (`eslint.config.mjs`) with `prettier` extension
- [ ] `.prettierrc` configured with `prettier-plugin-tailwindcss`
- [ ] treefmt configured (delegates to Prettier + nixfmt)
- [ ] lefthook pre-commit hooks run lint, format check, and type-check
- [ ] `next.config.ts` does NOT have `ignoreBuildErrors` or `ignoreDuringBuilds`
- [ ] All scripts present: `dev`, `build`, `start`, `lint`, `format`, `type-check`
- [ ] `bun run build` passes cleanly with no errors

## SEO & Social

- [ ] Custom favicon (not default Next.js/Vite icon)
- [ ] `<title>` is the actual project name
- [ ] `<meta name="description">` is meaningful
- [ ] OpenGraph metadata configured (title, description, image, url)
- [ ] Twitter card metadata configured
- [ ] `public/og-image.png` exists (1200x630px)
- [ ] Tested with [opengraph.xyz](https://opengraph.xyz)

## README

- [ ] Project name and description (not template boilerplate)
- [ ] Tech stack listed
- [ ] Getting Started instructions using `bun`
- [ ] Environment variables documented
- [ ] No leftover create-next-app content

## CI/CD

- [ ] `.github/workflows/ci.yml` exists
- [ ] CI runs: `bun install --frozen-lockfile` -> `lint` -> `type-check` -> `build`
- [ ] Deploy job uses `CLOUDFLARE_API_TOKEN` (org-level secret)
- [ ] `CLOUDFLARE_PROJECT_NAME` set as repo variable
- [ ] PR preview deployments working
- [ ] Deployment tested end-to-end

## Before Launch

- [ ] All env vars set in Cloudflare Pages dashboard
- [ ] Custom domain configured (if applicable)
- [ ] OG image preview verified on real social platforms
- [ ] Lighthouse score checked (aim for 90+ on Performance)
- [ ] No `console.log` statements left in production code
