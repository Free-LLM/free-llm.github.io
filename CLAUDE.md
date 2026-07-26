# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this repository is

This is the **website for the Free-LLM initiative**, published at
**https://free-ai.world** (see `CNAME`) via GitHub Pages. It is a Jekyll site
that documents the vision, architecture, roadmap, and progress of an open,
decentralized, community-owned AI ecosystem.

The code being documented lives in the sibling repository
**`Free-LLM/compute-all`** — the implementation of the **Distributed
Composable Neural Runtime (DCNR)**: orchestrator, physical nodes (PNodes),
virtual nodes (VNodes), CLI, tokenizer, and monitoring stack. When updating
project-status or blog content, source the facts from that repository (its
`README.md`, `docs/`, module READMEs, and commit history).

## Tech stack

- **Jekyll 4.4** with the **just-the-docs** theme (dark color scheme), see
  `_config.yml` and `Gemfile`.
- **Deployment**: GitHub Actions (`.github/workflows/jekyll.yml`) builds and
  deploys to GitHub Pages **on push to `main` only**. Feature branches do not
  deploy anywhere; there is no PR preview.
- **Mermaid diagrams** are rendered client-side: `_includes/head_custom.html`
  loads mermaid from a CDN and converts fenced ` ```mermaid ` code blocks.
- **Custom CSS** in `assets/css/custom.css` (wired via `custom_css` in
  `_config.yml`).

## Local development

```bash
bundle install
bundle exec jekyll serve   # http://localhost:4000
```

## Content layout and conventions

- Pages are **top-level Markdown files** (e.g. `architecture.md`,
  `roadmap.md`). There is no `_posts` collection; the blog lives under
  `blog/` as regular pages using just-the-docs parent/child navigation.
- Every page needs front matter:

  ```yaml
  ---
  title: Page Title      # used for nav and parent/child linking
  layout: default
  nav_order: 11          # position in the left sidebar (fractions allowed)
  ---
  ```

- Sidebar hierarchy uses just-the-docs conventions: `has_children: true` on
  the parent page, `parent: <Parent Title>` on children, `nav_exclude: true`
  to hide a page from the sidebar.
- **Internal links are absolute and extension-less** (`[Roadmap](/roadmap)`,
  not `roadmap.md` or `/roadmap.html`). GitHub Pages serves extension-less
  URLs; keep new pages consistent with this.
- Tone of the site: mission-driven but concrete. Prefer plain statements of
  what exists and what doesn't over hype.

## Special pages — do not break

- `free-llm/compute-protos.md` exists so the **Go toolchain can resolve the
  `free-ai.world/free-llm/compute-protos` module**: `head_custom.html`
  injects a `go-import` meta tag on that URL. Do not rename, move, or delete
  this page, and keep the conditional in `_includes/head_custom.html` intact.
- `CNAME` holds the custom domain (`free-ai.world`). Never modify or delete it.

## When updating project status / blog

- Ground every claim in the `compute-all` repository (code, `docs/`, commit
  history). Do not invent milestones, benchmarks, or dates.
- Also review the **open pull requests** in `compute-all` — significant work
  often lives in a long-running PR before it merges. Report such work as
  "in review"/"in flight", clearly distinguished from what is merged on
  `main`.
- The roadmap (`roadmap.md`) is living documentation — phases may be updated
  to reflect actual progress, but keep the compute-first framing.
- Blog articles go in `blog/` with `parent: Blog` front matter and a date in
  the file name (`blog/YYYY-MM-title.md`) and in the page intro.
- **Obsolete content goes to the Archive, not the trash**: pages describing
  superseded designs get `parent: Archive` front matter and an
  "⚠️ Archived" banner after the H1 (see `archive/index.md`). Keep archived
  files at their original path when possible so URLs don't break; update
  links on current pages so they stop pointing into the archive.
