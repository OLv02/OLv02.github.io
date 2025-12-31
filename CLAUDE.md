# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Astro-based blog site with Tailwind CSS v4, configured for deployment to GitHub Pages. The site uses content collections for blog posts and supports both Markdown and MDX. The codebase is in transition, containing both the original Bear Blog-inspired styling (Header/Footer components) and a newer Tailwind-based Layout.astro system.

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
- `src/pages/index.astro` → `/` (homepage with welcome message)
- `src/pages/about.astro` → `/about` (uses BlogPost layout)
- `src/pages/blog/index.astro` → `/blog` (blog listing page, sorted by pubDate desc)
- `src/pages/blog/[...slug].astro` → `/blog/:slug` (dynamic blog post pages)
- `src/pages/rss.xml.js` → `/rss.xml` (RSS feed endpoint)

### Dynamic Route Generation

Blog posts use static path generation (`getStaticPaths`):
- `src/pages/blog/[...slug].astro` calls `getCollection('blog')` at build time
- Each post's `id` becomes the `slug` parameter
- Post data passed as props to BlogPost layout
- Content rendered via `<Content />` component from `render(post)`

### Layouts

**Two layout systems currently in use:**

1. **Layout.astro** (src/layouts/Layout.astro)
   - Modern Tailwind-based layout
   - Navigation: Home, About, Resume links
   - Responsive mobile menu using `<details>` element
   - Footer with social links (GitHub, LinkedIn, Email)
   - Skip-to-content link for accessibility
   - Logo uses "OL" initials in circular badge
   - Currently used by: index page

2. **BlogPost.astro** (src/layouts/BlogPost.astro)
   - Original Bear Blog-inspired layout
   - Uses Header, Footer, BaseHead components
   - Renders hero image, title, dates, and content
   - Scoped styles for blog post typography
   - Currently used by: blog posts and about page

### Components

- **BaseHead.astro** - SEO metadata component
  - Handles Open Graph and Twitter cards
  - Auto-generates canonical URLs
  - Includes sitemap and RSS feed links
  - Preloads Atkinson fonts
  - Fallback image: `src/assets/blog-placeholder-1.jpg`
  - Imports `src/styles/global.css`

- **Header.astro** - Original blog header
  - Site title from `SITE_TITLE` constant
  - Navigation: Home, Blog, About
  - Social links: Mastodon, Twitter, GitHub
  - Active link styling via HeaderLink component
  - Hides social links on mobile (<720px)

- **Footer.astro** - Original blog footer
  - Copyright with dynamic year
  - Same social links as Header
  - Gradient background

- **HeaderLink.astro** - Navigation link with active state
  - Auto-detects active page via `Astro.url.pathname`
  - Handles BASE_URL for subpath deployments
  - Adds underline + bold to active links

- **FormattedDate.astro** - Date display component
  - Formats dates as "MMM DD, YYYY" (e.g., "Aug 08, 2021")
  - Uses semantic `<time>` element with ISO datetime

### Site Configuration

- **src/consts.ts** - Global site constants
  - `SITE_TITLE = 'Astro Blog'`
  - `SITE_DESCRIPTION = 'Welcome to my website!'`
  - Used across pages for consistency

- **astro.config.mjs** - Astro configuration
  - `site: 'https://example.com'` - Update for production URLs
  - Integrations: `@astrojs/mdx`, `@astrojs/sitemap`
  - Tailwind via Vite plugin (`@tailwindcss/vite`)

### Styling

**Hybrid approach combining Tailwind and custom CSS:**

- **Tailwind CSS v4** via `@tailwindcss/vite` plugin
  - No separate config file needed
  - Utility classes used in Layout.astro

- **Custom CSS** in `src/styles/global.css`
  - Based on Bear Blog's default styles
  - Tailwind layers (`@layer base`)
  - CSS custom properties for theming:
    - `--accent: #2337ff`
    - `--black`, `--gray`, `--gray-light`, `--gray-dark` (RGB values)
    - `--box-shadow` and `--gray-gradient`
  - Typography scale (h1: 3.052em down to h5: 1.25em)
  - Responsive breakpoint: 720px
  - Screen reader utility: `.sr-only`

- **Custom Font** - Atkinson Hyperlegible
  - Font files: `public/fonts/atkinson-{regular,bold}.woff`
  - Preloaded in BaseHead for performance
  - Weights: 400 (regular), 700 (bold)

### Images and Assets

- **Image handling** via `astro:assets`
  - Use `<Image>` component for optimized images
  - Sharp integration for image processing
  - Blog placeholder images: `src/assets/blog-placeholder-{1-5}.jpg` + about variant

- **Public assets** in `public/`
  - Favicon: `public/favicon.svg`
  - Fonts: `public/fonts/`
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
2. Filename becomes the post slug (e.g., `my-post.md` → `/blog/my-post/`)
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
- Update social links in Header.astro and Footer.astro (or Layout.astro)
- Update logo initials in Layout.astro (currently "OL")
- Update `site` URL in `astro.config.mjs` for production
- Update email/social URLs in Layout.astro footer

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

## Known Architectural Notes

- **Mixed layout approach**: The site currently uses two different layout paradigms. BlogPost.astro (with Header/Footer components) for blog content, and Layout.astro for other pages. Consider consolidating for consistency.
- **Social links duplicated**: Header, Footer, and Layout all contain social link markup. Extract to a shared component if modifying links.
- **Placeholder content**: Header/Footer link to Astro's official social accounts; Layout.astro has placeholder URLs (github.com/, linkedin.com/, you@example.com) that should be updated.
- **Resume page referenced** but not implemented: Layout.astro nav includes `/resume` link with no corresponding page.
