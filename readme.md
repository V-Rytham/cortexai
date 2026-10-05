# CortexAI

An AI assistant where you can chat, search the web, generate code, and work with PDFs and images in one place. It picks an agent based on your request, or lets you choose one yourself.

Built from scratch to learn how a full-stack AI app fits together: a React frontend, separate backend services, authentication, storage, payments, and multiple AI providers.

![CortexAI chat with a generated calculator preview](images/Screenshot%202026-10-05%20193723.png)

## What it does

- **Agent selection:** Choose Chat, Coding, Search, PDF, PPT, or Vision. Auto mode uses LangGraph to route your request, including uploaded files.
- **Chat and memory:** Conversations and messages are saved in MongoDB. Redis caches recent messages for follow-up questions.
- **Web search:** Tavily retrieves web results and images, which are passed to the chat agent to answer the question.
- **Coding assistant:** Generate code, ask for explanations, or get help with debugging and reviews. View generated files in Monaco Editor and preview HTML, CSS, and JavaScript.
- **PDF questions:** Uploaded PDFs are split into chunks and embedded with Gemini. Qdrant retrieves relevant passages for the answer.
- **Image analysis:** Upload an image and ask questions about it using Gemini's vision capabilities.
- **File generation:** Create PDFs with PDFKit, presentations with PptxGenJS, and images through Pollinations. Generated files are stored in an AWS S3 bucket.
- **Google login:** Firebase handles Google sign-in; the backend verifies the token and creates an HTTP-only cookie session stored in Redis.
- **Plans and credits:** Razorpay handles checkout, with payment signatures verified on the backend. Different agents consume different amounts of credits.
- **Rate limits:** Redis tracks requests per user and agent, with a retry message when the limit is reached.
- **Voice input:** Dictate a prompt through the browser's speech recognition API where supported.

## Backend breakdown

The backend uses a microservices architecture: an Express API gateway forwards requests to four separate Node.js services over HTTP.

| Folder | Responsibility |
| --- | --- |
| `backend/gateway` | Entry point for the frontend; checks sessions and forwards requests with user details. |
| `backend/services/auth` | Firebase token verification, user accounts, sessions, plans, and credit balances. |
| `backend/services/chat` | Creates conversations, updates titles, and stores messages and generated artifacts. |
| `backend/services/agent` | LangGraph routing, AI calls, search, file analysis, generation, and S3 uploads. |
| `backend/services/billing` | Razorpay orders, payment verification, and payment records; asks auth to update credits. |

Each service and the gateway has its own package and Dockerfile. The included Docker Compose file starts Redis.

## Frontend and integrations

- **React + Vite:** UI built with Tailwind CSS, Motion animations, Markdown responses, and syntax highlighting.
- **Redux Toolkit:** Separate slices keep user details, conversations, messages, loading state, and generated artifacts in sync.
- **AI providers:** Groq handles chat and routing, OpenRouter handles coding, and Gemini handles image analysis and PDF embeddings.
- **External services:** Firebase for login, Tavily for search, Pollinations for images, and Razorpay for payments.
- **Storage:** MongoDB for users, chats, and payments; Redis for sessions, memory, and rate limits; Qdrant for PDF vectors.
- **AWS S3:** Stores generated PDFs, presentations, and images, with expiring signed URLs for downloads.

## Run locally

You'll need Node.js, MongoDB, Redis, Qdrant, and credentials for the integrations above, Firebase, and Razorpay.

1. Run `npm install` in `frontend`, `backend`, `backend/gateway`, and each folder under `backend/services`.
2. Add `.env` files for the frontend, gateway, and services using the variables referenced in their source. Set distinct service ports and point the gateway's service URLs at them. Set `VITE_SERVER_URL` to the gateway URL and `FRONTEND_URL` to the frontend URL.
3. Set up Firebase authentication, update `frontend/utils/firebase.js` for your project, and add your Firebase Admin key at `backend/services/auth/serviceAccountKey.json`.
4. Start Redis with `docker compose -f backend/docker-compose.yml up -d` and have MongoDB and Qdrant running.
5. Run `npm run dev` in the gateway, each of the four services, and the frontend, using separate terminals.

## Plans and credits

<img src="images/Screenshot%202026-10-05%20193759.png" alt="Billing panel showing plans and credits" width="300">
