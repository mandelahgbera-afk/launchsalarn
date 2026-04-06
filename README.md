# Salarn — Crypto Trading Platform

A full-featured crypto investment platform with copy trading, portfolio management, and superadmin controls. Built with React + Vite + Supabase.

---

## Stack

- **Frontend**: React 18, Vite 5, Tailwind CSS v4, Framer Motion
- **Auth & DB**: Supabase (PKCE auth flow, PostgreSQL + RLS)
- **Deployment**: Vercel (static SPA)

---

## Deploy to Vercel — Complete Guide

### Step 1: Set Up Supabase

1. Create a new project at [supabase.com](https://supabase.com)
2. Go to **SQL Editor** → paste and run the entire `SUPABASE_SCHEMA.sql` file
   - This creates all tables, RLS policies, triggers, and indexes
   - Safe to re-run — everything is idempotent
3. Go to **Authentication → URL Configuration**:
   - **Site URL**: your production domain e.g. `https://salarn.vercel.app`
   - **Redirect URLs**: add `https://salarn.vercel.app/auth/callback`

### Step 2: Configure Email (Critical)

**Option A — Use Supabase built-in email (easiest, for testing):**
- No SMTP setup needed. Works out of the box.
- Rate-limited to ~4 emails/hour in free tier.

**Option B — Custom SMTP (Resend, SendGrid, Mailgun):**
1. Go to **Auth → SMTP Settings** in Supabase
2. Enter your SMTP credentials
3. Click **"Send Test Email"** — confirm it works before deploying
4. If the test email fails: `error {}` on signup is an SMTP delivery failure, not a code bug

**Custom email templates:** After saving templates, always test with a fresh signup. The `error {}` bug on second signup = your SMTP is failing to send the email.

### Step 3: Get Environment Variables

Go to **Supabase → Project Settings → API**:

| Variable | Where to find it |
|---|---|
| `VITE_SUPABASE_URL` | Project URL (`https://xxxx.supabase.co`) |
| `VITE_SUPABASE_ANON_KEY` | `anon` public key |
| `VITE_APP_URL` | Your Vercel domain (e.g. `https://salarn.vercel.app`) |

### Step 4: Deploy to Vercel

1. Push your project to GitHub (include: `src/`, `public/`, `index.html`, `package.json`, `vite.config.ts`, `tsconfig.json`, `components.json`, `vercel.json`)
2. Import the repo on [vercel.com](https://vercel.com)
3. Framework: **Vite** (auto-detected)
4. Go to **Project Settings → Environment Variables** → add all 3 variables from Step 3
5. Click **Redeploy** — env vars only apply after a fresh deploy

### Step 5: Set Admin Role

After your first signup:
1. Go to **Supabase → Table Editor → users**
2. Find your user row
3. Change `role` from `user` to `admin`
4. Sign out and sign back in — you'll be redirected to `/admin`

---

## Common Issues

| Error | Cause | Fix |
|---|---|---|
| `error {}` on signup | SMTP delivery failing | Check Supabase Auth → SMTP → send test email |
| Blank page after deploy | Missing env vars | Add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` to Vercel |
| Auth callback loop | Wrong redirect URL | Ensure Site URL + redirect URL match your Vercel domain exactly |
| Admin route → dashboard | Role not set | Set `role = 'admin'` in Supabase users table |
| 404 on page refresh | Missing SPA rewrite | `vercel.json` must have the rewrite rule (already included) |

---

## Architecture

```
src/
  lib/
    auth.tsx         — AuthContext, PKCE sign-in/up, race-condition safe
    supabase.ts      — Supabase client + TypeScript types
    api.ts           — Typed API layer (balances, transactions, portfolio, etc.)
    marketData.ts    — CoinGecko live market data
  pages/
    Auth.tsx         — Sign in / Sign up / Forgot password
    AuthCallback.tsx — PKCE code exchange + role-based redirect
    Dashboard.tsx    — User dashboard with live market data
    Portfolio.tsx    — Holdings overview
    Trade.tsx        — Buy / sell crypto
    CopyTrading.tsx  — Follow top traders
    Transactions.tsx — Deposit / withdrawal history
    Settings.tsx     — Profile settings
    admin/
      AdminDashboard.tsx    — Platform overview
      AdminTransactions.tsx — Approve/reject all transactions
      ManageUsers.tsx       — User management
      ManageCryptos.tsx     — Crypto price management
      ManageTraders.tsx     — Copy trader management
      PlatformSettings.tsx  — Global platform config
  components/
    AdminRoute.tsx    — Admin guard (role check)
    ProtectedRoute.tsx — Auth guard
    layout/           — Sidebar, BottomNav, AppLayout
    ui/               — shadcn/ui components
SUPABASE_SCHEMA.sql   — Complete idempotent schema (run once in SQL Editor)
```

---

## Fixed Bugs (v5)

- **Dashboard stale closure**: market interval now uses a `useRef` mirror of `selectedCoin` — coin selection is never reset by the 60s refresh
- **AdminTransactions TypeScript error**: removed duplicate `email` variable declaration
- **signIn balance guarantee**: ensures `user_balances` row exists on every sign-in (prevents missing balance edge case)
- **vite.config**: added `allowedHosts: true` for preview server compatibility
