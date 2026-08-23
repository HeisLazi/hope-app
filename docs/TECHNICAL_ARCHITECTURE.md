# Hope App — Technical Architecture

## 1. Architecture goals

The MVP should be:
- cheap enough to run on free tiers during development;
- easy for one developer + coding agent to understand;
- installable as a PWA;
- secure enough that AI and backend secrets never reach the browser;
- capable of reliable scheduled reminders;
- designed so note processing is done once and reused across all study features.

Do not introduce microservices, message brokers, Kubernetes, or a complex agent framework for the MVP.

## 2. Proposed stack

### Frontend
- React
- TypeScript
- Vite
- React Router
- Tailwind CSS
- TanStack Query for server state
- Zod for API/runtime validation
- PWA service worker / web app manifest

### Backend / persistence
Supabase:
- Postgres database;
- Auth;
- Storage for original uploads;
- Edge Functions for privileged server operations;
- vector storage/search in Postgres;
- scheduled job support for reminder dispatch.

### AI
Gemini Developer API called **only from server-side functions**.

Model routing is defined in `AI_SYSTEM.md`.

## 3. High-level system

```text
                   ┌─────────────────────┐
                   │ React / TypeScript  │
                   │ PWA                 │
                   └─────────┬───────────┘
                             │ authenticated API calls
                             ▼
                  ┌───────────────────────┐
                  │ Supabase              │
                  │                       │
                  │ Auth                  │
                  │ Postgres + vectors    │
                  │ Storage               │
                  │ Edge Functions        │
                  │ Scheduler             │
                  └───────┬───────────────┘
                          │ server-side
                          ▼
                  ┌───────────────────────┐
                  │ Gemini Developer API  │
                  └───────────────────────┘
```

## 4. Core data model

The exact SQL may evolve, but these concepts should remain stable.

### users
Supabase Auth owns identity. Add a profile table only for application preferences.

`profiles`
- id UUID PK -> auth.users
- display_name
- timezone
- default_reminder_days integer default 7
- created_at
- updated_at

### subjects
- id UUID PK
- user_id UUID
- name
- code nullable
- lecturer nullable
- accent nullable
- archived boolean
- created_at
- updated_at

### notes
- id UUID PK
- user_id
- subject_id
- title
- note_type
- original_filename nullable
- storage_path nullable
- mime_type nullable
- extracted_text
- processing_status
- processing_error nullable
- created_at
- updated_at

### note_chunks
- id UUID PK
- note_id
- subject_id
- user_id
- chunk_index
- heading nullable
- content
- token_estimate
- embedding vector
- page_number nullable
- source_locator JSONB nullable
- created_at

### study_assets
One table can hold summaries, flashcard sets and tests initially.

- id UUID PK
- user_id
- subject_id
- note_id nullable
- asset_type enum: summary | flashcards | practice_test | key_concepts
- title
- settings JSONB
- content JSONB
- source_chunk_ids UUID[] / relation table if needed
- model_used
- created_at
- updated_at

### test_attempts
- id UUID PK
- user_id
- study_asset_id
- answers JSONB
- score numeric
- topic_breakdown JSONB
- started_at
- completed_at nullable

### candidate_events
AI detections awaiting human review.

- id UUID PK
- user_id
- subject_id
- note_id
- title
- event_type
- proposed_start_at timestamptz nullable
- proposed_due_at timestamptz nullable
- date_text_raw
- source_excerpt
- source_locator JSONB
- confidence numeric
- ambiguity_reason nullable
- review_status enum: pending | confirmed | ignored
- created_at

### academic_events
Only user-confirmed or manually created events belong here.

- id UUID PK
- user_id
- subject_id nullable
- candidate_event_id nullable
- title
- event_type
- starts_at timestamptz nullable
- due_at timestamptz
- details nullable
- source_note_id nullable
- created_by enum: manual | confirmed_ai
- created_at
- updated_at

### reminders
- id UUID PK
- user_id
- event_id
- remind_at timestamptz
- channel enum: in_app | web_push
- status enum: scheduled | sent | failed | cancelled
- sent_at nullable
- error nullable
- created_at

### push_subscriptions
- id UUID PK
- user_id
- endpoint
- p256dh
- auth
- user_agent nullable
- created_at
- last_seen_at

### tutor_threads
- id UUID PK
- user_id
- subject_id nullable
- note_id nullable
- title nullable
- created_at
- updated_at

### tutor_messages
- id UUID PK
- thread_id
- role user | assistant
- content
- source_chunk_ids UUID[] nullable
- model_used nullable
- created_at

## 5. Row-level security

Every user-owned table must enforce RLS.

Minimum rule:
- authenticated user may only select/insert/update/delete rows where `user_id = auth.uid()`.

Do not rely on frontend filtering as a security boundary.

Storage paths should be namespaced by user id and protected with corresponding policies.

## 6. File ingestion pipeline

### Upload request
1. Client validates rough file type/size.
2. Original file uploads to private Supabase Storage.
3. A note row is created with `processing_status = uploading`.
4. Client invokes a server-side processing function.

### Processing function
1. Validate authenticated ownership.
2. Download/read file from private storage.
3. Extract text based on file type.
4. Normalize whitespace while preserving headings/page/source metadata where possible.
5. Save extracted text.
6. Split into semantic chunks.
7. Generate embeddings for chunks.
8. Run metadata/topic/deadline extraction.
9. Write candidate events, never academic events directly.
10. Set note to `ready`.

