---
title: "Reading Vercel Web Analytics data programmatically (no public API on Hobby)"
date: 2026-06-14
tags: [analytics, vercel, integration]
category: integration
module: deployment
symptoms:
  - "Want page-views-by-day on demand but Vercel Hobby has no public Web Analytics API"
  - "Dashboard doesn't live-update; first events lag ~1-2 min"
---

# Reading Vercel Web Analytics data programmatically

## Problem
Vercel Web Analytics has no public/token-based query API on the Hobby plan. The goal was to fetch "how many page views, on which days" on demand instead of eyeballing the dashboard chart.

## Solution
Replay the same internal endpoint the dashboard itself calls, from an **authenticated browser session** (the user's logged-in Chrome). The dashboard's data comes from:

```
GET https://vercel.com/api/web-analytics/v2/timeseries
    ?projectId=<project>&teamId=<team>&from=<ISO>&to=<ISO>&tz=<IANA tz>
```
Call it with `fetch(url, { credentials: "include" })` so the session cookies authorize it. Response shape:
```
{ data: { groups: { all: [ { key: <ISO timestamp>, total: <page views>, devices: <unique visitors> }, ... ] } } }
```
Sum `total` for page views and `devices` for visitors; bucket by the `key`'s date.

- **Bucket granularity is adaptive**: hourly for ranges ≲ 7 days, daily for longer ranges. Account for this when grouping by day, and cross-check the grand total against the dashboard's "Page Views".
- Realtime ("online now"): `https://vercel.com/api/web-analytics/v2/realtime?projectId=...&teamId=...&tz=...`.
- The exact `projectId`/`teamId` are project-specific and live in private notes, not this (public) repo.

## Why It Works
The analytics dashboard is a normal SPA hitting first-party JSON endpoints under `vercel.com/api`. With the user's cookies present, those endpoints authorize the same way they do for the dashboard — no separate API token needed. A fresh headless/unauthenticated browser hits the login wall, so this only works in the user's signed-in session.

## Gotchas worth remembering
- Setup: add the cookieless script to the page (`/_vercel/insights/script.js` + the `window.va` shim) AND enable Web Analytics in the dashboard; then **redeploy** — the `/_vercel/insights/*` endpoints only activate on a build made after enabling.
- The dashboard does NOT live-refresh; reload it, and allow ~1-2 min after a visit.
- `curl`/server-side requests don't run JS, so they don't register as visits — handy for testing without polluting counts.

## Related
- `2026-06-14-vercel-hobby-static-deploy-gotchas.md`
