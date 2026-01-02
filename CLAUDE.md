# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro-based blog site with Tailwind CSS v4, configured for deployment to GitHub Pages. The site uses content collections for blog posts (Markdown and MDX). The UI supports a modern theme (default) and a retro theme, switched via a client-side toggle that persists in localStorage.

## Development Commands

All commands run from the project root:

- `npm run dev` - Start dev server at `localhost:4321`
- `npm run build` - Build production site to `./dist/`
- `npm run preview` - Preview production build locally
- `npm run astro ...` - Run Astro CLI commands (e.g., `npm run astro check`)

## Architecture

### Content Collections

Blog posts are managed via Astro's content collections system:
- Collection definition: `src/content.config.ts`
- Blog posts location: `src/content/blog/` (supports `.md` and `.mdx` files)
- Schema validation (Zod) enforces required frontmatter:
  - `title` (string, required)
  - `description` (string, required)
  - `pubDate` (date, required - coerced from string)
  - `updatedDate` (date, optional)
  - `heroImage` (ImageMetadata, optional)
- Retrieve posts: `getCollection('blog')` returns array of posts
- Render content: `render(post)` returns `{ Content }` component

### File-Based Routing

Pages in `src/pages/` are automatically exposed as routes:
- `src/pages/index.astro` -> `/` (homepage)
- `src/pages/about.astro` -> `/about`
- `src/pages/blog/index.astro` -> `/blog` (blog listing page)
- `src/pages/blog/[...slug].astro` -> `/blog/:slug` (dynamic blog post pages)
- `src/pages/rss.xml.js` -> `/rss.xml` (RSS feed endpoint)

### Dynamic Route Generation

Blog posts use static path generation (`getStaticPaths`):
- `src/pages/blog/[...slug].astro` calls `getCollection('blog')` at build time
- Each post's `id` becomes the `slug` parameter
- Post data passed as props to BlogPost layout
- Content rendered via `<Content />` component from `render(post)`

### Layouts

- **BaseLayout.astro** (`src/layouts/BaseLayout.astro`)
  - Only place where the `<html>` tag is defined
  - Imports `src/styles/global.css`
  - Injects `ThemeScript` in `<head>` before other content
  - Applies `class:list={["site-shell", bodyClass]}` to `<body>`

- **Layout.astro** (`src/layouts/Layout.astro`)
  - Modern Tailwind-based layout
  - Uses BaseLayout + BaseHead + Header + Footer
  - Provides skip-to-content link and a centered content column
  - Currently used by: `src/pages/index.astro`

- **BlogPost.astro** (`src/layouts/BlogPost.astro`)
  - Uses BaseLayout + BaseHead + Header + Footer
  - Renders hero image, title, dates, and slot content
  - Currently used by: `src/pages/blog/[...slug].astro`, `src/pages/about.astro`

### Components

- **BaseHead.astro** (`src/components/BaseHead.astro`)
  - SEO metadata (Open Graph, Twitter cards, canonical URL)
  - Links sitemap + RSS feed
  - Preloads Atkinson fonts
  - Fallback image: `src/assets/blog-placeholder-1.jpg`

- **Header.astro** (`src/components/Header.astro`)
  - Site title from `SITE_TITLE`
  - Navigation: Home, Blog, About
  - Social links: Mastodon, Twitter, GitHub
  - Renders `ThemeToggle` inside the nav
  - Renders two nav UIs:
    - `.header-nav--modern` keeps the full modern nav row and ThemeToggle
    - `.header-nav--retro` uses `<details>/<summary>` with a collapsible "Site Map" list (Home, Blog, About, plus explicit Modern/Retro theme buttons)
  - Retro nav theme buttons use `data-theme-set="modern|retro"` attributes and are handled by the ThemeToggle script

- **ThemeToggle.astro** (`src/components/ThemeToggle.astro`)
  - Toggle button for modern/retro
  - Uses localStorage key `site-theme` and updates `<html data-theme>`
  - Inline styles for the toggle state
  - Also listens for `[data-theme-set]` clicks to explicitly set the theme (used by the retro Site Map list)

- **ThemeScript.astro** (`src/components/ThemeScript.astro`)
  - Inline script that runs before CSS to prevent FOUC
  - Reads localStorage key `site-theme` and sets `<html data-theme>`

- **Footer.astro** (`src/components/Footer.astro`)
  - Dynamic year + social links

- **HeaderLink.astro** (`src/components/HeaderLink.astro`)
  - Active-state nav links (based on `Astro.url.pathname`)
  - Handles `BASE_URL` for subpath deployments

- **FormattedDate.astro** (`src/components/FormattedDate.astro`)
  - Formats dates as "MMM DD, YYYY"
  - Uses semantic `<time>` element

### Site Configuration

- **src/consts.ts** - Global site constants
  - `SITE_TITLE = 'Astro Blog'`
  - `SITE_DESCRIPTION = 'Welcome to my website!'`
  - Used across pages for consistency

- **astro.config.mjs** - Astro configuration
  - `site: 'https://example.com'` (placeholder)
  - Integrations: `@astrojs/mdx`, `@astrojs/sitemap`
  - Tailwind via Vite plugin (`@tailwindcss/vite`)

### Theme System (Modern/Retro)

- **State + persistence**
  - Default theme attribute: `<html data-theme="modern">` in `src/layouts/BaseLayout.astro`
  - LocalStorage key: `site-theme`
  - Stored values: `modern` or `retro`
  - ThemeScript sets the attribute before CSS loads

