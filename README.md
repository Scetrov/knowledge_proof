# Knowledge Proof

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/Scetrov/knowledge_proof/actions)
[![Version](https://img.shields.io/badge/version-0.1.0-blue)](https://github.com/Scetrov/knowledge_proof/releases)

> A SaaS-style learning platform with structured collections, interactive lessons, quiz-based verification, and progress tracking. Built with Sui zkLogin authentication for web3-native identity.

## 🎯 Project Overview

Knowledge Proof provides an interactive learning experience where users can:

- **Explore structured learning collections** (e.g., "Sui 101", "Rust Primer")
- **Complete interactive lessons** with rich markdown content
- **Verify knowledge through quizzes** with instant feedback
- **Track progress** across lessons and collections
- **Sign in with Sui** using zkLogin for seamless web3 authentication

**For the complete design specification**, see [docs/design/overview.md](docs/design/overview.md).

## 🏗️ Architecture

This is an **Nx monorepo** with a clear separation of concerns:

```
knowledge_proof/
├── apps/
│   ├── frontend/          # Next.js 14+ (App Router, TypeScript, React)
│   └── api/               # Rust Axum API with JWT validation
├── libs/
│   ├── shared-types/      # TypeScript interfaces for API contracts
│   ├── api-client/        # Type-safe API client with auth support
│   └── config/            # Shared configuration and env parsing
├── content/               # YAML/JSON learning content
│   └── sui-101/          # Example: collection and lesson files
├── docs/
│   └── design/
│       └── overview/      # Modular design documentation (10 chapters)
└── docker-compose.yml     # Local development environment
```

### Key Services

- **Frontend (Next.js)**: Handles zkLogin UI flow, session management, and content rendering
- **Backend (Rust/Axum)**: REST API with JWT validation, database access, and progress tracking
- **Database (PostgreSQL)**: Stores users, content, quizzes, and progress data
- **Authentication (Sui zkLogin)**: Two-tier JWT system for web3-native sign-in


## 🚀 Quick Start for Contributors

### Prerequisites

Ensure you have the following installed:

- **Node.js**: v18+ ([Download](https://nodejs.org/))
- **pnpm**: v8+ (`npm install -g pnpm`)
- **Rust**: 1.70+ ([Install via rustup](https://rustup.rs/))
- **Docker**: 20.10+ ([Download](https://www.docker.com/))
- **Docker Compose**: v2+
- **PostgreSQL**: 14+ (via Docker or local installation)
- **Nx CLI** (optional): `npm install -g nx`

### Initial Setup

```bash
# 1. Clone the repository
git clone https://github.com/Scetrov/knowledge_proof.git
cd knowledge_proof

# 2. Install Node.js dependencies
pnpm install

# 3. Set up environment variables
cp apps/api/.env.example apps/api/.env
cp apps/frontend/.env.example apps/frontend/.env
# Edit .env files with your configuration

# 4. Start PostgreSQL via Docker Compose
docker-compose up -d db

# 5. Run database migrations
cd apps/api
sqlx migrate run

# 6. Seed initial content
cargo run --bin seeder
cd ../..
```

### Development Workflow

```bash
# Start all services in development mode
nx run-many --target=serve --all --parallel

# Or start services individually:
nx serve frontend    # Frontend at http://localhost:3000
nx run api:serve     # API at http://localhost:8080

# Run tests
nx run-many --target=test --all

# Lint all code
nx run-many --target=lint --all

# Format code
nx format:write

# Build for production
nx run-many --target=build --all
```

### Using Docker Compose

```bash
# Start all services (frontend, API, database)
docker-compose up

# Rebuild after changes
docker-compose up --build

# Stop services
docker-compose down

# View logs
docker-compose logs -f
```

## 🛠️ Development Guide

### Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Monorepo** | Nx | Task orchestration, dependency graph, caching |
| **Frontend** | Next.js 14+ (App Router) | Server-side rendering, routing, UI |
| **UI Library** | React 18+ | Component-based UI |
| **Language** | TypeScript | Type safety across frontend and shared libs |
| **Styling** | TailwindCSS | Utility-first styling (recommended) |
| **Backend** | Rust (Axum) | High-performance REST API |
| **Database** | PostgreSQL 14+ | Relational data storage |
| **ORM** | sqlx | Compile-time checked SQL queries |
| **Auth** | Sui zkLogin + JWT | Web3-native authentication |
| **Container** | Docker + Compose | Local development and deployment |
| **CI/CD** | GitHub Actions | Automated testing and builds |

### Project Structure

#### Frontend (`apps/frontend`)

```text
apps/frontend/
├── app/                   # Next.js App Router
│   ├── (auth)/           # Auth-related routes (login, callback)
│   ├── collections/      # Collection listing and detail pages
│   ├── lessons/          # Lesson viewer with quizzes
│   └── api/              # API route handlers (optional)
├── components/           # React components
├── lib/                  # Utilities and helpers
├── public/               # Static assets
└── styles/               # Global styles
```

#### Backend (`apps/api`)

```text
apps/api/
├── src/
│   ├── main.rs           # Application entry point
│   ├── routes/           # Axum route handlers
│   ├── models/           # Database models
│   ├── middleware/       # JWT validation, logging
│   ├── db.rs             # Database connection pool
│   └── config.rs         # Configuration management
├── migrations/           # sqlx database migrations
└── Cargo.toml            # Rust dependencies
```

#### Shared Libraries (`libs/`)

```text
libs/
├── shared-types/         # TypeScript interfaces for API
├── api-client/           # Type-safe API client
│   ├── src/
│   │   ├── client.ts    # HTTP client with auth
│   │   └── endpoints.ts # API endpoint methods
└── config/               # Environment configuration
```

### Database Schema

The database has 7 main tables:

- **`users`**: Sui address, zkLogin subject, profile information
- **`collections`**: Learning collections (e.g., "Sui 101")
- **`lessons`**: Individual lessons with markdown content
- **`quizzes`**: Multiple-choice questions per lesson
- **`quiz_answers`**: Answer options with correctness flag
- **`user_lesson_progress`**: Tracks lesson completion
- **`user_quiz_attempts`**: Records quiz attempt history

See [docs/design/overview/chapter-5_backend-design.md](docs/design/overview/chapter-5_backend-design.md) for the full schema.


### Authentication Flow

Knowledge Proof uses a two-tier JWT system:

1. **zkLogin JWT** (from Sui): Proves web2 login, binds to Sui address (used once at login)
2. **App JWT** (application-issued): Contains `user_id` and `sui_address`, signed with app secret

The flow:

1. User clicks "Sign in with Sui"
2. Frontend redirects to Sui zkLogin provider
3. User authenticates with OAuth provider (Google, etc.)
4. Sui returns zkLogin JWT
5. Frontend sends zkLogin JWT to API `/api/auth/exchange`
6. API validates zkLogin JWT, creates/finds user, issues app JWT
7. App JWT stored in HttpOnly cookie
8. All subsequent API requests include `Authorization: Bearer <app_jwt>`

See [docs/design/overview/chapter-4_authentication.md](docs/design/overview/chapter-4_authentication.md) for details.

## 📝 Common Tasks

### Adding a New API Endpoint

1. Define route in `apps/api/src/routes/mod.rs`
2. Create handler function in appropriate module
3. Add request/response types to `libs/shared-types`
4. Add client method to `libs/api-client`
5. Write tests for the endpoint

### Adding a New Frontend Page

1. Create route in `apps/frontend/app/` following App Router conventions
2. Fetch data using the API client from `libs/api-client`
3. Create or reuse components from `apps/frontend/components/`
4. Handle loading and error states appropriately
5. Add navigation links

### Creating a Database Migration

```bash
cd apps/api
sqlx migrate add descriptive_migration_name
# Edit the generated migration file in migrations/
sqlx migrate run
```

### Adding Learning Content

1. Create or edit YAML files in `content/` following the schema
2. Run the seeder to populate the database:

   ```bash
   cd apps/api
   cargo run --bin seeder
   ```

3. Verify content appears via the API or frontend

See [docs/design/overview/chapter-9_content-authoring.md](docs/design/overview/chapter-9_content-authoring.md) for content format details.

## 🧪 Testing

```bash
# Run all tests
nx run-many --target=test --all

# Test frontend only
nx test frontend

# Test Rust API
cd apps/api
cargo test

# Run with coverage
cargo test --coverage

# E2E tests (if configured)
nx e2e frontend-e2e
```

## 📋 Code Standards

### TypeScript/React

- Use **TypeScript strict mode**
- Follow **React best practices**: hooks, functional components
- Use **ESLint** and **Prettier** for consistency
- Prefer **named exports** over default exports
- Keep components focused (< 200 lines)
- Document complex logic with comments

### Rust

- Follow **Rust 2021 edition** idioms
- Use **rustfmt** (`cargo fmt`) and **clippy** (`cargo clippy`)
- Prefer **explicit error handling** with `Result<T, E>`
- Use **async/await** with Tokio runtime
- Document public APIs with doc comments (`///`)
- Use **`tracing`** for structured logging, not `println!`


### Git Workflow

- Branch naming: `feature/description`, `fix/description`, `docs/description`
- Commit messages: Follow [Conventional Commits](https://www.conventionalcommits.org/)
  - Prefixes: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`
- Keep commits atomic and focused
- Squash before merging to main


## 🗺️ Implementation Roadmap

We follow a milestone-based approach. See [docs/design/overview/chapter-10_milestones.md](docs/design/overview/chapter-10_milestones.md) for detailed prompts.

1. **Milestone 1**: Monorepo + skeleton apps (Nx, Next.js, Rust "hello world")
2. **Milestone 2**: Database schema + migrations (sqlx, PostgreSQL)
3. **Milestone 3**: Sui zkLogin authentication (zkLogin flow, JWT middleware)
4. **Milestone 4**: Public content endpoints (collections, lessons)
5. **Milestone 5**: Progress and quizzes (authenticated endpoints, UI)
6. **Milestone 6**: Docker + docker-compose (containerization)
7. **Milestone 7**: CI/CD (GitHub Actions) and polish

## 📚 Documentation

- **[Design Overview](docs/design/overview.md)**: Canonical design specification (10 chapters)
- **[Contributing Guidelines](CONTRIBUTING.md)**: How to contribute code
- **[Code of Conduct](CODE_OF_CONDUCT.md)**: Community standards
- **[Security Policy](SECURITY.md)**: Reporting vulnerabilities
- **[Changelog](CHANGELOG.md)**: Version history

## 🤝 Contributing

We welcome contributions of all kinds! Here's how to get started:

1. **Fork the repository** and create a new branch
2. **Read the design docs** in [docs/design/overview.md](docs/design/overview.md)
3. **Set up your dev environment** following the Quick Start above
4. **Make your changes** following our code standards
5. **Test your changes** (`nx run-many --target=test --all`)
6. **Submit a pull request** with a clear description

Before contributing, please read:

- [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) for community expectations


## 🐛 Reporting Issues

Found a bug or have a feature request? Please use our issue templates:

- [Bug Report](.github/ISSUE_TEMPLATE/bug_report.md)
- [Feature Request](.github/ISSUE_TEMPLATE/feature_request.md)

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Links

- **Repository**: [https://github.com/Scetrov/knowledge_proof](https://github.com/Scetrov/knowledge_proof)
- **Issue Tracker**: [https://github.com/Scetrov/knowledge_proof/issues](https://github.com/Scetrov/knowledge_proof/issues)
- **Discussions**: [https://github.com/Scetrov/knowledge_proof/discussions](https://github.com/Scetrov/knowledge_proof/discussions)

## 💡 Getting Help

- Check the [documentation](docs/design/overview.md)
- Search [existing issues](https://github.com/Scetrov/knowledge_proof/issues)
- Ask in [Discussions](https://github.com/Scetrov/knowledge_proof/discussions)
- Read the [Contributing Guidelines](CONTRIBUTING.md)

---

Built with ❤️ by the Knowledge Proof community
