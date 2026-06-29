# Full Stack AI System : System Design Document

## Overview

A production-style AI system built across 10 days consisting of:
- A REST API backend with authentication and persistent storage
- A React frontend with protected routes
- A hybrid RAG pipeline combining dense and sparse retrieval
- A streaming LLM endpoint
- A tool-calling agent with persistent conversation memory
- A real-time document ingestion pipeline via Kafka

---

## Services and responsibilities

| Service | Tech | Port | Responsibility |
|---------|------|------|----------------|
| Auth + Posts API | FastAPI + PostgreSQL | 8000 | User registration, login, JWT auth, CRUD |
| RAG Service | FastAPI + FAISS + BM25 | 8001 | Document retrieval and LLM streaming |
| Agent Service | FastAPI + Groq | 8002 | Tool-calling ReAct agent with memory |
| Streaming Pipeline | Kafka + FAISS | 8003 | Real-time document ingestion |
| Frontend | React + TypeScript | 5173/3000 | UI for all services |
| Database | PostgreSQL | 5432 | Users, posts, conversation memory |

---

## Architecture decisions and tradeoffs

### Why FastAPI over Flask or Django?
FastAPI is async-native, which matters for LLM streaming, blocking I/O would hold up other requests while waiting for tokens. It also generates OpenAPI docs automatically and has the best Pydantic integration for request validation. Flask has no async support. Django is too heavy for a microservice-style API.

### Why PostgreSQL over SQLite?
SQLite does not support concurrent writes, multiple async workers would cause locking errors. PostgreSQL handles concurrent connections properly and supports JSONB for storing conversation history as structured data.

### Why hybrid search (FAISS + BM25) over dense-only retrieval?
Dense retrieval alone fails on exact keyword queries, the embedding model focuses on semantic meaning and may miss documents that contain the exact query terms. BM25 alone fails on semantic queries where the words do not match. Combining both with normalised scores gives better coverage across both query types.

### Why Groq over OpenAI?
Free tier with no credit card required. Uses the same OpenAI-compatible API format so switching later requires changing one line. Llama 3.3 70B is competitive with GPT-4o on most tasks.

### Why Kafka over Redis Streams or a REST call for the pipeline?
Kafka decouples the producer and consumer completely. The producer publishes and moves on regardless of consumer state. Kafka retains messages so a consumer that restarts can replay from its last offset, no data is lost. Redis Streams would work at this scale but has weaker durability guarantees. A direct REST call is synchronous and creates tight coupling.

### Why Docker Compose over running services directly?
Single command starts the entire stack. Environment variables are centralised. Services communicate over an internal network by service name rather than localhost ports. Reproducing the environment on any machine takes one command.

---

## What I would change at 10x scale

### Auth + Posts API
Add Redis caching for frequently read data. Move from a single PostgreSQL instance to a read replica setup. Add rate limiting on auth endpoints to prevent brute force attacks.

### RAG Service
Replace in-memory FAISS with a persistent vector database (Qdrant or Weaviate) so the index survives restarts without replaying Kafka. Add a reranking step using a cross-encoder model for better result quality. Add query caching, identical queries should not re-embed and re-search.

### Agent Service
Add streaming to the agent endpoint, currently the entire answer is buffered before returning. Move conversation storage from a single PostgreSQL table to a dedicated cache like Redis for faster reads. Add timeout handling, LLM calls can hang.

### Streaming Pipeline
Add multiple Kafka partitions and consumer group members to process documents in parallel. Add a dead letter queue for documents that fail to embed. Persist the FAISS index to S3 on a schedule so recovery after restart does not require replaying the entire topic.

### Infrastructure
Add an Nginx reverse proxy in front of all services. Containerise everything and deploy to a single VM or a small Kubernetes cluster. Add Prometheus metrics and Grafana dashboards for monitoring latency and error rates. Add GitHub Actions for CI/CD, every push to main should rebuild and redeploy.

---

## Security gaps in the current implementation

- JWT secret key is hardcoded as a string in auth.py: should be in environment variables
- No rate limiting on any endpoint
- CORS allows localhost origins only : production would need the actual domain
- Calculator tool uses eval : sandboxed but still risky for untrusted input at scale
- No input validation length limits : a very long query would slow down embedding

---

## What each day built and why it was in that order

| Day | Built | Why at this stage |
|-----|-------|-------------------|
| 1 | FastAPI basics | Foundation : everything else is a FastAPI service |
| 2 | PostgreSQL + SQLAlchemy | Data persistence before auth |
| 3 | JWT auth | Security layer before building anything user-facing |
| 4 | React frontend | Needed a UI to test the auth flow end to end |
| 5 | Docker | Containerise before adding more services |
| 6 | FAISS + BM25 | Retrieval foundation before adding LLM |
| 7 | LLM streaming | RAG requires retrieval to already work |
| 8 | Agent + tools | Agents require both retrieval and LLM to work |
| 9 | Persistent memory | Memory requires the agent to work first |
| 10 | Kafka pipeline | Streaming ingestion extends the retrieval system |



## Interview preparation

### If asked: walk me through your system in 2 minutes

### If asked: what was the hardest technical problem you solved?

### If asked: what would you do differently if you had 3 months instead of 12 days?

### If asked: explain RAG to someone who has never heard of it
