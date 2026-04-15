# MediNotes Pro

MediNotes Pro is a web app for clinicians: you sign in, enter visit details and consultation notes, and get a **streaming AI response** with a structured summary for the record, suggested next steps, and a patient-friendly email draft. The UI is **Next.js** (static export) with **Clerk** for authentication; the consultation API is **FastAPI** + OpenAI (see `api/server.py`). **Vercel** picks up the ASGI app via `pyproject.toml` (`app = "api.server:app"`) and Python deps via `requirements.txt`; **Docker** serves the same API plus the exported static site (see `Dockerfile`).

## Environment variables

Create files in the **project root** (they are gitignored). Do not commit real keys.

### Next.js (local `npm run dev` / Vercel)

Add a **`.env.local`** file with your Clerk application keys (from the [Clerk dashboard](https://dashboard.clerk.com)):

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk publishable key (safe for the browser) |
| `CLERK_SECRET_KEY` | Clerk secret key (server-side; required for Clerk in Next.js) |

### FastAPI (Vercel and/or Docker)

Set these wherever the Python API runs (Vercel **Production** env, or Docker `-e` / `.env`):

| Variable | Purpose |
| --- | --- |
| `CLERK_JWKS_URL` | Clerk JWKS URL for validating JWTs in FastAPI |
| `OPENAI_API_KEY` | OpenAI API key (used by the streaming completion) |

### Docker only (combined static + API on one port)

The Docker image builds the static site and serves it from **FastAPI** on port 8000. You still need the runtime secrets above, plus:

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Passed as a Docker **build arg** so the static bundle includes Clerk |

## Run locally

**Frontend only** (landing + Clerk; consultation calls `/api/consultation` on the same origin):

```bash
npm ci
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

**Full app** (single origin: static files + FastAPI + consultation API), after setting build-time and runtime env as in the Dockerfile:

```bash
docker build --build-arg NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..." -t medinotes .
docker run -p 8000:8000 \
  -e CLERK_JWKS_URL="https://..." \
  -e OPENAI_API_KEY="sk-..." \
  medinotes
```

Then open [http://localhost:8000](http://localhost:8000).

## Deploy on Vercel (CLI)

```bash
vercel --prod
```

The Next.js app uses **static export**; Vercel hosts that bundle. **FastAPI** is configured for the same project via `pyproject.toml` and `requirements.txt` ([Vercel FastAPI](https://vercel.com/docs/frameworks/backend/fastapi)). Ensure `OPENAI_API_KEY` and `CLERK_JWKS_URL` are set in the Vercel project for **Production** (and Preview if you use it). For local `npm run dev`, the browser still calls `/api/consultation` on the Next dev server unless you proxy to FastAPI or use Docker on port 8000.

## Deploy on Vercel (GitHub Actions)

Pushes to `main` run `.github/workflows/deploy-vercel.yml`. In the GitHub repo, add these **Actions secrets**: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, and `VERCEL_PROJECT_ID` (from the Vercel project and a Vercel account token). Also configure the same Clerk (and any other) **environment variables** in the Vercel project dashboard for production builds.

## Learn More

- [Next.js Documentation](https://nextjs.org/docs)
- [Clerk + Next.js](https://clerk.com/docs/quickstarts/nextjs)
