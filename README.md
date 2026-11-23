# FocusNFlow — Campus Study Organizer

> Find an open study room, form a course study group, agree on a time, and run a shared focus timer — all in one Flutter + Firebase app.

**Course:** Mobile App Development — Flutter & Firebase Final Group Project (Georgia State University, Winter 2026)
**Track:** Undergraduate (required presentation questions **FOC-U1** and **FOC-U2**)
**Team:** Saketh & Sai

| Member | Role / ownership |
| --- | --- |
| Saketh Sagar Kokkireni | Session scheduling, shared focus timer, Featured AI study planner + evaluation, Analytics, Crashlytics, CI |
| Sai Athota | Authentication & profiles, room discovery & occupancy, course groups, group chat, Storage, FCM, Remote Config |

Security rules, emulator tests, this README and the APK release are shared work.

---

## Deliverables

| Item | Link / location | Status |
| --- | --- | --- |
| Proposal | `docs/FocusNFlow_Proposal.pdf` | Submitted Nov 22, 2026 |
| Signed team acknowledgment | `docs/FocusNFlow_Team_Acknowledgment.pdf` (replace with the signed scan) | Nov 22, 2026 |
| Presentation deck (PowerPoint) | `docs/FocusNFlow_SakethSai_Deck.pptx` | Due Dec 13, 2026 |
| Tested APK | GitHub Release `v1.0-final` | Due Dec 13, 2026 |
| Demo video (20–25 min) | _Link added before submission_ | Due Dec 13, 2026 |

---

## The problem

Group study breaks down at three points:

1. Nobody knows which campus rooms are free right now.
2. Finding a time that works for every member takes dozens of messages.
3. Focus sessions drift when members start and pause at different times.

**Primary user:** an undergraduate taking 4–5 courses who studies with classmates.

**Success:** a student signs in, finds a room with live occupancy, joins a course group, accepts a session that doesn't clash with their schedule, and starts a timer that stays in sync for everyone — in under two minutes.

---

## Features

| ID | Feature | Owner | Week |
| --- | --- | --- | --- |
| F1 | Sign in (email/password + Google), password reset, profile with courses, weekly availability, contact preferences | Sai | 1 |
| F2 | Room discovery by capacity, amenities and live occupancy (Available / Full / Stale / Unknown) | Sai | 2 |
| F3 | Course groups: create/join, capacity enforced server-side, owner removal, member exit | Sai | 2 |
| F4 | Members-only group chat with ordered, offline-safe messages and deleted-message handling | Sai | 3 |
| F5 | Propose, accept and revise sessions without schedule conflicts; push notifications to affected members | Saketh | 3 |
| F6 | Server-authoritative shared focus timer with synced pause/resume and reconnect recovery | Saketh | 3 |
| F7 | **Featured AI Logic:** study-sequence planner compared against deadline-first sorting | Saketh | 3 |

### Must-Solve Challenges

| ID | Challenge | Approach |
| --- | --- | --- |
| C1 | Real-time occupancy tracking | Check-ins with expiry; scheduled Cloud Function clears expired check-ins; `occupancyUpdatedAt` drives a *Stale* badge |
| C2 | Course-centric data organization | Groups keyed by `courseCode`; profile courses drive group search and planner task tags |
| C3 | Coordinating group schedules | Free/busy overlap of members' availability; conflicts checked server-side on accept |
| C4 | Synchronized shared timer | Server stores `startedAt`, `elapsedMs`, `state`; clients compute remaining time using the server-time offset |

> **Disclosure:** there is no real campus sensor or booking integration. Room data is seeded and occupancy comes from user check-ins. All demo accounts and data are synthetic.

---

## Tech stack

| Layer | Choice |
| --- | --- |
| UI | Flutter 3.x (Dart), Material 3 |
| State management | Riverpod |
| Navigation | go_router (with push-notification deep links) |
| Backend | Firebase Auth, Cloud Firestore, Cloud Storage, Cloud Messaging, Cloud Functions, Remote Config, Analytics, Crashlytics (App Check as a stretch goal) |
| Server logic | Cloud Functions — TypeScript, Node 20 |
| Maps | google_maps_flutter (list view fallback) |
| Testing | flutter_test, fake_cloud_firestore, Firebase Emulator Suite |
| AI logic | On-device, rule-based constraint scheduler in Dart (no external AI API) |

