# AGENTS.md — Anna Jay portfolio

A **static, single-file site**: everything lives in `index.html` (inline CSS + vanilla JS, no build step, no framework, no tests). Assets in `public/`, icon at `favicon.svg`.

## Conventions
- Edit `index.html` directly. Project data (cards + work list) is the `TILES` array near the bottom; the featured "AI Builds" cards are static markup in `<section class="featured">`.
- Colors are CSS variables in `:root` (`--bg`, `--fg`, `--muted`, …). Change a color once there.
- Copy style: no em dashes (use commas/periods/middots); date ranges use en dashes. `Campminder` is capital-C, lowercase-m.
- Keep external links `target="_blank" rel="noopener"`.

## Deploy
- Hosted on Vercel (Hobby). Pushing to `main` auto-deploys. The site is served from the **repo root** via `vercel.json` (`{"outputDirectory": "."}`) — do not remove that, or `public/` gets served as root and `index.html` 404s.
- `.vercelignore` keeps `docs/`, fonts, and unused images out of the deploy.

## Before re-discovering anything, read `docs/solutions/`
- `2026-06-14-vercel-hobby-static-deploy-gotchas.md` — why deploys can hang/401/404/block.
- `2026-06-14-vercel-web-analytics-programmatic-read.md` — how to pull page-view counts.
