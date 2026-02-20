# Voice AI Agent

A real-time voice AI agent that uses **LiveKit** for WebRTC communication, **Groq** for AI inference (STT, LLM, TTS), and **RAG** over uploaded documents using **Jina AI embeddings** and **Firestore vector search**.

## Architecture

```
Frontend (React + Vite)  ←→  LiveKit Server (Docker)  ←→  LiveKit Agent (Python)
         |                                                        |
         └── FastAPI Backend ──┬── Knowledge Base (Firestore)     |
                               └── RAG Retrieval ←────────────────┘
```

## Prerequisites

- **Node.js** ≥ 20.x
- **Python** ≥ 3.11
- **Docker** (for LiveKit server)
- **API Keys:**
  - [Groq API Key](https://console.groq.com) — for STT, LLM, and TTS
  - [Jina AI API Key](https://jina.ai) — for embeddings
  - [Firebase Service Account JSON](https://console.firebase.google.com) — for Firestore

## Project Structure

```
voice-ai-agent/
├── client/                  # React Frontend (Vite + TailwindCSS)
│   └── src/
│       ├── components/      # UI components
│       ├── lib/             # Utilities & API helpers
│       └── App.jsx          # Main app layout
├── server/                  # FastAPI Backend + LiveKit Agent
│   ├── routers/             # API endpoints
│   ├── services/            # KB & RAG services
│   ├── agent/               # LiveKit voice agent
│   └── main.py              # FastAPI app
├── docker-compose.yml       # LiveKit server
└── livekit.yaml             # LiveKit config
```

## Setup & Run

### 1. Environment Variables

```bash
cp server/.env.example server/.env
```

Edit `server/.env` with your actual keys:

```env
GROQ_API_KEY=gsk_xxxxx
JINA_API_KEY=jina_xxxxx
LIVEKIT_URL=ws://localhost:7880
LIVEKIT_API_KEY=devkey
LIVEKIT_API_SECRET=secret
GOOGLE_APPLICATION_CREDENTIALS=/path/to/service-account.json
FIRESTORE_PROJECT_ID=your-project-id
```

### 2. Start LiveKit Server (Docker)

```bash
docker compose up -d
```

This starts the LiveKit SFU server on port 7880.

### 3. Start Backend (FastAPI)

```bash
cd server
python -m venv venv
source venv/bin/activate   # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### 4. Start Voice Agent

In a separate terminal:

```bash
cd server
source venv/bin/activate
python agent/voice_agent.py dev
```

### 5. Start Frontend

```bash
cd client
npm install
npm run dev
```

Open **http://localhost:5173** in your browser.

## Usage

1. **Edit System Prompt** — Use the left sidebar to customize what the agent knows and how it responds
2. **Upload Documents** — Drop PDF, TXT, MD, or DOCX files to create the knowledge base
3. **Start Call** — Click "Start Call" to begin a voice conversation
4. **Ask Questions** — Ask questions that require info from uploaded documents; the agent will search the KB automatically
5. **View Transcript** — Real-time transcript appears in the right sidebar

## Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | React + Vite, TailwindCSS v4, plain JS |
| WebRTC | LiveKit (`@livekit/components-react`) |
| Backend | FastAPI (Python) |
| Voice Agent | `livekit-agents` + `livekit-plugins-groq` |
| STT | Groq `whisper-large-v3-turbo` |
| LLM | Groq `llama-3.3-70b-versatile` |
| TTS | Groq `playai-tts` |
| Embeddings | Jina AI `jina-embeddings-v3` |
| Vector Store | Google Firestore (KNN vector search) |
| Infra | Docker Compose (LiveKit Server) |

## Known Limitations

- LiveKit server runs in dev mode (single API key). For production, configure proper key management.
- Firestore vector search requires a vector index to be created (the first query may give an error with instructions to create it).
- File size for uploads is limited by FastAPI default (can be configured).
- The voice agent reads the system prompt from a file at session start; changes take effect on the next call.
