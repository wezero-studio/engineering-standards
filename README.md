# Wezero Studio - Standard Operating Procedures

This repository contains the engineering standards, best practices, and SOPs for all Wezero Studio projects.

Every new project should follow these guidelines. Existing projects should be migrated incrementally.

## Table of Contents

### CI/CD
- [CI/CD & Deployment](./ci-cd/) - GitHub Actions + Cloudflare Pages

### Code Quality
- [Code Quality](./code-quality/) - ESLint, treefmt/Prettier, lefthook, TypeScript strict
- [Package Manager](./code-quality/package-manager.md) - Use bun exclusively

### Standards
- [README Standards](./standards/readme-standards.md) - Every repo needs a real README
- [SEO & Social Sharing](./standards/seo-social.md) - Favicon, title, OpenGraph

### Architecture
- [Common Anti-Patterns](./architecture/anti-patterns.md) - What to avoid
- [New Project Checklist](./architecture/checklist.md) - Quick-start checklist

### Environment
- [Environment Variables](./environment/) - `.env.example` required

## Quick Start

Starting a new project? Clone the template:

```bash
git clone https://github.com/wezero-studio/template-nextjs my-project
cd my-project

# With Nix (recommended)
nix develop
bun install

# Without Nix
bun install
lefthook install
```

Then go through the [New Project Checklist](./architecture/checklist.md).
