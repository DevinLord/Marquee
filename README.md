# Marquee — a personal movie diary

A single-user movie diary in the style of Letterboxd: log what you've watched,
rate it, review it, browse alternate posters, build lists, and keep a
ranked Top 5. Runs as one self-contained HTML file — no build step, no
server, no database.

## Live setup (Netlify)

1. Get a free TMDB API key — see below.
2. This repo's `index.html` **is** the site. Push it to GitHub as-is.
3. In Netlify: **Add new site → Import an existing project → GitHub** →
   pick this repo.
4. Build command: leave blank. Publish directory: leave as `/` (root).
   Click **Deploy site**.
5. Open the Netlify URL, click **Settings** in the app, paste your TMDB
   key, and click **Test connection**.

That's it — no environment variables, no functions, no backend.

## Running it locally

Double-click `index.html` to open it in a browser. Note: some browsers
restrict `localStorage` for files opened directly from disk (`file://`),
so entries may not save until it's served over `https://` (i.e. deployed
to Netlify). Local opening is fine for a quick look, not for daily use.

## Getting a TMDB API key

1. Create a free account at [themoviedb.org](https://www.themoviedb.org)
   and verify your email.
2. Profile avatar (top right) → **Settings** → **API**.
3. Request a key, choose **Developer**, accept the terms.
4. On the application form: Application URL can be `N/A` or
   `http://localhost` — TMDB accepts that for personal, non-commercial
   apps. Describe it as a personal, non-commercial movie diary.
5. Approval is instant. The API settings page then shows two credentials
   — either works in this app's Settings screen:
   - **API Key (v3 auth)** — a short key
   - **API Read Access Token (v4 auth)** — a long token starting `eyJ`

## How data is stored

Everything — entries, ratings, reviews, lists, and your Top 5 — is saved
in this browser's `localStorage`, on whichever device/browser you're
using. Nothing is sent anywhere except to TMDB (to search movies and
fetch posters) and, once you paste it in, your API key lives only in
that same local storage.

Consequences worth knowing:
- Data is **per-browser**, not synced. Using the site from your phone and
  your laptop means two separate diaries unless you export/import.
- Clearing your browser's site data for this URL erases your diary.
- There's no login and no multi-user support — it's built for one person.

## Features

- **Search & log** — TMDB search, watch date, half-star ratings (5-star
  scale), written reviews.
- **Grid view** — poster grid sorted by watch date by default; sort by
  rating, title, or year; filter by minimum rating or title search.
- **Movie detail** — edit rating/date/review; browse every alternate
  poster TMDB has for that title and set one as the entry's poster.
- **Lists** — create, rename, reorder, delete; add movies from any
  detail page.
- **Top 5** — five ranked slots, reorderable, swappable, drawn from your
  logged movies.
- **Stats** — total logged, average rating, top genre, top decade,
  movies-per-month (last 12 months), rating distribution.

## Tech notes

- Single React app, loaded via CDN (unpkg) with Babel standalone doing
  in-browser JSX compilation — that's what lets it run as one plain
  `.html` file with no build tooling.
- TMDB errors (bad key, rate limit, network) surface as plain-language
  messages in the UI rather than console-only failures.