### Firebase services

| # | Service | Use in FocusNFlow |
| --- | --- | --- |
| 1 | Authentication | Email/password + Google; roles `student`, `admin`; password reset |
| 2 | Cloud Firestore | Users, rooms, groups, messages, sessions, timers, tasks, plans |
| 3 | Cloud Storage | Profile photos and group banners, with upload progress and size/type rules |
| 4 | Cloud Messaging | Topic per group (`group_<id>`); session alerts deep-link to the session |
| 5 | Cloud Functions | `onSessionWrite`, `onGroupJoin`, `timerControl` (callable), `expireStaleOccupancy` (scheduled) |
| 6 | Remote Config | `planner_enabled`, `stale_minutes`, seasonal theme |
| 7 | Analytics | `room_checkin`, `group_join`, `session_accept`, `timer_start`, `plan_generated`; join → session → timer funnel |
| 8 | Crashlytics | Debug-only "force test crash" |
| 9 | App Check (stretch) | Play Integrity for Firestore and Storage |

---

## Featured AI Logic — Study Sequence Planner

| Item | Definition |
| --- | --- |
| Inputs | Tasks (title, course, deadline, estimated hours, priority 1–3), weekly availability, accepted group sessions (busy), horizon (default 7 days), daily cap (default 6 h) |
| Output | Ordered study blocks, unscheduled tasks with reasons, metrics panel, one-line explanation per block |
| Method | Urgency + priority scoring, tasks split into 30–90 min chunks, placed to keep daily load balanced without passing deadlines or the daily cap, then a repair pass on the busiest day |
| Baseline | Deadline-first: sort by deadline, fill the earliest free slots |
| Test data | 8 fixed synthetic scenarios in `test/planner_scenarios/` |
| Metrics | Conflicts · Deadline misses · Workload balance (std. dev. of daily hours) · Uncovered hours |
| Failure case | Scenario 8 — total hours exceed free time |
| Fallback | Unfittable tasks listed as *Unscheduled* with the shortfall; missing estimate defaults to 2 h (marked *estimated*); if disabled via Remote Config or on error, the deadline-first list is shown |
| Privacy | Runs on device; data stays in the owner's Firestore path; nothing sent to external AI services |

Results are produced by `test/planner_eval_test.dart` so they can be re-run. Effort values are user estimates, not measured cognitive load.

---

## Getting started

### Prerequisites

- Flutter SDK **3.x** — record the exact version here once fixed: `Flutter X.Y.Z • Dart X.Y.Z`
- Android Studio with an Android emulator (API 34 recommended) or a physical Android device
- Node.js 20 and the Firebase CLI: `npm install -g firebase-tools`
- FlutterFire CLI: `dart pub global activate flutterfire_cli`

Check your setup:

```bash
flutter doctor
flutter devices
```

### 1. Clone

```bash
git clone https://github.com/SakethSagar05/Project-2.git
cd Project-2
```

### 2. Connect Firebase (secrets are not committed)

`google-services.json`, `firebase_options.dart` and any service-account keys are **git-ignored**. Ask a team member for access to the Firebase project, then:

```bash
firebase login
flutterfire configure --project=<firebase-project-id>
```

This generates `lib/firebase_options.dart` and `android/app/google-services.json` locally.

### 3. Install and run

```bash
flutter pub get
flutter analyze
flutter run
```

If several devices are connected: `flutter run -d <device-id>`.

### 4. Cloud Functions

```bash
cd functions
npm install
npm run build
firebase deploy --only functions
```

### 5. Seed demo data (synthetic only)

```bash
cd functions
npm run seed   # loads demo rooms and test accounts into the emulator
```

### Demo accounts

Test accounts (owner, member, nonmember) are created by the seed script in the emulator. Credentials for the graded APK will be shared privately with the grader, not in this repository.

---

## Testing

```bash
# Unit and widget tests
flutter test

# AI planner evaluation (prints the planner vs deadline-first table)
flutter test test/planner_eval_test.dart

# Security rules and functions against the emulator
firebase emulators:start
cd functions && npm test
```

Planned test cases:

