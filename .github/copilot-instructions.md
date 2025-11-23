# Copilot Instructions

This file provides custom instructions to GitHub Copilot for this repository.

## Project Overview

Knowledge Proof is a SaaS-style learning platform. It provides:

- **Structured learning collections** (e.g., "Sui 101")
- **Interactive lessons** with markdown content
- **Quiz-based knowledge verification** with instant feedback
- **Progress tracking** per user (lesson completion + quiz attempts)

The platform uses **Sui zkLogin** for authentication, enabling "Sign in with Sui" functionality with web3-native identity.

**Design Reference**: See [docs/design/overview.md](../docs/design/overview.md) for the canonical design document.

## Repository Structure

This is an **Nx monorepo** with the following structure:

```
knowledge_proof/
├── apps/
│   ├── frontend/          # Next.js (TypeScript, React) application
│   └── api/               # Rust Axum API
├── libs/
│   ├── shared-types/      # Common TypeScript types for API payloads
│   ├── api-client/        # Typed TS client to call the Rust API
│   └── config/            # Shared config (endpoints, env var parsing)
├── content/               # Content source files (YAML/JSON)
│   └── sui-101/       # Example collection
│       ├── collection.yaml
│       └── lessons/       # Lesson files with quizzes
├── docs/
│   └── design/
│       └── overview/      # Modular design chapters
├── .github/
│   ├── workflows/         # CI/CD (GitHub Actions)
│   └── copilot-instructions.md
└── docker-compose.yml     # Local development environment
```

### Key Directories

