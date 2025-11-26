# Deploying the Streamlit Frontend

This repository contains the Streamlit frontend for IR Downloader at `frontend/app.py`.

Quick steps to deploy on Streamlit Cloud:

1. Push the repository to GitHub (make sure your `frontend/app.py` is committed).
2. Go to https://streamlit.io/cloud and click **New app**.
3. Select the GitHub repo and the branch, then set the **Main file path** to `frontend/app.py`.
4. Set the following **Environment Variables / Secrets** in the Streamlit Cloud app settings:
   - `IR_API` — URL of your deployed backend API (for example `https://my-backend.example.com`). If omitted, the app defaults to `http://127.0.0.1:8008` which will not work on Streamlit Cloud.
   - Any provider keys you want to keep secret (OpenAI, Tavily, Google) as environment variables or in the backend settings if the backend exposes a `/settings` endpoint.
5. Deploy the app. Streamlit will install dependencies from the repository `requirements.txt`.

Local testing (optional):

1. Start the backend API locally (from repo root):

```bash
cd /workspaces/IR-FETCHER
/workspaces/IR-FETCHER/.venv/bin/python -m uvicorn backend.main:app --host 0.0.0.0 --port 8008
```

2. Start the frontend locally:

```bash
cd frontend
/workspaces/IR-FETCHER/.venv/bin/streamlit run app.py
```

Notes:
- The app reads `IR_API` from environment and falls back to `http://127.0.0.1:8008` by default.
- If your backend is not deployed yet, deploy it first (e.g., Render, Fly, Railway, or a cloud VM) and then set `IR_API` in Streamlit Cloud to that URL.
- If you want the Streamlit app to call the backend without exposing API keys to the browser, keep keys on the backend and use its `/settings` endpoint as designed.
