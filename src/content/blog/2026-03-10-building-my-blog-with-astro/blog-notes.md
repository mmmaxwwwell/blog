# mmmaxwwwell's Blog — Notes

## Architecture decisions
- **Astro v5 + MDX** chosen for component-friendly content, strong content collections, and agent-friendly JS/TS/HTML/CSS stack.
- **Tailwind CSS v4** for styling — utility-first, easy dark/light mode via `dark:` variant, good for systematic retrowave theming.
- **GitHub Pages deployment** via GitHub Actions (not Jekyll). Repo is `mmmaxwwwell/blog`, so base path is `/blog/`.
- **Content collections** with typed frontmatter for posts — enforces consistent metadata across all posts.
- **Shiki** for syntax highlighting (built into Astro) — supports many languages, theme-able.
- **No comments system** — intentional. Blog is a publishing platform, not a discussion forum.
- **Two-agent model:** One agent (using `blog-prompt.md`) scaffolds the site. A separate agent (using `CLAUDE.md`) creates content post-scaffolding.

## Design: Retrowave theme

### Color palette
| Token | Dark mode | Light mode | Usage |
|-------|-----------|------------|-------|
| `primary-pink` | `#FF2D95` | `#C4206E` | Links, active states, primary accents |
| `primary-cyan` | `#00F0FF` | `#0098A6` | Secondary accents, hover states |
| `primary-purple` | `#B026FF` | `#7B2FBE` | Highlights, gradients |
| `bg-base` | `#0D0221` | `#F8F8FF` | Page background |
| `bg-surface` | `#1A1A2E` | `#EEEEF5` | Cards, code blocks, surfaces |
| `text-primary` | `#E8E8F0` | `#1A1A2E` | Body text |
| `text-muted` | `#8888AA` | `#6666880` | Dates, secondary text |
| `border` | `#2A2A4E` | `#D0D0E0` | Card borders, dividers |

### Dark/light mode implementation
- Use `class` strategy (not `media`) so user can override system preference
- On load: check `localStorage('theme')` → fall back to `prefers-color-scheme` → default dark
- Toggle button in header swaps class on `<html>` and saves to `localStorage`
- Tailwind `dark:` variant maps to `.dark` class on `<html>`

### Typography
- Body: system font stack (`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`) or Inter if loaded
- Code: `'JetBrains Mono', 'Fira Code', 'Cascadia Code', monospace`
- Prose max-width: ~720px

## Deployment

### GitHub Pages config
- **Site URL:** `https://mmmaxwwwell.github.io`
- **Base path:** `/blog`
- All internal links and asset paths must account for the `/blog` base
- Astro handles this via `base: '/blog'` in config — use Astro's URL helpers, don't hardcode paths

### GitHub Actions workflow
- Trigger: push to `main`
- Steps: checkout → setup Node → install deps → `astro build` → deploy to `gh-pages` branch (or use `actions/deploy-pages`)
- Use `actions/configure-pages` + `actions/upload-pages-artifact` + `actions/deploy-pages` (official GitHub Pages deployment)

## Content structure

### Frontmatter schema
```yaml
---
title: "Post Title"
description: "Brief description for cards and SEO"
date: 2026-03-10
updated: 2026-03-15  # optional
category: "software-dev"
tags: ["astro", "tailwind", "blog"]
draft: false
heroImage: "/images/post-hero.jpg"  # optional
---
```

### Category conventions
- `electrical` — hobbyist electronics projects
- `openscad` — 3D model design with OpenSCAD
- `3d-printing` — 3D printing projects, settings, materials
- `software-dev` — software development, tools, frameworks
- Categories can be expanded as needed — they're just strings in frontmatter

### Tag conventions
- Lowercase, hyphenated (e.g., `astro`, `tailwind-css`, `esp32`, `openscad`)
- Tags are more granular than categories — a post in `electrical` might be tagged `esp32`, `mqtt`, `home-automation`

