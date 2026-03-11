# mmmaxwwwell's Blog — Step-by-Step Prompt

## What we're building
A personal tech blog hosted on GitHub Pages at `https://mmmaxwwwell.github.io/blog/`. The blog covers hobbyist electronics, OpenSCAD 3D modeling, 3D printing, software development, and other tech projects. Built with Astro v5, MDX, and Tailwind CSS v4 with a retrowave-inspired design featuring dark/light mode.

## Tech stack & architecture
- **Framework:** Astro v5.x with MDX integration
- **Styling:** Tailwind CSS v4
- **Content:** MDX files in Astro content collections with typed frontmatter
- **Deployment:** GitHub Actions → GitHub Pages
- **Base URL:** `/blog/` (repo is `mmmaxwwwell/blog`, not `mmmaxwwwell.github.io`)

### Key paths (target structure)
```
/
├── .github/workflows/deploy.yml    # GitHub Actions deploy workflow
├── src/
│   ├── components/                 # Astro/MDX components
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── ThemeToggle.astro       # Dark/light mode toggle
│   │   ├── PostCard.astro          # Blog post preview card
│   │   ├── TagList.astro           # Tag pills/chips
│   │   └── CategoryNav.astro       # Category navigation
│   ├── content/
│   │   ├── config.ts               # Content collection schema
│   │   └── blog/                   # MDX blog posts go here
│   │       └── example-post.mdx
│   ├── layouts/
│   │   ├── BaseLayout.astro        # HTML shell, theme, meta
│   │   └── PostLayout.astro        # Blog post layout with tags, date, etc.
│   ├── pages/
│   │   ├── index.astro             # Home — recent posts
│   │   ├── about.astro             # About page
│   │   ├── blog/
│   │   │   ├── [...slug].astro     # Individual post pages
│   │   │   └── index.astro         # All posts list
│   │   ├── category/
│   │   │   └── [category].astro    # Category filter pages
│   │   └── tags/
│   │       ├── index.astro         # All tags overview
│   │       └── [tag].astro         # Posts filtered by tag
│   └── styles/
│       └── global.css              # Tailwind directives + retrowave theme tokens
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
├── package.json
├── CLAUDE.md                       # Instructions for content-creation agent
├── blog-prompt.md                  # This file — scaffolding agent prompt
└── memory/
    ├── blog-tasks.md               # Task checklist
    └── blog-notes.md               # Decisions, notes, reference
```

## Design
- **Aesthetic:** Professional, slick, minimal with retrowave accent colors
- **Retrowave palette:**
  - Neon pink: `#FF2D95` / `#FF6EB4`
  - Cyan: `#00F0FF` / `#00C8D4`
  - Purple: `#B026FF` / `#7B2FBE`
  - Dark background: `#0D0221` / `#1A1A2E`
  - Light background: `#F8F8FF` / `#EEEEF5`
- **Dark mode:** Default. Deep dark backgrounds with neon accents for links, borders, highlights
- **Light mode:** Light backgrounds with muted retrowave accents (darker versions of the neon colors for contrast/accessibility)
- **Toggle:** Persistent dark/light toggle in header, respects `prefers-color-scheme` on first visit, saves preference to `localStorage`
- **Typography:** Clean sans-serif body (system font stack or Inter), monospace for code blocks
- **Layout:** Max-width content container (~720px prose, ~1080px for wider layouts), generous whitespace

## Pages & user flows

### Home (`/blog/`)
- Header with site title, nav (Home, Blog, About), theme toggle
- Hero or tagline area (subtle retrowave gradient or accent line)
- Recent posts list (5-10 most recent) as cards showing: title, date, description, tags, category
- Footer with socials/links

### Blog index (`/blog/blog/`)
- All posts, paginated or infinite scroll
- Filter by category (top nav or sidebar)
- Tags shown on each post card

### Post page (`/blog/blog/[slug]/`)
- Post title, date, category, tags
- MDX rendered content with syntax highlighting (Shiki)
- Prev/next post navigation

### Category pages (`/blog/category/[category]/`)
- Posts filtered by category
- Category name as page title

### Tag pages (`/blog/tags/[tag]/`)
- Posts filtered by tag

### Tags index (`/blog/tags/`)
- All tags with post counts, clickable

### About (`/blog/about/`)
- Brief bio: "Software engineer riding the AI wave"
- Can be expanded later

## Content collection schema
```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    updated: z.coerce.date().optional(),
    category: z.string(),
    tags: z.array(z.string()),
    draft: z.boolean().default(false),
    heroImage: z.string().optional(),
  }),
});

export const collections = { blog };
```

## Reference files
- `memory/blog-tasks.md` — task checklist with progress
- `memory/blog-notes.md` — detailed decisions, architecture notes, reference details

---

## How to work

1. **Read the task list** at `memory/blog-tasks.md` — find the FIRST unchecked task (`- [ ]`)
2. **Read the memory file** at `memory/blog-notes.md` — review decisions, architecture notes, and any blockers from previous sessions
3. **Pre-task review** — Before starting, think about:
   - Does this task have everything it needs? Are there missing details, ambiguous requirements, or design decisions that should be made first?
   - Are there dependencies on earlier tasks that aren't done yet?
   - Will this task affect other tasks or require changes to the plan?
   - Is there anything the user should weigh in on before you start?
   - **If anything is unclear or needs a decision, ASK the user before proceeding.**
4. **Execute that ONE task** — implement it, test it, verify it works
5. **Update the task list** — mark the task `- [x]` with a brief note of what was done
6. **Update the memory file** — add any new findings, decisions, gotchas, or file paths discovered during implementation
7. **Post-task review** — After completing the task, think about:
   - Did anything come up during implementation that changes the plan? (new tasks needed, tasks that should be reordered, tasks that are now unnecessary)
   - Are there open questions or risks for upcoming tasks?
   - Does the next task still make sense, or should something else come first?
   - **Update the task list and memory file with any changes. Flag anything the user should know about.**
8. **Stop and report** — tell the user what you did, what worked, what didn't, what questions came up, and what the next task is

## Rules

- ONE task per invocation. Do not skip ahead.
- If a task is blocked, write the blocker in the memory file, mark the task `- [?]` with the reason, and move to the next unblocked task.
- If you discover a task is unnecessary, mark it `- [~]` with why, and move on.
- If a task needs to be split into subtasks, add them as indented items under the parent task (e.g., 1.2.1, 1.2.2).
- If you discover NEW tasks are needed during implementation, add them as subtasks in the task list, update the memory file, and note them in the prompt context.
- Always read both files before starting — context from previous sessions matters.
- Prefer minimal changes. Don't refactor unrelated code.
- Test your work before marking complete — run `npm run build` to verify the site builds without errors. Run `npm run dev` and check the browser if the task involves visual changes.
- If you need user input (e.g., design decision), ask and do NOT proceed until answered.
- **Always verify the license** of any external code/repo before cloning or vendoring it. Confirm the license is compatible (MIT, Apache-2.0, BSD, etc.) and note it in the memory file.
- Use conventional commits (e.g., `feat: add tag filter page`, `fix: dark mode toggle persistence`).
- Do not modify `CLAUDE.md` — that file is for the content-creation agent and is maintained separately.
