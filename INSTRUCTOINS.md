# TECH BLITZ — The Jane Protocol: Vercel Deployment & Hosting Guide

This guide provides a comprehensive, step-by-step walkthrough to deploy and host **TECH BLITZ: The Jane Protocol** on [Vercel](https://vercel.com) with [Supabase](https://supabase.com).

---

## 1. Project Architecture Overview

- **Frontend:** Single-file vanilla HTML/CSS/ES2022 (`tech-blitz-jane-protocol.html`). No build step or bundler needed.
- **Backend:** Node.js Serverless Functions in `/api` (optimized for Vercel's Hobby plan function limits using dynamic routing in `/api/admin/[action].js`).
- **Database & Realtime:** Supabase (PostgreSQL) for user authentication, live countdown sync, score caching, and fullscreen violation queue.
- **Routing & Security:** Handled by [`vercel.json`](file:///d:/Kalej/Events/TechBlitz/Website/Techblitz/vercel.json) with security headers (`X-Frame-Options: DENY`, `nosniff`) and route rewrites.

---

## 2. Prerequisites

Before starting, ensure you have:
1. A **GitHub account** with access to this repository.
2. A **[Vercel](https://vercel.com)** account.
3. A **[Supabase](https://supabase.com)** account (free tier is fully sufficient).
4. Node.js 18+ installed on your local machine (optional, for local testing).

---

## 3. Step 1: Set Up the Supabase Database

1. Log in to [Supabase Dashboard](https://supabase.com/dashboard) and click **"New Project"**.
2. Enter a project name (e.g. `tech-blitz`), choose a strong database password, and select your preferred region.
3. Wait 1–2 minutes for the database to provision.
4. Open the **SQL Editor** (icon on the left sidebar: `>_`).
5. Open the [`supabase/schema.sql`](file:///d:/Kalej/Events/TechBlitz/Website/Techblitz/supabase/schema.sql) file from this repository, copy its entire contents, paste it into the Supabase SQL editor, and click **Run**.
   
   This script creates:
   - `event_state` (Authoritative case clock, live timer state, and score reveal status)
   - `users` (Whitelist and roles; seeds default admin `Gowtham123`)
   - `participant_sessions` (Live participant scoring and blocked statuses)
   - `submissions` (Unique logged puzzle submissions)
   - `fullscreen_requests` (Proctoring and fullscreen violation queue)
   - Enables Realtime Replication for tables (`event_state`, `participant_sessions`, `fullscreen_requests`).

6. Obtain your API Credentials:
   - In Supabase, go to **Project Settings** (gear icon) → **API**.
   - Copy the following values:
     - **Project URL** (e.g., `https://xyzcompany.supabase.co`)
     - **`anon` `public` Key**
     - **`service_role` Secret Key** (Click *Reveal* — keep this private!)

---

## 4. Step 2: Deploy to Vercel

### Method A: Deploy via Vercel Web Dashboard (Recommended)

1. Log in to your [Vercel Dashboard](https://vercel.com).
2. Click **"Add New..."** → **"Project"**.
3. Under **"Import Git Repository"**, select **The Jane Protocol** repo.
4. Configure the project settings:
   - **Framework Preset:** Select **"Other"**
   - **Root Directory:** `./`
5. Expand the **"Build and Output Settings"**:
   - **Build Command:** Leave empty / default (no compilation needed).
   - **Output Directory:** Leave empty / default (`.`).
   - **Install Command:** Leave default (`npm install`).
6. Expand **"Environment Variables"** and add the following 4 variables:

   | Key | Value | Description |
   |---|---|---|
   | `SUPABASE_URL` | `https://your-project.supabase.co` | Your Supabase Project URL |
   | `SUPABASE_SERVICE_ROLE_KEY` | `ey...` | Your Supabase `service_role` secret key |
   | `SUPABASE_ANON_KEY` | `ey...` | Your Supabase `anon` public key |
   | `JWT_SECRET` | Any strong 32+ char secret string | Used to sign session tokens securely |

   *(Tip: You can generate a random JWT secret using terminal: `openssl rand -hex 32`)*

7. Click **"Deploy"**.
8. Wait ~30 seconds for the deployment to complete. Vercel will assign a live URL (e.g., `https://the-jane-protocol.vercel.app`).

---

### Method B: Deploy via Vercel CLI (Alternative)

If you prefer deploying directly from your terminal:

```bash
# 1. Install Vercel CLI globally
npm install -g vercel

# 2. Login to Vercel
vercel login

# 3. Link and configure project
vercel

# 4. Set environment variables
vercel env add SUPABASE_URL
vercel env add SUPABASE_SERVICE_ROLE_KEY
vercel env add SUPABASE_ANON_KEY
vercel env add JWT_SECRET

# 5. Deploy to production
vercel --prod
```

---

## 5. Step 3: Verification & Testing

Once deployed, run through these quick checks to ensure full functionality:

### 1. Test Serverless Clock Sync
Navigate in your browser to:
```
https://<your-vercel-domain>.vercel.app/api/clock/sync
```
You should receive a JSON response:
```json
{"server_time": 1791908994846}
```

### 2. Test Administrator Access
1. Visit `https://<your-vercel-domain>.vercel.app`
2. In the participant entry / login dialog, enter the administrator ID:
   ```
   Gowtham123
   ```
3. Verify access to the **Admin Dashboard**:
   - Check the live participant roster.
   - Test starting the authoritative event timer.
   - Verify controls: Add User, Remove User, Reveal Scores, and Unblock Participants.

### 3. Test Participant Flow
1. Open an Incognito window and visit the site.
2. Enter a test participant ID (e.g. `21BCE1001` or any ID added via the Admin console).
3. Test accessing the Briefing (`#/briefing`), Evidence (`#/evidence`), Suspects (`#/suspects`), and Detective Puzzle Workspaces (`#/challenge/cho-a`).
4. Submit an answer to verify submission recording and score calculation.

---

## 6. Step 4: Running Locally

If you need to test or modify code locally:

```bash
# 1. Install dependencies
npm install

# 2. Create your .env file
copy .env.example .env
# Edit .env with your Supabase credentials

# 3. Start local development server
npm start
```
The server will be available at `http://localhost:8080`.