| ID | Case | Expected |
| --- | --- | --- |
| T1 | Invalid email / short password | Inline error, no account |
| T2 | Nonmember opens group chat | Rules deny; "Join to view" |
| T3 | 7th user joins a group of 6 | Rejected; count stays 6 |
| T4 | Occupancy updated 20 min ago | Shows *Stale* |
| T5 | Accept an overlapping session | Blocked; clash named |
| T6 | Offline during running timer, then reconnect | Resyncs within 2 s |
| T7 | Task exceeds free time before deadline | Listed as unscheduled with reason |
| T8 | Send chat message offline | Pending, delivered in order on reconnect |

> Only report results that were actually run.

---

## Project structure (planned)

```
lib/
  main.dart
  app/            # router, theme, Remote Config bootstrap
  features/
    auth/         # F1
    profile/      # F1 availability, contact preferences
    rooms/        # F2, C1
    groups/       # F3, C2
    chat/         # F4
    sessions/     # F5, C3
    timer/        # F6, C4
    planner/      # F7 — scheduler, baseline, metrics
  services/       # Firestore, Storage, FCM, Analytics wrappers
test/
  planner_scenarios/   # 8 fixed JSON scenarios
  planner_eval_test.dart
functions/
  src/                 # onSessionWrite, onGroupJoin, timerControl, expireStaleOccupancy
firestore.rules
storage.rules
firestore.indexes.json
docs/
```

---

## Data model (Firestore)

| Collection | Key fields | Access |
| --- | --- | --- |
| `users/{uid}` | name, email, courses[], availability, contactPrefs, role, fcmTokens[] | Owner only |
| `publicProfiles/{uid}` | displayName, photoUrl, courses[] | Signed-in read |
| `rooms/{roomId}` | name, building, capacity, amenities[], occupancy, occupancyUpdatedAt, status | Signed-in read; writes via function |
| `rooms/{roomId}/checkins/{uid}` | checkedInAt, expiresAt | Owner creates/deletes own |
| `groups/{groupId}` | courseCode, name, ownerId, capacity, memberCount, memberIds[] | Signed-in read; owner update; capacity via function |
| `groups/{groupId}/messages/{msgId}` | senderId, text, createdAt, deleted | Members only |
| `groups/{groupId}/sessions/{sessionId}` | roomId, start, end, proposedBy, status, acceptedBy[] | Members read; proposer/owner write |
| `timers/{sessionId}` | state, startedAt, pausedAt, elapsedMs, durationMs, updatedBy | Members read; writes via `timerControl` only |
| `users/{uid}/tasks/{taskId}` | title, courseCode, deadline, estHours, priority | Owner only |
| `users/{uid}/plans/{planId}` | generatedAt, blocks[], unscheduled[], metrics | Owner only |

---

## Security

- Firestore and Storage rules deny by default; every rule checks `request.auth.uid` and group membership.
- Capacity, conflict and timer changes run in Cloud Functions transactions, so a modified client cannot overfill a group or fake timer state.
- Availability, tasks and plans are readable only by their owner; other members see only free/busy overlap.
- No API keys, service-account files, keystores or `google-services.json` are committed (see `.gitignore`).
- Rules are tested in the Emulator Suite with owner, member and nonmember accounts.

---

## Workflow

- One branch per task: `feature/<short-name>` from an up-to-date `main`.
- Small, descriptive commits; every feature has a GitHub issue (F1–F7, C1–C4).
- Open a pull request describing what changed, why, and how it was tested; the other member reviews before merge.
- Never force-push to `main`.

```bash
git switch main
git pull --ff-only
git switch -c feature/room-discovery
# ...commit work...
git push -u origin HEAD
```

### Milestones

| Tag | Target date | Scope |
| --- | --- | --- |
| `v0.1-foundation` | Nov 25, 2026 | Firebase setup, auth, schema, rules draft, navigation, seeded rooms |
| `v0.2-core` | Dec 2, 2026 | Rooms, groups, basic sessions, Storage, real-time sync |
| `v0.3-feature-complete` | Dec 9, 2026 | Chat, conflict function, FCM, shared timer, AI planner + evaluation, tests |
| `v1.0-final` | Dec 13, 2026 | Analytics, Crashlytics, Remote Config toggle, edge cases, deck, video, APK |

---

## Known limitations

- Occupancy depends on students checking in and out; it is not sensor data.
- Android only for the graded build.
- Planner effort values are user estimates, not measured workload or medical guidance.

Design changes after wireframe review are tracked in [`docs/design-log.md`](docs/design-log.md).