- **Toggle UI**
  - Rendered in `src/components/Header.astro` via `ThemeToggle`
  - Toggle updates `document.documentElement.dataset.theme` and localStorage
  - Toggle updates `aria-label` for screen readers

- **CSS scoping**
  - `src/styles/global.css` imports Tailwind and then `src/styles/theme-retro.css`
  - Retro styles are scoped to `html[data-theme="retro"]`
  - Default (modern) styling relies on Tailwind utilities plus the base CSS

### Retro Mode UI Specifics

- **Header adjustments** (`src/styles/theme-retro.css`)
  - `.site-header` becomes a centered, columnar layout with gradient background
  - `h2::after` injects "(please wait 60 seconds for page to load)"
  - `nav::before` draws a striped progress line
  - `nav::after` places `/assets/retro/runner.svg` at `top: 95px`
  - `.social-links` are hidden in retro mode
  - `.header-nav--modern` is hidden and `.header-nav--retro` is shown only when `html[data-theme="retro"]`
  - Retro Site Map styles live here (summary brackets, list separators, focus outlines)

- **Assets + sizing**
  - Background tile: `/assets/retro/bg-tile.svg` (applied to `html[data-theme="retro"]`)
  - Runner icon: `/assets/retro/runner.svg` (used in header `nav::after`)
  - `public/assets/retro/globe.svg` exists but is not referenced in CSS
  - Retro logo styles exist for `.retro-logo-container`/`.retro-logo` (240x240, `object-fit: cover`), but no markup uses those classes yet
  - `public/images/site-logo.gif` is present but not referenced in markup (TODO if intended)

### Styling

- **Tailwind CSS v4** via `@tailwindcss/vite` plugin
  - Utility classes used in `src/layouts/Layout.astro` and `src/pages/index.astro`

- **Custom CSS** in `src/styles/global.css`
  - Tailwind layers (`@layer base`)
  - CSS custom properties for theming (`--accent`, `--black`, `--gray`, `--box-shadow`, etc.)
  - Screen reader utility: `.sr-only`
  - Hides `.retro-logo-container` by default (modern theme)

- **Custom Font** - Atkinson Hyperlegible
  - Font files: `public/fonts/atkinson-{regular,bold}.woff`
  - Preloaded in `src/components/BaseHead.astro`
  - Weights: 400 (regular), 700 (bold)

### Images and Assets

- **Image handling** via `astro:assets`
  - Use `<Image>` component for optimized images
  - Blog placeholder images: `src/assets/blog-placeholder-{1-5}.jpg` + about variant

- **Public assets** in `public/`
  - Favicon: `public/favicon.svg`
  - Fonts: `public/fonts/`
  - Retro assets: `public/assets/retro/`
  - Served at root path (e.g., `/favicon.svg`)

### RSS Feed

- Generated at `src/pages/rss.xml.js`
- Uses `@astrojs/rss` package
- Auto-includes all blog posts with link: `/blog/{post.id}/`
- Pulls title/description from `SITE_TITLE`/`SITE_DESCRIPTION`

### SEO Features

- Sitemap auto-generated by `@astrojs/sitemap` integration
- Canonical URLs on all pages via BaseHead
- Open Graph and Twitter card meta tags
- RSS feed linked in page head
- Semantic HTML (main, nav, article, time elements)

## Content Guidelines

### Adding Blog Posts

1. Create `.md` or `.mdx` file in `src/content/blog/`
2. Filename becomes the post slug (e.g., `my-post.md` -> `/blog/my-post/`)
3. Required frontmatter:
   ```yaml
   ---
   title: 'Post Title'
   description: 'Post description for SEO'
   pubDate: 2024-01-15
   # Optional:
   updatedDate: 2024-01-20
   heroImage: ../../assets/my-image.jpg
   ---
   ```
4. Images can be imported from `src/assets/` for optimization

### Customizing Site Identity

- Update site name/description in `src/consts.ts`
- Update social links in `src/components/Header.astro` and `src/components/Footer.astro`
- Update `site` URL in `astro.config.mjs` for production URLs

### Homepage Structure (Semantic + Tailwind)

- `src/pages/index.astro` uses separate `<p>` and `<ul>` blocks (no lists nested inside paragraphs)
- Tailwind utilities used for spacing and typography, e.g. `max-w-2xl`, `text-center`, `leading-relaxed`, `list-disc`, `pl-5`

## Deployment

Automatic deployment to GitHub Pages via `.github/workflows/deploy.yml`:
- Triggers on push to `main` branch or manual dispatch
- Uses `withastro/action@v2` for building
- Deploys via `actions/deploy-pages@v4`
- Requires GitHub Pages enabled in repo settings
- Sets `contents: read`, `pages: write`, `id-token: write` permissions

## TypeScript Configuration

- Extends `astro/tsconfigs/strict`
- Additional option: `strictNullChecks: true`
- Includes `.astro/types.d.ts` for auto-generated content types
- Excludes `dist/` directory

## Troubleshooting

- **No GitHub Pages deploy after push**: workflow only runs on `main` or via `workflow_dispatch` in `.github/workflows/deploy.yml`.
- **Incorrect canonical/RSS/sitemap URLs**: `astro.config.mjs` still sets `site: 'https://example.com'` and should be updated for the actual Pages URL.
