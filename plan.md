# Teacher Helper Web App — Project Plan

A full-stack web application for teachers to communicate with parents, manage curriculum visibility, track test results, and publish upcoming activities — organized by grade and semester.

---

## 1. Tech Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Framework** | Next.js 15 (App Router) | Full-stack React framework with server components, API routes, and SSR |
| **Language** | TypeScript | Type safety across frontend and backend |
| **Styling** | Tailwind CSS v4 | Utility-first CSS for responsive UI |
| **UI Components** | shadcn/ui + Radix UI | Accessible, composable components styled with Tailwind |
| **Database** | Neon Serverless PostgreSQL | Managed Postgres with serverless driver, branching, and connection pooling |
| **ORM** | Drizzle ORM | Type-safe SQL queries, lightweight, excellent Neon integration |
| **Auth** | Auth.js (NextAuth v5) | Session-based authentication for teachers and admins |
| **Validation** | Zod | Runtime schema validation for forms and API payloads |
| **Forms** | React Hook Form | Performant form handling with Zod resolver |
| **Email** | Resend (or Nodemailer + SMTP) | Deliver parent notification emails |
| **Date/Time** | date-fns | Format and manipulate dates in UI |
| **Icons** | Lucide React | Consistent icon set |
| **Deployment** | Vercel + Neon | Zero-config Next.js hosting with serverless DB |

### Why These Choices

- **Next.js App Router** — Server Components reduce client bundle size; Route Handlers replace a separate Express backend.
- **Neon + Drizzle** — Neon's `@neondatabase/serverless` driver works over HTTP (edge-friendly); Drizzle generates migrations and keeps types in sync with the schema.
- **Auth.js** — Credentials or magic-link auth for teachers; role-based access (teacher vs admin) via session callbacks.
- **shadcn/ui** — Copy-paste components that match Tailwind conventions; no heavy component library lock-in.

---

## 2. Complete File Structure

