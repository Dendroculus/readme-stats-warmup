# Readme Stats Warmup

A tiny GitHub Action that **periodically hits** a `github-readme-stats` endpoint (your personal Vercel deployment or the official public endpoint) to *warm* that endpoint's cache so profile cards show fresher numbers for viewers.

This repo provides a **simple, non-invasive** workflow that:

- The warming workflow itself does **NOT** edit your README (no extra commits) — it only performs GET requests.
- If you **deploy your own `github-readme-stats` to Vercel**, add a GitHub Personal Access Token in Vercel (as `PAT_1`) so the deployed service can call the GitHub API and avoid rate limits.
  - For public repo stats only, `read:user` (or similarly minimal scopes) is typically enough.
  - To include private-repo language/stats you will need `repo` (or fine-grained equivalents).
  - **Never** commit the token to the repo — set it as a secret/sensitive environment variable in Vercel.
- If you enable the optional **auto-increment `cache_bust` workflow**, that workflow **requires write permission** to your repo (it will commit and push changes).
- The warming workflow pings your stats endpoint on a schedule (default: every 5 minutes).

---

## Why this exists

The official `github-readme-stats` deployment (and many personal deployments) cache results to reduce GitHub API usage. Warming the cache means that when someone opens your profile, the image is more likely to be fresh — without you manually editing anything.

**Important:** the official public deployment caches heavily (~6 hours). Running a warming workflow against your **own** Vercel deployment plus setting `CACHE_SECONDS` on Vercel gives much faster automatic updates.

---

## ⚡ 30-Second Setup

1. **Create file** `.github/workflows/warm-readme-stats.yml`
2. **Paste** the YAML below (replace `YOUR_STATS_URL_HERE`)
3. **Commit** - the workflow starts automatically
   
```yaml
name: Warm Readme Stats Cache

on:
  schedule:
    - cron: '*/5 * * * *'    # runs every 5 minutes
  workflow_dispatch:          # allow manual run

jobs:
  warm:
    runs-on: ubuntu-latest
    steps:
      - name: Curl stats endpoint to warm Vercel cache
        run: |
          # Replace the URL below with your generated stats image URL.
          curl -sS "YOUR_STATS_URL_HERE" --fail >/dev/null
          echo "warmed cache"
```

### Example stats image URLs

- Official (public):  
  `https://github-readme-stats.vercel.app/api?username=YOUR_USER&show_icons=true&theme=tokyonight`

- Your Vercel deployment (example):  
  `https://github-yourname-readme-stats.vercel.app/api?username=YOUR_USER&show_icons=true&theme=tokyonight`

---

## Notes on timing, caching, and rate limits

This warming workflow speeds up how quickly visitors see fresh numbers, but it is not a real-time feed — expect updates within a few minutes, not instantly.

Multiple caches are involved:

- Your Vercel/server cache (`CACHE_SECONDS`)
- GitHub API propagation and rate limits
- GitHub’s image proxy (camo) and browser caches

**Why 5 minutes as default:** Running the workflow more frequently (e.g., every 30s) increases the chance of hitting GitHub or Vercel rate limits and is unnecessary for a README. A 5-minute schedule is a balance: frequent enough to feel near-real-time, but conservative enough to avoid API throttling.

### If you need faster updates

- Deploy your own `github-readme-stats` to Vercel and set a low `CACHE_SECONDS` (e.g., 60–120 seconds).
- Use `&cache_bust=N` in the image URL to force a fresh fetch. (Example: `...?username=foo&cache_bust=12345`)

> Warning: auto-incrementing `cache_bust` requires a workflow that commits to your README and will create a commit each run — only enable if you accept commit churn.

---
