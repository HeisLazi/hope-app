# Hope App

Hope is a student study companion that turns uploaded class notes into usable study material, keeps school deadlines visible, and provides a note-grounded AI tutor.

## Product goal

A student should be able to upload notes once and then use Hope to:

1. generate flashcards;
2. generate practice tests and quizzes;
3. generate summaries and key-topic breakdowns;
4. ask a tutor questions grounded in the uploaded notes;
5. detect assignments, quizzes, tests, and due dates from notes;
6. confirm those detected deadlines before they are saved;
7. receive a default reminder seven days before a confirmed deadline; and
8. see all confirmed academic events in a calendar.

The MVP is intentionally focused. Hope is **not** an LMS, university portal, social network, or full productivity suite.

## Core product loop

`Upload notes -> Process -> Review detected deadlines -> Study -> Ask tutor -> Track upcoming work`

## Recommended stack

- **Frontend:** React + TypeScript + Vite
- **App form:** installable PWA for desktop and mobile
- **Styling:** Tailwind CSS + accessible headless UI primitives
- **Backend:** Supabase (Postgres, Auth, Storage, Edge Functions)
- **AI:** Gemini Developer API
- **Tutor / generation model:** `gemini-3.6-flash`
- **Fast extraction model:** `gemini-3.5-flash-lite`
- **Retrieval embeddings:** `gemini-embedding-2`
- **Development agent:** Google Antigravity; use its strongest/default coding model unless a task specifically benefits from a faster model

## Documentation

- [`docs/PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md) — product requirements and MVP acceptance criteria
- [`docs/UX_UI_SPEC.md`](docs/UX_UI_SPEC.md) — screen structure, interactions, and prototype behavior
- [`docs/TECHNICAL_ARCHITECTURE.md`](docs/TECHNICAL_ARCHITECTURE.md) — frontend, backend, data model, reminders, and ingestion
- [`docs/AI_SYSTEM.md`](docs/AI_SYSTEM.md) — tutor grounding, generation pipelines, model routing, and safety rules
- [`docs/ANTIGRAVITY_HANDOFF.md`](docs/ANTIGRAVITY_HANDOFF.md) — coding-agent instructions and implementation sequence
- [`prototype/index.html`](prototype/index.html) — static interactive UI direction for the implementation agent

## MVP non-negotiables

- AI-created deadlines are never silently trusted; the student confirms or edits them first.
- Tutor answers should be grounded in the selected subject's uploaded notes and identify when the notes do not contain enough information.
- Generated study content keeps a source link back to the note/chunk it came from.
- Seven-days-before is the default academic reminder.
- The app remains useful without AI after material has already been generated.
- Gemini API keys must never be exposed in client-side code.

## Status

Documentation and UI-prototype phase. Application implementation has not started yet.
