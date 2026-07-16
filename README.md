# Marquee — a personal movie diary

A single-user movie diary in the style of Letterboxd: log what you've watched,
rate it, review it, browse alternate posters, build lists, and keep a
ranked Top 5. Runs as one self-contained HTML file — no build step, no
server, no database.

## Live setup (GitHub Pages)

1. Get a free TMDB API key — see below.
2. This repo's `index.html` **is** the site — push it and `README.md` to
   the repo root.
3. On the repo: **Settings → Pages → Build and deployment → Source:
   Deploy from a branch → Branch: `main`, folder: `/ (root)` → Save.**
4. GitHub builds it in a minute or two; refresh the Pages settings page
   for the live URL (`https://yourname.github.io/repo-name/`).
5. Open that URL, click **Settings** in the app, paste your TMDB key,
   and click **Test connection**.

That's it — no environment variables, no build step, no backend.
Note: with a public repo, the page is reachable by anyone with the
URL (not indexed or linked anywhere, just technically public) — fine
for a personal project, just don't expect real privacy from it.

## Cloud sync across devices (optional)

By default your diary lives only in this browser's `localStorage` — it
won't show up on a different device or browser. To see the same diary
everywhere, connect a free Supabase project:

1. Sign up free at [supabase.com](https://supabase.com) — no credit
   card required.
2. **New project** — pick any name, set a database password (you won't
   need it again for this), pick the region closest to you, and wait
   ~2 minutes while it provisions.
3. In the left sidebar, open the **SQL Editor**, paste the snippet
   below, and click **Run**:

   ```sql
   create table kv_store (
     key text primary key,
     value jsonb not null,
     updated_at timestamptz not null default now()
   );

   alter table kv_store enable row level security;

   create policy "allow anon read" on kv_store
     for select using (true);
   create policy "allow anon insert" on kv_store
     for insert with check (true);
   create policy "allow anon update" on kv_store
     for update using (true);
   ```

4. In the left sidebar: **Settings → API**. Copy the **Project URL**
   and the **anon / public** key (not the `service_role` key — that
   one's more privileged than this app needs).
5. In the app: **Settings → Cloud sync**, paste both values in, click
   **Connect**.
6. On any other device, open the same site, go to Settings → Cloud
   sync, paste the *same* two values, click **Connect** — it pulls
   down what's already there instead of overwriting it.

**Connect your main device — the one your existing entries are
already on — first.** The first device to connect a given Supabase
project uploads what's already logged there as the shared copy;
every device after that downloads that copy instead of overwriting it.

Two things worth knowing:
- These two Supabase values aren't secret in a strong sense — they sit
  in the page like the TMDB key does. Row Level Security is wide open
  (any request with the anon key can read/write), which is the
  necessary trade-off for a no-login single-user app. Don't share the
  URL somewhere public and expect the diary itself to stay private.
- Supabase's free tier pauses a project after 7 days with no activity.
  Nothing is lost — the next load just takes ~30 seconds longer while
  it wakes back up.

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

By default, everything — entries, ratings, reviews, lists, and your
Top 5 — is saved in this browser's `localStorage`, on whichever
device/browser you're using. Nothing is sent anywhere except to TMDB
(to search movies and fetch posters) and, once you paste it in, your
API key lives only in that same local storage.

If you set up **Cloud sync** (see below), entries/lists/Top 5 are also
mirrored to a small Postgres table in your own Supabase project, so
the same diary follows you across devices. `localStorage` still acts
as the offline cache either way — the app never depends on the network
being up to show you what you've already logged.

Without cloud sync, consequences worth knowing:
- Data is **per-browser**, not synced. Using the site from your phone
  and your laptop means two separate diaries.
- Clearing your browser's site data for this URL erases your diary.
- There's no login and no multi-user support — it's built for one
  person, with or without sync turned on.

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
