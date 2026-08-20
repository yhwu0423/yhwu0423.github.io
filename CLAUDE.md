# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal blog of **wyh** (https://yhwu0423.github.io), built with **Jekyll 4** on the Hux Blog template (forked from `Huxpro/huxpro.github.io`). Deployed via GitHub Pages from the `master` branch (see `.github/workflows/jekyll.yml`).

## Commands

- Local preview: `bundle exec jekyll serve` (or `npm start`)
- Build only: `bundle exec jekyll build` (output goes to `_site/`, which is git-ignored build output — never edit it)
- Install deps: `bundle install` (gems: jekyll ~> 4.0, jekyll-paginate, webrick)
- New post scaffold: `rake post title="My Title" subtitle="..."` — note it hardcodes `author: "Hux"` and `header-img: img/post-bg-2015.jpg`; change author to `wyh` after generating
- CSS/JS build: `npm install` then `npx grunt` — compiles `less/hux-blog.less` → `css/hux-blog.min.css` and `js/hux-blog.js` → `js/hux-blog.min.js`. **Never edit the `.min.*` / compiled `.css` files directly**; the Gruntfile derives filenames from `pkg.name` in `package.json` (still `"hux-blog"` — renaming it breaks the pipeline)
- `npm run dev` = grunt watch + jekyll serve together

There are no tests.

## Content model

- Posts live in `_posts/YYYY-MM-DD-slug.md`. `_config.yml` sets `future: true`, so future-dated posts still publish — the existing 2026-dated posts depend on this; do not remove that setting.
- Post front matter conventions (see existing 2026 posts):
  - `header-img: "img/..."` and `header-style: text` are mutually exclusive — plain-text header vs. image header
  - `author: "wyh"`, `catalog: true` (auto TOC, requires header ids from kramdown GFM), `tags:` list
  - `mathjax: true` enables math on a post
- Bilingual posts: a post with `multilingual: true` pulls its body from `_includes/posts/<slug>/{zh,en}.md` and renders a language switcher via `_includes/multilingual-sel.html`. The About page (`about.html` → `_includes/about/{zh,en}.md`) works the same way.

## Architecture notes

- All site identity (title, description, SNS usernames, sidebar avatar, featured-tag threshold, friends links) is driven by `_config.yml`. SNS icons in `_includes/sns-links.html` render only when the corresponding `*_username` key exists — to hide a network, comment it out in the config rather than editing the template.
- `sidebar-avatar` must be an **absolute URL** (it's referenced from both `/` and `/about/`); currently `https://github.com/yhwu0423.png`.
- Disqus comments and Google Analytics are intentionally commented out in `_config.yml` (they were the template author's accounts). Re-enable only with wyh's own IDs.
- Layout chain: `_layouts/default.html` (nav + footer + head includes) → `page.html` / `post.html` / `keynote.html`. Search is client-side via `js/simple-jekyll-search.min.js` reading `search.json`.
- **Service worker**: `sw.js` precaches a hardcoded `PRECACHE_LIST` of static assets. When adding or deleting top-level static files (images, JS, CSS), update that list or offline mode will 404 / serve stale assets. PWA metadata is in `pwa/manifest.json`.
- No `CNAME` file — the site is served at `yhwu0423.github.io`; adding one would change the custom domain.
- `_doc/` contains the original template's user manual (Manual.md, Chinese README) — reference documentation, not part of the built site.
- File/asset names containing `hux` (e.g. `js/hux-blog.js`, `css/hux-blog.min.css`, `#huxblog_navbar`) are internal template identifiers wired together across Gruntfile, includes, and JS — do not rename them.
