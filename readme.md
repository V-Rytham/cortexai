# CortexAI

> Multi-agent AI workspace with autonomous intent routing, microservice architecture, dynamic code execution sandboxes, and vector-backed document intelligence.

[![React](https://img.shields.io/badge/React_18-61DAFB?style=flat-square&logo=react&logoColor=black)](https://react.dev/)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)](https://expressjs.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-1C3C3C?style=flat-square&logo=langchain&logoColor=white)](https://langchain-ai.github.io/langgraphjs/)
[![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io/)
[![Qdrant](https://img.shields.io/badge/Qdrant_Vector_DB-DC2626?style=flat-square&logo=qdrant&logoColor=white)](https://qdrant.tech/)
[![AWS S3](https://img.shields.io/badge/AWS_S3-569A31?style=flat-square&logo=amazons3&logoColor=white)](https://aws.amazon.com/s3/)
[![Razorpay](https://img.shields.io/badge/Razorpay-02042B?style=flat-square&logo=razorpay&logoColor=white)](https://razorpay.com/)

---

## Overview

Instead of jumping between separate tools for conversation, live web search, coding sandboxes, and PDF Q&A, I built **CortexAI** as an integrated multi-agent platform:
- **Autonomous Intent Routing**: A central LangGraph state graph analyzes prompts and file attachments to dispatch tasks to specialized agent pipelines automatically.
- **Decoupled Microservices**: Express API Gateway with session authentication, Redis caching, and independent services for Auth, Chat, Agents, and Billing.
- **Live Code Sandbox**: Generates and previews HTML/CSS/JS in Monaco Editor with live iframe rendering.
- **Document Intelligence**: Chunks uploaded PDFs, embeds them with Gemini, and stores vectors in Qdrant for semantic RAG retrieval.

---

## Screenshots

| Live Code Sandbox & Artifact Preview | Credits & Razorpay Billing Drawer |
| :---: | :---: |
| ![Live Coding Artifact](images/Screenshot%202026-10-05%20193723.png) | ![Billing and Credits](images/Screenshot%202026-10-05%20193759.png) |

---

## Architecture

```mermaid
flowchart TD
    Client["React + Vite Frontend (Redux Toolkit + Tailwind)"]

    subgraph Edge ["API Gateway Layer"]
        Gateway["Express API Gateway (:5000)<br/>Session Auth, Rate Limiting, Proxy"]
    end

    subgraph Services ["Microservices Layer"]
        AuthSvc["Auth Service (:5001)<br/>Firebase OAuth, Redis Sessions, Users"]
        ChatSvc["Chat Service (:5002)<br/>Conversations, Messages, Artifacts"]
        AgentSvc["Agent Orchestrator (:5003)<br/>LangGraph State Graph & Routing"]
        BillingSvc["Billing Service (:5004)<br/>Razorpay Orders & Verification"]
    end

    subgraph AgentPipelines ["Specialized Agents"]
        ChatAgent["Chat (Groq / LLaMA 3)"]
        CodingAgent["Coding Sandbox (OpenRouter / Monaco)"]
        SearchAgent["Web Search (Tavily Engine)"]
        PDFRAG["Document RAG (Gemini + Qdrant)"]
        VisionAgent["Vision Analysis (Gemini)"]
        FileGen["File Generation (PDFKit / PptxGenJS / S3)"]
    end

    subgraph DataStore ["Databases & Storage"]
        MongoDB[("MongoDB Atlas")]
        RedisCache[("Redis Cache & Rate Limiting")]
        QdrantDB[("Qdrant Vector DB")]
        S3Bucket[("AWS S3")]
    end

    Client -->|HTTPS + HttpOnly Cookies| Gateway
    Gateway --> AuthSvc
    Gateway --> ChatSvc
    Gateway --> AgentSvc
    Gateway --> BillingSvc

    AgentSvc --> ChatAgent
    AgentSvc --> CodingAgent
    AgentSvc --> SearchAgent
    AgentSvc --> PDFRAG
    AgentSvc --> VisionAgent
    AgentSvc --> FileGen

    AuthSvc & ChatSvc & BillingSvc --> MongoDB
    Gateway & AuthSvc --> RedisCache
    PDFRAG --> QdrantDB
    FileGen --> S3Bucket
```

---

## Key Highlights

- **LangGraph State Routing**: Dynamically routes requests to `chat`, `coding`, `search`, `pdf`, `ppt`, or `vision` pipelines based on prompt semantics and multipart file payloads.
- **Document RAG with Qdrant**: Recursive text chunking, dense vector embeddings via Google Gemini, and cosine similarity lookup with source citations.
- **Monaco Code Sandbox**: Live in-browser editing and sandboxed iframe rendering with syntax highlighting and instant download.
- **Multi-Provider Fallback**: Groq for low-latency dialogue, OpenRouter for code generation, Gemini for vision and embeddings, and Tavily for live web search.
- **Credit Allocation & Billing**: Sliding-window rate limiting via Redis, atomic credit deductions per model tier, and cryptographically verified Razorpay orders.

---

## Service Breakdown

| Directory | Responsibility | Port | Tech Stack |
| :--- | :--- | :--- | :--- |
| `backend/gateway` | Public entry point; validates JWT cookies, verifies Redis sessions, reverse-proxies requests | `:5000` | Express, Redis, Axios |
| `backend/services/auth` | Firebase OAuth token verification, user profile management, credit balance sync | `:5001` | Firebase Admin, MongoDB |
| `backend/services/chat` | Persistent conversations, message histories, and generated artifact snapshots | `:5002` | Express, Mongoose |
| `backend/services/agent` | LangGraph routing, tool calling, vector search, file generation, and S3 uploads | `:5003` | LangGraph, Gemini, Qdrant, S3 |
| `backend/services/billing`| Razorpay checkout order generation, webhook signature verification | `:5004` | Razorpay SDK, Mongoose |
| `frontend` | Single-page application, Monaco Editor code preview, chat streaming UI | `:5173` | React 18, Vite, Redux Toolkit, Tailwind |

---

## Setup & Local Run

### Prerequisites
- Node.js 20+
- Docker & Docker Compose (for Redis and Qdrant)
- MongoDB instance (local or Atlas)

### 1. Install Dependencies
```bash
git clone https://github.com/V-Rytham/cortexai.git
cd cortexai

npm install --prefix backend/gateway
npm install --prefix backend/services/auth
npm install --prefix backend/services/chat
npm install --prefix backend/services/agent
npm install --prefix backend/services/billing
npm install --prefix frontend
```

### 2. Start Supporting Services
```bash
docker compose -f backend/docker-compose.yml up -d
```

### 3. Start Development Servers
```bash
npm run dev --prefix backend/gateway
npm run dev --prefix backend/services/auth
npm run dev --prefix backend/services/chat
npm run dev --prefix backend/services/agent
npm run dev --prefix backend/services/billing
npm run dev --prefix frontend
```

Web client runs at `http://localhost:5173`.
