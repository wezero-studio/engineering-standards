# Code Quality

**Standard: All projects must have ESLint, Prettier (via treefmt), TypeScript strict mode, and lefthook pre-commit hooks.**

## In This Section

- [Package Manager](./package-manager.md) - Use bun exclusively

---

## TypeScript

### Strict Mode Required

`tsconfig.json` must have `"strict": true`. No exceptions.

This catches null/undefined bugs, implicit `any` types, and other issues at compile time rather than in production.

### No Build Error Suppression

**Never** put this in `next.config.ts`:

```ts
// DO NOT DO THIS
typescript: {
  ignoreBuildErrors: true,
},
eslint: {
  ignoreDuringBuilds: true,
},
```

This hides real errors and lets broken code ship to production. If the build has errors, fix them. If you're in a rush, fix them anyway - shipping broken code creates more work later.

### Minimize `any` and `@ts-ignore`

- Avoid `any` - use `unknown` and narrow the type
- Never use `@ts-ignore` or `@ts-expect-error` without a comment explaining why
- If a third-party library has bad types, create a `types/` declaration file

## ESLint

Use the flat config format (`eslint.config.mjs`):

```js
import { dirname } from "path";
import { fileURLToPath } from "url";
import { FlatCompat } from "@eslint/eslintcompat";

const __dirname = dirname(fileURLToPath(import.meta.url));
const compat = new FlatCompat({ baseDirectory: __dirname });

export default [
  ...compat.extends("next/core-web-vitals", "next/typescript", "prettier"),
  {
    ignores: [".next/", "out/", "build/"],
  },
];
```

The `prettier` extension disables ESLint rules that conflict with Prettier, since formatting is handled by treefmt.

## Formatting: treefmt + Prettier

We use [treefmt](https://github.com/numtide/treefmt) as the unified formatting entry point. It delegates to Prettier for JS/TS/CSS/JSON/MD and nixfmt for Nix files.

### For Nix users (via `treefmt.nix`)

The `flake.nix` imports `treefmt.nix` which configures Prettier with our settings. Run:

```bash
nix fmt          # format everything
treefmt          # same thing inside nix develop
```

### For non-Nix users (via `treefmt.toml`)

A `treefmt.toml` is provided as a fallback. It expects `prettier` and `nixfmt` on PATH. The `.prettierrc` file contains the shared Prettier settings.

### Prettier Config (`.prettierrc`)

```json
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100,
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

The `prettier-plugin-tailwindcss` plugin auto-sorts Tailwind classes.

## Git Hooks: lefthook

We use [lefthook](https://github.com/evilmartians/lefthook) for pre-commit hooks. It runs lint, format check, and type-check in parallel before every commit.

### `lefthook.yml`

```yaml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,tsx,js,jsx,mjs}"
      run: bun run lint
    format:
      glob: "*.{ts,tsx,js,jsx,mjs,cjs,json,css,md,yaml,yml}"
      run: treefmt --fail-on-change
    type-check:
      run: bun run type-check
```

### Setup

If using Nix, `lefthook install` runs automatically on `nix develop`. Otherwise:

```bash
# Install lefthook (macOS)
brew install lefthook

# Install lefthook (any platform)
bun add -g lefthook

# Activate hooks
lefthook install
```

## Package.json Scripts

Every project must have these scripts:

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "treefmt",
    "format:check": "treefmt --fail-on-change",
    "type-check": "tsc --noEmit"
  }
}
```

CI runs `lint`, `type-check`, and `build`. All three must pass.

## Editor Config

Add `.editorconfig` to enforce basic formatting across editors:

```ini
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```
