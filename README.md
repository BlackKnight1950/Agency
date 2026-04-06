# ARC Meeting Review App

Stakeholder decision interface for the **ARC Meeting Review Log** in Smartsheet.  
Fetches live pending requests, lets reviewers enter decisions and comments, and writes them back to Smartsheet — all through a secure Express backend.

---

## Architecture

```
Browser (public/index.html)
       │  fetch /api/report/:id          fetch /api/sheet/:id/row/:id
       ▼                                          ▼
  Express server (server.js)  ──────►  Smartsheet REST API
       │
  SMARTSHEET_TOKEN (env var, never in browser)
```

Your Smartsheet token lives **only** on the server. The browser never sees it.

---

## Local development

### 1. Clone and install

```bash
git clone https://github.com/YOUR_USERNAME/arc-meeting-review-app.git
cd arc-meeting-review-app
npm install
```

### 2. Create your `.env` file

```bash
cp .env.example .env
```

Open `.env` and paste your Smartsheet API token:

```
SMARTSHEET_TOKEN=your_actual_token_here
```

> Generate a token at **Smartsheet → Account → Personal Settings → API Access**.  
> Keep this file local — it is in `.gitignore` and will never be committed.

### 3. Run the app

```bash
npm run dev       # uses nodemon — auto-restarts on file changes
# or
npm start         # plain node
```

Open [http://localhost:3000](http://localhost:3000).

---

## Deploy to GitHub + Render

### Step 1 — Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/arc-meeting-review-app.git
git push -u origin main
```

### Step 2 — Create a Render Web Service

1. Go to [render.com](https://render.com) → **New → Web Service**
2. Connect your GitHub repo (`arc-meeting-review-app`)
3. Render auto-detects `render.yaml` — confirm the settings:
   - **Environment:** Node
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
4. Under **Environment Variables**, add:
   | Key | Value |
   |-----|-------|
   | `SMARTSHEET_TOKEN` | *(paste your token here — never in code)* |
5. Click **Create Web Service**

Render assigns a URL like `https://arc-meeting-review-app.onrender.com`.

### Step 3 — (Optional) Lock down CORS

Once deployed, set this env var in Render to restrict API access to your domain only:

```
ALLOWED_ORIGIN=https://arc-meeting-review-app.onrender.com
```

---

## Adding more Smartsheet reports

To add another log/report to the app:

1. Find the new **Report ID** in Smartsheet (from the URL or search)
2. Identify the **Column IDs** for any writable fields
3. Add a new route in `server.js` or reuse `/api/report/:reportId`
4. Update `public/index.html` with a nav tab or selector for multiple reports

---

## Smartsheet IDs used

| Name | ID |
|------|----|
| ARC Meeting Review Log (Report) | `7288028541177732` |
| Agency Staff Request Process 2.0 (Source Sheet) | `4795627573563268` |
| Agency Request Committee Decision (Column) | `2002017168084868` |
| Agency Request Committee Comments (Column) | `6505616795455364` |

---

## Security notes

- `SMARTSHEET_TOKEN` is **never** in the browser or in the repo
- Rate limiting: 60 API requests/min per IP
- Helmet.js sets security headers on all responses
- CORS is configurable via `ALLOWED_ORIGIN`
- The `.env` file is in `.gitignore` — always double-check before pushing

---

## Tech stack

| Layer | Tech |
|-------|------|
| Frontend | Vanilla HTML/CSS/JS (no framework) |
| Backend | Node.js + Express |
| Smartsheet | REST API v2 via Axios |
| Hosting | Render (free tier available) |
| Security | Helmet, express-rate-limit, dotenv |
