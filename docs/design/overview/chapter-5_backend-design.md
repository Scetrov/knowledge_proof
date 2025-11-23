# Chapter 5: Backend Design (Rust / Axum)

## 5.1 Crates and dependencies

In `apps/api/Cargo.toml`, include:

- `axum` – routing
- `tokio` – async runtime
- `tower` / `tower-http` – middleware
- `serde`, `serde_json` – JSON
- `sqlx` (with `postgres` & `runtime-tokio-native-tls` features) – DB
- `uuid` – UUIDs
- `jsonwebtoken` – JWT validation / issuing
- `chrono` – timestamps
- `dotenvy` or `config` – env configuration
- `tracing`, `tracing-subscriber` – logging

Ask the LLM:

> Create a `Cargo.toml` for an Axum-based API named `api` that uses sqlx + Postgres, jsonwebtoken, tracing, and serde. Include sensible versions and features.

## 5.2 Database schema

Use SQL migrations. Schema:

### `users`

- `id` UUID PK (default `gen_random_uuid()`).
- `sui_address` TEXT UNIQUE NOT NULL.
- `zklogin_subject` TEXT NULL.
- `display_name` TEXT NULL.
- `avatar_url` TEXT NULL.
- `created_at` TIMESTAMPTZ DEFAULT `now()`.
- `updated_at` TIMESTAMPTZ DEFAULT `now()`.

### `collections`

- `id` UUID PK.
- `slug` TEXT UNIQUE.
- `title` TEXT.
- `description` TEXT.
- `created_at` TIMESTAMPTZ.

### `lessons`

- `id` UUID PK.
- `collection_id` UUID FK → `collections(id)`.
- `slug` TEXT UNIQUE.
- `title` TEXT.
- `summary` TEXT.
- `estimated_minutes` INT.
- `order_index` INT.
- `body_md` TEXT.
- `created_at` TIMESTAMPTZ.

### `quizzes`

- `id` UUID PK.
- `lesson_id` UUID FK → `lessons(id)`.
- `question` TEXT.

### `quiz_answers`

- `id` UUID PK.
- `quiz_id` UUID FK → `quizzes(id)`.
- `answer_text` TEXT.
- `is_correct` BOOLEAN.

### `user_lesson_progress`

- `user_id` UUID FK → `users(id)`.
- `lesson_id` UUID FK → `lessons(id)`.
- `completed` BOOLEAN.
- `completed_at` TIMESTAMPTZ.
- PK (`user_id`, `lesson_id`).

### `user_quiz_attempts`

- `id` UUID PK.
- `user_id` UUID FK → `users(id)`.
- `quiz_id` UUID FK → `quizzes(id)`.
- `is_correct` BOOLEAN.
- `created_at` TIMESTAMPTZ.

Ask the LLM:

> Write sqlx migrations for Postgres to create the `users`, `collections`, `lessons`, `quizzes`, `quiz_answers`, `user_lesson_progress`, and `user_quiz_attempts` tables as specified.

## 5.3 API endpoints

Base path: `/api`.

Unauthenticated (public):

- `GET /api/collections`

  - Returns list of collections with basic info and counts.

- `GET /api/collections/{slug}`

  - Returns collection details + list of lessons (without full body).

- `GET /api/lessons/{slug}`

  - Returns full lesson data:

    - Title, summary, `body_md`, `quizzes` (questions + answers, but mark correct answer only server-side or omit `is_correct`).

Authenticated:

- `POST /api/auth/exchange`

  - See auth section.

- `GET /api/me`

  - Returns current user info based on app JWT.

- `GET /api/progress`

  - Returns per-lesson completion + quiz correctness for the current user.

- `POST /api/lessons/{lesson_id}/complete`

  - Marks lesson complete for current user.

- `POST /api/quizzes/{quiz_id}/answer`

  - Body: `{ "answer_id": "<uuid>" }`.
  - Checks correctness, records attempt, returns `{ "correct": bool, "explanation": string }`.

Ask the LLM:

> Generate Axum route setup and handler function signatures for the endpoints above, using sqlx for DB access and a middleware that injects `User` into the request when an app JWT is valid.