- **apps/frontend**: Next.js App Router application handling UI, zkLogin flow, and session management
- **apps/api**: Rust Axum API with JWT validation, database access, and REST endpoints
- **libs/shared-types**: TypeScript interfaces shared between frontend and backend
- **libs/api-client**: Type-safe API client with authentication support
- **libs/config**: Environment variable parsing and configuration helpers
- **content/**: YAML/JSON files defining collections, lessons, and quizzes

## Build and Development

### Prerequisites

- **Node.js**: v18+ (for Next.js and Nx)
- **pnpm**: v8+ (preferred package manager)
- **Rust**: 1.70+ (for API)
- **Cargo**: Latest stable
- **Docker**: 20.10+ (for containerization)
- **Docker Compose**: v2+ (for local development)
- **PostgreSQL**: 14+ (via Docker or local)
- **Nx CLI**: `npm install -g nx` (optional but recommended)

### Setup

```bash
# Clone repository
git clone https://github.com/yourusername/knowledge_proof.git
cd knowledge_proof

# Install Node dependencies
pnpm install

# Set up environment variables
cp apps/api/.env.example apps/api/.env
cp apps/frontend/.env.example apps/frontend/.env

# Start PostgreSQL via Docker Compose
docker-compose up -d db

# Run database migrations
cd apps/api
sqlx migrate run

# Seed initial content
cargo run --bin seeder
cd ../..
```

### Build Commands

```bash
# Build all projects
nx run-many --target=build --all

# Build frontend only
nx build frontend

# Build Rust API
nx run api:build
# or directly: cd apps/api && cargo build --release

# Build Docker images
docker-compose build
```

### Testing

```bash
# Run all tests
nx run-many --target=test --all

# Test frontend
nx test frontend

# Test Rust API
nx run api:test
# or directly: cd apps/api && cargo test

# Run E2E tests (if configured)
nx e2e frontend-e2e
```

### Common Tasks

```bash
# Start all services in development mode
nx run-many --target=serve --all --parallel

# Serve frontend (port 3000)
nx serve frontend

# Serve Rust API (port 8080)
nx run api:serve
# or: cd apps/api && cargo run

# Lint all projects
nx run-many --target=lint --all

# Format code
nx format:write

# View dependency graph
nx graph

# Database operations
cd apps/api
sqlx migrate run              # Run migrations
sqlx migrate add <name>       # Create new migration
sqlx database reset           # Reset database

# Seed content
cd apps/api
cargo run --bin seeder
```

## Coding Standards

### TypeScript/React (Frontend & Libs)

- Use **TypeScript strict mode**
- Follow **React best practices** (hooks, functional components)
- Use **ESLint** and **Prettier** for linting and formatting
- Prefer **named exports** over default exports
- Use **Zod** or similar for runtime type validation where needed
- Component file structure: `ComponentName.tsx`, `ComponentName.module.css`
- Keep components small and focused (< 200 lines)
- Document complex logic with comments

### Rust (API)

- Follow **Rust 2021 edition** idioms
- Use **rustfmt** for formatting (`cargo fmt`)
- Use **clippy** for linting (`cargo clippy`)
- Prefer **explicit error handling** with `Result<T, E>`
- Use **anyhow** for application errors, **thiserror** for library errors
- Keep functions focused and testable
- Use **async/await** with Tokio runtime
- Document public APIs with doc comments (`///`)
- Use `tracing` for structured logging, not `println!`

### Database

- All schema changes must go through **sqlx migrations**
- Use **UUIDs** for primary keys
- Use `TIMESTAMPTZ` for timestamps (not `TIMESTAMP`)
- Always add indexes for foreign keys
- Use meaningful constraint names
- Never modify existing migrations; create new ones

### Git Workflow

- Branch naming: `feature/description`, `fix/description`, `docs/description`
- Commit messages: Follow [Conventional Commits](https://www.conventionalcommits.org/)
  - `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`
- Keep commits atomic and focused
- Squash before merging to main

## Architecture

### Authentication Flow (Sui zkLogin)

1. **zkLogin JWT** (Sui infrastructure): Proves web2 login, binds to Sui address (used once at login)
2. **App JWT** (application-issued): Contains `user_id` and `sui_address`, signed with app secret
3. Frontend stores app JWT in **HttpOnly cookie**
4. All API requests include `Authorization: Bearer <app_jwt>`
5. Rust middleware validates JWT and injects `User` into request context

### API Design

- **RESTful JSON endpoints** at `/api/*`
- Public endpoints: collections and lessons (read-only)
- Authenticated endpoints: user info, progress, quiz submissions
- Consistent error responses: `{ "error": "message", "code": "ERROR_CODE" }`
- Use HTTP status codes correctly (200, 201, 400, 401, 403, 404, 500)

### Database Schema

- **users**: Sui address, zkLogin subject, profile info
- **collections**: Learning collections (e.g., "Sui 101")
- **lessons**: Individual lessons with markdown content
- **quizzes**: Multiple-choice questions per lesson
- **quiz_answers**: Answer options with correctness flag
- **user_lesson_progress**: Lesson completion tracking
- **user_quiz_attempts**: Quiz attempt history

### Frontend Architecture

- **Next.js App Router** (`app/` directory)
- **Server Components** for initial data fetching
- **Client Components** for interactivity (quizzes, auth)
- **API client library** for type-safe backend calls
- **Markdown rendering** with `react-markdown` or similar
- Use React Query or SWR for data caching (optional)

## Dependencies

### Frontend

- **Next.js**: React framework with SSR/SSG
- **React**: UI library
- **TypeScript**: Type safety
- **TailwindCSS** (recommended): Utility-first styling
- **react-markdown**: Markdown rendering
- **Zod**: Schema validation

### Backend (Rust)

- **axum**: Web framework (routing, handlers)
- **tokio**: Async runtime
- **sqlx**: SQL toolkit with compile-time query checking
- **tower/tower-http**: Middleware
- **serde/serde_json**: JSON serialization
- **jsonwebtoken**: JWT handling
- **uuid**: UUID generation
- **chrono**: Date/time handling
- **tracing/tracing-subscriber**: Structured logging
- **dotenvy**: Environment variable loading

### Development

- **Nx**: Monorepo tooling and task orchestration
- **Docker**: Containerization
- **PostgreSQL**: Relational database

## Validation

Before committing, ensure:

1. **Code compiles and builds**:

   ```bash
   nx run-many --target=build --all
   ```

2. **All tests pass**:

   ```bash
   nx run-many --target=test --all
   ```

3. **Linting passes**:

   ```bash
   nx run-many --target=lint --all
   ```

4. **Formatting is correct**:

   ```bash
   nx format:check
   ```

5. **Rust checks pass**:

   ```bash
   cd apps/api
   cargo clippy -- -D warnings
   cargo fmt --check
   ```

6. **Database migrations are valid**:

   ```bash
   cd apps/api
   sqlx migrate run
   ```

7. **Docker images build** (if modifying Dockerfiles):
   ```bash
   docker-compose build
   ```

## Important Notes

- **Design document is canonical**: See [docs/design/overview.md](../docs/design/overview.md) for authoritative specification
- **Nx graph dependencies**: `frontend` depends on `shared-types` and `api-client`; `api-client` depends on `shared-types`
- **Environment variables**: Never commit `.env` files; use `.env.example` as templates
- **Database URL**: Use `DATABASE_URL` environment variable for sqlx
- **JWT secret**: Use strong random secret in production (min 32 bytes)
- **zkLogin configuration**: Requires Sui network configuration and OAuth provider setup
- **Content format**: Collections and lessons are defined in YAML files in `content/` directory
- **Seeding**: Use `cargo run --bin seeder` to populate database from content files

## Common Patterns

### Adding a new API endpoint

1. Define route in `apps/api/src/routes/mod.rs`
2. Create handler in appropriate module
3. Add types to `libs/shared-types`
4. Add client method to `libs/api-client`
5. Update tests

### Adding a new frontend page

1. Create route in `apps/frontend/app/`
2. Fetch data using API client
3. Render with appropriate components
4. Handle loading and error states
5. Add navigation links

### Creating a database migration

```bash
cd apps/api
sqlx migrate add descriptive_name
# Edit the generated migration file
sqlx migrate run
```

### Adding content

1. Create/edit YAML files in `content/`
2. Run seeder: `cd apps/api && cargo run --bin seeder`
3. Verify in database or through API
