# Shivmudra Dhol Tasha Pathak — Attendance Portal

Practice attendance management for **Shivmudra Dhol Tasha Pathak** with geo-fenced check-in, role-based access, and daily Excel reports.

## Features

| Role | Capabilities |
|------|----------------|
| **Admin** | Register members (auto-approved), **review self-registrations**, set geo-fence, promote leaders, approve removals, download Excel |
| **Leader** | View today's attendance, raise removal requests for irregular members |
| **User** | **Self-register** → wait for approval → mark present daily **8:30–9:30 PM IST** within geo-fence |

### Daily Excel (5 sheets)
1. Global Attendance  
2. Dhol Attendance  
3. Tasha Attendance  
4. Dhawaj Attendance  
5. Defaulters (>5 consecutive absent days)

Auto-generated at **9:30 PM IST** via Vercel Cron.

### Registration & approval (Phase 1)
1. New members open **`/register`** — name, phone, Gmail, address, instrument, photo, password
2. Account is created with **`approval_status: pending`** — can sign in but **cannot mark attendance**
3. Admin reviews at **Admin → Registrations** — Approve or Reject
4. On approve: member becomes active + **approval email** sent (Resend API)

---

## Setup (15 minutes)

### 1. Supabase

1. Create project at [supabase.com](https://supabase.com)
2. **SQL Editor** → run `supabase/schema.sql` (new projects)  
   **Existing DB:** also run `supabase/migrations/001_registration_approval.sql`
3. **Storage** → create bucket `member-photos` (public read)
4. Copy **Project URL**, **anon key**, **service role key**

### 2. Environment

```bash
cp .env.example .env.local
```

Fill in:
```
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...
CRON_SECRET=your-random-secret
BOOTSTRAP_ADMIN_EMAIL=omkargavkhadkar@gmail.com
RESEND_API_KEY=re_xxxxxxxx
FROM_EMAIL=Shivmudra Pathak <noreply@yourdomain.com>
```

### 3. Install & run

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

### 4. First admin login

1. Go to `/login` → **Sign Up** with `omkargavkhadkar@gmail.com`
2. Trigger assigns **admin** role automatically (see `schema.sql`)
3. Set geo-fence at **Admin → Geo Fence** (click map on your sarav ground)
4. Register members at **Admin → Register**

### 5. Deploy to Vercel

```bash
npx vercel
```

Add env vars in Vercel dashboard. Cron runs at `30 16 * * *` UTC (= 9:30 PM IST).

---

## Project structure

```
src/app/
  admin/          Admin portal
  leader/         Leader portal
  user/           Mark attendance (PWA)
  api/            REST endpoints
supabase/
  schema.sql      Database + RLS
```

---

## Bootstrap admin

Email `omkargavkhadkar@gmail.com` is configured as bootstrap admin in `supabase/schema.sql`. First signup with this email gets `admin` role.

---

## Support

Digital COE Gen AI Team — Shivmudra Pathak MVP v0.1
