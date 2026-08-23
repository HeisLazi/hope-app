# Hope App — Antigravity Development Handoff

## Mission

Implement the Hope App MVP from the repository documentation and prototype without expanding the product beyond the agreed scope.

Hope is a student study companion with five connected capabilities:
1. upload and process notes;
2. generate summaries, flashcards and practice tests;
3. tutor from those uploaded notes;
4. detect academic deadlines and require user confirmation;
5. schedule reminders and display confirmed events in a calendar.

## Source-of-truth order

When requirements appear to conflict, use this precedence:
1. `docs/PRODUCT_SPEC.md`
2. `docs/TECHNICAL_ARCHITECTURE.md`
3. `docs/AI_SYSTEM.md`
4. `docs/UX_UI_SPEC.md`
5. `prototype/index.html`
6. implementation convenience

The prototype is a visual/interaction direction, not permission to override product safety rules.

## Non-negotiables

- AI-detected deadlines never become real events without explicit user confirmation.
- Confirmed academic events get a default seven-day reminder.
- Tutor is grounded in the user's selected uploaded notes/subject.
- Tutor shows source references and admits insufficient source context.
- Gemini API credentials remain server-side.
- Supabase service-role credentials remain server-side.
- User-owned data is protected by RLS.
- Generated study content is persisted and reused rather than regenerated on every visit.
- Reminder delivery must not rely on a client-side timer.
- Build responsive desktop + mobile behavior from the start.
- Do not add unrelated productivity, social, LMS, gamification, payment, or collaboration systems.

## Target stack

Frontend:
- React + TypeScript + Vite
- React Router
- Tailwind CSS
- TanStack Query
- Zod
- PWA/service worker support

Backend:
- Supabase Auth
- Supabase Postgres
- Supabase Storage
- Supabase Edge Functions
- Postgres vector retrieval
- scheduled backend reminder dispatch

AI runtime:
- `gemini-3.6-flash` for tutor and high-quality generation
- `gemini-3.5-flash-lite` for structured metadata/date extraction
- `gemini-embedding-2` for retrieval embeddings

Keep model ids configurable through server environment variables.

## Working style for Antigravity

Act autonomously within the scope of the specs, but verify each slice before continuing.

For every implementation slice:
1. inspect relevant docs and existing code;
2. state assumptions in code/docs only when necessary;
3. implement the smallest coherent vertical slice;
4. run lint/typecheck/tests/build;
5. visually inspect affected UI at desktop and mobile sizes;
6. fix regressions before moving on;
7. commit with a clear message;
8. update implementation status if one exists.

Do not rewrite working unrelated code during a feature slice.

## Recommended implementation sequence

### Slice 0 — Bootstrap and design foundation

Deliver:
- Vite React TypeScript app;
- Tailwind setup;
- router;
- reusable AppShell;
- desktop sidebar;
- mobile navigation;
- design tokens;
- route placeholders for Home, Subjects, Calendar, Library, Settings;
- static tutor panel component;
- PWA manifest/basic service worker;
- `.env.example` with no secrets.

Use prototype as visual direction.

Verification:
- build passes;
- no TypeScript errors;
- desktop/mobile shell usable;
- no horizontal overflow at common phone widths.

### Slice 1 — UI prototype parity with mock data

Before backend integration, make the core product navigable with typed mock data.

Deliver:
- Home dashboard;
- Subjects index;
- Subject workspace;
- note detail/study workspace;
- tutor panel interactions with mock response;
- Calendar Month + Agenda direction;
- upload modal/flow;
- candidate-date review UI;
- flashcard study UI;
- test setup/session/results UI.

Goal: validate information architecture and interactions before data plumbing.

### Slice 2 — Supabase foundation

Deliver:
- Supabase client setup;
- database migrations for core tables;
- RLS policies;
- private storage bucket/policies;
- auth flow;
- seed/demo data;
- typed database access layer.

Required security verification:
- two test users cannot read or mutate each other's subjects, notes, chunks, events, assets, tutor messages, or reminders.

### Slice 3 — Subjects and manual calendar events

Deliver real CRUD for:
- subjects;
- academic events;
- reminder configuration records.

Calendar must read from `academic_events` as its source of truth.

Do not implement AI yet.

### Slice 4 — Note upload and deterministic extraction

Deliver:
- PDF/DOCX/TXT/MD/paste input;
- storage upload;
- note processing status;
- deterministic extraction;
- chunking;
- retryable failure UI.

