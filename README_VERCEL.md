# Deploying the Backend to Vercel (Docker)

This project uses FastAPI (ASGI) for the backend. Vercel's serverless functions are not ideal for long-lived ASGI servers, so this repository is configured to build a Docker container for the backend using the community Docker builder `@vercel/docker`.

Steps to deploy:

1. Make sure the repo is pushed to GitHub (already done).
2. Sign in to Vercel and create a new project, selecting your GitHub repository.
3. Vercel will detect `vercel.json` and use the Docker builder to build the `backend/Dockerfile`.
4. In the Vercel project settings -> Environment Variables, set any required secrets (OpenAI, Tavily, supabase keys, etc.). Also set `PORT` if needed (Vercel provides a runtime port).
5. Deploy the project. The backend will be available at the Vercel-provided domain (e.g., `https://your-app.vercel.app`).

Notes and caveats:
- The `backend/Dockerfile` installs dependencies from `requirements.txt` and runs `uvicorn backend.main:app` on `$PORT`.
- Make sure large local data files (like `data/downloads`) are not required by the deployed instance. Use cloud storage if needed.
- After deployment, set the Streamlit app's `IR_API` environment variable to the Vercel backend URL.
