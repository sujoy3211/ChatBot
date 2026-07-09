# Documentation — Groq AI Chatbot

## Overview

A production-ready, beginner-friendly AI chatbot powered by [Groq](https://groq.com)'s ultra-fast inference API. Features streaming responses, conversation history, a clean UI, and zero build tools.

---

## Project Structure

```
groq-chatbot/
├── backend/
│   ├── server.js        # Express server + Groq API integration
│   ├── package.json
│   ├── .env.example     # copy to .env and add your key
│   └── .gitignore
├── frontend/
│   ├── index.html       # chat UI structure
│   ├── style.css        # modern responsive styling
│   └── script.js        # streaming fetch logic + state
├── setup.md
├── documentation.md
└── README.md
```

---

## How It Works

1. **User sends a message** — the frontend adds it to a `conversation` array and sends the *whole array* to `POST /api/chat`. Sending full history each time is what gives the bot "memory" of the conversation — the model itself is stateless.
2. **Backend processing** — the server validates the request, prepends a system prompt (the bot's personality), and calls Groq with `stream: true`.
3. **Streaming** — Groq streams tokens back as they're generated. The backend forwards each token to the browser immediately using chunked HTTP responses, without waiting for the full answer.
4. **Frontend rendering** — the frontend reads the stream via `response.body.getReader()` and appends each chunk to the message bubble live, producing the "typing" effect.
5. **Persistence** — once done, the full reply is saved into the conversation array and into `localStorage`, so refreshing the page keeps chat history.

---

## Features

- Real-time streaming responses (character-by-character "typing" effect)
- Conversation memory across the session
- Chat history persisted via `localStorage`
- Clean, responsive UI with no build tools required
- Input validation on every request
- Basic per-IP rate limiting
- Startup validation (fails fast with clear errors if misconfigured)
- Graceful error handling on both frontend and backend

---

## Tech Stack

- **Backend:** Node.js, Express
- **AI Provider:** Groq API (`openai/gpt-oss-120b` by default)
- **Frontend:** Vanilla HTML/CSS/JS (no framework, no build step)
- **Storage:** Browser `localStorage` for chat history

---

## Customization Guide

| What to change | Where |
|---|---|
| Bot personality | Edit `systemMessage.content` in `server.js` |
| AI model | Set `GROQ_MODEL` in `.env` — see [console.groq.com/docs/models](https://console.groq.com/docs/models) |
| Colors/theme | Edit CSS variables at the top of `style.css` |
| History length sent to model | Change `messages.slice(-20)` in `server.js` |
| Rate limit | Adjust `RATE_LIMIT` / `RATE_WINDOW_MS` in `server.js` |

---

## Architecture Decisions & Best Practices

- **API key stays server-side only** — never exposed to the browser.
- **Input validation** on every request (shape, type, emptiness).
- **Rate limiting** protects the Groq API bill from abuse.
- **Startup validation** ensures the app fails fast with a clear message if misconfigured, rather than failing silently later.
- **Conversation trimming** before sending to the model, to control cost and latency.
- **Streaming responses** for a modern, responsive UX instead of waiting for the full reply.

---

## Possible Future Improvements

- Swap the in-memory rate limiter for `express-rate-limit` + Redis for real production traffic.
- Add user authentication for per-user chat history instead of one shared browser's `localStorage`.
- Add a proper database (Postgres/SQLite) so history survives across devices.
- Render markdown (e.g. with `marked.js`) in assistant replies, since the system prompt already asks the model to use markdown.

---

## API Reference

### `POST /api/chat`
Sends the full conversation array and receives a streamed response.

```bash
curl -N -X POST http://localhost:3000/api/chat \
  -H "Content-Type: application/json" \
  -d '{"messages":[{"role":"user","content":"Say hi in 5 words"}]}'
```

### `GET /api/health`
Returns server health status.

```bash
curl http://localhost:3000/api/health
```
