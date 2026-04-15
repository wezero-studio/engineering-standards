# Package Manager: Bun

**Standard: All Wezero Studio projects must use [bun](https://bun.sh) as the package manager.**

## Why Bun

- Significantly faster installs and script execution than npm/yarn/pnpm
- Drop-in replacement for npm - same `package.json`, same registry
- Built-in TypeScript execution (no ts-node needed)
- Consistent lockfile format across all repos

## Rules

1. **Only `bun.lock` should exist** in the repo. Delete `package-lock.json`, `yarn.lock`, and `pnpm-lock.yaml` if present.
2. **CI/CD must use `bun install --frozen-lockfile`** - never `npm ci` or `yarn install`.
3. **README instructions must reference bun** - not npm/yarn.
4. **Scripts should use `bun run`** - e.g., `bun run dev`, `bun run build`.

## Migration

For existing repos switching from npm/yarn/pnpm:

```bash
# Remove old lockfiles
rm -f package-lock.json yarn.lock pnpm-lock.yaml

# Install with bun to generate bun.lock
bun install

# Verify everything works
bun run build
```

Update CI workflows to use `oven-sh/setup-bun@v2` instead of `actions/setup-node`.

## Common Mistakes

- Having multiple lockfiles (e.g., both `bun.lock` and `package-lock.json`) - causes confusion about which manager is canonical
- CI using `npm ci` while the repo has `bun.lock` - installs may differ
- README saying `npm install` when the project uses bun
