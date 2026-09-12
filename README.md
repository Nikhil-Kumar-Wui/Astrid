# Astrid — Multi-Agent RAG Browser Agent 

Web agent that takes a plain-language goal, browses the real web with Playwright,
using multiple LLM agents (orchestrator, retriever, planner, verifier) backed
by a Postgres+pgvector memory (Supabase), with Google login.

## Architecture
```
User (Google login via Supabase Auth)
   -> Frontend (React) -- WebSocket --> Backend (FastAPI)
        -> Orchestrator agent (Groq LLM): goal -> subtasks
        -> Retriever agent: pgvector similarity search (RAG)
        -> Planner agent (Groq LLM): subtask + context -> next browser action
        -> Browser agent (Playwright): executes action, returns page state
        -> Verifier agent (Groq LLM): checks result isn't hallucinated
        -> Memory writer: embeds + stores outcome back into pgvector
```

## Setup

### 1. Supabase (auth + user DB + vector DB — all free tier)
1. Create project at supabase.com
2. Go to Authentication > Providers > enable Google, add your OAuth client id/secret
   (create one free at Google Cloud Console > APIs & Services > Credentials)
3. Go to SQL Editor, paste and run `backend/db/schema.sql`
4. Copy your Project URL, anon key, and service_role key into `.env`

### 2. Groq (free tier LLM)
1. Sign up at console.groq.com, generate an API key
2. Put it in `.env` as `GROQ_API_KEY`

### 3. Backend
```bash
cd backend
cp .env.example .env   # fill in real keys
pip install -r requirements.txt
playwright install chromium
uvicorn main:app --reload --port 8080
```

### 4. Frontend
```bash
cd frontend
npm install @supabase/supabase-js react react-dom
# set REACT_APP_SUPABASE_URL, REACT_APP_SUPABASE_ANON_KEY, REACT_APP_WS_URL in .env
npm start
```

## Deploy (Google Cloud, free tier)
```bash
# Backend -> Cloud Run
cd backend
gcloud run deploy astrid-backend \
  --source . \
  --region us-central1 \
  --allow-unauthenticated \
  --set-env-vars GROQ_API_KEY=...,SUPABASE_URL=...,SUPABASE_ANON_KEY=...,SUPABASE_SERVICE_ROLE_KEY=...

# Frontend -> Firebase Hosting (or Vercel, also free)
cd frontend
npm run build
firebase deploy
```

## Cost right now
Everything above is on free tiers: Supabase free project, Groq free tier,
Cloud Run free monthly quota, Firebase Hosting free tier. $0 to start.
Upgrade path later: swap Groq for Claude/GPT API, swap Supabase free tier
for a paid plan if you outgrow storage/rate limits — architecture doesn't change.
