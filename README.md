# Yas-Links

A full-stack SaaS marketing link management platform — campaigns, branded short links (`yas-link.to/{slug}`), UTM parameters, QR codes, analytics, and admin team management.

---

## Folder Structure

```
yas-links/
├── frontend/          # React 18 + Vite + Tailwind CSS → deploy to Vercel
│   ├── src/
│   │   ├── components/   # UI components (shadcn/ui + custom)
│   │   ├── hooks/        # React hooks (auth, toast, etc.)
│   │   ├── lib/
│   │   │   └── api-client/  # Generated API client (types + fetch helpers)
│   │   ├── pages/        # Dashboard, Campaigns, Links, Settings, Login
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
│
├── backend/           # Express 5 + PostgreSQL API → deploy to Railway/Render
│   ├── src/
│   │   ├── db/           # Drizzle ORM schema + PostgreSQL client
│   │   │   └── schema/   # users, campaigns, links, clicks tables
│   │   ├── middleware/   # requireAuth, admin guards
│   │   ├── routes/       # auth, campaigns, links, analytics, reports, admin
│   │   ├── validators/   # Zod request/response validators
│   │   └── lib/          # Utility helpers (slug generator, etc.)
│   ├── drizzle.config.ts
│   ├── package.json
│   └── tsconfig.json
│
├── vercel.json        # Vercel deployment config (frontend)
├── .env.example       # All required environment variables (no real values)
├── .gitignore
└── README.md
```

---

## Environment Variables

Copy `.env.example` and fill in your values:

```bash
cp .env.example .env
```

| Variable | Where used | Description |
|---|---|---|
| `DATABASE_URL` | Backend | PostgreSQL connection string |
| `SESSION_SECRET` | Backend | Random secret for signing session cookies |
| `PORT` | Backend | Port the Express server listens on (default 8080) |
| `NODE_ENV` | Backend | Set to `production` when deployed |
| `VITE_API_URL` | Frontend (build) | Full URL of your deployed backend |

> **Never commit `.env` files.** `.gitignore` already excludes them.

---

## Local Development

### Prerequisites
- Node.js 20+
- A PostgreSQL database ([Neon](https://neon.tech) free tier works great)

### 1. Set up the database schema

```bash
cd backend
npm install
cp ../.env.example .env   # Fill in DATABASE_URL
npm run db:push
```

### 2. Start the backend

```bash
cd backend
npm run dev               # Starts Express on http://localhost:8080
```

### 3. Start the frontend

```bash
cd frontend
npm install
npm run dev               # Starts Vite on http://localhost:5173
                          # /api requests are proxied to :8080 automatically
```

Open http://localhost:5173 in your browser.

### First admin account

After setting up the database, create your first user account (registration is disabled from the UI), then run:

```sql
UPDATE users SET is_admin = true WHERE email = 'your@email.com';
```

---

## Deploying to Vercel + Railway

### Step 1 — Deploy the backend on Railway

1. Create a new project at [railway.app](https://railway.app)
2. Connect your GitHub repo and select the `backend/` folder as the root
3. Add a PostgreSQL database plugin
4. Set these environment variables in Railway:
   - `DATABASE_URL` (Railway auto-fills this if you use their plugin)
   - `SESSION_SECRET` (generate: `openssl rand -hex 32`)
   - `PORT=8080`
   - `NODE_ENV=production`
5. Deploy — Railway will run `npm run build && npm start`
6. Copy your Railway backend URL (e.g. `https://yas-links-api.railway.app`)

### Step 2 — Deploy the frontend on Vercel

1. Push your project to a GitHub repository
2. Go to [vercel.com](https://vercel.com) → New Project → Import your repo
3. Set the **Root Directory** to `frontend/`
4. Framework preset: **Vite**
5. Add this environment variable in Vercel:
   - `VITE_API_URL` = `https://your-backend.railway.app` (from Step 1)
6. Deploy

The `vercel.json` at the project root handles SPA routing (all paths → `index.html`).

### Step 3 — Run database migrations on Railway

In your Railway project, open the terminal and run:

```bash
npm run db:push
```

### Step 4 — Set up the first admin

In Railway's PostgreSQL query tool:

```sql
UPDATE users SET is_admin = true WHERE email = 'your@email.com';
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, Vite 6, Tailwind CSS v4, shadcn/ui, TanStack Query, Recharts, Framer Motion |
| Backend | Express 5, TypeScript, express-session (PostgreSQL-backed) |
| Database | PostgreSQL + Drizzle ORM |
| Auth | Email/password + bcrypt, server-side sessions |
| QR Codes | `qrcode` library, error correction level H |

## Features

- **Campaigns** — organise links into marketing campaigns
- **Short Links** — branded short URLs (`yas-link.to/{slug}`) with copy & QR
- **UTM Parameters** — automatically append UTM tags to destination URLs
- **QR Codes** — download QR with logo overlay, error correction level H
- **Analytics** — total and unique click tracking, charts over time
- **CSV Reports** — per-campaign reports with date range filtering
- **Team Management** — admin-only: create/delete team member accounts
- **Access Control** — public registration disabled; admin creates accounts
