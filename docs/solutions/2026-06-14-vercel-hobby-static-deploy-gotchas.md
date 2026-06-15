---
title: "Deploying a static HTML site to Vercel Hobby: three blocking gotchas"
date: 2026-06-14
tags: [deployment, vercel, static-site]
category: gotcha
module: deployment
symptoms:
  - "Deployment status stuck on UNKNOWN, hangs forever on 'Building…'"
  - "Every URL returns 401 even though the site is meant to be public"
  - "Live URL returns 404 NOT_FOUND / serves a generic Vercel page, not index.html"
  - "Deployment Blocked: commit author did not have contributing access"
---

# Deploying a static HTML site to Vercel Hobby: three blocking gotchas

## Problem
Getting a plain static site (`index.html` at repo root + a `public/` folder of assets) live on Vercel Hobby surfaced three *separate* failures whose symptoms overlapped and pointed in misleading directions — deploys reported `UNKNOWN` / hung on "Building…", URLs 401'd, and even when "Ready" the site 404'd. Each had a distinct root cause.

## Solution

### 1. A `public/` folder gets served AS the site root (root `index.html` 404s)
With no framework detected, Vercel's zero-config treats an existing `public/` directory as the **output directory**, serving its contents at `/`. A root-level `index.html` is then never served (404 / `x-vercel-error: NOT_FOUND`), and assets resolve at `/images/...` instead of `/public/images/...`.

Fix — pin the output directory to the repo root with `vercel.json`:
```json
{ "outputDirectory": "." }
```
Now `/index.html` is served and `<img src="public/images/...">` paths resolve.

### 2. Hobby plan BLOCKS private-repo deploys when the commit author isn't the connected account
If the GitHub repo is **private** and the commit author identity differs from the GitHub account connected to the Vercel project, Hobby refuses to build:
> "Deployment Blocked — the commit author did not have contributing access to the project on Vercel. The Hobby Plan does not support collaboration for private repositories."

This blocks BOTH git-triggered and CLI deploys (the CLI attaches git metadata). Only the very first deploy (before the repo was git-connected) went through.

Fix (cheapest): **make the repo public** — Hobby fully supports public-repo deploys, and the block disappears. (Alternatives: ensure commits are authored by the connected GitHub identity, or upgrade to Pro.)

### 3. Deployment Protection ("Vercel Authentication") makes the CLI misreport status
With Deployment Protection / "Require Log In" enabled, every deployment URL returns **401** to anyone not logged into the Vercel account — including the **CLI's own status-polling**. The result: `vercel --prod` hangs on "Building…", `vercel inspect` reports `status UNKNOWN` with no logs, and `vercel alias set` fails with "deployment is not ready" — even though the build actually succeeded.

Fix — turn it off for a public site: Project → Settings → Deployment Protection → Vercel Authentication → **Disabled**.

## Why It Works
All three stem from Vercel's defaults assuming a *framework app on a paid plan*: zero-config prefers `public/` as build output; Hobby gates private-repo collaboration; and Deployment Protection gates the deployment domain (which the CLI also reads). A static, public, unprotected site needs each default flipped.

## Related
- The misleading part: symptoms (UNKNOWN/Building/401/404) cross-pollinated. Diagnose by viewing the deployment in the dashboard (it showed "Deployment Blocked" plainly) rather than trusting CLI status.
- See `2026-06-14-vercel-web-analytics-programmatic-read.md` for reading analytics on Hobby.
