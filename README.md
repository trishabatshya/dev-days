# 12-Day Full Stack AI System

Built in 12 days as preparation for Inter IIT Tech Meet 15.0.

## Repositories

| Repo | What it is |
|------|-----------|
| [day2-api](https://github.com/trishabatshya/day2-api) | FastAPI backend with PostgreSQL, JWT auth, full CRUD |
| [day4-frontend](https://github.com/trishabatshya/day4-frontend) | React + TypeScript frontend with auth, dashboard, RAG chat, agent UI |
| [day6-rag](https://github.com/trishabatshya/day6-rag) | Hybrid RAG pipeline: FAISS + BM25 + Groq LLM streaming |
| [day8-agent](https://github.com/trishabatshya/day8-agent) | ReAct agent with tool calling and PostgreSQL conversation memory |
| [day10-pipeline](https://github.com/trishabatshya/day10-pipeline) | Kafka streaming document ingestion with real-time FAISS indexing |
| [day10-kafka](https://github.com/trishabatshya/day10-kafka) | Docker Compose for local Kafka development |
| [fullstack](https://github.com/trishabatshya/fullstack) | Docker Compose orchestrating the full stack |

## Running the full system locally

Start these in order, each in its own terminal:

```bash
# 1. Auth backend
cd day2-api && source .venv/Scripts/activate && uvicorn app.main:app --reload

# 2. RAG service  
cd day6-rag && source .venv/Scripts/activate && uvicorn app.main:app --reload --port 8001

# 3. Agent service
cd day8-agent && source .venv/Scripts/activate && uvicorn app.main:app --reload --port 8002

# 4. Streaming pipeline (requires Kafka running)
cd day10-pipeline && source .venv/Scripts/activate && uvicorn app.main:app --reload --port 8003

# 5. Frontend
cd day4-frontend && npm run dev
```

Then open http://localhost:5173.

## System design

See [system-design.md](./system-design.md) for architecture decisions, tradeoffs, and scaling considerations.