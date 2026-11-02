# Video Intelligence

Video Intelligence is a React + TypeScript dashboard connected to a Django REST backend. It does not seed fake records or return mock analysis. The library starts empty and displays records saved in Django.

## Local setup

In local development Vite proxies `/api` requests to Django at `http://127.0.0.1:8000`, so no frontend API secret or `.env.local` file is needed. If Django uses a different host or port, set `VITE_DEV_API_PROXY_TARGET` in the Vite environment (for example `http://127.0.0.1:8011`). Development requests always use this same-origin proxy; production builds use `VITE_API_BASE_URL`.

Frontend terminal:

```powershell
npm install
npm run dev
```

Backend terminal:

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
# Set OPENAI_API_KEY in backend/.env before using analysis or uploads.
python manage.py migrate
python manage.py runserver 8000
```

Configure `OPENAI_API_KEY` only in `backend/.env`; never put it in a `VITE_` variable. `OPENAI_MODEL` defaults to `gpt-4o-mini`. Without a key, AI endpoints return a setup error rather than mock results.

## Video processing

Pasted transcripts go to Django for OpenAI analysis and are saved with the resulting video. For video uploads, Django uses the bundled FFmpeg binary from `imageio-ffmpeg` to extract audio, sends it to OpenAI Whisper for transcription, then analyzes and saves the transcript and video metadata. Supported formats are MP4, MOV, and WEBM. Uploads are capped at 500 MB; extracted audio must fit OpenAI's 25 MB transcription limit.

## API endpoints

- `GET /api/health/`
- `POST /api/analyze/` with `{ "title": "...", "transcript": "..." }`
- `POST /api/videos/upload/` as multipart form data (`file`, optional `title`, optional `sourceUrl`)
- `GET|POST /api/videos/`
- `GET|PUT|PATCH|DELETE /api/videos/<uuid>/`
- `POST /api/videos/<uuid>/reanalyze/`

The backend database is SQLite for this proof of concept. Transcripts are retained for reanalysis. SharePoint/Graph integration, per-user libraries, token authentication, and automatic background synchronization are implemented in Django. Keep the backend worker service running for initial and daily scans; synchronization does not depend on the dashboard being open. Before scaling to multiple backend instances, move SQLite and the worker queue to shared production database/queue infrastructure.

## Build and deploy

Run `npm run build` for a frontend production build. Deploy the Vite frontend to Vercel and set `VITE_API_BASE_URL` to the separately hosted Django service. Django is not deployed by the Vercel frontend project; set `DJANGO_SECRET_KEY`, `DJANGO_DEBUG=false`, `DJANGO_ALLOWED_HOSTS`, `CORS_ALLOWED_ORIGINS`, `OPENAI_API_KEY`, and optionally `OPENAI_MODEL` on the backend host.

OpenAI summaries are a first pass and the knowledge readiness score is heuristic. SharePoint app-only access must be granted the minimum required read permission for the configured folder.
