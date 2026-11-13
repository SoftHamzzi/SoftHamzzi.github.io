# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

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

## `_posts/game_dev/{devlog,portfolio}` — writing style

See **`WRITING_STYLE.md`** at repo root before drafting or editing a post in these two folders.
It covers the DevLog(안 고친다)/Portfolio(고친다) split, sentence-level rules (no em-dash, no
middot, avoid "~인가" endings, no vague-teaser headings), folding code into
`<details markdown="1">`, table/mermaid usage, and the rule that DevLog posts must not reference
devlog chapters written later than the one being edited. Read it in full before touching these
posts — don't rely on memory of past sessions.

### `_posts/routine/` — personal retrospective journal

Not code documentation — a recurring personal-retrospective log with four cadences (`daily`, `weekly`, `monthly`, `yearly`), each further nested by year/month for `daily`. Every entry follows the same **5F template** (Fact / Feeling / Finding / Future action / Feedback), with the Feedback section structured as an S/B/I/N/F table plus a closing direct-comment paragraph, and a final "🌙 남기는 말" one-liner. When drafting a new entry in this folder, match the existing tone and structure exactly rather than inventing a new format — look at a recent file in the same cadence folder as the template.

회고 피드백 원본은 `.claude/skills/retro-feedback/SKILL.md`, 현재 AI용 진입점은 `.agents/skills/retro-feedback/SKILL.md`다. **Feedback만 작성하고 Future action과 선택 학습은 수정하지 않는다.** 기존 피드백이 있으면 덮어쓰기 전에 확인한다.

## 블로그 세션의 전용 탭

Tern의 `블로그` 세션에서 `회고`와 `퀴즈` 탭을 분리해 사용한다. 화면의 세션 묶음만 같으며, 각 탭의 대화 기록과 작업 폴더는 별도로 유지한다.

| 요청 | 작업 폴더와 절차 |
|---|---|
| 회고 피드백 | 이 블로그 루트에서 `retro-feedback` 스킬. 사용자가 지정한 회고 파일을 처리한다 |
| 말랑퀴즈 생성·채점 | 기본은 `C:/Github/MallangQuiz`의 같은 탭에서 수행. 이 블로그에서 요청해도 `mallang-quiz`·`mallang-grade` 스킬이 원본 절차로 연결한다 |

퀴즈 요청은 회고와 구분한다. `_posts/routine/mallang_quiz`는 회고 대상이 아니다. 문제지 블로그 이동은 기존처럼 사용자가 직접 한다. 실제 채점할 때만 해답 파일과 오답노트를 갱신하며, 설정 중에는 기존 글·해답·점수·오답 기록을 바꾸지 않는다. 커밋·푸시·게시는 별도 요청 없이 하지 않는다.

