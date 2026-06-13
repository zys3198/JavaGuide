# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository identity

JavaGuide is a **content + docs site** repo, not an application. It publishes a Chinese-language Java/AI backend interview & learning guide at https://javaguide.cn/ built with **VuePress 2 + vuepress-theme-hope**. Almost all work here is editing Markdown under `docs/`; the JS/TS under `docs/.vuepress/` only configures the site.

The author/upstream is `Snailclimb/JavaGuide`. This clone tracks it read-only.

## Git remotes & write policy (read this first)

- `origin` → `zys3198/JavaGuide` (the user's fork — only this is writable, and even then per-change authorization required)
- `upstream` → `Snailclimb/JavaGuide` (the official repo — **NEVER** push, open PRs, file issues, comment, or otherwise write without explicit per-action user authorization)
- Pulling from `upstream`/`origin` to sync local is allowed. Default stance: this is a study clone, keep it in sync, don't push.

## Toolchain

- **pnpm 10.0.0** is the pinned package manager (`packageManager` field). Don't use npm/yarn — `pnpm-lock.yaml` is authoritative.
- Node 20+ recommended (VuePress 2 + Vite 7 bundler).

## Commands

| Task                                  | Command                                                 |
| ------------------------------------- | ------------------------------------------------------- |
| Start dev server (hot reload)         | `pnpm dev` (alias for `pnpm docs:dev`)                  |
| Dev with cache wiped                  | `pnpm docs:clean-dev`                                   |
| Production build → `dist/`            | `pnpm build` (alias for `pnpm docs:build`)              |
| Build wiping `.temp`/`.cache` first   | `pnpm build:clean`                                      |
| Lint everything                       | `pnpm lint`                                             |
| Lint markdown only                    | `pnpm lint:md`                                          |
| Format + check via prettier           | `pnpm lint:prettier`                                    |
| Lint a single file                    | `pnpm markdownlint-cli2 path/to/file.md`                |
| Submit URLs to IndexNow (post-deploy) | `pnpm indexnow:submit --sitemap` (needs `INDEXNOW_KEY`) |
| Rebuild DocSearch index               | `pnpm docsearch:index`                                  |
| Upgrade VuePress theme                | `pnpm update`                                           |

There is **no test suite** — verification is `pnpm build` succeeding + visual check in the dev server.

## Pre-commit hook

`.husky/pre-commit` runs `pnpm nano-staged`:

- All files → `prettier --write --ignore-unknown`
- Markdown → `markdownlint-cli2`

A commit that fails markdownlint or prettier will be blocked. Fix the lint errors rather than bypassing with `--no-verify`.

## Markdown lint rules (`.markdownlint-cli2.mjs`)

Defaults on, with these notable relaxations — safe to rely on these:

- MD010 (no hard tabs), MD013 (line length), MD036 (emphasis as heading), MD040 (fenced-code-language), MD045 (image alt), MD046 (code block style) — **disabled**
- MD024 allows siblings with same heading text (different nesting)
- `*.snippet.md` and `TODO.md` are ignored — snippets are markdown-include fragments, not full pages
- ATX headings (`#`), dash bullets (`-`), `---` thematic breaks are enforced

## Site architecture (`docs/.vuepress/`)

- **`config.ts`** — top-level VuePress config. Sets Vite bundler with SCSS preprocessor (sass-embedded), `pagePatterns` exclude `*.snippet.md` / `TODO.md`, alias for `@vuepress/plugin-markdown-chart`'s Mermaid client, 百度统计 injected via `head`. Output dir is `./dist`.
- **`theme.ts`** (~470 lines) — the heart. Configures `hopeTheme`. Notable custom logic:
  - **`buildSeoDescription(page, app)`** generates `<meta description>` / OGP / JSON-LD descriptions on the fly, targeting a 150–160 char window, falling back through `frontmatter.description` → page title + headers + tags → excerpt/page text. Don't hand-write per-page descriptions unless overriding — the auto-builder is the source of truth.
  - JSON-LD `ItemList` blocks are injected for `/`, `/home.html`, `/ai/`, `/cs-basics/` to feed search engines.
  - `copyright` plugin is **disabled** (upstream bug during hydration).
  - `docsearch` (Algolia) only activates when `DOCSEARCH_APP_ID`/`DOCSEARCH_API_KEY`/`DOCSEARCH_INDEX_NAME` env vars are all set; otherwise site search is off.
- **`navbar.ts`** — top nav.
- **`sidebar/index.ts`** (~600 lines) — the big one. Path-specific sidebar maps for `/ai/`, `/ai-coding/`, `/cs-basics/`, `/open-source-project/`, `/books/`, `/about-the-author/`, `/high-quality-technical-articles/`, `/zhuanlan/`, then a catch-all `/` covering Java/db/tools/framework/system-design/distributed/high-performance/high-availability. **Order matters**: more specific path prefixes must come before `/`. Helpers `createImportantSection` / `createSourceCodeSection` (from `constants.ts`) wrap grouped child entries.
- **`sidebar/constants.ts`** — `ICONS` map and section-builder helpers; new sidebar entries should reuse these.
- **`client.ts` + `components/`** — client-side enhancements (`LazyMermaid`, `ClickImagePreview`, layout toggles).
- **`styles/`** — `palette.scss` (theme tokens), `config.scss`, `index.scss`.
- **`public/`** — static assets served as-is (logo, favicon).

## Content layout (`docs/`)

Top-level directories are topic areas, mirroring the sidebar structure: `java/` (basis/collection/concurrent/io/jvm/new-features), `database/` (mysql/redis/elasticsearch/mongodb/sql), `system-design/`, `distributed-system/`, `high-performance/`, `high-availability/`, `cs-basics/`, `ai/`, `ai-coding/`, `tools/`, `interview-preparation/`, etc. `docs/snippets/` holds `@`-prefixed markdown fragments pulled in via the theme's `include` feature (`@foo` → `docs/snippets/foo`).

`docs/home.md` is the landing page content; `README.md` at repo root is the GitHub-facing index.

## `my-submission/` — user's personal workspace

Not part of the published site. Contains `completed/` and `todo/` subfolders for the user's own article drafts. Git-tracked locally but irrelevant to the upstream VuePress build (lives outside `docs/`, so `pagePatterns` won't pick it up). Treat as scratch space — don't "clean up" without asking.

## Editing content — practical notes

- New pages: create the `.md`, then **register it in the relevant `sidebar/*.ts`** or it won't appear in navigation. Match the existing entry style (string link vs. `{ text, link }` object).
- Cross-page links use relative `.md` paths; VuePress rewrites them to `.html` at build.
- Code blocks: language tag is **not** required (MD040 disabled), but adding one enables highlighting — prefer to tag.
- Images: drop into `docs/.vuepress/public/` and reference as `/image.png`, or co-locate per-section.
- Mermaid diagrams are supported via the markdown-chart plugin (` ```mermaid` fence).

## Deploy / hosting

Static output in `dist/` is served from `javaguide.cn`. `scripts/indexnow-submit.mjs` POSTs sitemap URLs to IndexNow after deploy when `INDEXNOW_KEY`/`INDEXNOW_*` env vars are set; without the key it no-ops safely. `.github/workflows/test.yml` runs CI on push (verify before assuming deploy steps).
