# Chapter 1: Goals and Requirements

Build a small SaaS-style learning platform:

- Monorepo using **Nx**
- **Frontend**: Next.js (TypeScript, React)
- **Backend**: Rust API (Axum or similar)
- **Auth**: Sui **zkLogin** ("Sign in with Sui")
- **Data**: Postgres
- **Deployment**: Docker images for frontend + backend (+ Postgres via compose)
- Features:

  - Collections (e.g. "Sui 101")
  - Lessons with markdown content
  - Quizzes per lesson
  - User progress (lesson completion + quiz attempts)
