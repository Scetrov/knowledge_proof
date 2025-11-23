# Knowledge Proof Design Overview

This is the canonical design document for the Knowledge Proof learning platform. The content is organized into separate chapters for easier navigation and maintenance.

## Table of Contents

1. **[Goals and Requirements](overview/chapter-1_goals-and-requirements.md)**

   - Project overview
   - Technology stack
   - Core features

2. **[High-Level Architecture](overview/chapter-2_architecture.md)**

   - Monorepo structure (Nx)
   - Service responsibilities
   - Component interaction

3. **[Directory Layout and Tooling](overview/chapter-3_directory-layout.md)**

   - Nx workspace initialization
   - Project structure
   - Library organization

4. **[Authentication with Sui zkLogin](overview/chapter-4_authentication.md)**

   - Token model (zkLogin JWT + App JWT)
   - Complete authentication flow
   - Security considerations

5. **[Backend Design (Rust / Axum)](overview/chapter-5_backend-design.md)**

   - Crates and dependencies
   - Database schema (users, collections, lessons, quizzes, progress)
   - API endpoints (public and authenticated)

6. **[Frontend Design (Next.js)](overview/chapter-6_frontend-design.md)**

   - App Router structure
   - Shared types and API client
   - Lesson rendering and quiz components

7. **[Docker and Local Environment](overview/chapter-7_docker-deployment.md)**

   - Multi-stage Dockerfiles
   - docker-compose configuration
   - Development environment setup

8. **[Nx Integration and CI](overview/chapter-8_nx-integration-ci.md)**

   - Nx project graph and dependencies
   - GitHub Actions workflow
   - Testing and build automation

9. **[Content Authoring and Seeding](overview/chapter-9_content-authoring.md)**

   - Content file format (YAML)
   - Database seeding
   - Content management

10. **[Milestones with LLM Prompts](overview/chapter-10_milestones.md)**
    - Implementation roadmap
    - Step-by-step guidance
    - LLM prompt examples for each milestone

## Quick Reference

### Key Technologies

- **Monorepo**: Nx
- **Frontend**: Next.js 14+ (App Router, TypeScript, React)
- **Backend**: Rust (Axum framework)
- **Database**: PostgreSQL 14+ (with sqlx)
- **Authentication**: Sui zkLogin + JWT
- **Deployment**: Docker + docker-compose

### Project Structure

```
knowledge_proof/
├── apps/
│   ├── frontend/          # Next.js application
│   └── api/               # Rust Axum API
├── libs/
│   ├── shared-types/      # TypeScript types
│   ├── api-client/        # API client library
│   └── config/            # Shared configuration
├── content/               # Learning content (YAML)
├── docs/
│   └── design/
│       └── overview/      # This documentation
└── docker-compose.yml
```

### Development Workflow

```bash
# Install dependencies
pnpm install

# Start all services
nx run-many --target=serve --all --parallel

# Run tests
nx run-many --target=test --all

# Build for production
nx run-many --target=build --all
```

## Contributing

When updating this design document:

1. Update the relevant chapter file in `overview/`
2. Keep this index synchronized with chapter titles
3. Maintain consistency across all chapters
4. Update LLM prompts to reflect design changes

## Version History

- **v1.0** (2025-11-23): Initial design document
- **v1.1** (2025-11-23): Decomposed into modular chapters
