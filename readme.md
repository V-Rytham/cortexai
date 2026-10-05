# CortexAI

An AI assistant where you can chat, search the web, generate code, and work with PDFs and images in one place. It picks an agent based on your request, or lets you choose one yourself.

![CortexAI chat with a generated calculator preview](images/Screenshot%202026-10-05%20193723.png)

## What it does

- Chat with saved conversation history.
- Search the web for answers that need current information.
- Generate code and preview HTML, CSS, and JavaScript in the app.
- Upload a PDF and ask questions about its contents, or upload an image for analysis.
- Generate downloadable PDFs, presentations, and images.
- Sign in with Google and buy credits through Razorpay.

## Built with

React, Redux Toolkit, Tailwind CSS, and Monaco Editor on the frontend. Node.js and Express on the backend, with MongoDB and Redis.

LangGraph routes requests between agents. Groq, Gemini, and OpenRouter handle the AI responses; Tavily handles search, Qdrant stores PDF embeddings, and AWS S3 stores generated files.

The backend is split into an API gateway and four services: auth, chat, agent, and billing.

## Run locally

You'll need Node.js, MongoDB, Redis, Qdrant, and credentials for the integrations above, Firebase, and Razorpay.

1. Run `npm install` in `frontend`, `backend`, `backend/gateway`, and each folder under `backend/services`.
2. Add `.env` files for the frontend, gateway, and services using the variables referenced in their source. Set distinct service ports and point the gateway's service URLs at them. Set `VITE_SERVER_URL` to the gateway URL and `FRONTEND_URL` to the frontend URL.
3. Set up Firebase authentication, update `frontend/utils/firebase.js` for your project, and add your Firebase Admin key at `backend/services/auth/serviceAccountKey.json`.
4. Start Redis with `docker compose -f backend/docker-compose.yml up -d` and have MongoDB and Qdrant running.
5. Run `npm run dev` in the gateway, each of the four services, and the frontend, using separate terminals.

## Plans and credits

<img src="images/Screenshot%202026-10-05%20193759.png" alt="Billing panel showing plans and credits" width="300">
