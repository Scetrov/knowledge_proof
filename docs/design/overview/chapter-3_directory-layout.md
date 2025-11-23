# Chapter 3: Directory Layout and Tooling

## 3.1 Initial Nx workspace

Use npm or pnpm at root.

- Initialize workspace:

  - `npx create-nx-workspace@latest sui-workspace`

- Choose integrated repo.

Add Next.js app via Nx generator:

- `npx nx generate @nx/next:app frontend`

Add Rust app manually:

- `cargo new --bin api` inside `apps/api`.
- Add `apps/api/project.json` with Nx `run-commands` targets (`build`, `test`, `serve`).

Add libraries:

- `libs/shared-types`
- `libs/api-client`
- `libs/config`

You can ask the LLM to:

> Generate a minimal Nx `project.json` for a Rust app in `apps/api` that has `build`, `test`, and `serve` targets using `nx:run-commands` and `cargo`.