If a stage fails, store a user-readable processing error and preserve enough state to retry.

### File extraction approach

Prefer deterministic local parsing before AI:
- PDF with a stable PDF text parser;
- DOCX via mammoth or equivalent;
- TXT/MD direct decode.

If a PDF has little/no extractable text, flag it as likely scanned. Multimodal extraction can be a secondary path, but the UI must show that the file required vision processing.

## 7. Chunking / retrieval

Chunking goal is retrieval quality, not arbitrary equal byte sizes.

Guidelines:
- preserve section headings;
- aim around 500–900 tokens per chunk;
- small overlap around 80–150 tokens;
- keep note/page/source metadata;
- do not split formulas, bullet groups, or definitions unnecessarily.

Tutor query flow:
1. embed user query;
2. filter chunks to current user + selected note/subject scope;
3. similarity search top candidates;
4. optionally diversity-rerank;
5. pass compact source context to Gemini;
6. validate structured response;
7. return answer + source chunk metadata.

## 8. AI endpoint design

Suggested server functions:
- `process-note`
- `generate-summary`
- `generate-flashcards`
- `generate-practice-test`
- `generate-key-concepts`
- `tutor-chat`
- `detect-academic-events`

All endpoints:
- authenticate user;
- verify ownership;
- enforce request size limits;
- use server environment variables for Gemini keys;
- validate model output before database insertion;
- return useful error codes.

## 9. Structured AI output

Never parse critical outputs from prose using regex if Gemini can return structured JSON.

Example candidate event contract:

```ts
{
  title: string;
  type: 'assignment' | 'quiz' | 'test' | 'exam' | 'presentation' | 'other';
  dateTextRaw: string;
  isoDate: string | null;
  sourceExcerpt: string;
  confidence: number; // 0..1
  ambiguityReason: string | null;
}
```

Validate with Zod before storing.

## 10. Reminder system

### Why server scheduling is required
JavaScript timers or client-only notifications are not reliable if the browser is closed, sleeping, or the PWA is terminated.

### Creation
When an academic event is created/confirmed:
1. create default reminder at `due_at - 7 days`;
2. if that timestamp is already past, do not create a stale reminder; suggest a nearer default instead;
3. allow 3-day, 1-day and custom reminders.

### Dispatch
A scheduled backend job runs periodically and:
1. selects `scheduled` reminders whose `remind_at <= now()`;
2. creates an in-app notification record / state;
3. sends web push when subscription exists;
4. marks sent or failed;
5. uses idempotency so the same reminder is never sent twice.

### Timezones
Store timestamps in UTC. Store the user's preferred timezone. Convert only for display and reminder-rule creation.

## 11. PWA notifications

Requirements:
- request notification permission only after explaining value;
- do not ask on first page load;
- store Web Push subscription server-side;
- allow notification test in Settings;
- allow channel opt-out;
- clicking a notification deep-links to the relevant event.

Use a VAPID-based web push implementation. Private keys remain server-side.

## 12. Calendar architecture

Calendar is a view over `academic_events`; do not create a second calendar-specific source of truth.

Queries:
- fetch events by visible date range;
- filter by subject/type;
- optimistic UI for manual event edits where safe.

Future Google Calendar sync should map external ids onto the same `academic_events` domain rather than replacing it.

## 13. Caching and free-tier efficiency

To reduce Gemini usage:
- process a note once;
- store extracted text/chunks/embeddings;
- save generated study assets;
- reuse existing assets unless the user requests regeneration;
- route extraction/tagging to cheaper model;
- send only retrieved chunks to tutor rather than entire subject corpus;
- cap tutor history sent per request and summarize older conversation context if needed.

## 14. Security requirements

- no Gemini key in Vite `VITE_*` variables;
- no service-role Supabase key in browser;
- private upload bucket;
- RLS enabled before real user data testing;
- validate mime type and extension server-side;
- file size limit;
- escape/render generated markdown safely;
- do not execute uploaded content;
- rate-limit AI endpoints per user;
- no autonomous deadline creation from unconfirmed AI output.

## 15. Error handling

Errors should be domain-specific.

Examples:
- `UPLOAD_UNSUPPORTED_TYPE`
- `NOTE_EXTRACTION_FAILED`
- `NOTE_SCAN_REQUIRES_VISION`
- `AI_RATE_LIMITED`
- `AI_INVALID_RESPONSE`
- `TUTOR_NO_RELEVANT_CONTEXT`
- `PUSH_PERMISSION_DENIED`

User-facing messages must offer the next action.

## 16. Testing priorities

Highest-risk tests:
- RLS isolation between two users;
- candidate event never bypasses confirmation;
- timezone math for seven-day reminder;
- reminder idempotency;
- tutor retrieval cannot fetch another user's chunks;
- answer key does not leak into active test UI;
- invalid Gemini JSON cannot corrupt persistent data;
- processing retries do not duplicate chunks/events.

## 17. Suggested repo structure

```text
hope-app/
  src/
    app/
    components/
    features/
      auth/
      subjects/
      notes/
      study/
      tutor/
      calendar/
      reminders/
    lib/
    routes/
    styles/
  supabase/
    functions/
    migrations/
    seed.sql
  docs/
  prototype/
  tests/
  .env.example
```

Keep domain code under `features/` rather than creating a giant global components/services folder.
