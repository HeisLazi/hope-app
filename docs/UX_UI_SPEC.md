# Hope App — UX / UI Specification

## 1. Design direction

Hope should feel like a calm academic workspace, not an admin dashboard and not a children's study game.

Target feel:
- focused;
- reassuring;
- modern;
- lightweight;
- high information density without visual clutter;
- comfortable for long study sessions.

The interface should use soft surfaces, strong typography, generous spacing, restrained motion, and clear hierarchy. Avoid excessive gradients, giant cards, neon effects, or gamification-heavy visuals.

## 2. Desktop shell

Desktop layout:

```text
┌──────────────┬────────────────────────────────────────┬─────────────────────┐
│ Left nav     │ Main workspace                         │ Tutor panel         │
│              │                                        │ contextual/optional │
│ Home         │ Header / breadcrumbs                   │                     │
│ Subjects     │                                        │ Ask Hope            │
│ Calendar     │ Active screen                          │ Sources             │
│ Library      │                                        │ Quick actions       │
│              │                                        │                     │
│ Settings     │                                        │                     │
└──────────────┴────────────────────────────────────────┴─────────────────────┘
```

Tutor panel is collapsible. When closed, the main workspace expands.

## 3. Mobile shell

Mobile uses:
- compact top bar;
- bottom navigation for Home, Subjects, Calendar, Library;
- floating or header Tutor button when inside a subject/note;
- Tutor opens as full-screen sheet;
- upload uses a full-screen flow rather than a tiny modal.

## 4. Primary screens

### 4.1 Home

Purpose: answer "what do I need to care about now?"

Layout order:
1. Greeting + current date.
2. **Next deadline** hero card.
3. **Upcoming** horizontal/stacked event list.
4. **Continue studying** cards.
5. Subject grid/list.
6. Recent uploads.

Quick actions:
- Upload notes;
- Add deadline;
- Start quick test.

A new user sees an onboarding empty state with one clear action: **Create your first subject**.

### 4.2 Subjects index

Each subject card/list row shows:
- subject name/code;
- number of notes;
- next deadline;
- recent study progress;
- last opened time.

Actions:
- open;
- add note;
- edit metadata;
- archive.

### 4.3 Subject workspace

Header:
- subject name + code;
- next assessment;
- `Upload notes` action;
- `Ask Hope` action.

Tabs:
- Overview;
- Notes;
- Study;
- Assessments.

Overview widgets:
- next deadline;
- latest notes;
- weak/recent topics;
- saved study sets.

### 4.4 Upload flow

Step 1 — Select source
- drag/drop file;
- choose file;
- paste notes.

Step 2 — Metadata
- subject (preselected if opened from subject);
- optional title override;
- note type: Lecture / Tutorial / Reading / Revision / Other.

Step 3 — Processing
Show actual stages rather than a generic spinner:
- Uploading file;
- Extracting text;
- Finding topics;
- Checking for academic dates;
- Preparing study tools.

Step 4 — Results
Show:
- extracted title;
- topic chips;
- detected deadline count;
- generation shortcuts.

If dates were detected, primary CTA becomes **Review detected dates**.

### 4.5 Detected dates review

Use a sheet/modal with one card per candidate event.

Card contents:
- proposed title;
- type badge;
- date/time;
- subject;
- confidence indicator in plain language (High / Check this / Ambiguous);
- exact supporting note excerpt;
- source note link.

Actions:
- Confirm;
- Edit;
- Ignore.

Never use a one-click `Accept all` for ambiguous dates. A bulk-confirm option is acceptable only for high-confidence dates with explicit user action.

### 4.6 Note detail / study workspace

