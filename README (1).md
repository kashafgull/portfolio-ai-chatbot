# Portfolio AI Chatbot

A lightweight, embeddable AI chatbot for portfolio websites. It runs as a floating chat widget (plain JavaScript, no dependencies) and talks to an AI model through a small Cloudflare Worker, so the API key is never exposed in the browser or in this repository.

**Live demo:** https://YOURUSERNAME.github.io/portfolio-ai-chatbot/

```
Visitor's browser           Cloudflare Worker              AI provider
┌──────────────┐  POST   ┌────────────────────┐  API call  ┌────────────┐
│   chat.js    │ ──────▶ │     worker.js      │ ─────────▶ │  Gemini /  │
│ (chat widget)│ ◀────── │ key + system prompt│ ◀───────── │ Claude /   │
└──────────────┘  reply  │ CORS + rate limit  │            │ OpenAI     │
       │                 └────────────────────┘            └────────────┘
       └─ if the Worker or AI is unavailable, falls back to built-in FAQ answers
```

## Features

- **Embeddable with one line**: works on any HTML, WordPress, or Shopify site
- **API key stays server-side**: stored as a Cloudflare secret, never committed or sent to the browser
- **Provider-agnostic**: switch between Gemini (free tier), Claude, and OpenAI with one environment variable
- **Graceful fallback**: keyword-based FAQ answers keep the widget useful if the AI service is down or rate limited
- **Basic abuse protection**: origin allowlist (CORS), per-IP rate limiting, message length and history caps
- **Grounded answers**: a system prompt restricts the bot to topics about the site owner and tells it not to invent facts

## Files

| File | Purpose |
|---|---|
| `chat.js` | Chat widget: UI, conversation history, fallback FAQ |
| `worker.js` | Cloudflare Worker: proxy, system prompt, rate limiting |
| `index.html` | Demo page |

## Setup

### 1. Get an API key
For a free option, create a Gemini API key in [Google AI Studio](https://aistudio.google.com). The free tier has rate limits. Check AI Studio for the current model name.

### 2. Deploy the Worker
1. Create a free Cloudflare account, then go to **Workers & Pages > Create > Start with Hello World**.
2. Open **Edit code**, replace the code with the contents of `worker.js`, and deploy.
3. Under **Settings > Variables and Secrets**, add:

| Name | Type | Value |
|---|---|---|
| `GEMINI_API_KEY` | Secret | your key |
| `PROVIDER` | Text | `gemini` (or `claude` / `openai`) |
| `ALLOWED_ORIGIN` | Text | comma-separated site origins, e.g. `https://yourname.github.io,https://yourdomain.com` |
| `GEMINI_MODEL` | Text (optional) | override the default model |

4. Redeploy and copy the Worker URL (`https://<name>.<account>.workers.dev`).

### 3. Add the widget to your site
```html
<script src="chat.js" data-worker="https://YOUR-WORKER.workers.dev"></script>
```
Optional attributes: `data-name="My Assistant"`, `data-color="#7c3aed"`.

### 4. Customize
- Edit `SYSTEM_PROMPT` in `worker.js` with your own bio, skills, and projects. The bot only knows what you put there.
- Edit the `FAQ` list in `chat.js` for the offline fallback answers.

## Using other providers
Set `PROVIDER` to `claude` (secret `ANTHROPIC_API_KEY`) or `openai` (secret `OPENAI_API_KEY`). Optional model overrides: `CLAUDE_MODEL`, `OPENAI_MODEL`. If a client wants their own bot, deploy a separate Worker with their key so billing stays with them.

## Security notes
- Never put API keys in `chat.js`, `index.html`, or this repository.
- `ALLOWED_ORIGIN` stops other websites from calling your Worker from a browser, but it is not authentication: a script outside a browser can still send requests. The per-IP rate limit is best-effort (it resets when the Worker restarts), so add a Cloudflare rate limiting rule if you expect real traffic.
- The widget renders replies with `textContent`, so AI output is never injected as HTML.

## Tech
Vanilla JavaScript, Cloudflare Workers, Google Gemini API (also supports Anthropic Claude and OpenAI).

## License
MIT
