# Shivmudra Pathak — Installation & Go-Live Guide

Complete steps to install and run the **Senior/Junior Attendance Portal** on any laptop (Windows, Mac, or Linux).

---

## What you need

| Tool | Version | Download |
|------|---------|----------|
| **Node.js** | 18+ (20 LTS recommended) | https://nodejs.org |
| **npm** | Comes with Node | — |
| **Git** | Any recent version | https://git-scm.com |
| **Python** | 3.10+ (optional, for setup scripts) | https://python.org |
| **Supabase account** | Free tier | https://supabase.com |
| **Code editor** | VS Code / Cursor | — |

---

## 1. Get the project on your laptop

```bash
# Clone (or copy the shivmudra-pathak folder)
git clone <your-repo-url>
cd shivmudra-pathak

# Install dependencies
npm install
```

---

## 2. Supabase — database & auth (one-time)

### 2.1 Create project

1. Go to [supabase.com](https://supabase.com) → **New project**
2. Choose a name (e.g. `shivmudra-pathak`)
3. Set a database password and region (closest to India)
4. Wait for the project to finish provisioning

### 2.2 Run database schema

Open **SQL Editor** in Supabase and run these files **in order**:

| Order | File | Purpose |
|-------|------|---------|
| 1 | `supabase/schema.sql` | Tables, roles, RLS (new projects) |
| 2 | `supabase/migrations/001_registration_approval.sql` | Registration approval flow |
| 3 | `supabase/migrations/002_idempotent_bootstrap.sql` | Safe re-run helpers |
| 4 | `supabase/migrations/003_member_photos_storage.sql` | Photo storage policies |
| 5 | `supabase/migrations/004_blood_group_member_category.sql` | Senior/Junior + blood group |

> **Existing project?** Skip `schema.sql` if tables already exist; run only migrations you have not applied yet.

### 2.3 Create photo storage bucket

1. Supabase → **Storage** → **New bucket**
2. Name: `Shivmudra_attendance_bucket_2026` (or your chosen name)
3. Enable **Public bucket** (for member photo URLs)
4. Allowed MIME types: `image/jpeg`, `image/png`, `image/webp`

### 2.4 Copy API keys

Supabase → **Project Settings** → **API**:

- **Project URL** → `NEXT_PUBLIC_SUPABASE_URL`
- **anon public** key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- **service_role** key → `SUPABASE_SERVICE_ROLE_KEY` (keep secret)

### 2.5 Enable Google Sign-In (verified Gmail)

Members can sign in with **Google** so the email is verified by Google.

1. **Google Cloud Console** — https://console.cloud.google.com
   - Create a project (or use existing)
   - **APIs & Services** → **OAuth consent screen** → configure (External, add your email)
   - **Credentials** → **Create Credentials** → **OAuth client ID** → **Web application**
   - **Authorized redirect URI** (add exactly):
     ```
     https://YOUR_PROJECT_REF.supabase.co/auth/v1/callback
     ```
     Find `YOUR_PROJECT_REF` in Supabase URL (e.g. `ixywmygrxyfnffiqyobl`)
   - Copy **Client ID** and **Client Secret**

2. **Supabase Dashboard** → **Authentication** → **Providers** → **Google**
   - Enable Google
   - Paste Client ID and Client Secret → **Save**

3. **Supabase** → **Authentication** → **URL Configuration**
   - **Site URL**: `http://localhost:3000` (local) or your Vercel URL (production)
   - **Redirect URLs** — add:
     ```
     http://localhost:3000/auth/callback
     https://your-app.vercel.app/auth/callback
     ```

> Only **@gmail.com** accounts are accepted. **Registration** and **login** use email + password (no Google sign-in buttons).

---

## 3. Environment file

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

SUPABASE_PHOTOS_BUCKET=Shivmudra_attendance_bucket_2026

CRON_SECRET=change-me-to-a-long-random-string
BOOTSTRAP_ADMIN_EMAIL=omkargavkhadkar@gmail.com

# Optional — approval emails via https://resend.com
RESEND_API_KEY=re_xxxxxxxx
FROM_EMAIL=Shivmudra Pathak <noreply@yourdomain.com>

# Set after deploy (production URL)
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

> Never commit `.env.local` to Git. It contains secrets.

---

## 4. Run locally

```bash
npm run dev
```

Open: **http://localhost:3000**

Corporate network SSL issues:

```bash
npm run dev:corp
```

---

## 5. First-time admin setup

| Step | Action |
|------|--------|
| 1 | Open `/login` → **Sign Up** with `BOOTSTRAP_ADMIN_EMAIL` |
| 2 | First login with that email gets **admin** role automatically |
| 3 | Go to **Admin → Geo Fence** → click your **sarav ground** on the map → **Save** |
| 4 | Go to **Admin → Registrations** → **Approve** pending members |

### Portal URLs

| Portal | URL |
|--------|-----|
| Senior register | `/register/senior` |
| Junior register | `/register/junior` |
| Senior sign in | `/login/senior` |
| Junior sign in | `/login/junior` |
| Senior attendance | `/user/senior` |
| Junior attendance | `/user/junior` |
| Senior admin | `/admin/senior` |
| Junior admin | `/admin/junior` |
| Geo-fence | `/admin/geo-fence` |
| Registrations | `/admin/registrations` |

---

## 6. Go-live checklist (production ready)

Run this automated setup (approves pending members, verifies geo-fence):

```bash
python scripts/go_live_setup.py
```

Manual checks:

- [ ] **Time window** — Attendance only **8:30–9:30 PM IST** (enabled in production API)
- [ ] **Geo-fence** — Admin set exact sarav ground pin on map (150 m radius)
- [ ] **Registrations** — All real members approved; test accounts removed if needed
- [ ] **Photos** — Registration uploads work (bucket name matches `.env.local`)
- [ ] **GPS** — Test on a **phone** with location allowed (HTTPS required in production)

---

## 7. Deploy to Vercel (recommended hosting)

### 7.1 Push to GitHub

```bash
git add .
git commit -m "Shivmudra Pathak ready for deploy"
git push origin main
```

### 7.2 Import on Vercel

1. [vercel.com](https://vercel.com) → **Add New Project** → import repo
2. Framework: **Next.js** (auto-detected)
3. Add all variables from `.env.local` in **Environment Variables**
4. Set `NEXT_PUBLIC_APP_URL` to `https://your-app.vercel.app`
5. **Deploy**

### 7.3 Configure Supabase for production

Supabase → **Authentication** → **URL Configuration**:

| Setting | Value |
|---------|-------|
| Site URL | `https://your-app.vercel.app` |
| Redirect URLs | `https://your-app.vercel.app/**` |

### 7.4 Daily Excel report (automatic)

`vercel.json` already configures cron at **9:30 PM IST** (`30 16 * * *` UTC).

Ensure `CRON_SECRET` is set in Vercel env vars.

---

## 8. How attendance works (for members)

1. Member opens `/user/senior` or `/user/junior` on their **phone**
2. Browser asks for **location permission** → tap **Allow**
3. During **8:30–9:30 PM IST**, tap **उपस्थिती नोंदवा**
4. App checks GPS is within **150 m** of admin geo-fence
5. Attendance saved for today

---

## 9. Troubleshooting

| Problem | Fix |
|---------|-----|
| `Location permission denied` | Browser site settings → Location → Allow; enable Windows/Mac location services |
| `Practice ground not configured` | Admin → Geo Fence → save map pin |
| `You are Xm away` | Move closer to sarav ground or admin widens radius (max 300 m) |
| `Attendance only 8:30–9:30 PM` | Normal outside window; wait for practice time |
| `Registration pending` | Admin → Registrations → Approve |
| Photo upload fails | Check `SUPABASE_PHOTOS_BUCKET` matches Supabase Storage bucket name |
| Supabase connection error | Verify URL/keys in `.env.local`; restart `npm run dev` |
| White logo box | Hard refresh: `Ctrl+Shift+R` |
| Google login blank white page | Google provider not set up in Supabase — see **§2.5** below |
| Google `redirect_uri_mismatch` | Google Cloud redirect URI must be `https://YOUR_REF.supabase.co/auth/v1/callback` |
| Google login returns to login with `error=auth` | Add `http://localhost:3000/auth/callback` in Supabase **Redirect URLs** |

### Google login shows blank page with JSON

If the browser stops on `supabase.co/auth/v1/authorize?provider=google` with a **white page** and small JSON text, Supabase rejected the request **before** Google opens. Fix in order:

1. **Supabase** → Authentication → **Providers** → **Google** → toggle **ON**
2. Paste **Google Client ID** and **Client Secret** (from Google Cloud Console)
3. **Supabase** → Authentication → **URL Configuration** → **Redirect URLs** → add:
   ```
   http://localhost:3000/auth/callback
   ```
4. **Google Cloud** → Credentials → OAuth client → **Authorized redirect URIs**:
   ```
   https://ixywmygrxyfnffiqyobl.supabase.co/auth/v1/callback
   ```
5. Save all, wait 1 minute, hard refresh login page and try again.

---

## 10. Project structure

```
shivmudra-pathak/
  src/app/
    admin/           Admin dashboards (senior/junior, geo-fence, registrations)
    user/            Member attendance portals
    register/        Self-registration (senior/junior)
    login/           Sign-in pages
    api/             REST APIs (attendance, registration, reports)
  supabase/
    schema.sql       Full database schema
    migrations/      Incremental SQL updates
  public/branding/   Logo and group photo
  scripts/
    go_live_setup.py Approve pending + verify geo-fence
    test_core.py     Quick logic tests (no server needed)
```

---

## 11. Quick reference — commands

```bash
npm install              # Install dependencies
npm run dev              # Start dev server (localhost:3000)
npm run build            # Production build test
npm run start            # Run production build locally
python scripts/go_live_setup.py   # Go-live: approve pending, check geo-fence
python scripts/test_core.py       # Run core logic tests
npx vercel               # Deploy to Vercel
```

---

## Support

**Digital COE Gen AI Team** — Shivmudra Pathak Attendance Portal

Bootstrap admin: `omkargavkhadkar@gmail.com`