Desktop recommended structure:

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ Subject / Note title                          Generate ▾    Ask Hope     │
├───────────────────────────────┬─────────────────────────────────────────┤
│ Note / generated content      │ Tutor panel                             │
│                               │                                         │
│ Tabs:                         │ Question history                        │
│ Original | Summary | Cards    │ Source-backed responses                 │
│ Test | Key concepts           │                                         │
│                               │ Suggested prompts                       │
└───────────────────────────────┴─────────────────────────────────────────┘
```

`Generate` menu:
- Quick summary;
- Detailed summary;
- Flashcards;
- Practice test;
- Key concepts.

Generated content is saved as a separate study asset and does not overwrite original notes.

### 4.7 Summary view

Summary header:
- source note;
- generated timestamp;
- summary type;
- regenerate menu.

Body:
- key takeaways callout;
- structured sections;
- definitions/formulas when present;
- source chips where useful.

### 4.8 Flashcard study view

Focus mode. Remove unnecessary chrome.

Card behavior:
- question front;
- tap/click/spacebar reveals answer;
- answer shows source link;
- rating buttons: Again / Hard / Good / Easy;
- progress `12 / 30`;
- keyboard shortcuts on desktop.

End screen:
- cards completed;
- accuracy/self-rating distribution;
- cards marked difficult;
- retry difficult cards.

### 4.9 Practice test setup

Controls:
- 5 / 10 / 20 / custom questions;
- Easy / Mixed / Hard;
- question types toggles;
- topic selector;
- optional timer.

Primary CTA: **Start test**.

### 4.10 Practice test session

One-question-at-a-time is default on mobile. Desktop can support question navigator.

Before submission:
- no answer key or tutor reveal;
- allow flag for review;
- progress visible.

Results:
- score;
- topic breakdown;
- wrong answers with explanation;
- `Study weak topics` action;
- `Ask Hope about mistakes` action.

### 4.11 Tutor panel

Header:
- Hope Tutor;
- scope indicator, e.g. `Using: Data Networks > Lecture 4`;
- change scope menu.

Message response anatomy:
1. direct answer;
2. explanation;
3. optional example;
4. source chips;
5. suggested follow-up.

Quick modes:
- Explain;
- Socratic;
- Quiz me;
- Exam prep.

When context is insufficient, show:
> I can't verify that from the notes currently selected.

Then offer:
- search all notes in subject;
- upload more material;
- answer from general knowledge only if the user explicitly opts in.

### 4.12 Calendar

Top controls:
- Today;
- previous / next;
- Month / Agenda;
- subject filter;
- event-type filter;
- Add event.

Event details drawer:
- title;
- subject;
- type;
- date/time;
- notes/source;
- reminders;
- edit/delete.

Event types:
- Assignment;
- Quiz;
- Test;
- Exam;
- Presentation;
- Other.

Use icons/text labels in addition to color.

### 4.13 Notifications

Notification center groups:
- Today;
- Upcoming;
- Earlier.

Examples:
- `ARI test in 7 days`;
- `DSA assignment due tomorrow`;
- `3 possible dates need review`.

Clicking a reminder should open the event, not just dismiss the notification.

## 5. Global component inventory

Implementation agent should build reusable components for:
- AppShell;
- Sidebar;
- MobileNav;
- PageHeader;
- SubjectCard;
- DeadlineCard;
- EventBadge;
- NoteCard;
- ProcessingStepper;
- DetectionReviewCard;
- StudyAssetCard;
- TutorPanel;
- SourceChip;
- Flashcard;
- TestQuestion;
- CalendarEvent;
- EmptyState;
- ConfirmationDialog;
- Toast / inline error state.

## 6. Motion

Use motion sparingly:
- 150–220ms panel transitions;
- flashcard flip/reveal;
- tutor panel slide;
- processing-step transitions;
- small success confirmation after deadline save.

Respect `prefers-reduced-motion`.

## 7. Accessibility

Minimum requirements:
- keyboard usable;
- visible focus states;
- labels on icon-only buttons;
- AA contrast targets;
- semantic form labels;
- no information encoded by color alone;
- dialogs trap focus and restore focus on close;
- flashcards and tests usable without pointer input.

## 8. Responsive breakpoints / behavior

Do not design separate products for desktop and mobile. Components should reflow.

Suggested behavior:
- `< 768px`: mobile nav, tutor full-screen sheet, single-column layouts;
- `768–1199px`: compact sidebar, overlay tutor;
- `>= 1200px`: persistent sidebar + optional persistent tutor panel.

## 9. Prototype dataset

The prototype should use realistic demo content so the flows are obvious:

Subjects:
- Data Networks — `DTN611S`;
- Artificial Intelligence & Reasoning — `ARI`;
- Data Structures & Algorithms — `DSA`.

Example events:
- DSA Assignment 3 — due Aug 30;
- ARI Quiz — Sep 2;
- DTN Test 1 — Sep 7.

Example notes:
- `IPv4 Subnetting & CIDR`;
- `A* Search and Heuristics`;
- `AVL Trees and Rotations`.

## 10. UI implementation rule

The prototype is a directional artifact, not a pixel-perfect contract. Antigravity may improve spacing, responsiveness, and component implementation, but it should preserve:
- information architecture;
- core flows;
- deadline confirmation safety;
- contextual tutor placement;
- fast access to study generation;
- calm, study-focused visual character.
