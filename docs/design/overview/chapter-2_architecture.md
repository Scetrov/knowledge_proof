# Chapter 2: High-Level Architecture

## 2.1 Monorepo structure (Nx)

- `apps/frontend` – Next.js app
- `apps/api` – Rust Axum API
- `libs/shared-types` – common TypeScript types for API payloads
- `libs/api-client` – typed TS client to call the Rust API
- `libs/config` – shared config (endpoints, env var parsing helpers)

## 2.2 Service responsibilities

- **Frontend (Next.js)**:

  - Handles zkLogin UI flow.
  - Exchanges zkLogin data for an app JWT.
  - Stores session (JWT) and attaches it to API calls.
  - Renders collections, lessons, quizzes, progress.

- **Rust API**:

  - Verifies app JWT and maps to internal `user` table.
  - Manages content data (collections, lessons, quizzes).
  - Records user progress and quiz attempts.
  - Offers REST JSON endpoints.

- **Postgres**:

  - Persist users, content, progress.
