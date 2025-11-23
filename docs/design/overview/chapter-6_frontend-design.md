# Chapter 6: Frontend Design (Next.js)

Assume App Router (`app/`), but you can adapt to Pages Router.

## 6.1 Routes

- `/` – Landing:

  - Explanation, "Sign in with Sui" button.

- `/login`:

  - Triggers zkLogin redirect.

- `/auth/callback`:

  - Handles zkLogin callback, exchanges token via `/api/auth/complete`, then redirects to `/dashboard`.

- `/dashboard`:

  - Requires auth.
  - Shows list of collections with progress.

- `/collections/[slug]`:

  - Requires auth.
  - Shows collection details and ordered lesson list with completion status.

- `/lessons/[slug]`:

  - Requires auth.
  - Renders lesson markdown.
  - Renders quizzes and "Mark as complete" UI.

Ask the LLM:

> Generate a Next.js App Router structure for the routes above, with placeholder components and TypeScript types for data returned from the Rust API.

## 6.2 Shared types and API client

In `libs/shared-types`:

- TS interfaces mirroring backend DTOs:

  - `CollectionSummary`
  - `CollectionDetail`
  - `LessonSummary`
  - `LessonDetail`
  - `Quiz`
  - `QuizAnswer`
  - `UserProgress`, etc.

In `libs/api-client`:

- `getCollections()`
- `getCollection(slug)`
- `getLesson(slug)`
- `getProgress()`
- `markLessonComplete(lessonId)`
- `answerQuiz(quizId, answerId)`

All functions:

- Accept a base URL / fetch impl.
- Attach `Authorization` header when JWT is present.

Ask the LLM:

> Define TypeScript interfaces and a minimal API client for the endpoints, assuming they return JSON in a straightforward REST style.

## 6.3 Lesson rendering and quizzes

- Use a markdown renderer (e.g., `react-markdown`) to render `body_md`.
- Quiz component:

  - Displays question and answers as buttons.
  - On click:

    - Calls `answerQuiz`.
    - Shows feedback (correct/incorrect + explanation).

- Lesson completion:

  - "Mark as complete" button calls `markLessonComplete`.
  - UI updates completion state locally and via `/api/progress`.

Ask the LLM:

> Implement a `LessonPage` component in Next.js that fetches lesson data, renders markdown, shows quizzes, and wires up `answerQuiz` and `markLessonComplete` with optimistic updates.
