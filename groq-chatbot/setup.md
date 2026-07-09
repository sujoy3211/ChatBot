# Setup Guide — Groq AI Chatbot

This guide walks you through installing and running the chatbot locally, and deploying it.

---

## 1. Prerequisites

- **Node.js 18+** installed. Check with:
  ```bash
  node -v
  ```
  If not installed, get it from [nodejs.org](https://nodejs.org).

- A **free Groq API key**:
  1. Sign up at [console.groq.com](https://console.groq.com)
  2. Go to **API Keys** → **Create API Key**
  3. No credit card required for the free tier.

---

## 2. Installation (5 minutes)

```bash
# 1. Go into the backend folder
cd groq-chatbot/backend

# 2. Install dependencies
npm install

# 3. Create your environment file
cp .env.example .env

# 4. Open .env and paste your real key:
#    GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxx

# 5. Start the server
npm start
```

You should see:
```
✅ Groq chatbot server running at http://localhost:3000
   Using model: openai/gpt-oss-120b
```

Open **http://localhost:3000** in your browser. The same server serves both the API and the chat UI — nothing else needs to run.

> Tip: use `npm run dev` instead of `npm start` during development — it auto-restarts the server whenever you save a file.

---

## 3. Testing the Setup

**Manual test:**
1. Send "Hello, who are you?" — you should see a typing indicator, then streamed text.
2. Send a follow-up like "What did I just ask you?" — confirms conversation memory works.
3. Click **Clear** — confirms history resets.
4. Refresh the page mid-conversation — confirms `localStorage` persistence.

**API test with curl** (server must be running):
```bash
curl -N -X POST http://localhost:3000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hi in 5 words"}]}'
```

**Health check:**
```bash
curl http://localhost:3000/api/health
```

---

## 4. Common Setup Issues

| Symptom | Cause | Fix |
|---|---|---|
| Server won't start, "Missing GROQ_API_KEY" | `.env` missing or empty | Run `cp .env.example .env` and paste your key |
| `401` error in the browser | Invalid/expired API key | Regenerate a key at console.groq.com/keys |
| `429 Too many requests` | Hit the rate limiter or Groq's own rate limit | Wait a minute, or raise `RATE_LIMIT` in `server.js` |
| Chat bubble stays empty forever | Model name deprecated/invalid | Check current models at console.groq.com/docs/models, update `GROQ_MODEL` in `.env` |
| CORS error in console | Frontend served from a different origin than backend | Always use `http://localhost:3000` for both — don't open `index.html` via `file://` |
| Changes to server.js not taking effect | Server still running old code | Stop it (Ctrl+C) and restart, or use `npm run dev` |

---

## 5. Deployment

### Option A: Render.com (simplest)
1. Push this project to a GitHub repo.
2. On [render.com](https://render.com) → **New → Web Service** → connect your repo.
3. Set **Root Directory** to `backend`.
4. **Build Command:** `npm install`  **Start Command:** `npm start`
5. Add environment variable `GROQ_API_KEY` (and optionally `GROQ_MODEL`) in the Render dashboard — never commit `.env` to git.
6. Deploy. Render gives you a public URL serving both your API and frontend.

### Option B: Railway.app
1. Push to GitHub, then **New Project → Deploy from GitHub repo** on [railway.app](https://railway.app).
2. Set the root/service directory to `backend`.
3. Add `GROQ_API_KEY` under **Variables**.
4. Railway auto-detects `npm start`. Deploy and you'll get a public URL.

### Pre-deployment checklist
- [ ] `.env` is in `.gitignore` — **never** commit your API key.
- [ ] `npm install` runs cleanly with no errors.
- [ ] You've tested `npm start` locally one final time.
