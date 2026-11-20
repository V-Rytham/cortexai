# CortexAI

> **Production-grade multi-agent AI workspace with autonomous intent routing, microservice architecture, dynamic code execution sandboxes, and vector-backed document intelligence.**

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

## 📌 Why I Built This

Most modern AI apps are thin wrappers around a single LLM API endpoint. As a final-year CS student wanting to understand how real-world, enterprise-grade AI platforms operate, I designed **CortexAI** from first principles to explore:
1. **Decoupled Microservices**: Splitting monolithic AI logic into isolated services (API Gateway, Auth, Chat/Artifacts, Agent Orchestrator, Billing) communicating over internal networks.
2. **Autonomous Intent Routing**: Instead of forcing the user to manually toggle models, a central **LangGraph** state machine analyzes prompt semantics and multimodal attachments (PDFs, images) to dynamically select the optimal agent pipeline.
3. **End-to-End Multimodal Execution**: Beyond simple text output, CortexAI generates downloadable slide decks (PPTX), formatted PDFs, runs live code in an interactive Monaco sandbox, searches the live web, and indexes complex documents into a vector database for semantic retrieval.

---

## 📸 Product Walkthrough

| Live Interactive Code Sandbox & Artifact Preview | Credit-Based Tiering & Razorpay Billing Drawer |
| :---: | :---: |
| ![Live Coding Artifact](images/Screenshot%202026-10-05%20193723.png) | ![Billing and Credits](images/Screenshot%202026-10-05%20193759.png) |

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    Client["React + Vite Frontend (Redux Toolkit + Tailwind)"]

    subgraph Edge ["API Gateway"]
        Gateway["Express API Gateway (:5000)<br/>• Session Auth Verification<br/>• Rate Limiting (Redis)<br/>• Microservice Reverse Proxy"]
    end

    subgraph Services ["Microservices Layer"]
        AuthSvc["Auth Service (:5001)<br/>• Firebase Google OAuth<br/>• Redis Session Tokens<br/>• User Model"]
        ChatSvc["Chat Service (:5002)<br/>• Conversations & Messages<br/>• Artifact Snapshots<br/>• MongoDB Storage"]
        AgentSvc["Agent Orchestrator (:5003)<br/>• LangGraph State Graph<br/>• Prompt Routing Engine"]
        BillingSvc["Billing Service (:5004)<br/>• Razorpay Orders & Webhooks<br/>• Credit Allocation Engine"]
    end

    subgraph AgentPipelines ["Specialized Agent Engines"]
        ChatAgent["General Chat (Groq / LLaMA 3)"]
        CodingAgent["Coding Sandbox (OpenRouter / Monaco)"]
        SearchAgent["Live Web Search (Tavily Engine)"]
        PDFRAG["Document RAG (Gemini + Qdrant Vector DB)"]
        VisionAgent["Vision Analysis (Gemini 1.5 Pro)"]
        FileGen["Media Generation (PDFKit / PptxGenJS / S3)"]
    end

    subgraph DataStore ["Databases & Cloud Storage"]
        MongoDB[(MongoDB Atlas)]
        RedisCache[(Redis Cache & Rate Limiting)]
        QdrantDB[(Qdrant Vector DB)]
        S3Bucket[(AWS S3 Presigned URLs)]
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

## ⚙️ Key Engineering Highlights

### 1. LangGraph Autonomous Router
Rather than relying on basic `if/else` keyword matching, requests pass through a compiled LangGraph state graph. The routing node inspects user prompts, conversational context from Redis, and multipart file payloads to classify user intent:
- `Auto Mode`: Routes to `chat`, `coding`, `search`, `pdf`, `ppt`, or `vision`.
- `Deterministic Override`: Users can explicitly pin specific agents when they need strict task execution.

### 2. Retrieval-Augmented Generation (RAG) on Documents
- Uploaded PDFs are parsed, split into semantically coherent text chunks, and embedded via Google Gemini Embeddings.
- Chunk vectors and metadata are stored in a dedicated **Qdrant Vector Database** collection.
- Top-k cosine similarity matches are injected into the agent prompt context with source page citations.