## Key references
- [Astro docs](https://docs.astro.build/)
- [Astro content collections](https://docs.astro.build/en/guides/content-collections/)
- [Astro GitHub Pages deploy guide](https://docs.astro.build/en/guides/deploy/github/)
- [Tailwind CSS v4 docs](https://tailwindcss.com/docs)
- [MDX in Astro](https://docs.astro.build/en/guides/integrations-guide/mdx/)
- [@astrojs/rss](https://docs.astro.build/en/guides/rss/)
- [Shiki syntax highlighting](https://shiki.style/)

## Implementation notes

### Task 1.1 — Project init (2026-03-10)
- Astro v6 is now latest, but prompt specifies v5. Pinned `astro@5`, `@astrojs/mdx@4` (v5 requires MDX integration v4).
- `@astrojs/tailwind` integration doesn't support Astro v5+ with Tailwind v4. Used `@tailwindcss/vite` plugin directly in `astro.config.mjs` `vite.plugins` instead.
- Nix flake added for Node 22 dev environment. All npm commands run via `nix develop --command bash -c "..."`.
- Shiki dual theme configured: `github-light` (light) / `tokyo-night` (dark).
- Minimal `src/pages/index.astro` created as placeholder — will be replaced in task 3.1.
- `.gitignore` added for `dist/`, `node_modules/`, `.astro/`, `result`.

### Task 1.2 — Tailwind retrowave theme (2026-03-10)
- Tailwind v4 uses `@theme` directive in CSS instead of `tailwind.config.mjs` — no config file needed.
- Dark mode uses `:root` (default), light mode uses `:root.light` class strategy. Toggle will add/remove `.light` on `<html>`.
- Semantic colors (`--bg-base`, `--bg-surface`, `--text-primary`, `--text-muted`, `--border-color`, `--accent-*`) swap between dark/light via CSS custom properties.
- Tailwind theme colors reference CSS vars so utilities like `bg-bg-base`, `text-text-primary`, `border-border` work.
- Base styles for body, links, code blocks, and selection highlight applied in `global.css`.
- Body has `transition` on background-color and color for smooth theme switching.

### Task 1.3 — BaseLayout (2026-03-10)
- `BaseLayout.astro` at `src/layouts/BaseLayout.astro` — accepts `title` and `description` props.
- Page title format: `"Page Title | mmmaxwwwell's blog"` (or just site title if they match).
- Inline `<script is:inline>` for theme detection runs before paint — avoids flash of wrong theme.
- Theme logic: check `localStorage('theme')` for `'light'` → check `prefers-color-scheme: light` → default dark (no class = dark).
- Global CSS imported via `<style is:global>` block.
- Body has `min-h-screen flex flex-col` for sticky footer pattern.
- `index.astro` updated to use `BaseLayout`.
- Build warning about auto-generated content collections — will resolve in task 2.1 when we create `src/content/config.ts`.

### Task 1.4 — Header & ThemeToggle (2026-03-10)
- `Header.astro` at `src/components/Header.astro` — sticky header with `backdrop-blur-sm`, border-bottom.
- Desktop nav: inline links with active state detection (pink accent for current page). Mobile: hamburger button toggles a drawer.
- `ThemeToggle.astro` at `src/components/ThemeToggle.astro` — toggles `.light` class on `<html>`, saves to `localStorage`. Sun icon in dark mode, moon icon in light mode.
- Both components use `is:inline` scripts for immediate DOM access without Astro's module bundling.
- Active link detection uses `Astro.url.pathname` — exact match for Home, `startsWith` for Blog/About.
- ThemeToggle is rendered in both desktop and mobile sections of the header (two instances on mobile screens, but only one visible due to layout).

### Task 1.5 — Footer (2026-03-10)
- `Footer.astro` at `src/components/Footer.astro` — dynamic copyright year, GitHub icon link, RSS icon link.
- Uses `mt-auto` to push footer to bottom (works with `flex flex-col min-h-screen` on body in BaseLayout).
- RSS link points to `/blog/rss.xml` — will be created in task 4.3.
- Added Footer import and component to `BaseLayout.astro` after `<slot />`.

### Task 2.1 — Content collection schema (2026-03-10)
- Astro v5 uses `src/content.config.ts` (not `src/content/config.ts`) with `glob` loader from `astro/loaders`.
- Schema file location: `src/content.config.ts`. Blog posts directory: `src/content/blog/`.
- Glob loader config: `glob({ pattern: '**/*.mdx', base: './src/content/blog' })`.
- Added a draft placeholder post (`2026-03-10-hello-world.mdx`) so the collection isn't empty.
- Build now shows "Syncing content" / "Synced content" instead of the auto-generation warning from earlier tasks.

### Task 2.2 — PostLayout (2026-03-10)
- `PostLayout.astro` at `src/layouts/PostLayout.astro` — wraps `BaseLayout`, accepts typed props from content collection frontmatter.
- Displays: title (h1), date, optional updated date, category link (`/blog/category/{cat}/`), tag pills (`/blog/tags/{tag}/`), optional hero image.
- Prose styles scoped via `.prose` class with `:global()` selectors for MDX-rendered elements (headings, lists, blockquotes, tables, links, images, hr).
- Links in prose have subtle underline with pink accent, hover switches to cyan.
- Blockquotes use purple left border accent.
- Shiki dual theme: was already configured in `astro.config.mjs` (task 1.1) with `github-light` / `tokyo-night`. Added CSS override in `global.css` so `html.light` class switches Shiki to light colors via `--shiki-light` CSS variables (overriding the default `prefers-color-scheme` behavior).
- Max-width `720px` for readable prose line length.

### Task 2.3 — PostCard (2026-03-10)
- `PostCard.astro` at `src/components/PostCard.astro` — accepts title, description, date, category, tags, slug, optional heroImage.
- Card structure: border with hover accent (pink), date + category on top line, title (bold, hover pink), description, then tag pills at bottom.
- Title/description wrapped in a single `<a>` linking to the post. Tags are separate `<a>` elements linking to tag pages.
- Tag pill styling matches PostLayout (rounded-full, border, hover pink) — consider extracting to `TagList.astro` in task 2.4.
- `heroImage` prop accepted but not rendered in the card — available for future enhancement if desired.

### Task 2.4 — TagList (2026-03-10)
- `TagList.astro` at `src/components/TagList.astro` — accepts `tags: string[]`, renders tag pills with links to `/blog/tags/{tag}/`.
- Extracted duplicated tag pill markup from PostCard and PostLayout into this shared component.
- Consistent styling: `rounded-full`, border, hover pink accent. Uses `var(--accent-pink)` bracket notation for Tailwind compatibility.
- PostCard wraps TagList in a `px-5 pb-4` div for card padding. PostLayout wraps in `mt-3` div for spacing below metadata.
- TagList handles the empty-tags check internally (`tags.length > 0`).

### Task 3.1 — Home page (2026-03-10)
- Replaced placeholder `index.astro` with full home page using `BaseLayout`, `PostCard`.
- Hero: site title ("mmmaxwwwell"), tagline, gradient accent line (pink → purple → cyan).
- Posts: fetches from `blog` collection, filters out drafts, sorts by date desc, takes first 10.
- Empty state: "No posts yet. Check back soon!" (currently shown since the only post is draft).
- Uses `post.id` as slug (Astro v5 content collections use `id` not `slug`).

### Task 3.2 — Blog index (2026-03-10)
- `src/pages/blog/index.astro` — lists all non-draft posts sorted by date desc using PostCard.
- Category filter nav at top: "All" pill (highlighted as active on this page) + one pill per category linking to `/blog/category/{cat}/`.
- Initially tried query-param filtering (`?category=...`) but this doesn't work with Astro's static output — query params aren't available at build time. Switched to linking to category pages instead (task 3.4).
- The "All" pill on the blog index is a `<span>` (not a link) since you're already on the all-posts page.

### Task 3.3 — Individual post pages (2026-03-11)
- `src/pages/blog/[...slug].astro` — uses `[...slug]` rest parameter so the slug can contain slashes (future-proofing, though current IDs are flat).
- `getStaticPaths()` fetches non-draft posts sorted by date desc, passes prev/next post as props.
- Uses Astro v5 `render()` function (not `post.render()`) to get `Content` component from collection entry.
- Prev/next nav: "Newer" (prev in array = more recent) on left, "Older" (next in array = less recent) on right. Pink hover accent.
- Verified build with non-draft post — generates `/blog/2026-03-10-hello-world/index.html` correctly.
- PostCard links (`/blog/blog/${slug}/`) match the generated paths.

### Task 3.4 — Category pages (2026-03-11)
- `src/pages/category/[category].astro` — `getStaticPaths()` generates one page per unique category from non-draft posts.
- Category nav pills at top mirror the blog index nav — "All" links back to `/blog/blog/`, current category is highlighted (pink bg), others link to their category page.
- Same PostCard layout as blog index. Empty state handled but shouldn't occur (no empty categories generated).
- No category pages generated yet since the only post (`hello-world`) is a draft — will work once non-draft posts exist.

### Task 3.5 — Tags index (2026-03-11)
- `src/pages/tags/index.astro` — shows all tags with post counts, sorted alphabetically.
- Builds a `Map<string, number>` of tag → count from all non-draft posts. Renders as pill links matching existing tag pill styling.
- Empty state: "No tags yet." (currently shown since only post is draft).
- Generated at `/tags/index.html` (with `/blog` base = `/blog/tags/`).

### Task 3.6 — Tag filter page (2026-03-11)
- `src/pages/tags/[tag].astro` — `getStaticPaths()` builds a `Map<string, posts[]>` from all non-draft posts, generates one page per unique tag.
- Page title uses `#tag` format. Includes "All tags" back link to `/blog/tags/`.
- Same PostCard layout as category pages. Posts sorted by date desc.
- No tag pages generated yet since only post is draft — will work once non-draft posts exist.

### Task 3.7 — About page (2026-03-11)
- `src/pages/about.astro` — simple about page with heading, gradient accent line (matching home page), bio text in `.prose` div, GitHub link.
- Content kept brief per the prompt ("can be expanded later"). Three short paragraphs: tagline, what the blog covers, GitHub link.
- Generated at `/about/index.html` (with `/blog` base = `/blog/about/`).

### Task 4.1 — Responsive design (2026-03-11)
- Applied mobile-first responsive adjustments across all pages using Tailwind `sm:` breakpoint (640px).
- **Headings:** All page h1s now scale: `text-2xl sm:text-3xl` (listing pages) or `text-3xl sm:text-4xl` (home, about). PostLayout already had this.
- **Vertical spacing:** All `py-12` → `py-6 sm:py-12`. Margins like `mb-8` → `mb-6 sm:mb-8`.
- **Gradient accent bars:** `w-24` → `w-16 sm:w-24` on home and about pages.
- **PostCard:** Padding `p-5` → `p-4 sm:p-5`, bottom area `px-5 pb-4` → `px-4 sm:px-5 pb-3 sm:pb-4`.
- **Prev/next nav:** `flex` → `flex flex-col sm:flex-row`, removed `max-w-[45%]` on mobile so titles get full width, `items-end` and `ml-auto` only on `sm:`.
- **Tables:** Added `.table-wrapper` overflow-x style in PostLayout prose for responsive tables.
- **Header:** Already had solid mobile hamburger menu — no changes needed.
- **Category/tag nav pills:** Already used `flex-wrap` — works well on mobile.

### Task 4.2 — SEO basics (2026-03-11)
- Added to `BaseLayout.astro`: canonical URL (`<link rel="canonical">`), OG tags (`og:type`, `og:url`, `og:title`, `og:description`, `og:image`), Twitter Card tags (same set with `twitter:` prefix), sitemap link.
- New `image` prop on BaseLayout — optional, used for `og:image` and `twitter:image`. PostLayout passes `heroImage` through.
- Installed `@astrojs/sitemap` — generates `sitemap-index.xml` and `sitemap-0.xml` in dist. Added to `astro.config.mjs` integrations.
- Created `public/robots.txt` — allows all crawlers, references sitemap URL.
- Canonical URLs built from `Astro.url.pathname` + `Astro.site`.

### Task 4.3 — RSS feed (2026-03-11)
- Installed `@astrojs/rss` package. Created endpoint at `src/pages/rss.xml.ts`.
- Uses `getCollection('blog')` with draft filter, sorted by date desc.
- Each item includes: title, pubDate, description, link, and categories (category + tags combined).
- Link format: `/blog/blog/${post.id}/` — matches the post page routes.
- Footer already had RSS link (`/blog/rss.xml`) from task 1.5.
- Feed is empty when no non-draft posts exist (expected).

### Task 4.4 — GitHub Actions deploy workflow (2026-03-11)
- Created `.github/workflows/deploy.yml` — two-job workflow: `build` then `deploy`.
- Triggers: `push` to `main` branch + manual `workflow_dispatch`.
- Build job: checkout → Node 22 setup (with npm cache) → `npm ci` → `npm run build` → upload `dist/` as pages artifact.
- Deploy job: uses `actions/deploy-pages@v4` to deploy the artifact to GitHub Pages.
- Permissions: `contents: read`, `pages: write`, `id-token: write` (required for Pages deployment).
- Concurrency group `pages` with `cancel-in-progress: false` prevents overlapping deploys.
- **Note:** Repo must have GitHub Pages configured to deploy from GitHub Actions (not from a branch). Go to repo Settings → Pages → Source → "GitHub Actions".

### Task 4.5 — Sample posts (2026-03-11)
- Created 3 sample posts covering all non-openscad categories:
  1. `2026-03-08-esp32-temperature-mqtt.mdx` — electrical category, tags: esp32, mqtt, home-assistant, arduino. ~1200 words about building a wireless temp sensor with ESP32 + DHT22 + MQTT for Home Assistant.
  2. `2026-03-09-parametric-coaster-holder-openscad.mdx` — 3d-printing category, tags: openscad, 3d-printing, parametric-design. ~900 words about designing a parametric coaster holder in OpenSCAD. Based on real `/home/max/git/3d/hello_kitty_coaster_holder.scad` file.
  3. `2026-03-10-building-my-blog-with-astro.mdx` — software-dev category, tags: astro, tailwind-css, mdx, github-pages. ~1000 words about building this blog.
- Left `2026-03-10-hello-world.mdx` as draft — serves as a template reference.
- **Bug fix:** `src/pages/category/[category].astro` had `categories.sort()` called inside `.map()` callback. `.sort()` mutates the array in place, which re-orders elements while `.map()` is iterating by index. This caused `3d-printing` (sorted to index 0 after the first `.sort()` call) to be skipped because `.map()` had already passed index 0. Fixed by sorting before `.map()` and passing the pre-sorted array.
- All 3 categories (3d-printing, electrical, software-dev) now generate pages correctly.
- All 11 tags generate pages correctly.
- 21 total pages built.

### Task 4.6 — Final review (2026-03-11)
- Build passes: 21 pages, no errors, no warnings.
- **Bug fix:** `ThemeToggle.astro` used `id="theme-toggle"`, `id="icon-sun"`, `id="icon-moon"` but the component is rendered twice in the Header (desktop + mobile nav). `getElementById` only finds the first instance, so the mobile toggle button was non-functional. Fixed by switching to class-based selectors (`.theme-toggle`, `.icon-sun`, `.icon-moon`) with `querySelectorAll`. Added `window.__themeToggleInit` guard to prevent duplicate event listeners from the `is:inline` script running twice.
- Shiki dual theme CSS verified correct — dark mode uses default Shiki vars (no override needed), light mode overridden via `html.light .shiki` selector.
- All internal links correctly use `/blog` base path.
- Active link detection in Header works correctly for the current page structure.
- SEO meta tags (OG, Twitter Card, canonical, sitemap, robots.txt) all present and correct.

## Open questions
- Hero section on home page — gradient banner? Subtle accent line? Animated retrowave sun/grid? Keep it simple for now, can enhance later.
- Social links in footer — which platforms? GitHub at minimum (`https://github.com/mmmaxwwwell`). Can add later.
- Font loading — system font stack (zero network requests) vs Inter/custom font (better look, extra request). Start with system fonts, upgrade if desired.
