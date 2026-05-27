# SmartHire – AI-Powered Job Application Tracker

A full-stack MERN application to track job applications, visualize hiring pipeline progress, and generate AI-powered cover letters.

![Tech Stack](https://img.shields.io/badge/React-Vite-61DAFB)
![Tech Stack](https://img.shields.io/badge/Node-Express-339933)

## Features

- **Authentication** – Register, login, logout with JWT + bcrypt
- **Dashboard** – Stats cards and charts (pie + bar)
- **Job CRUD** – Full application management
- **Kanban Pipeline** – Drag & drop between Applied → Interviewed → Offered → Rejected
- **AI Cover Letters** – OpenAI (optional) or local Ollama generation with copy & download
- **Dark/Light Mode** – Theme toggle with persistence
- **Security** – Helmet, CORS, rate limiting, input validation

## Project Structure

```
SmartHire/
├── client/                 # React + Vite + Tailwind
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── context/
│   │   └── utils/
│   ├── netlify.toml
│   └── package.json
├── server/                 # Express API
│   ├── config/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   ├── routes/
│   ├── utils/
│   ├── tests/
│   └── server.js
├── render.yaml             # Render backend deploy
├── .github/workflows/      # GitHub Pages deploy
└── README.md
```

## Hosting Architecture

| Layer    | Platform      | Notes                                      |
|----------|---------------|--------------------------------------------|
| Frontend | **Netlify**   | Static React build, SPA redirects          |
| Frontend | **GitHub Pages** | Optional; subpath `/RepoName/`        |
| Backend  | **Render**    | Node.js API (Netlify does not run Express) |
| Storage  | **Local JSON file** | `server/storage/db.json` (no MongoDB required) |

> **Important:** Netlify and GitHub Pages only host the **frontend**. Your Express API must run on **Render** (or Railway, Fly.io, etc.) and the frontend calls it via `VITE_API_URL`.

## Quick Start (Local)

### Prerequisites

- Node.js 18+
- Ollama running locally (free alternative for cover letters)
  - `OLLAMA_URL` defaults to `http://localhost:11434`
  - `OLLAMA_MODEL` defaults to `llama3`
- Optional: `OPENAI_API_KEY` (if you prefer OpenAI instead)

### 1. Clone & install

```bash
cd SmartHire
npm run install:all
```

### 2. Backend setup

```bash
cd server
cp .env.example .env
```

Edit `server/.env`:

```env
NODE_ENV=development
PORT=5000
JWT_SECRET=your_super_secret_jwt_key_min_32_chars
JWT_EXPIRE=7d
CLIENT_URL=http://localhost:5173
OLLAMA_URL=http://localhost:11434
OLLAMA_MODEL=llama3
# OPENAI_API_KEY=sk-...   # optional
```

Start API:

```bash
npm run dev
```

### 3. Frontend setup

```bash
cd client
cp .env.example .env
```

Edit `client/.env`:

```env
VITE_API_URL=http://localhost:5000/api
```

Start UI:

```bash
npm run dev
```

Open **http://localhost:5173**

## API Documentation

Base URL: `http://localhost:5000/api` (production: your Render URL + `/api`)

### Auth

| Method | Endpoint            | Auth | Description        |
|--------|---------------------|------|--------------------|
| POST   | `/auth/register`    | No   | Create account     |
| POST   | `/auth/login`       | No   | Login, get JWT     |
| POST   | `/auth/logout`      | Yes  | Logout             |
| GET    | `/auth/me`          | Yes  | Current user       |
| PUT    | `/auth/profile`     | Yes  | Update name/email  |
| PUT    | `/auth/password`    | Yes  | Change password    |

### Jobs

| Method | Endpoint              | Auth | Description           |
|--------|-----------------------|------|-----------------------|
| GET    | `/jobs`               | Yes  | List (search, filter) |
| POST   | `/jobs`               | Yes  | Create application    |
| GET    | `/jobs/:id`           | Yes  | Get one               |
| PUT    | `/jobs/:id`           | Yes  | Update                |
| PATCH  | `/jobs/:id/status`    | Yes  | Update status only    |
| DELETE | `/jobs/:id`           | Yes  | Delete                |
| PATCH  | `/jobs/bulk-status`   | Yes  | Bulk status update    |

### Dashboard

| Method | Endpoint            | Auth | Description      |
|--------|---------------------|------|------------------|
| GET    | `/dashboard/stats`  | Yes  | Count by status  |
| GET    | `/dashboard/chart`  | Yes  | Chart data       |
| GET    | `/dashboard/recent` | Yes  | Recent jobs      |

### AI

| Method | Endpoint                    | Auth | Description           |
|--------|-----------------------------|------|-----------------------|
| POST   | `/ai/generate-cover-letter` | Yes  | Generate letter       |
| POST   | `/ai/improve-cover-letter`  | Yes  | Improve existing text |

### Health

| Method | Endpoint  | Auth | Description |
|--------|-----------|------|-------------|
| GET    | `/health` | No   | API status  |

**Auth header:** `Authorization: Bearer <token>`

See `server/tests/api-examples.http` for copy-paste requests.

## Testing

```bash
cd server
npm test
```

## Deploy Backend (Render)

1. Push repo to GitHub.
2. Go to [render.com](https://render.com) → **New Web Service**.
3. Connect repo; set **Root Directory** to `server`.
4. **Build Command:** `npm install`
5. **Start Command:** `npm start`
6. Add environment variables from `server/.env.example`.
7. Set `CLIENT_URL` to your frontend URL(s), comma-separated:
   ```
   https://your-app.netlify.app,https://username.github.io
   ```
8. Copy your Render URL, e.g. `https://smarthire-api.onrender.com`.

## Deploy Frontend (Netlify)

1. Netlify → **Add new site** → Import from Git.
2. **Base directory:** `client`
3. **Build command:** `npm run build`
4. **Publish directory:** `client/dist`
5. Environment variables:
   ```
   VITE_API_URL=https://smarthire-api.onrender.com/api
   ```
6. Deploy. `netlify.toml` includes SPA redirects.

## Deploy Frontend (GitHub Pages)

GitHub Pages hosts **static files only** (same limitation as Netlify for backend).

### Option A – GitHub Actions (included)

1. Repo → **Settings** → **Pages** → Source: **GitHub Actions**.
2. Add repository variable: **Settings → Secrets and variables → Actions → Variables**
   - Name: `VITE_API_URL`
   - Value: `https://your-render-api.onrender.com/api`
3. Push to `main`. Workflow `.github/workflows/deploy-gh-pages.yml` builds and deploys.
4. Site URL: `https://<username>.github.io/<repo-name>/`

If your repo is **not** named `SmartHire`, set `VITE_GH_REPO_NAME` in the workflow to match the repo name (used for Vite `base` path).

### Option B – Manual build

```bash
cd client
# Set API URL for production
echo VITE_API_URL=https://your-api.onrender.com/api > .env
echo VITE_GH_REPO_NAME=YourRepoName >> .env
npm run build:gh-pages
# Deploy dist/ to gh-pages branch or use Actions
```

### GitHub Pages checklist

- [ ] Render API live with CORS `CLIENT_URL` including `https://<user>.github.io`
- [ ] `VITE_API_URL` points to Render `/api`
- [ ] Repo name matches `VITE_GH_REPO_NAME` if not `SmartHire`
- [ ] Pages enabled with GitHub Actions source

## Environment Variables Summary

### Server (`server/.env`)

| Variable       | Required | Description                    |
|----------------|----------|--------------------------------|
| JWT_SECRET     | Yes      | Min 32 random characters       |
| OLLAMA_URL     | No       | Ollama base URL (free AI)     |
| OLLAMA_MODEL   | No       | Ollama model name             |
| OPENAI_API_KEY | No*      | Optional OpenAI API key      |
| CLIENT_URL     | Yes      | Frontend URL(s), comma-separated |
| PORT           | No       | Default 5000                   |

### Client (`client/.env`)

| Variable     | Required | Description              |
|--------------|----------|--------------------------|
| VITE_API_URL | Yes      | Backend API base URL     |

## Scripts

| Command              | Location | Description        |
|----------------------|----------|--------------------|
| `npm run dev`        | server   | API with watch     |
| `npm run dev`        | client   | Vite dev server    |
| `npm run build`      | client   | Production build   |
| `npm run build:gh-pages` | client | Build for GH Pages |
| `npm test`           | server   | Validation tests   |