### 3. In-Browser Live Code Sandbox
- Generates HTML, CSS, JavaScript, and React snippets on the fly.
- Rendered in a sandboxed iframe with Monaco Editor integration, allowing real-time code editing, live DOM preview, and one-click copy/download.

### 4. Resilient Multi-Provider LLM Orchestration
- **Groq**: Ultra-low-latency conversational inference.
- **OpenRouter**: Specialized programming and code reasoning models.
- **Google Gemini**: Multimodal vision analysis and dense text embeddings.
- **Tavily AI**: Context-rich web search extraction with image indexing.

### 5. Production Billing & Redis Rate Limiting
- Dynamic credit deduction matrix: Different models consume credits proportionally based on token and compute intensity.
- Cryptographic verification of Razorpay signatures on the backend before crediting balances.
- Sliding-window rate limiter powered by Redis to guard endpoints against abuse.

---

## 🛠️ Microservice Directory Breakdown

| Directory | Responsibility | Port | Primary Tech |
| :--- | :--- | :--- | :--- |
| `backend/gateway` | Public entry point; validates JWT cookies, verifies Redis sessions, reverse-proxies requests | `:5000` | Express, Redis, Axios |
| `backend/services/auth` | Firebase OAuth token verification, user profile management, credit balance sync | `:5001` | Firebase Admin, MongoDB |
| `backend/services/chat` | Persistent conversations, message histories, and generated artifact snapshots | `:5002` | Express, Mongoose |
| `backend/services/agent` | LangGraph routing, tool calling, vector search, file generation, and S3 uploads | `:5003` | LangGraph, Gemini, Qdrant, S3 |
| `backend/services/billing`| Razorpay checkout order generation, webhook signature verification | `:5004` | Razorpay SDK, Mongoose |
| `frontend` | Single-page application, Monaco Editor code preview, chat streaming UI | `:5173` | React 18, Vite, Redux Toolkit, Tailwind |

---

## 🚀 Local Development Setup

### Prerequisites
- Node.js `20.x` or higher
- Docker & Docker Compose (for Redis and Qdrant)
- MongoDB instance (local or MongoDB Atlas URI)
- API Keys: Groq, Gemini, OpenRouter, Tavily, AWS S3, Firebase Admin, Razorpay

### 1. Clone & Install Dependencies
```bash
git clone https://github.com/V-Rytham/cortexai.git
cd cortexai

# Install root dependencies
npm install --prefix backend/gateway
npm install --prefix backend/services/auth
npm install --prefix backend/services/chat
npm install --prefix backend/services/agent
npm install --prefix backend/services/billing
npm install --prefix frontend
```

### 2. Start Supporting Infrastructure
Run Redis via Docker Compose:
```bash
docker compose -f backend/docker-compose.yml up -d
```

### 3. Environment Configuration
Populate `.env` files in:
- `backend/gateway/.env` (`PORT=5000`, `REDIS_URL`, microservice URLs)
- `backend/services/auth/.env` (`PORT=5001`, `MONGO_URI`, Firebase keys)
- `backend/services/chat/.env` (`PORT=5002`, `MONGO_URI`)
- `backend/services/agent/.env` (`PORT=5003`, `GROQ_API_KEY`, `GEMINI_API_KEY`, `QDRANT_URL`, `AWS_ACCESS_KEY_ID`, etc.)
- `backend/services/billing/.env` (`PORT=5004`, `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`)
- `frontend/.env` (`VITE_SERVER_URL=http://localhost:5000`)

### 4. Run Development Servers
Start each service using npm scripts:
```bash
npm run dev --prefix backend/gateway
npm run dev --prefix backend/services/auth
npm run dev --prefix backend/services/chat
npm run dev --prefix backend/services/agent
npm run dev --prefix backend/services/billing
npm run dev --prefix frontend
```

The web client will be available at `http://localhost:5173`.

---

## 🧑‍💻 Author

**Rytham Maalpani**  
- Final Year Computer Science Undergraduate  
- GitHub: [@V-Rytham](https://github.com/V-Rytham)