```
teacher-web-app/
├── .env.local                    # Local secrets (DATABASE_URL, AUTH_SECRET, RESEND_API_KEY)
├── .env.example                  # Template for required env vars
├── .gitignore
├── drizzle.config.ts             # Drizzle Kit config (migrations, introspection)
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── tailwind.config.ts
├── tsconfig.json
├── plan.md                       # This file
│
├── drizzle/                      # Generated SQL migrations
│   └── 0000_initial.sql
│
├── public/
│   └── logo.svg
│
├── src/
│   ├── app/
│   │   ├── layout.tsx            # Root layout (fonts, providers, Toaster)
│   │   ├── page.tsx              # Landing / redirect to dashboard
│   │   ├── globals.css           # Tailwind imports + CSS variables
│   │   │
│   │   ├── (auth)/               # Auth route group (no sidebar)
│   │   │   ├── layout.tsx
│   │   │   ├── login/
│   │   │   │   └── page.tsx
│   │   │   └── register/         # Admin-only or invite-based
│   │   │       └── page.tsx
│   │   │
│   │   ├── (dashboard)/          # Protected route group (sidebar layout)
│   │   │   ├── layout.tsx        # Auth guard + sidebar + header
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx      # Overview: recent messages, upcoming activities
│   │   │   │
│   │   │   ├── messages/
│   │   │   │   ├── page.tsx      # Message history list
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx  # Compose message to parents
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx  # Message detail + delivery status
│   │   │   │
│   │   │   ├── curriculum/
│   │   │   │   ├── page.tsx      # Grade + semester topic browser
│   │   │   │   └── manage/
│   │   │   │       └── page.tsx  # CRUD topics (teacher/admin)
│   │   │   │
│   │   │   ├── tests/
│   │   │   │   ├── page.tsx      # Test list by grade/semester
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx  # Create test
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx  # Test detail + results table
│   │   │   │       └── results/
│   │   │   │           └── page.tsx  # Bulk enter/edit scores
│   │   │   │
│   │   │   ├── activities/
│   │   │   │   ├── page.tsx      # Upcoming & past activities calendar/list
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx
│   │   │   │
│   │   │   ├── students/
│   │   │   │   ├── page.tsx      # Student roster by grade
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx  # Student profile + parent contact + results
│   │   │   │
│   │   │   └── settings/
│   │   │       └── page.tsx      # Profile, grade assignments
│   │   │
│   │   ├── (public)/             # Optional public parent-facing views
│   │   │   └── parent/
│   │   │       └── [token]/
│   │   │           └── page.tsx  # Read-only view via secure link (future)
│   │   │
│   │   └── api/
│   │       ├── auth/
│   │       │   └── [...nextauth]/
│   │       │       └── route.ts
│   │       ├── messages/
│   │       │   ├── route.ts              # GET list, POST send
│   │       │   └── [id]/
│   │       │       └── route.ts          # GET detail
│   │       ├── curriculum/
│   │       │   ├── topics/
│   │       │   │   ├── route.ts          # GET, POST
│   │       │   │   └── [id]/
│   │       │   │       └── route.ts      # GET, PATCH, DELETE
│   │       │   ├── grades/
│   │       │   │   └── route.ts          # GET grades
│   │       │   └── semesters/
│   │       │       └── route.ts          # GET semesters
│   │       ├── tests/
│   │       │   ├── route.ts              # GET, POST
│   │       │   └── [id]/
│   │       │       ├── route.ts          # GET, PATCH, DELETE
│   │       │       └── results/
│   │       │           └── route.ts      # GET, POST (bulk upsert)
│   │       ├── activities/
│   │       │   ├── route.ts              # GET, POST
│   │       │   └── [id]/
│   │       │       └── route.ts          # GET, PATCH, DELETE
│   │       ├── students/
│   │       │   ├── route.ts              # GET, POST
│   │       │   └── [id]/
│   │       │       └── route.ts          # GET, PATCH, DELETE
│   │       └── dashboard/
│   │           └── stats/
│   │               └── route.ts          # GET summary counts
│   │
│   ├── components/
│   │   ├── ui/                   # shadcn/ui primitives (button, input, table, etc.)
│   │   ├── layout/
│   │   │   ├── sidebar.tsx
│   │   │   ├── header.tsx
│   │   │   └── mobile-nav.tsx
│   │   ├── messages/
│   │   │   ├── message-compose-form.tsx
│   │   │   ├── message-list.tsx
│   │   │   └── recipient-picker.tsx
│   │   ├── curriculum/
│   │   │   ├── topic-card.tsx
│   │   │   ├── topic-form.tsx
│   │   │   └── grade-semester-filter.tsx
│   │   ├── tests/
│   │   │   ├── test-form.tsx
│   │   │   ├── results-table.tsx
│   │   │   └── score-input-grid.tsx
│   │   ├── activities/
│   │   │   ├── activity-card.tsx
│   │   │   ├── activity-form.tsx
│   │   │   └── activity-calendar.tsx
│   │   ├── students/
│   │   │   ├── student-table.tsx
│   │   │   └── student-form.tsx
│   │   └── shared/
│   │       ├── data-table.tsx
│   │       ├── empty-state.tsx
│   │       ├── loading-spinner.tsx
│   │       ├── page-header.tsx
│   │       └── confirm-dialog.tsx
│   │
│   ├── lib/
│   │   ├── db/
│   │   │   ├── index.ts          # Neon + Drizzle client singleton
│   │   │   └── schema.ts         # Drizzle table definitions
│   │   ├── auth/
│   │   │   ├── config.ts         # Auth.js configuration
│   │   │   └── session.ts        # getSession helper for server components
│   │   ├── email/
│   │   │   └── send.ts           # Resend wrapper for parent emails
│   │   ├── validations/
│   │   │   ├── message.ts
│   │   │   ├── topic.ts
│   │   │   ├── test.ts
│   │   │   ├── activity.ts
│   │   │   └── student.ts
│   │   └── utils.ts              # cn(), formatDate(), etc.
│   │
│   ├── hooks/
│   │   ├── use-grade-filter.ts
│   │   └── use-semester-filter.ts
│   │
│   ├── types/
│   │   └── index.ts              # Shared TypeScript types / enums
│   │
│   └── middleware.ts             # Protect /dashboard routes, redirect unauthenticated
│
└── scripts/
    └── seed.ts                   # Seed grades, semesters, sample data
```

---

## 3. Backend API Endpoints

All endpoints require authentication unless noted. Responses follow `{ data, error }` or standard HTTP status codes with JSON error bodies.

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET/POST` | `/api/auth/[...nextauth]` | Auth.js session handlers (login, logout, callback) |

### Dashboard

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/dashboard/stats` | Summary: unread messages sent, upcoming activities count, recent test avg |

