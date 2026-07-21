# 🤟 SignLearn (SignOLight) — AI Sign Language Video Player

> **Build with Gemma 4 Hackathon** · Inclusive Education Tool for Deaf & Hard-of-Hearing Students in Bangladesh

 you paste any YouTube educational video URL and watch it **side-by-side with a real-time 3D sign language avatar** that synchronizes with the video's captions. **Google's Gemma 4** is the only LLM powering the app — it converts captions into ASL gloss notation, simplifies lecture language, and generates plain-language definitions for concepts the avatar can't sign.

---

## 🖼️ What It Does

- 📺 **YouTube video player** on the left (with built-in captions)
- 🤟 **Animated 3D sign language avatar** on the right — VRM by default, with an alternate Mixamo/FBX avatar mode — signs every word in sync
- 🤖 **Gemma 4** converts subtitles → ASL gloss notation, simplifies language, and writes concept-card definitions for words outside the sign dictionary — all in real time
- 🈶 **Bangla code-switching support** — detects and translates mixed Bangla-English captions
- ⏱️ Optional **WhisperX** word-level timestamp alignment for tighter sync (non-LLM speech processing, runs alongside Gemma 4)
- 🎛️ Three layout modes: Side-by-Side / Picture-in-Picture / Focus Sign
- 📜 **Caption timeline bar** at the bottom — click any caption to jump to it
- ⚡ Works with any YouTube video that has CC subtitles enabled
- 🛟 **No AI key? No problem** — falls back to a deterministic heuristic gloss so the app still functions

---

## 🗂️ Project Structure

```
SignOLight-v2/
├── backend/                    ← Express.js API server
│   ├── server.js               ← Entry point, caching, vocabulary-gap tracker
│   ├── routes/
│   │   ├── video.js            ← YouTube metadata fetch
│   │   ├── captions.js         ← Caption extraction (VTT/XML/transcript/Bangla)
│   │   └── sign.js             ← Gemma 4 gloss/simplify/concept-card pipeline
│   ├── lib/signability.js      ← Vocabulary-constrained signability enforcement
│   ├── data/sign-vocabulary.json ← Avatar's full signable word list
│   ├── cache/                  ← Persistent per-video JSON cache (partially committed)
│   ├── __tests__/               ← Jest unit + integration + comprehension tests
│   ├── .env.example
│   └── package.json
│
├── frontend/                   ← React app
│   ├── src/
│   │   ├── App.js              ← Hand-rolled router
│   │   ├── pages/               ← Landing, Player, SignDemo, PoseTuner, MixamoDemo, HowItWorks
│   │   ├── components/
│   │   │   ├── YouTubePlayer.js       ← YouTube IFrame API wrapper (100ms clock)
│   │   │   ├── SignAvatar.js          ← 3D VRM avatar (@pixiv/three-vrm)
│   │   │   ├── MixamoAvatar.js / MixamoSignAvatar.js ← Alternate FBX avatar mode
│   │   │   ├── CaptionBar.js          ← Caption timeline + simplified text
│   │   │   └── ControlPanel.js        ← Layout & sign toggle controls
│   │   ├── services/            ← timelineScheduler, playbackPacing, banglaAlphabet/Captions
│   │   ├── utils/                ← sync.js (binary search), notation.js
│   │   └── styles/global.css
│   ├── public/
│   │   ├── models/               ← sign.vrm, Mixamo Ch09_nonPBR.fbx
│   │   └── signs/*.json          ← Authored sign clip files
│   ├── .env.example
│   └── package.json
│
├── backend_nlp/                 ← Optional Python FastAPI microservice (WhisperX)
│                                    Non-LLM speech processing, proxied via /api/nlp/*
├── docs/                        ← Project vision, research, and architecture docs
├── scripts/                     ← VRM verification/brow-rig tooling, vocabulary sync
├── render.yaml                  ← Render.com deployment config
├── package.json                 ← Root scripts
├── QUICKSTART.md                ← Detailed local-dev + deployment walkthrough
└── README.md
```

---

## 🚀 Local Setup (Step-by-Step)

