# Margin Monitor — Deployment Guide
Cross-device EL/MM · EL/NLV daily monitor with cloud-synced log.

---

## Step 1 — Create a Supabase project

1. Go to https://supabase.com and sign up (free, no credit card)
2. Click **New project**, give it a name (e.g. `margin-monitor`), set a database password, choose a region close to Singapore (e.g. Southeast Asia)
3. Wait ~2 minutes for the project to provision
4. Go to **SQL Editor** (left sidebar) and run this query:

```sql
create table margin_log (
  id bigint generated always as identity primary key,
  ts text not null unique,
  raw_nlv numeric,
  raw_el  numeric,
  raw_mm  numeric,
  elmm    text,
  elnlv   text,
  c1      text,
  c2      text,
  overall text,
  created_at timestamptz default now()
);
alter table margin_log enable row level security;
create policy "allow all" on margin_log
  for all using (true) with check (true);
```

5. Go to **Project Settings → API**
   - Copy **Project URL** (looks like `https://xxxx.supabase.co`)
   - Copy **anon public** key (long string starting with `eyJ...`)

---

## Step 2 — Configure index.html

Open `index.html` and find the config block near the top:

```javascript
const SUPABASE_URL  = 'YOUR_SUPABASE_URL';
const SUPABASE_KEY  = 'YOUR_SUPABASE_ANON_KEY';
const ACCESS_PIN    = 'YOUR_PIN';
```

Replace with your actual values:
```javascript
const SUPABASE_URL  = 'https://xxxx.supabase.co';
const SUPABASE_KEY  = 'eyJhbGc...your-full-anon-key...';
const ACCESS_PIN    = '2025';   // choose any PIN you will remember
```

Save the file.

---

## Step 3 — Deploy to Vercel

### Option A — Vercel CLI (fastest)
```bash
npm install -g vercel
cd margin-monitor
vercel --prod
```
Follow the prompts. Done. You get a URL like `https://margin-monitor-xxxx.vercel.app`

### Option B — GitHub + Vercel dashboard
1. Push this folder to a GitHub repository
2. Go to https://vercel.com, sign up with GitHub
3. Click **Add New Project → Import Git Repository**
4. Select your repo, click **Deploy**
5. Done. Vercel auto-deploys on every push.

---

## Step 4 — Bookmark the URL

Open the Vercel URL on any device. Enter your PIN. Log entries are saved to Supabase and appear instantly on any other device using the same URL.

---

## Usage

- Enter EL, MM, NLV from IBKR each trading day
- Click **Log today's reading** — saved to cloud immediately
- **Export CSV** — downloads full history as a spreadsheet
- **Clear log** — permanently deletes all entries from database

## Security note

The anon key is visible in the HTML source. For a personal tool this is acceptable — Supabase's Row Level Security limits what the key can do. Do not share the URL publicly if you want to keep your financial data private. For stronger security, add Supabase Auth email/password login in a future version.
