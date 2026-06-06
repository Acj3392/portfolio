# Implementation Plan — Anna Jay Portfolio

**Date:** 2026-06-06
**Deliverable:** Static HTML mockup (approve the look + interaction) → production Next.js page deployed to Vercel.
**Reference:** Godly-style minimal portfolio (Santi Jaramillo) — oversized type, generous whitespace, smooth reveals.

---

## Concept

A fully public single-page personal portfolio for **Anna Jay**.

- **Hero:** "Anna Jay — Group Product Manager" + a bridging line (product leader with a camera-crew past).
- **Body:** a **7-tile icon-area grid**. Clicking a tile enters *detail mode*: the tiles collapse into a left rail and a detail panel slides in on the right.
- **Close behaviors:** × button, outside-click, or clicking another tile (swaps the active detail, keeping detail mode).
- **Single active tile** at any time.

### The 7 icon areas
| # | Tile | Detail content | Outbound link |
|---|------|----------------|---------------|
| 1 | City Notes | City-discovery app: authors, verbatim Reddit "internet voices," strange facts; no-paraphrase verification pipeline | city-notes.vercel.app + Lisbon zine |
| 2 | MacGuffin Kitchen | Co-founder, 2021–24, "Best hot sauce in CO" | — |
| 3 | DriveSlide | Founder, 2020–present; ideation→crowdfunding→mass production→patent | driveslideofficial.com |
| 4 | Otic Acoustic Art | "Spaces that sound as good as they look" — acoustic treatment representing data of our world | otic.studio |
| 5 | Campminder | Client Rep → PM → Sr PM → Group PM (2021–present) | campminder.com |
| 6 | Colorado Product | Board member, Colorado Product Foundation (501c3), 2024–25 | — |
| 7 | Film | All camera-crew/digital-loader work bucketed (House of Cards, Mandalorian, Boba Fett, Yes Day, Lucifer, Penny Dreadful, All American, Interrogation, On the Basis of Sex; RED field tester) | — |

---

## Architecture

- **Database:** none. Content is static.
- **API:** none.
- **Auth:** none (fully public).
- **AI:** none.
- **Rendering:** static / SSG. Content lives in a typed `content.ts` data file; the page maps over it.

### Stack (production target)
- Next.js 15 (App Router), React 19, TypeScript.
- Tailwind CSS for styling; Framer Motion (or CSS transitions) for the slide-left animation.
- Single route `/` (Server Component shell) + one Client Component for the interactive grid.
- Deploy: Vercel (zero-config, static output).

### Component hierarchy (production)
```
app/layout.tsx          fonts, metadata, base styles
app/page.tsx            Server Component — hero + <IconGrid/> + footer
components/hero.tsx      name, role, bridging line
components/icon-grid.tsx (client) — owns active-tile state + interaction
components/icon-tile.tsx  single tile (thumbnail + label)
components/detail-panel.tsx  right-side content for active tile
content/portfolio.ts    typed array of the 7 tiles + hero copy
public/                  tile artwork (otic hero image, etc.)
```

---

## UI/UX Delivery Plan

**User Journey:** A visitor (recruiter, collaborator, curious peer) lands → reads hero → scans the 7 tiles → clicks one → tiles slide left, detail panel reveals context + outbound link → closes or swaps to another tile.

**Interaction state machine:**
- `idle` → grid centered, all tiles equal.
- `active(tileId)` → grid collapses to left rail; detail panel for `tileId` visible on right.
- Transitions: click tile → `active(id)`; click another tile → `active(otherId)`; click ×, press Esc, or click outside panel/rail → `idle`.

**State Matrix:**
- Loading: n/a (static).
- Empty: n/a (always 7 tiles).
- Error: n/a; outbound links open in new tab.
- Success/active: detail panel rendered with copy + link.

**UX Constraints:**
- Responsive: desktop 3-across grid → tablet 2 → mobile 1. In detail mode on mobile, the rail becomes a top horizontal scroller and the detail stacks below.
- Accessibility: tiles are real `<button>`s; Esc closes; focus moves into the panel on open and returns to the tile on close; `aria-expanded`/`aria-controls` wiring; respects `prefers-reduced-motion`.
- Motion: ~300–400ms ease transitions; reduced-motion users get instant state change.

---

## Tasks (in order)

1. **Static mockup** — `index.html` (self-contained: inline CSS + vanilla JS) — manual browser check — visitor sees hero + 7 tiles, clicking slides left into detail, ×/outside/swap behaviors work. *(This task is the approval gate.)*
2. **Scaffold Next.js** — `package.json`, `app/layout.tsx`, `app/page.tsx`, `tailwind.config`, `content/portfolio.ts` — `npm run build` passes — page renders hero + grid.
3. **IconGrid client component** — `components/icon-grid.tsx`, `components/icon-tile.tsx` — component test for active-state toggle — clicking a tile sets active state.
4. **DetailPanel + interactions** — `components/detail-panel.tsx`, wire Esc/outside-click/swap — interaction test — all close/swap behaviors verified.
5. **Polish + a11y + responsive** — focus management, reduced-motion, mobile layout, real assets in `public/` — Lighthouse/axe check — passes a11y + responsive at 3 breakpoints.
6. **Deploy** — `vercel.json` (if needed), deploy via Vercel — live URL — page reachable publicly.

---

## Test Strategy
- **Unit:** content data shape (7 tiles, required fields, valid URLs).
- **Integration/component:** IconGrid active-state reducer; close-on-Esc, close-on-outside, swap-on-other-tile.
- **E2E (Playwright):** land → click Film tile → detail visible → click City Notes → detail swaps → press Esc → idle.

---

## Ready for: Work Phase
Start with Task 1 (mockup) as the visual approval gate before scaffolding Next.js.
