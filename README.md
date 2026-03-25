# Scout Challenge — MVP Setup Guide

## Project Structure
```
scout-challenge/
├── index.html          # Main app
├── css/main.css        # All styles
├── js/
│   ├── config.js       # ⚙️ YOUR CONFIG HERE
│   ├── db.js           # Supabase database layer
│   └── app.js          # App logic
└── vercel.json         # Deployment config
```

## Step 1 — Supabase Setup (free)

1. Go to https://supabase.com → Create account → New project
2. Go to **Settings → API** → Copy your:
   - Project URL
   - anon/public key
3. Open `js/config.js` and replace:
   ```js
   const SUPABASE_URL = 'https://xxxx.supabase.co';
   const SUPABASE_ANON_KEY = 'eyJxxx...';
   ```

## Step 2 — Create Supabase Tables

In your Supabase project → **SQL Editor** → Run this:

```sql
-- Users table
create table users (
  id uuid default gen_random_uuid() primary key,
  pseudo text not null,
  email text unique not null,
  created_at timestamp default now()
);

-- Players table
create table players (
  id text primary key,
  nom text not null,
  club text,
  championnat text,
  poste text,
  date_naissance date,
  nationalite text,
  image_url text,
  actif text default 'OUI'
);

-- Selections table
create table selections (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references users(id),
  gameweek integer not null,
  captain_id text references players(id),
  starter1_id text references players(id),
  starter2_id text references players(id),
  starter3_id text references players(id),
  sub_id text references players(id),
  locked boolean default false,
  updated_at timestamp default now(),
  unique(user_id, gameweek)
);

-- Scores table
create table scores (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references users(id),
  gameweek integer not null,
  gw_pts integer default 0,
  total_pts integer default 0,
  flair_bonus integer default 0
);

-- Enable Row Level Security
alter table users enable row level security;
alter table players enable row level security;
alter table selections enable row level security;
alter table scores enable row level security;

-- Allow public read on players
create policy "Players are public" on players for select using (true);

-- Allow public read on scores
create policy "Scores are public" on scores for select using (true);
```

## Step 3 — Import Your Players

1. Export your `Scout_Challenge_DB_v2.xlsx` → Joueurs sheet → as CSV
2. In Supabase → **Table Editor → players → Import CSV**

## Step 4 — Deploy on Vercel (free)

1. Go to https://vercel.com → Create account
2. Install Vercel CLI: `npm i -g vercel`
3. In the project folder: `vercel`
4. Follow the prompts → your app is live!

Or drag & drop the folder on https://vercel.com/new

## Step 5 — Each GameWeek

1. Update `js/config.js`:
   - `CURRENT_GW` → increment by 1
   - `GW_DEADLINE` → next Friday 14:00
2. Fill in `Stats GW` sheet in your Excel file
3. Calculate scores and update the `scores` table in Supabase

## Notes

- Without Supabase configured, the app runs in **demo mode** with sample data
- The app works as a PWA — users can add it to their home screen
- Add `<link rel="manifest" href="manifest.json">` later for full PWA support