### Prerequisites
- **Node.js** v18+ ([download](https://nodejs.org))
- **npm** v8+
- A **Google AI Studio API key** for Gemma 4 (free tier works) → [aistudio.google.com/apikey](https://aistudio.google.com/apikey)

---

### Step 1 — Clone / Download the project

```bash
# If using git:
git clone https://github.com/YOUR_USERNAME/signlearn.git
cd signlearn

# Or just unzip the downloaded folder and cd into it
cd signlearn
```

---

### Step 2 — Install dependencies

```bash
# Install root tools
npm install

# Install backend dependencies
cd backend
npm install
cd ..

# Install frontend dependencies
cd frontend
npm install
cd ..
```

Or all at once:
```bash
npm run install:all
```

---

### Step 3 — Configure environment variables

**Backend:**
```bash
cd backend
cp .env.example .env
```

Open `backend/.env` and fill in:
```env
GEMMA_API_KEY=xxxxxxxxxxxxxxxxx               # Your Google AI Studio key
GEMMA_MODEL=                                  # Exact Gemma 4 model ID from AI Studio's model picker
GEMMA_BATCH_SIZE=10
FRONTEND_URL=http://localhost:3000
PORT=5000
```

**Frontend:**
```bash
cd frontend
cp .env.example .env
```

Open `frontend/.env` and set:
```env
REACT_APP_API_URL=http://localhost:5000
```

---

### Step 4 — Run in development mode

**Terminal 1 — Start backend:**
```bash
cd backend
npm run dev
# ✅ Server starts at http://localhost:5000
```

**Terminal 2 — Start frontend:**
```bash
cd frontend
npm start
# ✅ React app opens at http://localhost:3000
```

Or run both at once from root:
```bash
npm run dev
```

---

### Step 5 — Test it!

1. Open **http://localhost:3000**
2. Paste a YouTube URL with captions, e.g.:
   - `https://www.youtube.com/watch?v=kqtD5dpn9C8` (Khan Academy)
   - `https://www.youtube.com/watch?v=HluANRwPyNo` (TED-Ed)
3. Click **Start Learning** — the sign avatar will appear!

> **Tip:** Make sure to use videos that have **CC (Closed Captions)** enabled.
> Khan Academy, TED-Ed, and most major educational channels work great.

---

## ☁️ Deploy for Free

### Option A — Render.com (Recommended, one-click)

1. Push your code to GitHub
2. Go to [render.com](https://render.com) → New → Blueprint
3. Connect your GitHub repo
4. Render reads `render.yaml` automatically and creates both services (API + frontend)
5. In the **Render dashboard**, add environment variables:
   - Backend service → Environment → Add:
     - `GEMMA_API_KEY` = your Google AI Studio key
     - `GEMMA_MODEL` = exact Gemma 4 model ID
     - `GEMMA_BATCH_SIZE` = `10`
     - `FRONTEND_URL` = your frontend Render URL (set after first deploy)
   - Frontend service → Environment → Add:
     - `REACT_APP_API_URL` = your backend Render URL

6. Trigger a redeploy on both services
7. Your app is live! 🎉

**Optional — Enable WhisperX (free, Phase B2):**
Deploy `backend_nlp/` to [Hugging Face Spaces](https://huggingface.co/spaces) (free, 16 GB RAM).
Then set `NLP_SERVICE_URL` on the Render backend to your Space URL.
See [`docs/WHISPERX_DEPLOYMENT.md`](docs/WHISPERX_DEPLOYMENT.md) for step-by-step instructions.

---

### Option B — Vercel (Frontend) + Render (Backend)

**Backend on Render:**
1. New Web Service → Connect GitHub repo
2. Root Directory: `backend`
3. Build Command: `npm install`
4. Start Command: `node server.js`
5. Add env vars: `GEMMA_API_KEY`, `GEMMA_MODEL`, `GEMMA_BATCH_SIZE`, `FRONTEND_URL`
6. Copy your Render backend URL (e.g. `https://signlearn-api.onrender.com`)

**Frontend on Vercel:**
1. Go to [vercel.com](https://vercel.com) → New Project → Import GitHub repo
2. Root Directory: `frontend`
3. Framework Preset: Create React App
4. Add Environment Variable:
   - `REACT_APP_API_URL` = your Render backend URL
5. Deploy!

---

## 🔑 API Keys Needed

| Key | Required | Where to Get | Free? |
|-----|----------|--------------|-------|
| `GEMMA_API_KEY` | ✅ Yes | [aistudio.google.com/apikey](https://aistudio.google.com/apikey) | Free tier available |
| YouTube Data API | ❌ Optional | Google Cloud Console | Free (10k/day) |

> **Without a Gemma key:** The app still works! It falls back to a simple word-by-word gloss system (no AI, but functional).

---

## 🎛️ How It Works (Technical Flow)

```
User pastes YouTube URL
        ↓
Backend: GET /api/video/info
  → YouTube oEmbed API (no key needed)
  → Returns title, author, thumbnail, videoId
        ↓
Frontend: Loads YouTube IFrame API player
        ↓
Backend: GET /api/captions?videoId=xxx
  → YouTube Timedtext API (no key needed)
  → Parses XML → [{start, end, text}]
        ↓
Backend: POST /api/sign/batch
  → Gemma 4 simplifies each caption + converts it to ASL gloss (one call, JSON-schema output)
  → Unknown words become [FINGERSPELL:x] or [CONCEPT:x] typed tokens
  → Gemma 4 writes a plain-language definition for every [CONCEPT:x] found
  → WhisperX (if deployed) runs in parallel to refine word-level timing
  → Returns [{start, end, text, simplified, gloss, words[], conceptExplanations}]
        ↓
Frontend: Polls player.getCurrentTime() every 100ms
        ↓
SignAvatar: Binary-searches the active caption by timestamp
  → Animates the 3D VRM (or Mixamo) avatar through each sign
  → Cross-fades between signs, applies non-manual markers (WH/YN question expressions)
  → Falls back to fingerspelling or a concept card when a word has no authored sign
```

---

## 🛠️ Customization

### Add more signable words
Either drop an authored clip JSON into `frontend/public/signs/`, or add a procedural motion to the `SIGN_MOTIONS` object in `frontend/src/components/SignAvatar.js`. After adding words, run:
```bash
npm run sync:vocab
```
This regenerates `backend/data/sign-vocabulary.json` so Gemma 4's prompt knows the new word is signable — a word missing from that list always renders as a silent `[CONCEPT:x]` pause.

### Change the avatar model
Swap `frontend/public/models/sign.vrm` (VRM avatar) or `frontend/public/models/mixamo/Ch09_nonPBR.fbx` (Mixamo avatar). Use `npm run vrm:verify` to check a new VRM file exposes the blendshapes `SignAvatar.js` expects.

### Target a different sign language
In `backend/routes/sign.js`, edit the ASL grammar rules inside `buildGlossPrompt()` and the batch prompt in `simplifyAndGlossBatch()` (topic-comment order, WH-final, etc. are ASL-specific — BSL, ISL, or BdSL each have their own grammar).

---

## 📋 Sample Videos That Work Well

| Video | URL |
|-------|-----|
| Khan Academy - Basic Math | `youtube.com/watch?v=kqtD5dpn9C8` |
| TED-Ed - How the brain works | `youtube.com/watch?v=HluANRwPyNo` |
| Python for Beginners | `youtube.com/watch?v=1BfCnjr_Vjg` |
| NASA Space Science | `youtube.com/watch?v=_tmkDIgZFLE` |
| Crash Course Biology | `youtube.com/watch?v=QnQe0xW_JY4` |

---

## 🐛 Troubleshooting

**"No captions found"**
→ The video doesn't have CC enabled. Try another video with the CC icon.

**Sign avatar not moving**
→ Check that captions loaded (green status bar). Check browser console for errors.

**Backend connection error**
→ Make sure backend is running on port 5000. Check `REACT_APP_API_URL` in frontend `.env`.

**Render free tier sleeping**
→ Free Render services sleep after 15min. First request takes ~30s to wake up. Consider adding a health-ping cron job.

---

## 📄 License

MIT License — free to use, modify, and submit to hackathons!

---

Built with ❤️ for inclusive education · Powered by **Gemma 4** · **SignLearn 2026**