**Response example:**
```json
{
  "data": {
    "messagesSentThisMonth": 12,
    "upcomingActivities": 3,
    "studentsCount": 28,
    "recentTests": [{ "id": "...", "title": "Unit 2 Quiz", "avgScore": 78.5 }]
  }
}
```

### Messages

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/messages` | List messages sent by current teacher. Query: `?page=1&limit=20&gradeId=` |
| `POST` | `/api/messages` | Compose and send message to parent(s) |
| `GET` | `/api/messages/[id]` | Message detail with per-recipient delivery status |

**POST body:**
```json
{
  "subject": "Parent-Teacher Meeting Reminder",
  "body": "Dear parents, ...",
  "recipientType": "grade" | "students" | "all",
  "gradeId": "uuid",
  "studentIds": ["uuid", "uuid"]
}
```

### Curriculum (Topics)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/curriculum/grades` | List all grades |
| `GET` | `/api/curriculum/semesters` | List semesters. Query: `?academicYear=2025-2026` |
| `GET` | `/api/curriculum/topics` | List topics. Query: `?gradeId=&semesterId=` |
| `POST` | `/api/curriculum/topics` | Create topic |
| `GET` | `/api/curriculum/topics/[id]` | Single topic |
| `PATCH` | `/api/curriculum/topics/[id]` | Update topic |
| `DELETE` | `/api/curriculum/topics/[id]` | Delete topic |

**POST/PATCH body:**
```json
{
  "gradeId": "uuid",
  "semesterId": "uuid",
  "title": "Fractions and Decimals",
  "description": "Introduction to equivalent fractions...",
  "weekNumber": 4,
  "startDate": "2026-01-06",
  "endDate": "2026-01-17",
  "orderIndex": 1
}
```

### Tests & Results

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/tests` | List tests. Query: `?gradeId=&semesterId=` |
| `POST` | `/api/tests` | Create test |
| `GET` | `/api/tests/[id]` | Test detail with aggregated stats |
| `PATCH` | `/api/tests/[id]` | Update test metadata |
| `DELETE` | `/api/tests/[id]` | Delete test and associated results |
| `GET` | `/api/tests/[id]/results` | All student scores for a test |
| `POST` | `/api/tests/[id]/results` | Bulk upsert results |

**POST test body:**
```json
{
  "gradeId": "uuid",
  "semesterId": "uuid",
  "title": "Mid-Term Mathematics",
  "testDate": "2026-03-15",
  "maxScore": 100,
  "topicId": "uuid"
}
```

**POST results body (bulk):**
```json
{
  "results": [
    { "studentId": "uuid", "score": 85, "remarks": "Strong performance" },
    { "studentId": "uuid", "score": 72, "remarks": "" }
  ]
}
```

### Activities

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/activities` | List activities. Query: `?gradeId=&from=&to=&status=upcoming` |
| `POST` | `/api/activities` | Create activity |
| `GET` | `/api/activities/[id]` | Activity detail |
| `PATCH` | `/api/activities/[id]` | Update activity |
| `DELETE` | `/api/activities/[id]` | Delete activity |

**POST body:**
```json
{
  "gradeId": "uuid",
  "title": "Science Fair",
  "description": "Students present experiments...",
  "activityDate": "2026-04-20T09:00:00Z",
  "endDate": "2026-04-20T15:00:00Z",
  "location": "School Auditorium",
  "activityType": "event"
}
```

### Students

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/students` | List students. Query: `?gradeId=&search=` |
| `POST` | `/api/students` | Add student with parent contact |
| `GET` | `/api/students/[id]` | Student profile + recent results |
| `PATCH` | `/api/students/[id]` | Update student / parent info |
| `DELETE` | `/api/students/[id]` | Soft-delete or archive student |

**POST body:**
```json
{
  "gradeId": "uuid",
  "firstName": "Aarav",
  "lastName": "Sharma",
  "parentName": "Priya Sharma",
  "parentEmail": "priya@example.com",
  "parentPhone": "+91-9876543210"
}
```

---

## 4. Database Schema

PostgreSQL schema managed via Drizzle ORM. All primary keys are UUIDs (`gen_random_uuid()`). Timestamps use `timestamptz`.

### Entity Relationship Overview

```
users ──────────────┬──< messages
                    └──< teacher_grades >── grades

