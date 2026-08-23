# Hope App — Product Specification

**Status:** MVP specification  
**Primary target:** university / college students  
**Primary platform:** installable web app (PWA), responsive on desktop and mobile

## 1. Product statement

Hope is a study companion for students who already have class notes but need help turning those notes into revision material and remembering school deadlines.

The product solves two connected problems:

1. **Learning:** notes are passive; Hope converts them into active study tools and provides a tutor that understands the student's own material.
2. **Remembering:** academic dates are often buried in notes, PDFs, announcements, or manually written pages; Hope surfaces them, asks the student to confirm them, then creates reminders and calendar entries.

## 2. MVP promise

> Upload your notes. Hope turns them into summaries, flashcards and tests, gives you a tutor for those notes, and helps make sure important school dates do not sneak up on you.

## 3. Core user journey

1. Student creates or selects a **Subject**.
2. Student uploads a note file or pastes note text.
3. Hope extracts and stores the note content.
4. Hope analyzes the content for:
   - topics and subtopics;
   - important concepts;
   - possible assignments;
   - quizzes/tests/exams;
   - due dates and times.
5. If dates are detected, Hope shows a **Review detected dates** sheet.
6. Student confirms, edits, or rejects each detected event.
7. Confirmed events appear in Calendar and Upcoming.
8. Hope automatically creates a reminder seven days before the event by default.
9. Student can open the note workspace and choose:
   - Summary;
   - Flashcards;
   - Practice Test;
   - Tutor.
10. Generated study assets remain saved for later use.

## 4. MVP features

### 4.1 Subjects

A subject is the top-level study container.

Required behavior:
- create, rename, archive, and delete a subject;
- optional subject code, lecturer, and color/accent metadata;
- subject dashboard lists notes, generated study sets, and upcoming events;
- tutor can be scoped to one subject or one note.

### 4.2 Note upload and ingestion

MVP accepted input:
- PDF;
- DOCX;
- TXT;
- Markdown;
- pasted text.

Images / scanned documents are desirable but not required for the first implementation slice. If included, they must use multimodal extraction rather than pretending OCR succeeded.

For each upload Hope stores:
- original file metadata;
- extracted text;
- processing status;
- subject association;
- chunks used for retrieval;
- detected deadlines awaiting review;
- generated study assets.

Processing states:
`uploading -> extracting -> analyzing -> ready`

Failure states must be visible and retryable.

### 4.3 Summaries

Student can generate:
- Quick Summary — compact exam-revision view;
- Detailed Summary — structured headings and explanations;
- Key Concepts — terms, definitions, formulas/rules, and important relationships.

Summary rules:
- generated from selected source material;
- preserve the source note relationship;
- do not silently introduce unsupported facts;
- allow regenerate;
- allow copy/export later, but export is not MVP-critical.

### 4.4 Flashcards

Student can generate flashcards from a note or subject.

Each flashcard contains:
- front/question;
- back/answer;
- optional hint;
- topic;
- source reference;
- difficulty: easy / medium / hard.

Study interactions:
- reveal answer;
- mark Again / Hard / Good / Easy;
- track simple mastery counts.

Full spaced-repetition scheduling is post-MVP. The data model should not prevent adding it later.

### 4.5 Practice tests

Supported question types:
- multiple choice;
- true/false;
- short answer.

Test creation options:
- question count;
- difficulty;
- selected topics;
- source scope: note or whole subject.

Test behavior:
- one question at a time or full-page mode;
- submit and score;
- explain the correct answer;
- show which topic/source needs review;
- save attempt history.

The AI should generate an answer key separately from the question presentation so answers are never leaked to the user before submission.

### 4.6 Tutor chatbot

Tutor is a side panel available from a note/subject workspace.

Tutor principles:
- answer primarily from retrieved uploaded notes;
- show source chips/citations back to relevant notes/sections;
- say when the material does not contain enough information;
- explain at different depths when asked;
- support follow-up questions;
- be able to quiz the student instead of only answering;
- never alter events, notes, or study material without an explicit user action.

Useful quick actions:
- Explain this simply;
- Give me an example;
- Quiz me on this;
- What should I revise next?;
- Compare these concepts.

### 4.7 Deadline and assessment detection

