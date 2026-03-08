# 💍 Wedding Camera — Memory Wall

A real-time wedding photo wall. Guests scan a QR code → take a live photo with their phone camera → add a caption → it appears instantly on the shared gallery.

---

## Features

- 📸 **Camera-only capture** — no file picker, no gallery uploads. Live camera only.
- 🔄 **Front / back camera toggle**
- ✍️ **Name + caption** per photo
- 🖼️ **Masonry gallery** with lightbox viewer
- 🔁 **Auto-refreshes** every 8 seconds on the main wall
- ✨ Elegant wedding-themed UI

---

## Tech Stack

- **Next.js 14** (Pages Router)
- **Supabase** (free tier) — Postgres DB + image storage
- **Vercel** — deployment

---

## Setup Guide

### 1. Create a Supabase Project

1. Go to [supabase.com](https://supabase.com) → Create a free account
2. Create a new project (note the password)
3. Wait for the project to initialize (~1 min)

### 2. Set Up the Database Table

In your Supabase project, go to **SQL Editor** and run:

```sql
create table photos (
  id uuid primary key default gen_random_uuid(),
  image_url text not null,
  caption text default '',
  name text default 'Anonymous Guest',
  created_at timestamptz default now()
);

-- Allow anyone to read and insert
alter table photos enable row level security;

create policy "Anyone can read photos"
  on photos for select
  using (true);

create policy "Anyone can insert photos"
  on photos for insert
  with check (true);
```

### 3. Set Up Supabase Storage

1. In Supabase, go to **Storage** → **New Bucket**
2. Name it exactly: `wedding-photos`
3. Set it to **Public**
4. Go to **Storage → Policies** and add:
   - **SELECT policy**: `true` (allow all reads)
   - **INSERT policy**: `true` (allow all uploads)

Or run this SQL:

```sql
-- Storage policies for wedding-photos bucket
insert into storage.buckets (id, name, public) 
values ('wedding-photos', 'wedding-photos', true)
on conflict do nothing;

create policy "Public read wedding photos"
  on storage.objects for select
  using (bucket_id = 'wedding-photos');

create policy "Public upload wedding photos"
  on storage.objects for insert
  with check (bucket_id = 'wedding-photos');
```

### 4. Get Your Supabase Credentials

Go to **Settings → API** in your Supabase project:
- Copy the **Project URL**
- Copy the **anon / public key**

### 5. Deploy to Vercel

1. Push this project to a GitHub repo
2. Go to [vercel.com](https://vercel.com) → **New Project** → import your repo
3. Under **Environment Variables**, add:

| Name | Value |
|------|-------|
| `NEXT_PUBLIC_SUPABASE_URL` | `https://your-project.supabase.co` |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | `your-anon-key` |

4. Click **Deploy** 🚀

### 6. Create the QR Code

After deploying, your capture page will be at:
```
https://your-app.vercel.app/capture
```

Generate a QR code pointing to that URL using any free QR generator (e.g. [qr-code-generator.com](https://www.qr-code-generator.com)).

Print it and place it on tables at the wedding!

---

## Local Development

```bash
npm install
```

Create `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

```bash
npm run dev
```

Visit http://localhost:3000

---

## Pages

| URL | Description |
|-----|-------------|
| `/` | Main gallery wall — shows all photos, auto-refreshes |
| `/capture` | Camera page — QR code should link here |

---

## Notes

- The camera constraint uses `facingMode: environment` (back camera) by default on phones
- Only live camera capture is allowed — no file input anywhere in the UI
- Images are stored in Supabase Storage (free tier: 1 GB)
- Database rows are stored in Supabase Postgres (free tier: 500 MB)
