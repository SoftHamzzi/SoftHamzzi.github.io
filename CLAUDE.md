# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal Jekyll blog (`SoftHamzzi.github.io`, published via GitHub Pages) built directly on top of a vendored copy of the **Minimal Mistakes** theme source — the theme is not installed as a gem/`remote_theme`, its actual files (`_layouts`, `_includes`, `_sass`, `assets`) live in this repo and are used as-is. Only touch those theme internals if a genuine layout/styling change is needed; for everyday work the task is almost always adding or editing content under `_posts`/`_pages`.

`docs/` and `test/` are the upstream theme's own demo/test site scaffolding (carried over from the `mmistakes/minimal-mistakes` project this was forked from). They are explicitly excluded from the real build (`exclude:` in `_config.yml`) and are not part of the live site — don't edit them when making content changes.

## Commands

```bash
bundle install                 # install Ruby/Jekyll deps (see Gemfile / minimal-mistakes-jekyll.gemspec)
bundle exec jekyll serve       # local dev server for the real site (root _config.yml, http://localhost:4000)
bundle exec jekyll build       # production build -> _site/
```

Deployment is automatic: `.github/workflows/jekyll.yml` runs `bundle exec jekyll build` and publishes `_site/` via GitHub Pages Actions on every push to `main`. There is no test suite or linter to run for content changes.

`package.json` scripts (`npm run build:js` / `uglify`) only rebuild the theme's bundled/minified vendor JS (`assets/js/main.min.js`) — irrelevant unless you're editing theme JS.

## Content architecture

- Posts live under `_posts/<topic>/...` (e.g. `algorithm`, `game_dev/{cpp,devlog,moderncpp,portfolio,technique}`, `graphics`, `ps/{baekjoon,programmers}`, `server/game_server`, `routine/{daily,weekly,monthly,yearly}`). The subfolder path is purely organizational for the repo — it has no effect on the site's URL.
- URLs are controlled by `permalink: /:categories/:title/` in `_config.yml` plus each post's front-matter `categories:`. Every category used must have a matching taxonomy page at `_pages/categories/category-<slug>.md` (see existing ones, e.g. `category-cpp.md`, `category-devlog.md`) or its archive/listing page won't resolve — add one when introducing a new category.
- `defaults:` in `_config.yml` applies `layout: single, author_profile: true, read_time: true, comments: true, share: true, related: true, mathjax: true, sidebar_main: true` to every post automatically — no need to repeat these in front matter.
- Standard post front matter: `title`, `excerpt`, `categories`, `tags`, `toc`/`toc_sticky`, `date`, `last_modified_at`; add `mermaid: true` when a post embeds Mermaid diagrams.
- `_data/navigation.yml` drives the top nav bar.

### `_posts/routine/` — personal retrospective journal

Not code documentation — a recurring personal-retrospective log with four cadences (`daily`, `weekly`, `monthly`, `yearly`), each further nested by year/month for `daily`. Every entry follows the same **5F template** (Fact / Feeling / Finding / Future action / Feedback), with the Feedback section structured as an S/B/I/N/F table plus a closing direct-comment paragraph, and a final "🌙 남기는 말" one-liner. When drafting a new entry in this folder, match the existing tone and structure exactly rather than inventing a new format — look at a recent file in the same cadence folder as the template.