Hope may detect candidate events from note content, for example:
- assignment due dates;
- quiz dates;
- test dates;
- exam dates;
- presentation dates;
- project milestones.

A candidate event has:
- title;
- event type;
- subject;
- detected date/time;
- source note;
- supporting source excerpt;
- confidence;
- review status.

**Critical rule:** AI-detected events are suggestions only until the student confirms them.

Review actions:
- Confirm;
- Edit and confirm;
- Ignore.

Ambiguous dates must be marked as ambiguous rather than guessed.

### 4.8 Reminders

When a student confirms an academic event:
- create a default reminder **7 days before**;
- allow additional reminders at 3 days, 1 day, or custom times;
- allow reminders to be disabled per event.

MVP notification channels:
- in-app notification center;
- browser/PWA push notification when permission is granted.

Optional later channels:
- email;
- Google Calendar sync;
- native mobile notifications.

Reminder creation must be server-scheduled. A client-only timer is not reliable when the app/browser is closed.

### 4.9 Calendar

Calendar has:
- month view;
- agenda/upcoming view;
- subject filter;
- event-type filter;
- click event to inspect/edit;
- create event manually;
- create/edit reminders.

Event types should be visually distinguishable without relying on color alone.

### 4.10 Home dashboard

Home should answer three questions immediately:
1. What is coming up?
2. What should I study?
3. What did I recently upload/create?

Suggested dashboard sections:
- Next deadline hero card;
- Upcoming this week;
- Continue studying;
- Subjects;
- Recent notes;
- Quick Upload.

## 5. Information architecture

Primary navigation:
- Home
- Subjects
- Calendar
- Study Library
- Notifications
- Settings

The Tutor is contextual and should appear as a right-side panel on desktop and a full-screen drawer on mobile rather than as a completely separate primary page.

## 6. User stories

### Notes and study generation
- As a student, I can upload my lecture notes so I do not have to manually re-enter them.
- As a student, I can generate revision material from my notes so I can study actively.
- As a student, I can choose topics and difficulty before generating a test.
- As a student, I can return to previously generated material without spending another AI request.

### Tutor
- As a student, I can ask questions about a subject and receive answers based on my uploaded notes.
- As a student, I can see which notes an answer came from.
- As a student, I can ask the tutor to quiz me rather than reveal an answer immediately.

### Deadlines
- As a student, I can see possible dates detected from my notes.
- As a student, I approve a detected date before Hope treats it as real.
- As a student, I receive a reminder seven days before confirmed assignments, quizzes and tests.
- As a student, I can manually add dates that were not present in my notes.

## 7. MVP acceptance criteria

The MVP is considered functionally complete when all of the following are true:

1. A user can create a subject and upload a supported note file.
2. The uploaded note reaches a visible `ready` state or displays a recoverable error.
3. The user can generate and save a summary from the note.
4. The user can generate and study flashcards.
5. The user can generate, complete, submit, and score a practice test.
6. The tutor can answer a question using retrieved chunks and return source references.
7. The tutor explicitly handles insufficient-context questions.
8. The ingestion pipeline can produce candidate academic events.
9. Candidate events are not saved to the real calendar until confirmed.
10. A confirmed event appears in Calendar and Upcoming.
11. A confirmed academic event receives a seven-day reminder by default.
12. A reminder can be edited or disabled.
13. The app works responsively at phone and desktop widths.
14. AI API secrets do not ship to the browser.

## 8. Explicitly out of scope for MVP

Do not add these unless core MVP is stable:
- social/community features;
- teacher accounts;
- university LMS integrations;
- automatic grade scraping;
- full note editor / Notion replacement;
- gamification/RPG systems;
- live collaborative study rooms;
- video courses;
- complex spaced repetition;
- payment/subscriptions;
- native iOS/Android apps;
- autonomous changes to deadlines without confirmation.

## 9. Product principles

- **One upload should create leverage.** Reuse processed notes across tutor, flashcards, tests and summaries.
- **Deadlines must be trustworthy.** Detection may be automatic; confirmation is human.
- **Study first, configuration second.** Avoid deep settings screens.
- **Show provenance.** AI outputs should be traceable to notes.
- **Fast defaults.** One-click generation should work; advanced settings remain optional.
- **Do not punish free-tier users with waste.** Cache generated content and use cheaper models for cheap tasks.
