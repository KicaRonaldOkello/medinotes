# MediNotes Pro

MediNotes Pro is a web app for clinicians: you sign in, enter visit details and consultation notes, and get a **streaming AI response** with a structured summary for the record, suggested next steps, and a patient-friendly email draft. The UI is **Next.js** (static export) with **Clerk** for authentication; the consultation endpoint is implemented as **FastAPI** with OpenAI (see `backend/server.py` and the `Dockerfile` for the combined static + API layout).

## Environment variables

Create files in the **project root** (they are gitignored). Do not commit real keys.

### Next.js (local `npm run dev` / Vercel)

Add a **`.env.local`** file with your Clerk application keys (from the [Clerk dashboard](https://dashboard.clerk.com)):

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Clerk publishable key (safe for the browser) |
| `CLERK_SECRET_KEY` | Clerk secret key (server-side; required for Clerk in Next.js) |

### Full stack with Docker (UI + `/api/consultation`)

The Docker image builds the static site and serves it from **FastAPI** on port 8000. Use a **`.env`** file or `-e` flags at `docker run` time for runtime secrets:

| Variable | Purpose |
| --- | --- |
| `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` | Passed as a Docker **build arg** so the static bundle includes Clerk |
| `CLERK_JWKS_URL` | Clerk JWKS URL for validating JWTs in FastAPI |
| `OPENAI_API_KEY` | OpenAI API key (used by the streaming completion) |

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

The Next.js app uses **static export**; Vercel hosts that bundle. Live **consultation streaming** is provided by **FastAPI** in this repo (for example via the Docker image), not by the static export alone, unless you host the API elsewhere and point the client at it.

## Deploy on Vercel (GitHub Actions)

Pushes to `main` run `.github/workflows/deploy-vercel.yml`. In the GitHub repo, add these **Actions secrets**: `VERCEL_TOKEN`, `VERCEL_ORG_ID`, and `VERCEL_PROJECT_ID` (from the Vercel project and a Vercel account token). Also configure the same Clerk (and any other) **environment variables** in the Vercel project dashboard for production builds.

## Learn More

- [Next.js Documentation](https://nextjs.org/docs)
- [Clerk + Next.js](https://clerk.com/docs/quickstarts/nextjs)
