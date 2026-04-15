# Common Anti-Patterns

Avoid these in all new and existing projects.

## 1. Suppressing Build Errors

```ts
// next.config.ts - DO NOT DO THIS
typescript: { ignoreBuildErrors: true }
eslint: { ignoreDuringBuilds: true }
```

**Why it's bad**: Broken TypeScript and lint errors ship silently to production. The build "passes" but the code is wrong.

**Fix**: Remove these flags and fix the actual errors.

## 2. Multiple Lockfiles

Having both `bun.lock` and `package-lock.json` (or any combination).

**Why it's bad**: Unclear which package manager is canonical. CI may install different versions than local dev. Dependencies can drift.

**Fix**: Pick bun. Delete all other lockfiles.

## 3. Template READMEs

The default create-next-app README with "Getting Started" pointing to Next.js docs.

**Why it's bad**: No one knows what the project is, how to run it, or what env vars are needed.

**Fix**: Follow the [README Standards](../standards/readme-standards.md).

## 4. Missing OpenGraph Metadata

No `openGraph` or `twitter` properties in the metadata export.

**Why it's bad**: Links shared on Slack, Twitter, LinkedIn show as plain URLs with no preview image or description.

**Fix**: Follow the [SEO & Social Sharing guide](../standards/seo-social.md).

## 5. No `.env.example`

No documentation of required environment variables.

**Why it's bad**: New developers have to reverse-engineer which env vars are needed by reading the code.

**Fix**: Follow the [Environment Variables guide](../environment/).

## 6. Mismatched CI Package Manager

CI workflow using `npm ci` while the repo uses pnpm or bun.

**Why it's bad**: Different dependency resolution. What works in CI may break locally and vice versa.

**Fix**: CI must use the same package manager as the repo (bun).

## 7. Excessive Memory Allocation

```json
"dev": "NODE_OPTIONS='--max-old-space-size=8192' next dev"
```

**Why it's bad**: Masks memory leaks and bloated bundles. If you need 8GB to build, something is wrong.

**Fix**: Investigate what's consuming memory. Usually it's unoptimized imports or circular dependencies.

## 8. Disabled ESLint Rules for `any`

```js
"@typescript-eslint/no-explicit-any": "off"
```

**Why it's bad**: `any` disables TypeScript's type system. Widespread use of `any` means bugs that types would catch go undetected.

**Fix**: Use `unknown` and narrow types. Only disable for specific lines with a comment explaining why.

## 9. Broken or Missing Favicons

Empty favicon href, default Next.js favicon, or missing favicon entirely.

**Why it's bad**: Looks unprofessional. Browser tabs show a blank or generic icon.

**Fix**: Add a proper project favicon. Never leave the favicon href empty. See [SEO & Social Sharing](../standards/seo-social.md).

## 10. No CI/CD Pipeline

No GitHub Actions workflow, relying on platform Git integration or manual deploys.

**Why it's bad**: No linting or type-checking before deploy. Platform-specific Git integrations often require commit author matching or other account constraints.

**Fix**: Add the standard [CI/CD workflow](../ci-cd/) deploying to Cloudflare Pages.

## 11. Hardcoded External URLs

API endpoints, storage URLs, or third-party service URLs hardcoded in source files.

**Why it's bad**: Can't change per environment. Hard to find and update. May leak internal infrastructure details.

**Fix**: Move to environment variables.

## 12. Large Unoptimized Images

Images over 500KB committed directly to the repo.

**Why it's bad**: Kills page load performance and Core Web Vitals. Wastes bandwidth.

**Fix**: Compress images before committing. Use Next.js `<Image>` component for automatic optimization. Consider hosting large assets on a CDN.