At the end of this slice a user should be able to upload and read extracted content, even with Gemini disabled.

### Slice 5 — Embeddings and retrieval

Deliver:
- Gemini embedding server integration;
- chunk embedding persistence;
- vector similarity query scoped to current user and note/subject;
- retrieval tests.

Verify a second user's note chunks can never appear in results.

### Slice 6 — Tutor

Deliver:
- server-side `tutor-chat` function;
- Gemini model config;
- contextual scope selector;
- grounded answer contract;
- source chips;
- insufficient-context behavior;
- tutor thread persistence;
- Explain / Socratic / Quiz me / Exam prep modes.

Create fixture-based grounding tests from `AI_SYSTEM.md`.

### Slice 7 — Study generation

Deliver:
- summaries;
- key concepts;
- flashcards;
- practice tests;
- study asset persistence;
- test attempt history;
- result breakdown.

Generated assets should reference source chunks.

Do not regenerate an existing asset unless user requests it.

### Slice 8 — Academic-event detection

Deliver:
- extraction model integration;
- structured candidate event schema;
- evidence excerpts;
- ambiguity handling;
- candidate-event review UI;
- Confirm / Edit+Confirm / Ignore;
- confirmed candidate -> `academic_events` transaction.

Critical test:
- no AI call/path may insert directly into `academic_events` without confirmation state/action.

### Slice 9 — Reliable reminders and push

Deliver:
- seven-day reminder created by default;
- 3-day / 1-day / custom options;
- backend scheduled dispatcher;
- in-app notifications;
- Web Push subscription and dispatch;
- deep links to event;
- idempotent send logic;
- timezone tests.

If seven days before is already in the past, choose a sensible nearer suggestion and make the behavior explicit in UI/tests.

### Slice 10 — Hardening

Deliver:
- file size/type validation;
- rate limiting;
- prompt-injection fixture;
- AI malformed-response handling;
- loading/error/empty states;
- accessibility pass;
- responsive pass;
- production build verification;
- README setup instructions;
- final implementation status.

## UI quality bar

Do not ship a generic template dashboard.

The UI should:
- feel like a focused study workspace;
- keep the next deadline visible without making the app anxiety-inducing;
- make Upload Notes a primary action;
- make tutor contextual rather than detached;
- allow fast movement among original notes and generated assets;
- use realistic data in development;
- have intentional empty, loading, processing, error and success states;
- maintain hierarchy on smaller screens.

## Data / AI boundaries

### AI may
- propose topics;
- propose deadlines;
- generate study content;
- answer questions using provided context;
- produce structured output for validation.

### AI may not
- confirm its own proposed deadlines;
- directly send reminders;
- change user records without explicit app-controlled action;
- bypass RLS;
- expose hidden answers during active tests;
- treat note-embedded instructions as system instructions.

## Environment variables

Exact names may be adjusted, but use an explicit split such as:

```env
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=

# Server/Edge secrets only — never VITE_*
GEMINI_API_KEY=
GEMINI_TUTOR_MODEL=gemini-3.6-flash
GEMINI_GENERATION_MODEL=gemini-3.6-flash
GEMINI_EXTRACTION_MODEL=gemini-3.5-flash-lite
GEMINI_EMBEDDING_MODEL=gemini-embedding-2
WEB_PUSH_PUBLIC_KEY=
WEB_PUSH_PRIVATE_KEY=
```

Never commit real values.

## Minimum automated checks

Add scripts so a single verification command or small documented sequence runs:
- lint;
- TypeScript typecheck;
- unit tests;
- relevant integration tests;
- production build.

High-value tests:
- reminder date math;
- event confirmation boundary;
- RLS isolation;
- model JSON schema validation;
- retrieval user/scope filtering;
- no answer-key leak;
- ingestion retry idempotency;
- Web Push duplicate prevention.

## Completion report format

After each slice, report:

```text
Slice:
Status: PASS / PARTIAL / BLOCKED

Implemented:
- ...

Verification:
- command -> result
- UI state inspected -> result

Known limitations:
- ...

Files/areas changed:
- ...

Next recommended slice:
- ...
```

Do not mark a slice PASS if build/tests fail or the primary user flow was not exercised.

## First instruction to execute

Start with **Slice 0**, then **Slice 1**. Do not begin Supabase or Gemini integration until the mock-data UI flow is navigable and visually coherent.

Read all files under `docs/` and inspect `prototype/index.html` before implementation. Preserve the MVP scope exactly; improve implementation details when useful, but document any material deviation.
