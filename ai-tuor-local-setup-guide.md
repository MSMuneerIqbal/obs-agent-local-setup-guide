
# AI Tutor — AI-Powered LMS

A full-stack LMS with a standalone, reusable AI microservice. Students learn through AI chat, course browsing, quizzes, and progress tracking. Teachers manage classes, courses, and student analytics.

**The AI Tutor service is zero-LMS-code — plug it into any other system.**

---

## What You Need

| Service | Get At |
|---------|--------|
| **OpenAI** (GPT-4o + embeddings) | platform.openai.com |
| **Pinecone** (vector database) | app.pinecone.io |
| **MongoDB Atlas** (database) | mongodb.com/atlas |

### Software

| Tool | Min Version | Check |
|------|-----------|-------|
| Python | 3.12 | `python --version` |
| Node.js | 18 | `node --version` |

---

## Architecture

```
┌─────────────────────────────────────────┐
│           Frontend (Next.js 14)         │
│  / → Landing  /student → Login          │
│  /dashboard → Student Portal            │
│  /teacher → Teacher Portal              │
└──────┬──────────────────────────────────┘
       │ REST + WebSocket
       ▼
┌─────────────────────────────────────────┐
│        LMS Backend (:8000)              │
│  Auth, Content CRUD, Quiz, Classes      │
│  DB: MongoDB — NO AI calls directly     │
└──────┬──────────────────────────────────┘
       │ HTTP (httpx)
       ▼
┌─────────────────────────────────────────┐
│        AI Tutor (:8001)                 │
│  /explain  /chat  /generate-quiz        │
│  /explain-mistake  /search  /embed      │
│  Deps: OpenAI + Pinecone only           │
│  Reusable — zero LMS code               │
└─────────────────────────────────────────┘
```

### Why 3 Services?

The AI Tutor at `:8001` has **no knowledge of users, courses, or LMS logic**. Any LMS in any language can call its REST API for AI explanations, quiz generation, semantic search, etc.

---

## Quick Start — Run Everything Locally

This guide takes you from zero to a running app. You'll set up a **single Python virtual environment** for both backend services, then run the frontend.

### Step 1: Clone the Repo

```bash
git clone <your-repo-url>
cd Agentic-AI-Tutor
```

### Step 2: Create One Python Virtual Environment (Root)

Create a single venv at the project root. Both `ai-tutor/` and `backend/` will use it.

```bash
# Windows:
python -m venv .venv
.venv\Scripts\activate

# Mac / Linux:
python3 -m venv .venv
source .venv/bin/activate
```

> Keep this terminal open. All Python steps below run inside this activated venv.

### Step 3: Install Python Dependencies

Install dependencies for **both services** into the same venv:

```bash
# AI Tutor dependencies
pip install -r ai-tutor/requirements.txt

# LMS Backend dependencies
pip install -r backend/requirements.txt
```

Verify everything installed:

```bash
pip list | findstr "fastapi uvicorn openai pinecone motor beanie"
```

### Step 4: Create Environment Files

#### 4a. AI Tutor — `ai-tutor/.env`

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-4o
PINECONE_API_KEY=pc-...
PINECONE_INDEX=ai-tutor
```

#### 4b. LMS Backend — `backend/.env`

```env
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=pc-...
DATABASE_URL=mongodb+srv://<user>:<pass>@cluster.mongodb.net/tutor_lms
PINECONE_INDEX=ai-tutor
AI_SERVICE_URL=http://localhost:8001

OPENAI_MODEL=gpt-4o
ENVIRONMENT=development
PORT=8000
FRONTEND_URL=http://localhost:3000
```

> **Pinecone Index:** Create a serverless index in Pinecone console → **dimension 1536**, metric **cosine**, cloud **aws**, region **us-east-1**.

#### 4c. Frontend — `frontend/.env.local`

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_WS_URL=ws://localhost:8000
```

### Step 5: Install Frontend Dependencies

```bash
cd frontend
npm install
cd ..
```

### Step 6: Run All Three Services

**Open 3 terminals from the project root (`Agentic-AI-Tutor/`).** Activate the root `.venv` in the first two.

