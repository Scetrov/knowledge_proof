# Chapter 8: Nx Integration and CI

## 8.1 Nx project graph

- Ensure `nx.json` and project configs declare:

  - `frontend` depends on `shared-types` and `api-client`.
  - `api-client` depends on `shared-types`.

- Use tags (optional) to enforce boundaries (`scope:frontend`, `scope:backend`).

## 8.2 GitHub Actions CI

Workflow steps:

1. Checkout.
2. Setup Node and Rust.
3. Install JS deps: `npm ci` or `pnpm install`.
4. Cache Rust target dir.
5. `nx lint frontend`.
6. `nx test frontend`.
7. `nx run api:test` (or `cargo test`).
8. `nx build frontend`.
9. `nx run api:build` (or `cargo build --release`).
10. Build Docker images and push on `main`.

Ask the LLM:

> Create a GitHub Actions workflow `ci.yml` for an Nx monorepo with a Next.js frontend and Rust backend, running lint/test/build for both, and building Docker images on pushes to main.
