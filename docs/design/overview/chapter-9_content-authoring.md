# Chapter 9: Content Authoring and Seeding

## 9.1 Content source

Keep initial content in simple YAML or JSON fixture files committed to the repo, e.g.:

- `content/sui-101/collection.yaml`
- `content/sui-101/lessons/*.yaml` with fields:

  - `slug`, `title`, `summary`, `estimated_minutes`, `order_index`, `body_md`, `quizzes`.

## 9.2 Seeder

- Write a small Rust or Node script that:

  - Reads the content files.
  - Inserts rows into `collections`, `lessons`, `quizzes`, `quiz_answers` if they don't exist.

Ask the LLM:

> Produce a Rust command-line seeder that reads YAML files describing collections and lessons, and inserts/updates rows in the Postgres database used by the API.