#### Terminal 1 — AI Tutor (:8001)

```bash
# Windows:
.venv\Scripts\activate

# Mac / Linux:
source .venv/bin/activate

uvicorn app:app --port 8001
```

#### Terminal 2 — LMS Backend (:8000)

```bash
# Windows:
.venv\Scripts\activate

# Mac / Linux:
source .venv/bin/activate

uvicorn app.main:app --port 8000
```

#### Terminal 3 — Frontend (:3000)
first deactivate the environment by writting deactivate  then run the frontend

```bash
cd frontend
npm install
npm run dev
```

### Step 7: Open the App

Go to **http://localhost:3000**

---

## Running Services (Quick Reference)

| Service | Command (from project root) | Port |
|---------|----------------------------|------|
| AI Tutor | `uvicorn app:app --port 8001 --app-dir ai-tutor` | 8001 |
| Backend | `uvicorn app.main:app --port 8000 --app-dir backend` | 8000 |
| Frontend | `cd frontend && npm run dev` | 3000 |

**Startup order matters:** AI Tutor → Backend → Frontend. The backend calls the AI Tutor on startup, so the AI Tutor must be running first.

---

## Portals & Pages

| URL | Portal | Who |
|-----|--------|-----|
| `/` | Landing | Public — choose Student or Teacher |
| `/student` | Student Login | → redirects to `/dashboard` |
| `/teacher` | Teacher Login | → redirects to Teacher Portal |
| `/dashboard` | Student Portal | Courses, Progress, Quiz, AI Chat, Knowledge Base |
| `/teacher` | Teacher Portal | Classes, Courses, Students, Analytics |

---

## Features

### Student
- **Course Browser** — Card grid → Chapters → Topics with markdown rendering
- **AI Sidebar** — Select any text → AI explains using knowledge base
- **Quiz Engine** — Generate from topic, submit answers, score tracking
- **Progress Dashboard** — Stats, quiz history, weak area detection
- **WebSocket Chat** — Real-time AI tutoring with conversation memory

### Teacher
- **Class Management** — Create batches/sections, enroll students, assign courses
- **Course Builder** — Course → Chapter → Topic hierarchy, PDF/DOCX/file upload
- **Student Analytics** — Per-student quiz history, weak areas, progress
- **Content embedding** — Auto-chunk → embed → store in Pinecone

### AI Tutor Service (Standalone — 10 Endpoints)

| Endpoint | Old Agent | What it does |
|----------|-----------|-------------|
| `POST /chat` | Orchestrator + Tutor + Feedback | Brainstorm chat with memory, personalized coaching |
| `POST /explain` | Tutor | Text explanation with web search + real-life examples |
| `POST /generate-quiz` | Quiz | MCQ generation with adaptive difficulty |
| `POST /explain-mistake` | Quiz | Why wrong + correct concept + memory tip |
| `POST /assess` | Assessment | VARK learning style — 6 questions, returns learning style |
| `POST /plan` | Planning | Personalized study plan (topic + learning style + hours) |
| `POST /feedback` | Feedback | Progress coaching based on quiz history |
| `POST /search` | RAG | Pinecone semantic search |
| `POST /embed` | RAG | OpenAI embeddings (1536d) |
| `POST /web-search` | Web Search | Web search for supplementary content |

---

## Project Structure

