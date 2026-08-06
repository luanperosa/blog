# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

---

## Behavioral Guidelines

Bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding
Don't assume. Don't hide confusion. Surface tradeoffs.

Before implementing:
- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First
Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes
Touch only what you must. Clean up only your own mess.

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution
Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria enable independent looping. Weak criteria ("make it work") require constant clarification.

**Working signals:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, clarifying questions arriving before implementation rather than after mistakes.

---

## Commands

```bash
# Development
npm start          # Start local dev server at http://localhost:3000
npm run build      # Build production static site to /build
npm run serve      # Serve the built site locally
npm run clear      # Clear Docusaurus cache

# Testing
npm test                  # Run unit tests with Jest
npm run test:coverage     # Run tests with coverage report

# Deployment
USE_SSH=true npm run deploy               # Deploy to GitHub Pages via SSH
GIT_USER=<username> npm run deploy        # Deploy to GitHub Pages via HTTPS

# Content utilities
npm run write-translations    # Generate i18n translation files
npm run write-heading-ids     # Auto-generate heading IDs
npm run swizzle               # Eject/customize Docusaurus components
```

## Architecture

This is a [Docusaurus 3.9.2](https://docusaurus.io/) static documentation site.

**Content** lives in two places:
- `blog/` — Blog posts with author metadata in `blog/authors.yml` and tags in `blog/tags.yml`
- `src/pages/` — Custom React pages (rendered at their filename path, e.g. `index.js` → `/`)

Note: The docs plugin is disabled (`docs: false` in the preset config). The `docs/` folder and `sidebars.js` are unused.

**React source** in `src/`:
- `src/pages/index.js` — Homepage with hero banner + blog post list
- `src/pages/portfolio.js` — Portfolio page at `/portfolio`, renders cards from `src/data/portfolio.js`
- `src/components/HomepageFeatures/` — Homepage blog list component (renders posts from `usePluginData`)
- `src/css/custom.css` — Global Infima CSS variable overrides (primary color, dark mode)
- `src/theme/BlogListPage/` — Custom blog list page wrapper with JSON-LD structured data for SEO

**Portfolio page:**
- `src/data/portfolio.js` — Data file exporting `portfolioItems`. Each item has `title`, `description`, `image`, `bullets: string[]`, and optional `url`.
- `src/pages/portfolio.js` — Renders a 2-column grid of project cards (company name header, screenshot, bold description, bullet list, URL link).
- `src/pages/portfolio.module.css` — Scoped styles for the portfolio page and cards.

**Configuration:**
- `docusaurus.config.js` — Main config: site metadata, navbar, footer, theme (Prism), presets

**Homepage blog list:**
- `plugins/blog-posts-plugin.js` — Custom Docusaurus plugin that uses `allContentLoaded` to read blog posts from the built-in `docusaurus-plugin-content-blog` plugin's processed data (no filesystem parsing). Exposes the post list via `setGlobalData`.
- The homepage (`src/components/HomepageFeatures/`) consumes this data with `usePluginData('blog-posts-plugin')`.
- Adding a new `.md` or `.mdx` file to `blog/` (or a folder with `index.md`) will automatically include it on the homepage after the next build. The post must have the required frontmatter fields (see blog post conventions below). Posts retain the order from the blog plugin (newest first).

**Blog post conventions:**
- File naming: `blog/YYYY/YYYY-MM-DD-slug.md`
- Required frontmatter: `slug`, `title`, `authors` (key from `authors.yml`), `tags` (keys from `tags.yml`)
- Include `<!-- truncate -->` to control the preview excerpt — omitting it triggers a build warning (`onUntruncatedBlogPosts: 'warn'`)
- `onBrokenLinks: 'throw'` — the build hard-fails on any broken internal links
- Invoke the `blog-post-docusaurus` skill whenever creating a new blog post

**Styling** uses CSS Modules (`.module.css`) for component-scoped styles and Infima CSS framework variables for global theming. Dark/light mode is handled via CSS custom properties.
