# Lesson Studio

A local web app for building evidence-backed lessons. Enter a lesson title and objective; the app searches public reference pages and book records, plus supplementary YouTube videos when configured. Review the findings, generate an editable lesson, run separate factual and teaching-quality reviews, apply their recommendations, save drafts locally, and export editable Word-compatible `.doc` and Markdown files.

Research and lesson drafting are separate clicks so you can inspect the gathered evidence first. The lesson plan is capped at 45 minutes; its section timings include explanations, examples, practice, knowledge checks, and the opening/wrap-up. Optional YouTube links are listed at the end and are outside the timed plan.

## Cost and data

- The app calls Google Gemini through the backend. It tries Gemini 3.8 Flash first and makes one fallback attempt with Gemini 3.7 Flash. If Gemini is unavailable or its free quota is reached, it can use OpenRouter's `openrouter/free` route when `OPENROUTER_API_KEY` is set, and then Groq's `qwen/qwen3.8-27b` when `GROQ_API_KEY` is set. The OpenRouter route selects only free models; no paid model ID is called. Quota exhaustion stops generation without upgrading or billing.
- Create a Gemini API key in Google AI Studio using a project with no billing account linked. If Gemini's free quota is reached, configured free fallbacks are tried; if they are also unavailable, wait for a provider quota to reset.
- Automatic website discovery uses Wikimedia reference pages and scholarly abstracts from OpenAlex or Semantic Scholar. Book discovery uses Open Library's public catalog; it uses descriptions or first-sentence previews when available, not full copyrighted book text.
- OpenAlex works without a key for casual use and stops when its free daily request budget is reached. Open Library is queried at its published low-volume rate. The app does not attach billing or retry through a paid service.
- Wikimedia asks API clients to identify themselves and provide contact details. If its search is blocked, add a public contact URL or email as `WIKIMEDIA_CONTACT` in `.env`; scholarly abstracts are also searched as a fallback.
- YouTube discovery uses the official YouTube Data API. Create an API key in a project with YouTube Data API v3 enabled and no billing account linked, then set `YOUTUBE_API_KEY` in `.env`. The default quota allows 100 search calls per day; this app makes one search call per research action. Without this key, web and book research still work, but videos cannot be discovered.
- Groq is optional. Create a key on the [Groq Console](https://console.groq.com/keys) and set `GROQ_API_KEY` in `.env` to enable it. The lesson brief and source extracts are sent to Groq only if Gemini is unavailable or OpenRouter's free route fails. Keep the Groq account on its Free Plan; free-plan limits reject requests when reached.
- OpenRouter is optional. Create a free API key in [OpenRouter](https://openrouter.ai/settings/keys) and set `OPENROUTER_API_KEY` in `.env`. The app uses only `openrouter/free`, which chooses among free models such as GLM or DeepSeek when available. Free requests are limited to 50 per day and 20 per minute; failed requests also count toward the daily limit. Do not add credits; the app does not call paid model IDs.
- Video transcripts are not fetched; YouTube results are supplementary links and are not used as factual evidence in lesson generation.
- Google may use content sent through the free tier to improve its products. Use public sources and do not send private or sensitive material.
- Gemini Search grounding is not used; source discovery runs through the public APIs listed above.
- Saved lessons stay in `lesson_studio.sqlite3` on this computer. API keys stay in the backend `.env` file and are never sent to the browser.
- Google Fonts are loaded from Google Fonts. If you are offline, the site falls back to local system fonts.

## Start the app

Requirements: Python 3.9+ and Node.js 20+.

1. From this folder, create and activate a Python virtual environment, then install the packages:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

2. Create the local settings file and add your free-tier key:

   ```bash
   cp .env.example .env
   ```

   Open `.env` and add the Gemini key after `GEMINI_API_KEY=`. To enable automatic video discovery, also add a YouTube Data API key after `YOUTUBE_API_KEY=`. Keep both keys private.

3. In one terminal, start the backend:

   ```bash
   source .venv/bin/activate
   uvicorn backend.app:app --reload --host 127.0.0.1 --port 8000
   ```

4. In another terminal, install the website packages and start the site:

   ```bash
   cd frontend
   npm install
   npm run dev
   ```

5. Open the local address printed by Vite (normally `http://127.0.0.1:5173`).

The site can open without API keys so you can inspect the workspace. Automatic research needs an internet connection; lesson generation and reviews need the Gemini key. Video discovery additionally needs the YouTube key. Restart the backend after changing `.env`.

## Build a local production bundle

```bash
cd frontend
npm run build
```

Then start the backend from the project folder. If `frontend/dist` exists, FastAPI serves the built site and API together at `http://127.0.0.1:8000`.

## Main areas

- `frontend/src/App.jsx` — authoring, evidence, review, saved-draft and export flows.
- `frontend/src/styles.css` — Furrow visual theme and responsive app layout.
- `backend/app.py` — Gemini requests, source gathering, quality reviews, and local SQLite storage.
- `.env.example` — local Gemini key setting template.
