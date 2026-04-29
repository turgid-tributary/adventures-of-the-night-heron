# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — start dev server at `localhost:4321`
- `npm run build` — build static site to `./dist/`
- `npm run preview` — serve the built site locally
- `npm run astro check` — type-check `.astro` files (use `npm run astro -- <args>` to pass flags)

Requires Node `>=22.12.0`. There is no test runner or linter configured.

## Architecture

Astro 6 static blog scaffolded from the official `blog` starter. Site URL in `astro.config.mjs` is still the placeholder `https://example.com`, and `src/consts.ts` still has the default `SITE_TITLE`/`SITE_DESCRIPTION` — update both when branding the site.

Content flow:

- Posts live as Markdown/MDX in `src/content/blog/`. The collection is defined in `src/content.config.ts` using `glob` + a Zod schema (`title`, `description`, `pubDate`, optional `updatedDate`, optional `heroImage` resolved via `image()`). Frontmatter is type-checked at build time — schema changes propagate to consumers via `CollectionEntry<'blog'>`.
- `src/pages/blog/[...slug].astro` calls `getStaticPaths()` over the collection to generate one route per post, rendering through `src/layouts/BlogPost.astro`.
- `src/pages/rss.xml.js` builds the RSS feed from the same collection — keep post `link` paths in sync with the `[...slug]` route shape (`/blog/${post.id}/`).
- `src/pages/index.astro` and `src/pages/about.astro` are standalone pages; add new top-level routes by dropping `.astro`/`.md` files into `src/pages/`.

Shared chrome lives in `src/components/` (`BaseHead`, `Header`, `Footer`, `HeaderLink`, `FormattedDate`) and is composed by layouts — there is no per-page `<head>` duplication, so SEO/meta changes belong in `BaseHead.astro`.

Integrations & assets:

- `astro.config.mjs` registers `@astrojs/mdx` (enables `.mdx` posts), `@astrojs/sitemap`, and the `@tailwindcss/vite` plugin. It also declares a local Atkinson font via `fontProviders.local()` exposed as the `--font-atkinson` CSS variable. Font files live in `src/assets/fonts/`.
- Styling: Tailwind CSS v4 + DaisyUI 5, configured CSS-first in `src/styles/global.css` (no `tailwind.config.*` file). DaisyUI is loaded via `@plugin "daisyui"` with `light --default, dark --prefersdark` themes registered. The stylesheet is imported once from `src/components/BaseHead.astro`, which every page includes — there is no per-page CSS import. Prefer DaisyUI component classes (`btn`, `card`, `navbar`, etc.) and Tailwind utilities for new UI; reach for scoped Astro `<style>` blocks only when DaisyUI/Tailwind can't express the styling.
- Static assets that should be served as-is go in `public/`; assets imported through Astro (and thus optimized) go in `src/assets/`.

TypeScript extends `astro/tsconfigs/strict` with `strictNullChecks` on top.
