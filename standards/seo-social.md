# SEO & Social Sharing

**Standard: Every public-facing project must have proper favicon, title, description, and OpenGraph metadata.**

## Why This Matters

When someone shares a link to our project on Slack, Twitter, LinkedIn, or iMessage, the preview card is generated from OpenGraph tags. Without them, the link shows as a plain URL with no image or description. This looks unprofessional.

## Required Metadata

### Favicon

- Place a proper favicon at `src/app/favicon.ico` (for Next.js App Router)
- Also provide `apple-touch-icon.png` (180x180) for iOS home screen
- Use a real icon, not the default Next.js or Vite favicon
- **Never leave the favicon href empty**

### Title & Description

In Next.js App Router, set in `layout.tsx`:

```tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: {
    default: "Project Name - Tagline",
    template: "%s | Project Name",
  },
  description: "A clear, compelling description of what this project does.",
};
```

### OpenGraph Tags

Required for proper social sharing previews:

```tsx
export const metadata: Metadata = {
  title: {
    default: "Project Name - Tagline",
    template: "%s | Project Name",
  },
  description: "A clear, compelling description.",
  metadataBase: new URL("https://project.com"),
  openGraph: {
    title: "Project Name - Tagline",
    description: "A clear, compelling description.",
    url: "https://project.com",
    siteName: "Project Name",
    images: [
      {
        url: "/og-image.png",
        width: 1200,
        height: 630,
        alt: "Project Name",
      },
    ],
    locale: "en_US",
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
    title: "Project Name - Tagline",
    description: "A clear, compelling description.",
    images: ["/og-image.png"],
  },
};
```

### OG Image

- Create a `public/og-image.png` at **1200x630px**
- Should include the project logo/name and a brief tagline
- Use a solid background - avoid transparency
- Test with [opengraph.xyz](https://opengraph.xyz) before shipping

## Checklist

- [ ] Custom favicon (not default)
- [ ] `<title>` set to project name, not "Create Next App"
- [ ] `<meta name="description">` is meaningful
- [ ] OpenGraph title, description, image, url configured
- [ ] Twitter card configured
- [ ] OG image exists at correct path (1200x630)
- [ ] Tested sharing preview on opengraph.xyz
