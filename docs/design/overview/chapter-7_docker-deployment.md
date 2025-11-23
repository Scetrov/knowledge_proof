# Chapter 7: Docker and Local Environment

## 7.1 Dockerfiles

Backend: `apps/api/Dockerfile` (multi-stage):

- Stage 1: build

  - `FROM rust:…`
  - `WORKDIR /app`
  - Copy workspace files.
  - Build `api` in release mode.

- Stage 2: runtime

  - `FROM gcr.io/distroless/cc` (or small base).
  - Copy binary.
  - Expose port (e.g., 8080).
  - `CMD ["/api"]`.

Frontend: `apps/frontend/Dockerfile` (multi-stage):

- Stage 1: builder

  - `FROM node:…`
  - Install deps, run `nx build frontend`.

- Stage 2: runtime

  - `FROM node:…`
  - Copy `.next`, `package.json`, etc.
  - `CMD ["node", "server.js"]` or `next start`.

Ask the LLM:

> Create multi-stage Dockerfiles for the Next.js frontend and Rust API in an Nx monorepo, optimizing for small final images and production use.

## 7.2 docker-compose.yml (local dev)

- Services:

  - `db` (Postgres)
  - `api` (Rust)
  - `frontend` (Next.js)

- Networks:

  - Shared network.

- Env:

  - `DATABASE_URL` for `api`.
  - `API_BASE_URL` / `NEXT_PUBLIC_API_BASE_URL` for frontend.
  - JWT secrets and zkLogin configuration as env vars.

Ask the LLM:

> Produce a `docker-compose.yml` that runs Postgres, the Rust API, and the Next.js frontend together, wiring ports and environment variables appropriately.