grades ──┬──< students ──┬──< test_results
         │               └──< message_recipients
         ├──< topics
         ├──< tests ──────< test_results
         └──< activities

semesters ──┬──< topics
            └──< tests

messages ──< message_recipients >── students
```

### Tables

#### `users`
Teachers and administrators who log into the system.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK, default `gen_random_uuid()` | |
| `email` | `varchar(255)` | UNIQUE, NOT NULL | Login identifier |
| `password_hash` | `varchar(255)` | NOT NULL | bcrypt hash |
| `name` | `varchar(255)` | NOT NULL | Display name |
| `role` | `varchar(20)` | NOT NULL, default `'teacher'` | `teacher` \| `admin` |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

#### `grades`
Grade levels (e.g., Grade 1, Grade 5).

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `name` | `varchar(50)` | UNIQUE, NOT NULL | e.g., "Grade 5" |
| `level` | `integer` | UNIQUE, NOT NULL | Numeric sort order (1–12) |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |

#### `semesters`
Academic semesters within a school year.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `name` | `varchar(50)` | NOT NULL | e.g., "Semester 1" |
| `academic_year` | `varchar(20)` | NOT NULL | e.g., "2025-2026" |
| `start_date` | `date` | NOT NULL | |
| `end_date` | `date` | NOT NULL | |
| `is_current` | `boolean` | NOT NULL, default `false` | Flag active semester |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |

**Unique:** `(name, academic_year)`

#### `teacher_grades`
Many-to-many: which grades a teacher is assigned to.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `user_id` | `uuid` | FK → `users.id`, NOT NULL | |
| `grade_id` | `uuid` | FK → `grades.id`, NOT NULL | |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |

**Unique:** `(user_id, grade_id)`

#### `students`
Students enrolled in a grade, with parent contact info.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `grade_id` | `uuid` | FK → `grades.id`, NOT NULL | |
| `first_name` | `varchar(100)` | NOT NULL | |
| `last_name` | `varchar(100)` | NOT NULL | |
| `parent_name` | `varchar(255)` | NOT NULL | |
| `parent_email` | `varchar(255)` | NOT NULL | Message recipient |
| `parent_phone` | `varchar(20)` | | Optional |
| `is_active` | `boolean` | NOT NULL, default `true` | Soft archive |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

**Index:** `(grade_id)`, `(parent_email)`

#### `topics`
Curriculum topics taught per grade and semester.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `grade_id` | `uuid` | FK → `grades.id`, NOT NULL | |
| `semester_id` | `uuid` | FK → `semesters.id`, NOT NULL | |
| `title` | `varchar(255)` | NOT NULL | |
| `description` | `text` | | Detailed syllabus notes |
| `week_number` | `integer` | | Optional week in semester |
| `start_date` | `date` | | Planned start |
| `end_date` | `date` | | Planned end |
| `order_index` | `integer` | NOT NULL, default `0` | Display sort order |
| `created_by` | `uuid` | FK → `users.id` | |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

**Index:** `(grade_id, semester_id, order_index)`

#### `tests`
Assessments linked to a grade and semester.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `grade_id` | `uuid` | FK → `grades.id`, NOT NULL | |
| `semester_id` | `uuid` | FK → `semesters.id`, NOT NULL | |
| `topic_id` | `uuid` | FK → `topics.id`, NULL | Optional link to curriculum topic |
| `title` | `varchar(255)` | NOT NULL | |
| `test_date` | `date` | NOT NULL | |
| `max_score` | `numeric(6,2)` | NOT NULL, default `100` | |
| `created_by` | `uuid` | FK → `users.id` | |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

**Index:** `(grade_id, semester_id, test_date)`

#### `test_results`
Individual student scores for a test.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `test_id` | `uuid` | FK → `tests.id`, ON DELETE CASCADE, NOT NULL | |
| `student_id` | `uuid` | FK → `students.id`, NOT NULL | |
| `score` | `numeric(6,2)` | NOT NULL | |
| `remarks` | `text` | | Teacher notes |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

**Unique:** `(test_id, student_id)`

#### `activities`
Upcoming and past school activities/events.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `grade_id` | `uuid` | FK → `grades.id`, NOT NULL | |
| `title` | `varchar(255)` | NOT NULL | |
| `description` | `text` | | |
| `activity_date` | `timestamptz` | NOT NULL | Start date/time |
| `end_date` | `timestamptz` | | Optional end |
| `location` | `varchar(255)` | | |
| `activity_type` | `varchar(50)` | NOT NULL, default `'event'` | `event` \| `field_trip` \| `exam` \| `holiday` \| `other` |
| `created_by` | `uuid` | FK → `users.id` | |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |
| `updated_at` | `timestamptz` | NOT NULL, default `now()` | |

**Index:** `(grade_id, activity_date)`

#### `messages`
Messages composed by a teacher.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `sender_id` | `uuid` | FK → `users.id`, NOT NULL | |
| `subject` | `varchar(500)` | NOT NULL | |
| `body` | `text` | NOT NULL | |
| `recipient_type` | `varchar(20)` | NOT NULL | `grade` \| `students` \| `all` |
| `grade_id` | `uuid` | FK → `grades.id`, NULL | Set when sending to whole grade |
| `sent_at` | `timestamptz` | NOT NULL, default `now()` | |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |

#### `message_recipients`
Per-parent delivery tracking for each message.

| Column | Type | Constraints | Notes |
|--------|------|-------------|-------|
| `id` | `uuid` | PK | |
| `message_id` | `uuid` | FK → `messages.id`, ON DELETE CASCADE, NOT NULL | |
| `student_id` | `uuid` | FK → `students.id`, NOT NULL | Links to parent via student |
| `parent_email` | `varchar(255)` | NOT NULL | Snapshot at send time |
| `delivery_status` | `varchar(20)` | NOT NULL, default `'pending'` | `pending` \| `sent` \| `failed` |
| `error_message` | `text` | | If delivery failed |
| `sent_at` | `timestamptz` | | When email was delivered |
| `created_at` | `timestamptz` | NOT NULL, default `now()` | |

**Index:** `(message_id)`, `(student_id)`

---

## 5. Environment Variables

```env
# Neon PostgreSQL
DATABASE_URL=postgresql://user:pass@ep-xxx.region.aws.neon.tech/neondb?sslmode=require

