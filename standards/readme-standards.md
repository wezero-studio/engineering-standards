# README Standards

**Standard: Every repo must have a meaningful README - not the default template.**

## The Problem

Most of our repos have the default `create-next-app` README or a one-line stub. This makes it hard for anyone (including future you) to understand what the project is, how to run it, or how it's deployed.

## Required Sections

Every README must include:

### 1. Project Title & Description

What is this project? Who is it for? One paragraph max.

```markdown
# Project Name

Brief description of what this project does and who it's for.
Live: [https://project.com](https://project.com)
```

### 2. Tech Stack

List the major technologies. Keep it brief.

```markdown
## Tech Stack

- **Framework**: Next.js 15 (static export, no SSR)
- **Styling**: Tailwind CSS v4
- **Database**: Drizzle ORM + Turso
- **Auth**: Better Auth
- **Hosting**: Cloudflare Pages (static files)
- **CI/CD**: GitHub Actions
```

### 3. Getting Started

How to clone, install, and run locally. Must use bun.

```markdown
## Getting Started

```bash
git clone https://github.com/wezero-studio/project-name.git
cd project-name
cp .env.example .env.local
bun install
bun run dev
```

Open [http://localhost:3000](http://localhost:3000).
```

### 4. Environment Variables

Reference the `.env.example` file and explain any non-obvious variables.

```markdown
## Environment Variables

Copy `.env.example` to `.env.local` and fill in the values:

| Variable | Description | Where to get it |
|----------|-------------|-----------------|
| `DATABASE_URL` | Turso database URL | Turso dashboard |
| `AUTH_SECRET` | Random secret for auth | `openssl rand -base64 32` |
```

### 5. Project Structure (optional but encouraged)

Brief overview of the directory layout if non-standard.

## What NOT to Include

- Default create-next-app boilerplate
- Generic "Learn More" links to Next.js docs
- Deployment instructions that just say "deploy to Vercel"
- Empty sections or TODOs
