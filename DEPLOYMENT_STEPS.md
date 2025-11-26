# Deployment Guide — Frontend (Streamlit) & Backend (Vercel Docker)

This document explains, step-by-step, how to deploy the frontend Streamlit app and the backend FastAPI app (using a Docker build) to production. It covers local testing, Streamlit Cloud for the frontend, and Vercel (Docker) for the backend. It also includes alternatives, troubleshooting, and security notes.

---

## Summary

- Frontend: `frontend/app.py` — a Streamlit app. Recommended hosting: Streamlit Cloud (easy) or any VM/hosting that supports Python.
- Backend: `backend/main.py` — FastAPI app. Current repo contains `backend/Dockerfile` and `vercel.json` configured to build the backend using `@vercel/docker` on Vercel.

## Prerequisites

- A GitHub repository for this project (your repo is already pushed).
- An account on Streamlit Cloud and Vercel (or alternative host like Render/Railway).
- Environment variables / secrets you will use (OpenAI, Tavily, Google CSE, Supabase keys, etc.).
- Docker knowledge is helpful for the backend Docker file.
- Ensure `requirements.txt` includes all Python dependencies.

## Local testing (recommended before deploying)

1. Create and activate a Python virtual environment (if you haven't already):

```bash
cd /workspaces/IR-FETCHER
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

2. Start the backend locally (Uvicorn):

```bash
# from repo root
/workspaces/IR-FETCHER/.venv/bin/python -m uvicorn backend.main:app --host 0.0.0.0 --port 8008
```

3. Start the Streamlit frontend locally (in a separate terminal):

```bash
cd frontend
/workspaces/IR-FETCHER/.venv/bin/streamlit run app.py
```

4. In the frontend, if running locally, you can keep `IR_API` unset — the app uses `http://127.0.0.1:8008` by default. If your backend runs elsewhere, set `IR_API` accordingly.

## Deploying the Frontend to Streamlit Cloud

Overview: Streamlit Cloud pulls from your GitHub repository and runs `frontend/app.py`. In Streamlit Cloud you configure the main file and environment variables.

1. Push your local changes to GitHub (if you haven't done so):

```bash
cd /workspaces/IR-FETCHER
git add -A
git commit -m "Prepare deployment: add Streamlit config and docs"
git push origin main
```

2. On Streamlit Cloud (https://streamlit.io/cloud):
   - Create a new app and connect your GitHub account.
   - Select the `nuwal01/IR-FETCHER` repository and the `main` branch.
   - Set the **Main file path** to `frontend/app.py`.

3. Set the required environment variables in the Streamlit app settings (Environment Variables / Secrets):
   - `IR_API` — the public URL of your deployed backend (e.g., `https://your-backend.vercel.app`).
   - Optional secrets if the frontend needs them (but best practice: keep secrets on the backend and use `/settings`).

4. Deploy and test the app using the provided Streamlit URL.

Notes for Streamlit Cloud:
- Streamlit Cloud installs dependencies from `requirements.txt` at the repo root. Ensure `requirements.txt` lists `streamlit` and other required packages.
- If you need to restrict the Python version, set the runtime in a `runtime.txt` or use the Streamlit Cloud settings.

## Deploying the Backend to Vercel (Docker)

Overview: This repository includes `backend/Dockerfile` and `vercel.json` to build a Docker container for the FastAPI app using the `@vercel/docker` builder. This approach ensures Uvicorn runs as a long-lived process.

Before you start: review `backend/Dockerfile` to confirm it installs required system packages and Python dependencies from `requirements.txt`.

1. Push changes to GitHub (already done):

```bash
git push origin main
```

2. In Vercel:
   - Log into Vercel and click **New Project** → Import Git Repository → select `nuwal01/IR-FETCHER`.
   - Vercel detects the `vercel.json` and will use the Docker builder for your `backend/Dockerfile`.
   - Configure Environment Variables in Vercel project settings: set the secrets you need, for example:
     - `OPENAI_API_KEY` (if used)
     - `TAVILY_API_KEY`
     - `GOOGLE_API_KEY`
     - `SUPABASE_SERVICE_ROLE_KEY`
     - `DATABASE_BACKEND` (e.g., `sqlite` or `supabase`)

3. Deploy: click **Deploy**. Vercel will build the Docker image and publish your app. The app will be available at `https://<project>.vercel.app`.

4. Verify endpoints:

```bash
curl https://<project>.vercel.app/health
# Expected: {"ok": true}

curl https://<project>.vercel.app/settings
# Returns settings payload (masked keys)
```

Important backend production notes:
- Vercel containers are ephemeral. Any files written to disk (downloads, SQLite DB) will not be persisted reliably across builds or instances. Use cloud storage (S3/GCS) and an external database (Supabase, Postgres) for persistence.
- If you need persistent storage, set `DATABASE_BACKEND=supabase` and configure Supabase environment variables in Vercel.
- For large dependency builds or specialized system libs, consider using a custom image or switching to a host like Render or Railway.

## Alternative: Deploy Backend to Render / Railway (recommended for simpler ASGI deployments)

Render and Railway typically accept a standard `Dockerfile` or a start command to run Uvicorn, and they provide better filesystem and persistent disk options (Render offers persistent disks for paid plans).

Quick Render steps (summary):

1. Create a `render` service and connect your repo.
2. Set the build command (if not using Docker):

```bash
pip install -r requirements.txt
```

Start command:
```bash
uvicorn backend.main:app --host 0.0.0.0 --port $PORT
```

3. Configure environment variables on Render.
4. Deploy and test.

Railway has a similar flow: link repo, configure service, set start command or Dockerfile, configure secrets, deploy.

## Post-deploy: Connect Frontend and Backend

1. Set `IR_API` for the Streamlit app to the backend's public URL (e.g., `https://<project>.vercel.app`). On Streamlit Cloud set this in the app's **Advanced settings → Environment variables**.
2. Visit the Streamlit app URL and perform a sample download request.
3. Monitor logs on Vercel and Streamlit Cloud for failures.

## CI / GitHub Actions (optional)

- You can add GitHub Actions to test, lint, and run unit tests on push. For deployments, use Vercel and Streamlit integration (both can auto-deploy when you push to the selected branch).

Example (very small) GitHub Actions snippet to run tests:

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: pip install -r requirements.txt
      - name: Run tests
        run: pytest -q
```

## Troubleshooting

- Backend not reachable: ensure Vercel deployment succeeded and the project domain is correct. Check Vercel build logs for Docker errors.
- Frontend can't reach backend on Streamlit Cloud: verify `IR_API` is set properly and there's no firewall blocking outgoing requests.
- SQLite DB missing / files lost: this is expected on serverless/ephermeral hosts. Use external DB and cloud storage.
- Long startup times or memory errors on Vercel Docker: reduce image size, pin dependency versions, or consider Render.

## Security & Secrets

- Never store API keys in the frontend repository or in client-side code. Keep keys on the backend and expose only the necessary endpoints.
- Use Vercel Secrets / Streamlit Cloud Secrets to store credentials.
- Limit access to storage buckets and databases using least privilege.

## Maintenance & Monitoring

- Configure log forwarding and alerts in Vercel and Streamlit Cloud.
- Set up regular backups for your database (if using a hosted DB).

---

If you'd like, I can:

- Create a GitHub Actions workflow to automatically deploy the backend to Vercel on each push.
- Prepare Render/Railway deployment config instead of Vercel, if you prefer.
- Update the Streamlit app's `IR_API` in Streamlit Cloud once you provide the backend URL (or I can deploy the backend and return the URL).