```
Agentic-AI-Tutor/
├── ai-tutor/                    ← Standalone AI microservice
│   ├── app.py                   ← All AI endpoints (explain, chat, quiz, search)
│   └── requirements.txt
├── backend/                     ← LMS Backend
│   ├── app/
│   │   ├── main.py
│   │   ├── api/routes/
│   │   │   ├── auth.py          ← Register, login, student management
│   │   │   ├── ai.py            ← AI proxy → ai-tutor:8001
│   │   │   ├── content.py       ← Course/Chapter/Topic CRUD + Classes
│   │   │   ├── quiz_routes.py   ← Quiz generate + submit + history
│   │   │   └── websocket.py     ← Chat via AI service
│   │   ├── core/
│   │   │   ├── ai_client.py     ← HTTP client to AI Tutor
│   │   │   ├── mongodb.py       ← Beanie/MongoDB init
│   │   │   └── session_store.py
│   │   ├── models/              ← User, Quiz, Content models
│   │   └── services/
│   │       └── rag_service.py   ← RAG proxy (→ AI service /search + /embed)
│   ├── requirements.txt
│   └── pyproject.toml
├── frontend/                    ← Next.js 14 App
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx         ← Landing page
│   │   │   ├── layout.tsx       ← Root layout + Global AI Sidebar
│   │   │   ├── dashboard/
│   │   │   ├── student/         ← Student login page
│   │   │   └── teacher/         ← Teacher portal
│   │   └── components/
│   │       ├── dashboard/       ← Dashboard, tabs, stats
│   │       ├── student/         ← CourseBrowser, AISidebar
│   │       ├── auth/            ← LoginModal, RegisterModal
│   │       ├── chat/            ← Chat interface, QuizCard
│   │       └── landing/         ← Landing page
│   └── package.json
├── SPECIFICATION.md             ← Full technical reference
├── IMPLEMENTATION_PLAN.md       ← Phases, tasks, progress
└── README.md                    ← This file
```

---

## API Docs

Once running: **http://localhost:8000/docs** (LMS Backend — Swagger)

### LMS Backend Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/v1/auth/register` | Register |
| POST | `/api/v1/auth/login` | Login |
| GET | `/api/v1/auth/students` | List students (teacher) |
| GET | `/api/v1/auth/students/{id}` | Student detail + analytics |
| POST | `/api/v1/ai/explain` | AI explanation |
| POST | `/api/v1/ai/chat` | AI chat |
| POST | `/api/v1/ai/explain-mistake` | Mistake analysis |
| GET | `/api/v1/content/courses` | List courses |
| POST | `/api/v1/content/courses` | Create course |
| POST | `/api/v1/content/classes` | Create class |
| GET | `/api/v1/content/classes` | List classes |
| POST | `/api/v1/quiz/generate` | Generate quiz |
| POST | `/api/v1/quiz/submit` | Submit answers |
| WS | `/ws/sessions/{id}` | Real-time chat |

### AI Tutor Endpoints: `http://localhost:8001/docs`

---

## Troubleshooting

**Backend won't start**
- Check `.env` has all keys set
- Verify MongoDB URL is correct and IP is whitelisted
- Run `pip install -r backend/requirements.txt` in the activated venv

**AI Tutor returns errors**
- Make sure AI Tutor is running on port 8001 before LMS backend
- Check `OPENAI_API_KEY` is set in `ai-tutor/.env`
- Check `AI_SERVICE_URL=http://localhost:8001` in `backend/.env`

**Module not found errors**
- Make sure your venv is activated in the terminal
- Run `pip list` and verify `fastapi`, `uvicorn`, `openai`, `pinecone`, `motor`, `beanie` are all present
- If missing, re-run: `pip install -r ai-tutor/requirements.txt && pip install -r backend/requirements.txt`

**Frontend build errors**
```bash
cd frontend && rm -rf node_modules && npm install
```

**Pinecone is empty**
- Upload content via Teacher Portal → Courses → Create Topic
- Or use the AI Tutor `/embed` endpoint directly

**Port already in use**
```bash
# Windows — find and kill process on a port:
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# Mac / Linux:
lsof -i :8000
kill -9 <PID>
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| AI Service | Python 3.12, FastAPI, OpenAI GPT-4o |
| LMS Backend | Python 3.12, FastAPI, Beanie/Motor |
| Vector DB | Pinecone (serverless, 1536d) |
| Database | MongoDB Atlas |
| Frontend | Next.js 14, TypeScript, Tailwind CSS |
| Real-time | FastAPI WebSocket |
| Auth | bcrypt + session-based |

---

Full spec: **[SPECIFICATION.md](./SPECIFICATION.md)** | Plan: **[IMPLEMENTATION_PLAN.md](./IMPLEMENTATION_PLAN.md)**
