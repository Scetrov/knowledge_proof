# Chapter 10: Milestones with LLM Prompts

Use this sequence as an implementation roadmap.

## Milestone 1: Monorepo + skeleton apps

- Set up Nx.
- Add Next.js `frontend`.
- Add Rust `api` with Axum "hello world".

LLM prompt example:

> Help me initialize an Nx workspace with a Next.js app in `apps/frontend` and a Rust Axum app in `apps/api`, including Nx `project.json` for the Rust app with `build`, `serve`, and `test` targets.

## Milestone 2: DB schema + migrations

- Add Postgres, migrations, and sqlx integration.

LLM prompt example:

> Using sqlx and Postgres, write migrations and Rust setup code (connection pool, health check endpoint) for the schema we defined (`users`, `collections`, `lessons`, `quizzes`, `quiz_answers`, `user_lesson_progress`, `user_quiz_attempts`).

## Milestone 3: Sui zkLogin auth

- Implement `/login` and `/auth/callback` in Next.js.
- Implement `POST /api/auth/exchange`.
- Implement app JWT issuance and verification middleware.

LLM prompt example:

> Implement a zkLogin-based sign-in flow: Next.js routes for `/login` and `/auth/callback`, a Rust Axum endpoint `/api/auth/exchange` that verifies zkLogin tokens and returns an app JWT, and middleware to protect other endpoints using that app JWT.

## Milestone 4: Public content endpoints

- Implement `GET /api/collections`, `GET /api/collections/{slug}`, `GET /api/lessons/{slug}`.
- Frontend pages to list collections and show lesson content.

LLM prompt example:

> Implement Axum handlers for the public content endpoints and corresponding Next.js pages for `/collections/[slug]` and `/lessons/[slug]`, using a shared TypeScript/JSON schema.

## Milestone 5: Progress and quizzes

- Implement `GET /api/progress`, `POST /api/lessons/{id}/complete`, `POST /api/quizzes/{id}/answer`.
- Wire up quiz and progress UI on the frontend.

LLM prompt example:

> Implement the authenticated progress endpoints in Rust and update the Next.js lesson page to render quizzes, call `answerQuiz`, and mark lessons complete, updating a progress bar on the collection page.

## Milestone 6: Docker + docker-compose

- Write Dockerfiles for both apps.
- Write `docker-compose.yml`.

LLM prompt example:

> Produce production-ready multi-stage Dockerfiles for the Next.js and Rust apps, plus a docker-compose file to run them with Postgres locally.

## Milestone 7: CI (GitHub Actions) and polish

- Add Nx-aware CI workflow.
- Add logging, error handling, and minimal styling.

LLM prompt example:

> Generate a GitHub Actions workflow that installs Node and Rust, runs Nx lint/test/build for `frontend` and `api`, and builds Docker images, caching dependencies where appropriate.