# Auth.js
AUTH_SECRET=                          # openssl rand -base64 32
AUTH_URL=http://localhost:3000        # Production: https://your-domain.com

# Email (Resend)
RESEND_API_KEY=
EMAIL_FROM=Teacher Helper <noreply@yourdomain.com>
```

---

## 6. Implementation Phases

### Phase 1 — Foundation
- Initialize Next.js project with TypeScript, Tailwind, and shadcn/ui
- Connect Neon via Drizzle; run initial migration
- Set up Auth.js with credentials provider
- Build dashboard layout (sidebar, header, middleware guard)

### Phase 2 — Core Data
- Seed grades and semesters
- Students CRUD (roster management)
- Teacher-grade assignment

### Phase 3 — Curriculum & Activities
- Topics CRUD with grade/semester filtering
- Activities list, create, calendar view

### Phase 4 — Tests & Results
- Test creation
- Bulk score entry grid
- Results display with averages

### Phase 5 — Messaging
- Compose message UI with recipient picker
- Email delivery via Resend
- Message history and delivery status

### Phase 6 — Polish
- Dashboard stats API
- Loading/error states, empty states
- Responsive mobile layout
- Deploy to Vercel + Neon production branch

---

## 7. Key Design Decisions

1. **Monolith over microservices** — Next.js Route Handlers keep the stack simple for a single-school or small multi-grade deployment.
2. **Grade-scoped data** — Most entities belong to a grade; teachers only see grades they are assigned to (enforced in API layer).
3. **Parent email snapshot** — `message_recipients.parent_email` stores the email at send time so historical records remain accurate if parent info changes later.
4. **Semester as first-class entity** — Topics and tests are always scoped to both grade and semester for clear academic organization.
5. **Soft delete for students** — `is_active` flag preserves historical test results when a student leaves.
6. **Future: parent portal** — Optional public route with token-based read-only access can be added without schema changes.

---

## 8. Next Steps

Once this plan is approved:

1. Run `npx create-next-app@latest` with App Router and Tailwind
2. Install Drizzle, Neon driver, Auth.js, and shadcn/ui
3. Create Neon project and apply schema migration
4. Implement Phase 1 (foundation) and iterate through phases
