# EduVerse Student Portal — Flutter Application Development Instructions

> **Reference UI:** `eduverse_student_portal_interactive.html`
> **Tech Stack:** Flutter (all platforms) · Supabase (DB + Auth) · OSM / flutter_map (Maps) · LangChain Dart (AI)

---

## 1. PROJECT OVERVIEW

EduVerse is an AI-powered School ERP Student Portal. The Flutter application must:
- Run on **Android, iOS, Web, Windows, macOS, Linux**
- Match the HTML mockup UI exactly (colors, gradients, cards, typography)
- Be **fully responsive** — from 320px phones to 2560px desktops
- Use **Supabase** as backend/database/auth/realtime
- Use **flutter_map + OpenStreetMap** for transport/bus live tracking
- Use **LangChain Dart** for AI assistant with tool-calling
- Use **Outfit + DM Sans** fonts (Google Fonts package)

---

## 2. DESIGN SYSTEM

### 2.1 Color Palette
```dart
// lib/core/theme/app_colors.dart
static const primary     = Color(0xFF4F46E5);
static const primaryDeep = Color(0xFF3730A3);
static const accent      = Color(0xFF06B6D4);
static const accent2     = Color(0xFFF59E0B);
static const accent3     = Color(0xFF10B981);
static const accent4     = Color(0xFFEF4444);
static const accent5     = Color(0xFF8B5CF6);
static const bgLight     = Color(0xFFF0F2FF);
static const surface     = Color(0xFFFFFFFF);
static const card        = Color(0xFFFAFAFA);
static const textPrimary = Color(0xFF0F172A);
static const textSec     = Color(0xFF475569);
static const textTert    = Color(0xFF94A3B8);
static const border      = Color(0xFFE2E8F0);
```

### 2.2 Typography
```dart
// lib/core/theme/app_text_styles.dart
// Outfit — headings/labels
// DM Sans — body text
```

### 2.3 Responsive Breakpoints
```dart
// lib/core/utils/responsive.dart
class Breakpoints {
  static const mobile  = 480.0;
  static const tablet  = 768.0;
  static const desktop = 1200.0;
}
// Use LayoutBuilder + MediaQuery throughout
// On tablet/desktop: show sidebar nav instead of bottom nav
// Content max-width: 480px centered on large screens
```

### 2.4 Gradients (match HTML)
```dart
// Splash/Login: LinearGradient(0xFF0F0C29 → 0xFF302B63 → 0xFF24243E)
// Dashboard header: LinearGradient(0xFF4F46E5 → 0xFF302B63)
// Primary button: LinearGradient(0xFF4F46E5 → 0xFF06B6D4)
// AI FAB: LinearGradient(0xFF4F46E5 → 0xFF06B6D4)
```

---

## 3. SUPABASE DATABASE SCHEMA

### Table 1: `schools`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
name          text NOT NULL
address       text
logo_url      text
phone         text
email         text
academic_year text
created_at    timestamptz DEFAULT now()
```

### Table 2: `users`
```sql
id            uuid PRIMARY KEY REFERENCES auth.users(id)
school_id     uuid REFERENCES schools(id)
role          text CHECK (role IN ('student','teacher','parent','admin'))
full_name     text NOT NULL
avatar_url    text
phone         text
email         text
is_active     boolean DEFAULT true
created_at    timestamptz DEFAULT now()
```

### Table 3: `students`
```sql
id               uuid PRIMARY KEY DEFAULT gen_random_uuid()
user_id          uuid REFERENCES users(id) UNIQUE
school_id        uuid REFERENCES schools(id)
admission_no     text UNIQUE
roll_no          integer
class_name       text        -- e.g. "X-A"
section          text
academic_year    text
date_of_birth    date
gender           text CHECK (gender IN ('Male','Female','Other'))
blood_group      text
nationality      text DEFAULT 'Indian'
religion         text
category         text DEFAULT 'General'
address          text
house            text        -- Blue/Red/Green/Yellow
father_name      text
father_phone     text
father_occupation text
mother_name      text
mother_phone     text
mother_occupation text
guardian_name    text
xp_points        integer DEFAULT 0
streak_days      integer DEFAULT 0
rank_class       integer
rank_school      integer
created_at       timestamptz DEFAULT now()
```

### Table 4: `subjects`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id    uuid REFERENCES schools(id)
name         text NOT NULL       -- Mathematics, Physics, etc.
code         text
icon_emoji   text
teacher_id   uuid REFERENCES users(id)
class_name   text
chapters     integer DEFAULT 0
created_at   timestamptz DEFAULT now()
```

### Table 5: `timetable`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id    uuid REFERENCES schools(id)
class_name   text
day_of_week  integer CHECK (day_of_week BETWEEN 0 AND 6)
period_no    integer
start_time   time NOT NULL
end_time     time NOT NULL
subject_id   uuid REFERENCES subjects(id)
teacher_id   uuid REFERENCES users(id)
room         text
is_break     boolean DEFAULT false
break_label  text        -- "Break", "Lunch"
created_at   timestamptz DEFAULT now()
```

### Table 6: `attendance`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id   uuid REFERENCES students(id)
subject_id   uuid REFERENCES subjects(id)
date         date NOT NULL
status       text CHECK (status IN ('present','absent','late','half_day'))
marked_by    uuid REFERENCES users(id)
remarks      text
created_at   timestamptz DEFAULT now()
UNIQUE(student_id, subject_id, date)
```

### Table 7: `homework`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id     uuid REFERENCES schools(id)
subject_id    uuid REFERENCES subjects(id)
teacher_id    uuid REFERENCES users(id)
class_name    text
title         text NOT NULL
description   text
due_date      timestamptz NOT NULL
reference_url text
created_at    timestamptz DEFAULT now()
```

### Table 8: `homework_submissions`
```sql
id             uuid PRIMARY KEY DEFAULT gen_random_uuid()
homework_id    uuid REFERENCES homework(id)
student_id     uuid REFERENCES students(id)
file_url       text
notes          text
submitted_at   timestamptz DEFAULT now()
status         text DEFAULT 'submitted' CHECK (status IN ('submitted','graded','late'))
grade          text
teacher_remarks text
```

### Table 9: `exams`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id     uuid REFERENCES schools(id)
subject_id    uuid REFERENCES subjects(id)
class_name    text
title         text NOT NULL
exam_type     text CHECK (exam_type IN ('Class Test','Lab Test','Assignment','Mid-Term','End-Term','Other'))
exam_date     date
start_time    time
duration_mins integer
total_marks   integer
hall          text
academic_year text
is_online     boolean DEFAULT false
created_at    timestamptz DEFAULT now()
```

### Table 10: `exam_questions`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
exam_id       uuid REFERENCES exams(id)
question_no   integer
question_text text NOT NULL
question_type text CHECK (question_type IN ('objective','subjective'))
option_a      text
option_b      text
option_c      text
option_d      text
correct_option text
marks_positive numeric DEFAULT 4
marks_negative numeric DEFAULT 1
created_at    timestamptz DEFAULT now()
```

### Table 11: `exam_submissions`
```sql
id              uuid PRIMARY KEY DEFAULT gen_random_uuid()
exam_id         uuid REFERENCES exams(id)
student_id      uuid REFERENCES students(id)
started_at      timestamptz
submitted_at    timestamptz
answers         jsonb        -- {q_id: selected_option}
score           numeric
integrity_flags jsonb        -- AI monitoring data
submission_id   text UNIQUE  -- EV-EX-2024-XXXX
file_url        text         -- for uploaded answer sheets
created_at      timestamptz DEFAULT now()
```

### Table 12: `results`
```sql
id             uuid PRIMARY KEY DEFAULT gen_random_uuid()
exam_id        uuid REFERENCES exams(id)
student_id     uuid REFERENCES students(id)
subject_id     uuid REFERENCES subjects(id)
marks_obtained numeric
total_marks    numeric
grade          text
percentage     numeric
rank_class     integer
academic_year  text
published_at   timestamptz
created_at     timestamptz DEFAULT now()
```

### Table 13: `fees`
```sql
id             uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id     uuid REFERENCES students(id)
school_id      uuid REFERENCES schools(id)
fee_type       text      -- Tuition, Transport, Lab, etc.
amount         numeric NOT NULL
due_date       date
status         text CHECK (status IN ('pending','paid','partial'))
academic_year  text
quarter        text
created_at     timestamptz DEFAULT now()
```

### Table 14: `payments`
```sql
id              uuid PRIMARY KEY DEFAULT gen_random_uuid()
fee_id          uuid REFERENCES fees(id)
student_id      uuid REFERENCES students(id)
amount_paid     numeric
payment_method  text CHECK (payment_method IN ('upi','card','netbanking','cash'))
transaction_id  text
paid_at         timestamptz DEFAULT now()
status          text DEFAULT 'confirmed'
receipt_url     text
```

### Table 15: `notices`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id    uuid REFERENCES schools(id)
title        text NOT NULL
body         text NOT NULL
notice_type  text CHECK (notice_type IN ('urgent','general','event'))
published_by uuid REFERENCES users(id)
attachment_url text
published_at timestamptz DEFAULT now()
expires_at   timestamptz
```

### Table 16: `events`
```sql
id             uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id      uuid REFERENCES schools(id)
title          text NOT NULL
description    text
event_date     date
start_time     time
venue          text
banner_url     text
created_by     uuid REFERENCES users(id)
created_at     timestamptz DEFAULT now()
```

### Table 17: `event_registrations`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
event_id     uuid REFERENCES events(id)
student_id   uuid REFERENCES students(id)
registered_at timestamptz DEFAULT now()
UNIQUE(event_id, student_id)
```

### Table 18: `leave_applications`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id    uuid REFERENCES students(id)
leave_type    text CHECK (leave_type IN ('sick','casual','urgent','family','other'))
start_date    date NOT NULL
end_date      date NOT NULL
total_days    integer
reason        text NOT NULL
document_url  text
status        text DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected','cancelled'))
applied_at    timestamptz DEFAULT now()
reviewed_by   uuid REFERENCES users(id)
reviewed_at   timestamptz
remarks       text
```

### Table 19: `transport`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id     uuid REFERENCES schools(id)
bus_number    text NOT NULL   -- HR-29-3847
route_name    text            -- Route 7B
driver_name   text
driver_phone  text
capacity      integer
created_at    timestamptz DEFAULT now()
```

### Table 20: `transport_stops`
```sql
id           uuid PRIMARY KEY DEFAULT gen_random_uuid()
transport_id uuid REFERENCES transport(id)
stop_name    text NOT NULL
stop_order   integer
lat          double precision
lng          double precision
scheduled_time time
```

### Table 21: `student_transport`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id    uuid REFERENCES students(id)
transport_id  uuid REFERENCES transport(id)
stop_id       uuid REFERENCES transport_stops(id)
seat_no       text
academic_year text
UNIQUE(student_id, academic_year)
```

### Table 22: `bus_location` (realtime)
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
transport_id  uuid REFERENCES transport(id)
lat           double precision
lng           double precision
speed         double precision
heading       double precision
updated_at    timestamptz DEFAULT now()
```

### Table 23: `live_classes`
```sql
id             uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id      uuid REFERENCES schools(id)
subject_id     uuid REFERENCES subjects(id)
teacher_id     uuid REFERENCES users(id)
title          text NOT NULL
class_name     text
stream_url     text
is_live        boolean DEFAULT false
started_at     timestamptz
ended_at       timestamptz
recording_url  text
viewer_count   integer DEFAULT 0
created_at     timestamptz DEFAULT now()
```

### Table 24: `live_class_comments`
```sql
id              uuid PRIMARY KEY DEFAULT gen_random_uuid()
live_class_id   uuid REFERENCES live_classes(id)
user_id         uuid REFERENCES users(id)
message         text NOT NULL
created_at      timestamptz DEFAULT now()
```

### Table 25: `messages`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
sender_id     uuid REFERENCES users(id)
receiver_id   uuid REFERENCES users(id)
group_id      uuid REFERENCES message_groups(id)
content       text
file_url      text
is_read       boolean DEFAULT false
created_at    timestamptz DEFAULT now()
```

### Table 26: `message_groups`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
name          text
icon_url      text
created_by    uuid REFERENCES users(id)
school_id     uuid REFERENCES schools(id)
created_at    timestamptz DEFAULT now()
```

### Table 27: `message_group_members`
```sql
id         uuid PRIMARY KEY DEFAULT gen_random_uuid()
group_id   uuid REFERENCES message_groups(id)
user_id    uuid REFERENCES users(id)
role       text DEFAULT 'member'
joined_at  timestamptz DEFAULT now()
UNIQUE(group_id, user_id)
```

### Table 28: `library_books`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
school_id     uuid REFERENCES schools(id)
title         text NOT NULL
author        text
isbn          text
category      text
cover_url     text
total_copies  integer DEFAULT 1
available     integer DEFAULT 1
created_at    timestamptz DEFAULT now()
```

### Table 29: `library_issues`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
book_id       uuid REFERENCES library_books(id)
student_id    uuid REFERENCES students(id)
issued_at     timestamptz DEFAULT now()
due_date      date
returned_at   timestamptz
status        text DEFAULT 'issued' CHECK (status IN ('issued','returned','overdue'))
```

### Table 30: `achievements`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id    uuid REFERENCES students(id)
title         text NOT NULL
description   text
badge_emoji   text
badge_type    text CHECK (badge_type IN ('academic','sports','streak','event','other'))
xp_earned     integer DEFAULT 0
earned_at     timestamptz DEFAULT now()
is_locked     boolean DEFAULT false
unlock_criteria text
```

### Table 31: `notifications`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
user_id       uuid REFERENCES users(id)
title         text NOT NULL
body          text
type          text     -- homework, fee, result, bus, event, etc.
deep_link     text     -- screen to navigate to
is_read       boolean DEFAULT false
created_at    timestamptz DEFAULT now()
```

### Table 32: `student_documents`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
student_id    uuid REFERENCES students(id)
doc_type      text    -- birth_cert, aadhaar, marksheet, etc.
file_url      text
verified      boolean DEFAULT false
uploaded_at   timestamptz DEFAULT now()
```

### Table 33: `ai_chat_history`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
user_id       uuid REFERENCES users(id)
session_id    uuid
role          text CHECK (role IN ('user','assistant'))
content       text NOT NULL
tool_calls    jsonb
created_at    timestamptz DEFAULT now()
```

---

## 4. LANGCHAIN AI — TOOL CALLING FUNCTIONS

All tools are registered with the LangChain Dart agent. Each has a name, description, and input schema.

### Tool 1: `get_student_dashboard`
```dart
// Description: Returns student's dashboard data including streak, XP, stats
// Input: { student_id: String }
// Returns: { streak_days, xp_points, attendance_pct, avg_score, rank_class, today_schedule[], pending_homework[] }
// Supabase: SELECT from students JOIN attendance JOIN results
```

### Tool 2: `get_timetable`
```dart
// Description: Returns timetable for a specific day
// Input: { student_id: String, day_of_week: int }
// Returns: List<{ time, subject, teacher, room, status }>
// Supabase: SELECT from timetable WHERE class_name = student.class_name AND day_of_week = ?
```

### Tool 3: `get_homework`
```dart
// Description: Returns pending/submitted/graded homework
// Input: { student_id: String, status_filter: String? }
// Returns: List<{ title, subject, due_date, status, description }>
// Supabase: SELECT from homework LEFT JOIN homework_submissions
```

### Tool 4: `submit_homework`
```dart
// Description: Submits homework assignment on behalf of student
// Input: { homework_id: String, student_id: String, notes: String, file_url: String? }
// Returns: { success: bool, submission_id: String }
// Supabase: INSERT into homework_submissions
```

### Tool 5: `get_results`
```dart
// Description: Returns exam results filtered by year/category
// Input: { student_id: String, academic_year: String, exam_type: String? }
// Returns: { overall_score, grade, rank, subject_results[] }
// Supabase: SELECT from results JOIN exams WHERE student_id = ? AND academic_year = ?
```

### Tool 6: `get_attendance`
```dart
// Description: Returns attendance summary overall and per subject
// Input: { student_id: String, month: int?, year: int? }
// Returns: { overall_pct, present_days, absent_days, subject_breakdown[] }
// Supabase: SELECT from attendance WHERE student_id = ?
```

### Tool 7: `get_fees`
```dart
// Description: Returns fee details and outstanding balance
// Input: { student_id: String }
// Returns: { outstanding_balance, fee_items[], payment_history[] }
// Supabase: SELECT from fees LEFT JOIN payments WHERE student_id = ?
```

### Tool 8: `get_bus_location`
```dart
// Description: Returns real-time bus location and ETA for student's stop
// Input: { student_id: String }
// Returns: { bus_number, route, lat, lng, eta_minutes, stops_remaining, driver_phone }
// Supabase: SELECT from bus_location + student_transport + transport_stops
```

### Tool 9: `get_exam_schedule`
```dart
// Description: Returns upcoming exam schedule
// Input: { student_id: String, upcoming_days: int? }
// Returns: List<{ exam_title, subject, date, time, duration, hall }>
// Supabase: SELECT from exams WHERE class_name = student.class_name AND exam_date >= now()
```

### Tool 10: `get_ai_study_plan`
```dart
// Description: Generates personalized study plan based on results and upcoming exams
// Input: { student_id: String, subject: String? }
// Returns: { focus_topics[], weak_areas[], study_schedule[], tips[] }
// Logic: Analyze results + exam dates + syllabus coverage using LLM
```

### Tool 11: `get_notices`
```dart
// Description: Returns school notices filtered by type
// Input: { school_id: String, notice_type: String?, limit: int? }
// Returns: List<{ title, body, type, published_at, attachment_url }>
// Supabase: SELECT from notices WHERE school_id = ? ORDER BY published_at DESC
```

### Tool 12: `apply_leave`
```dart
// Description: Submits a leave application
// Input: { student_id: String, leave_type: String, start_date: String, end_date: String, reason: String }
// Returns: { success: bool, leave_id: String, message: String }
// Supabase: INSERT into leave_applications
```

### Tool 13: `get_leave_status`
```dart
// Description: Returns leave application history
// Input: { student_id: String }
// Returns: { quota: int, used: int, pending: int, balance: int, applications[] }
// Supabase: SELECT from leave_applications WHERE student_id = ?
```

### Tool 14: `get_achievements`
```dart
// Description: Returns student badges and XP
// Input: { student_id: String }
// Returns: { total_xp, class_rank, school_rank, badges[] }
// Supabase: SELECT from achievements WHERE student_id = ?
```

### Tool 15: `get_library_books`
```dart
// Description: Returns student's issued books and AI recommendations
// Input: { student_id: String }
// Returns: { issued_books[], ai_recommendations[] }
// Supabase: SELECT from library_issues + library_books WHERE student_id = ?
```

### Tool 16: `get_live_classes`
```dart
// Description: Returns live and upcoming classes
// Input: { student_id: String }
// Returns: { live_now[], upcoming_today[], recorded[] }
// Supabase: SELECT from live_classes WHERE school_id = ? AND class_name = ?
```

### Tool 17: `send_message`
```dart
// Description: Sends a message to a teacher or student
// Input: { sender_id: String, receiver_id: String, content: String }
// Returns: { success: bool, message_id: String }
// Supabase: INSERT into messages
```

### Tool 18: `get_messages`
```dart
// Description: Returns message threads/conversations
// Input: { user_id: String }
// Returns: List<{ contact_name, last_message, unread_count, timestamp }>
// Supabase: SELECT from messages WHERE sender_id = ? OR receiver_id = ?
```

### Tool 19: `get_exam_topic_weights`
```dart
// Description: Analyzes past exam patterns to predict topic weights
// Input: { subject: String, class_name: String }
// Returns: { topics: [{name, weight_pct, student_weakness}] }
// Logic: LLM analysis of historical results data
```

### Tool 20: `download_report_card`
```dart
// Description: Generates and returns report card PDF URL
// Input: { student_id: String, academic_year: String }
// Returns: { pdf_url: String, filename: String }
// Logic: Generate PDF using server function, store in Supabase storage
```

---

## 5. FOLDER STRUCTURE — `lib/`

```
lib/
├── main.dart
├── app.dart                          # MaterialApp, routes, theme
│
├── core/
│   ├── theme/
│   │   ├── app_colors.dart
│   │   ├── app_text_styles.dart
│   │   ├── app_theme.dart
│   │   └── app_gradients.dart
│   ├── utils/
│   │   ├── responsive.dart           # Breakpoints + layout helpers
│   │   ├── date_utils.dart
│   │   ├── string_utils.dart
│   │   └── validators.dart
│   ├── constants/
│   │   ├── supabase_constants.dart
│   │   └── app_constants.dart
│   ├── router/
│   │   └── app_router.dart           # go_router configuration
│   └── di/
│       └── injection.dart            # GetIt service locator
│
├── services/
│   ├── supabase_service.dart         # Supabase client init
│   ├── auth_service.dart             # Login/logout/OTP/biometric
│   ├── storage_service.dart          # File upload to Supabase storage
│   ├── notification_service.dart     # FCM + local notifications
│   ├── pdf_service.dart              # Report card PDF generation
│   └── ai/
│       ├── langchain_service.dart    # LangChain agent setup
│       ├── ai_tools.dart             # All 20 tool definitions
│       └── ai_memory.dart           # Conversation memory
│
├── models/
│   ├── student_model.dart
│   ├── subject_model.dart
│   ├── timetable_model.dart
│   ├── attendance_model.dart
│   ├── homework_model.dart
│   ├── exam_model.dart
│   ├── result_model.dart
│   ├── fee_model.dart
│   ├── payment_model.dart
│   ├── notice_model.dart
│   ├── event_model.dart
│   ├── leave_model.dart
│   ├── transport_model.dart
│   ├── live_class_model.dart
│   ├── message_model.dart
│   ├── library_model.dart
│   ├── achievement_model.dart
│   └── notification_model.dart
│
├── repositories/
│   ├── auth_repository.dart
│   ├── student_repository.dart
│   ├── timetable_repository.dart
│   ├── attendance_repository.dart
│   ├── homework_repository.dart
│   ├── exam_repository.dart
│   ├── result_repository.dart
│   ├── fee_repository.dart
│   ├── notice_repository.dart
│   ├── event_repository.dart
│   ├── leave_repository.dart
│   ├── transport_repository.dart
│   ├── live_class_repository.dart
│   ├── message_repository.dart
│   ├── library_repository.dart
│   └── achievement_repository.dart
│
└── features/
    └── students/
        ├── splash/
        │   ├── splash_screen.dart
        │   └── splash_cubit.dart
        │
        ├── auth/
        │   ├── login_screen.dart
        │   ├── login_cubit.dart
        │   ├── widgets/
        │   │   ├── role_selector_card.dart
        │   │   ├── biometric_modal.dart
        │   │   ├── otp_modal.dart
        │   │   └── qr_scan_modal.dart
        │
        ├── shell/
        │   ├── app_shell.dart            # Scaffold with bottom nav / sidebar
        │   ├── bottom_nav_bar.dart
        │   ├── sidebar_nav.dart          # Desktop/tablet sidebar
        │   └── ai_fab_button.dart
        │
        ├── dashboard/
        │   ├── dashboard_screen.dart
        │   ├── dashboard_cubit.dart
        │   └── widgets/
        │       ├── dashboard_header.dart    # Gradient header with greeting
        │       ├── streak_card.dart
        │       ├── stats_row.dart
        │       ├── quick_access_grid.dart
        │       ├── quick_access_item.dart
        │       ├── today_schedule_card.dart
        │       ├── schedule_item_row.dart
        │       ├── ai_insight_card.dart
        │       └── pending_homework_card.dart
        │
        ├── timetable/
        │   ├── timetable_screen.dart
        │   ├── timetable_cubit.dart
        │   └── widgets/
        │       ├── day_chip_selector.dart
        │       ├── period_card.dart
        │       ├── break_card.dart
        │       └── period_detail_modal.dart
        │
        ├── results/
        │   ├── results_screen.dart
        │   ├── results_cubit.dart
        │   └── widgets/
        │       ├── overall_score_card.dart
        │       ├── result_category_chips.dart
        │       ├── subject_result_card.dart
        │       ├── year_picker_modal.dart
        │       ├── download_modal.dart
        │       └── share_modal.dart
        │
        ├── exams/
        │   ├── exams_screen.dart
        │   ├── exams_cubit.dart
        │   ├── exam_instructions_screen.dart
        │   ├── online_exam_screen.dart
        │   ├── exam_success_screen.dart
        │   ├── online_exam_cubit.dart
        │   └── widgets/
        │       ├── countdown_widget.dart
        │       ├── exam_schedule_card.dart
        │       ├── ai_prep_card.dart
        │       ├── question_palette.dart
        │       ├── palette_pill.dart
        │       ├── mcq_option_card.dart
        │       ├── subjective_input.dart
        │       ├── exam_timer.dart
        │       ├── exam_bottom_bar.dart
        │       ├── section_switcher.dart
        │       ├── upload_answer_sheet_modal.dart
        │       ├── submit_exam_modal.dart
        │       └── exit_exam_modal.dart
        │
        ├── fees/
        │   ├── fees_screen.dart
        │   ├── fees_cubit.dart
        │   └── widgets/
        │       ├── outstanding_balance_card.dart
        │       ├── fee_item_card.dart
        │       ├── fee_status_badge.dart
        │       ├── payment_method_modal.dart
        │       └── payment_success_modal.dart
        │
        ├── notices/
        │   ├── notices_screen.dart
        │   ├── notices_cubit.dart
        │   └── widgets/
        │       ├── notice_card.dart
        │       ├── notice_type_chip.dart
        │       └── notice_detail_modal.dart
        │
        ├── homework/
        │   ├── homework_screen.dart
        │   ├── homework_cubit.dart
        │   └── widgets/
        │       ├── homework_tab_selector.dart
        │       ├── homework_card.dart
        │       ├── submit_homework_modal.dart
        │       ├── homework_success_modal.dart
        │       └── ai_helper_card.dart
        │
        ├── transport/
        │   ├── transport_screen.dart
        │   ├── transport_cubit.dart
        │   └── widgets/
        │       ├── osm_bus_map.dart          # flutter_map + OSM tiles
        │       ├── bus_info_card.dart
        │       ├── route_stops_list.dart
        │       ├── stop_item.dart
        │       ├── eta_badge.dart
        │       ├── fullmap_screen.dart       # Full-screen flutter_map
        │       └── bus_alert_modal.dart
        │
        ├── events/
        │   ├── events_screen.dart
        │   ├── events_cubit.dart
        │   └── widgets/
        │       ├── event_card.dart
        │       ├── event_banner.dart
        │       └── event_register_modal.dart
        │
        ├── achievements/
        │   ├── achievements_screen.dart
        │   ├── achievements_cubit.dart
        │   └── widgets/
        │       ├── xp_summary_card.dart
        │       ├── badge_card.dart
        │       └── locked_badge_card.dart
        │
        ├── attendance/
        │   ├── attendance_screen.dart
        │   ├── attendance_cubit.dart
        │   └── widgets/
        │       ├── attendance_ring_chart.dart
        │       ├── attendance_stats_row.dart
        │       ├── subject_attendance_card.dart
        │       └── attendance_alert_card.dart
        │
        ├── leave/
        │   ├── leave_screen.dart
        │   ├── leave_cubit.dart
        │   └── widgets/
        │       ├── leave_dashboard_card.dart
        │       ├── leave_tab_selector.dart
        │       ├── apply_leave_form.dart
        │       ├── leave_card.dart
        │       ├── past_leave_card.dart
        │       ├── leave_success_modal.dart
        │       └── leave_cancel_modal.dart
        │
        ├── profile/
        │   ├── profile_screen.dart
        │   ├── profile_cubit.dart
        │   └── widgets/
        │       ├── profile_header.dart
        │       ├── profile_stats_bar.dart
        │       ├── info_section.dart
        │       ├── info_row.dart
        │       ├── documents_section.dart
        │       └── edit_profile_modal.dart
        │
        ├── ai_chat/
        │   ├── ai_chat_screen.dart
        │   ├── ai_chat_cubit.dart
        │   └── widgets/
        │       ├── chat_message_bubble.dart
        │       ├── ai_bubble.dart
        │       ├── user_bubble.dart
        │       ├── typing_indicator.dart
        │       ├── suggestion_chips.dart
        │       └── chat_input_bar.dart
        │
        ├── library/
        │   ├── library_screen.dart
        │   ├── library_cubit.dart
        │   └── widgets/
        │       ├── book_card.dart
        │       ├── ai_book_recommendation.dart
        │       └── return_book_modal.dart
        │
        ├── courses/
        │   ├── courses_screen.dart
        │   ├── courses_cubit.dart
        │   └── widgets/
        │       ├── course_card.dart
        │       ├── syllabus_progress_bar.dart
        │       └── play_recording_button.dart
        │
        ├── notifications/
        │   ├── notifications_screen.dart
        │   ├── notifications_cubit.dart
        │   └── widgets/
        │       └── notification_item_card.dart
        │
        ├── settings/
        │   ├── settings_screen.dart
        │   ├── settings_cubit.dart
        │   └── widgets/
        │       ├── settings_section.dart
        │       ├── settings_toggle_row.dart
        │       ├── settings_nav_row.dart
        │       └── logout_modal.dart
        │
        ├── messaging/
        │   ├── messaging_screen.dart
        │   ├── messaging_cubit.dart
        │   ├── chat_detail_screen.dart
        │   ├── chat_detail_cubit.dart
        │   ├── create_group_screen.dart
        │   └── widgets/
        │       ├── chat_thread_tile.dart
        │       ├── message_bubble.dart
        │       ├── message_input_bar.dart
        │       ├── category_chip_row.dart
        │       └── participant_selector.dart
        │
        └── live_classes/
            ├── live_classes_screen.dart
            ├── live_classes_cubit.dart
            ├── live_class_detail_screen.dart
            └── widgets/
                ├── live_class_card.dart
                ├── upcoming_class_tile.dart
                ├── recorded_class_tile.dart
                ├── video_player_area.dart
                ├── live_badge.dart
                ├── comment_section.dart
                ├── comment_input_bar.dart
                └── action_buttons_row.dart
```

---

## 6. SHARED / REUSABLE WIDGETS

```
lib/shared/widgets/
├── ev_app_bar.dart               # Gradient AppBar with back button
├── ev_gradient_card.dart         # Glass/gradient card container
├── ev_button.dart                # Primary/Secondary/Danger/Success buttons
├── ev_input_field.dart           # Styled text field
├── ev_chip.dart                  # Filter chips
├── ev_badge.dart                 # Status badges (PAID/DUE/PENDING)
├── ev_modal_sheet.dart           # Bottom sheet wrapper with handle
├── ev_section_header.dart        # "Section Title" + "View All →"
├── ev_ai_tag.dart                # "🤖 AI INSIGHT" badge
├── ev_toggle.dart                # iOS-style toggle switch
├── ev_avatar.dart                # User avatar with gradient fallback
├── ev_loading.dart               # Shimmer skeleton loaders
├── ev_empty_state.dart           # Empty state with illustration
├── ev_error_state.dart           # Error state with retry
└── ev_confetti.dart              # Confetti animation widget
```

---

## 7. STATE MANAGEMENT

Use **flutter_bloc (Cubit)** for all screens.

### State Pattern
```dart
// Each cubit follows this pattern:
class ScreenState {
  final Status status;    // loading, success, failure
  final T? data;
  final String? error;
}
// BlocBuilder + BlocListener in screens
```

---

## 8. NAVIGATION (go_router)

```dart
// lib/core/router/app_router.dart
Routes:
  /splash
  /login
  /             → shell (bottom nav)
    /dashboard
    /timetable
    /results
    /exams
    /exam-instructions
    /exam-internal
    /exam-success
    /fees
    /notices
    /homework
    /transport
    /transport/fullmap
    /events
    /achievements
    /attendance
    /leave
    /profile
    /ai-chat
    /library
    /courses
    /notifications
    /settings
    /messaging
    /messaging/chat/:id
    /messaging/create-group
    /live-classes
    /live-classes/:id
    /live-classes/:id/detail
```

---

## 9. OSM / MAPS IMPLEMENTATION

```yaml
# pubspec.yaml
flutter_map: ^6.0.0
latlong2: ^0.9.0
```

```dart
// lib/features/students/transport/widgets/osm_bus_map.dart
FlutterMap(
  options: MapOptions(
    initialCenter: LatLng(lat, lng),
    initialZoom: 14,
  ),
  children: [
    TileLayer(
      urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
      userAgentPackageName: 'com.eduverse.app',
    ),
    PolylineLayer(polylines: [routePolyline]),
    MarkerLayer(markers: [busMarker, schoolMarker, studentStopMarker]),
  ],
)
// Supabase Realtime subscription for bus_location table
// Updates bus marker position every 5 seconds
```

---

## 10. LANGCHAIN DART AGENT SETUP

```yaml
# pubspec.yaml
langchain: ^0.7.0
langchain_openai: ^0.7.0  # or langchain_google
```

```dart
// lib/services/ai/langchain_service.dart
class LangChainService {
  late final ChatOpenAI llm;
  late final OpenAIToolsAgent agent;
  late final AgentExecutor executor;

  Future<void> init() async {
    llm = ChatOpenAI(apiKey: env.openaiKey, model: 'gpt-4o');
    final tools = AiTools.allTools(studentId: currentStudentId);
    agent = OpenAIToolsAgent.fromLLMAndTools(llm: llm, tools: tools);
    executor = AgentExecutor(agent: agent);
  }

  Future<String> chat(String userMessage, {String? sessionId}) async {
    // Store message in ai_chat_history
    // Run executor with memory context
    // Store response in ai_chat_history
  }
}
```

---

## 11. PLATFORM RESPONSIVE LAYOUT RULES

### Mobile (< 480px)
- Bottom navigation bar (5 items)
- Single column layout
- AI FAB fixed at bottom-right
- Full-screen modals as bottom sheets

### Tablet (480px – 1200px)
- Side navigation rail (collapsed icons)
- Two-column layout for grids (e.g. quick access 4→6 per row)
- Modals as centered dialogs

### Desktop (> 1200px)
- Full sidebar navigation with labels
- Content max-width 480px centered OR two-pane layout
- Modals as centered dialogs with fixed max-width
- Hover effects on all interactive elements

---

## 12. PUBSPEC DEPENDENCIES

```yaml
dependencies:
  flutter_bloc: ^8.1.6
  go_router: ^14.0.0
  supabase_flutter: ^2.5.0
  flutter_map: ^6.0.0
  latlong2: ^0.9.0
  google_fonts: ^6.2.1
  langchain: ^0.7.0
  langchain_openai: ^0.7.0
  flutter_dotenv: ^5.1.0
  file_picker: ^8.0.0
  image_picker: ^1.0.7
  pdf: ^3.10.8
  printing: ^5.12.0
  local_auth: ^2.2.0
  mobile_scanner: ^5.0.0
  fl_chart: ^0.69.0
  shimmer: ^3.0.0
  cached_network_image: ^3.3.1
  intl: ^0.19.0
  get_it: ^8.0.0
  equatable: ^2.0.5
  json_annotation: ^4.9.0
  dio: ^5.4.3
  path_provider: ^2.1.3

dev_dependencies:
  build_runner: ^2.4.9
  json_serializable: ^6.8.0
  flutter_lints: ^4.0.0
```

---

## 13. SCREENS & FEATURES DETAIL

### 13.1 Splash Screen
- Animated gradient background (0F0C29 → 302B63 → 24243E)
- EduVerse logo with spinning ring animation
- Progress bar that fills from 0→100% in 2.5 seconds
- Auto navigate to Login

### 13.2 Login Screen
- Dark gradient background
- Role selector: Student / Parent / Teacher (3 cards)
- Student ID + Password fields (dark style)
- "Login with AI Assist" primary button
- Face ID / OTP / QR Scan quick login options (3 modals)
- All 3 modals implemented as bottom sheets

### 13.3 Dashboard Screen
- Gradient header with: greeting, notification bell (badge), avatar
- Streak card + Stats row (Attendance%, Avg Score, Rank, XP)
- 4×4 Quick Access Grid (16 items with colored icon containers)
- Today's Schedule card (3 items, current class highlighted)
- AI Insight card (gradient border, indigo text)
- Pending Homework card (2 items)
- Floating AI Robot button (pulsing animation)
- Bottom navigation: Home / Courses / Results / Achieve / Profile

### 13.4 Timetable Screen
- Day chip selector (Mon-Sat, today highlighted)
- Period cards with left color accent bar
- Break/Lunch cards in amber style
- Tap any period → modal with full details
- "NOW" badge on current period

### 13.5 Results Screen
- Category filter chips: Class Test / Lab Test / Assignment / Mid-Term / End-Term / Other / Overall
- Year picker (2024/2025/2026)
- Overall score card (52px font, gradient text, A+ grade)
- Subject-wise analytics list with score + grade
- Download Report Card button → modal → success
- Share modal with social options

### 13.6 Exams Screen
- Countdown timer (Days/Hours/Mins) to next exam
- AI Exam Prep Tip card (dark green gradient)
- Exam schedule list (date badges, subject, hall)
- Online Exam Portal card → Take Exam / Upload Answer Sheet
- Exam Instructions → Online Exam → Exam Success flow

### 13.7 Online Exam Screen
- Full-screen locked mode (hides bottom nav, AI FAB)
- Top bar: exam title + live timer (red when <5 min)
- Section switcher: Objective / Subjective
- Question palette grid (8 columns) with color states:
  - Blue = current, Green = answered, Orange = review, Gray = unanswered
- MCQ options as selectable cards (border changes on select)
- Subjective: textarea + file attachment button
- Bottom bar: Exit / Flag Review / Prev / Next / Submit
- Exit confirmation modal
- Submit confirmation modal

### 13.8 Transport Screen
- LIVE badge with blinking dot
- flutter_map with OSM tiles showing bus route + animated bus marker
- Bus info card: ETA, students onboard, stops remaining
- Route stops timeline (school → stops → student stop highlighted)
- "Call Driver / Set Alert" button
- Bus Alert modal

### 13.9 AI Chat Screen
- Dark themed UI (0x020617 background)
- Chat message bubbles (user: indigo gradient, AI: white card)
- Typing indicator (3 bouncing dots)
- Suggestion chips: Show timetable / Pending homework / Where's bus?
- Input field + send button
- All 20 tools wired to AI responses

### 13.10 Live Classes Screen
- "LIVE NOW" section with thumbnail cards + viewer count
- Join Now button (navigates to detail screen)
- Upcoming Today section with times
- Recorded Classes section with Play button
- Live Class Detail: embedded video player, info row, like/share/save, comments section

---

## 14. UI ANIMATION REQUIREMENTS

Every interactive element must have:
- `AnimatedContainer` / `AnimatedOpacity` transitions (300ms)
- Page transitions: slide from right for forward, slide to right for back
- FAB pulse animation (CSS keyframe equivalent using AnimationController)
- Scheduled period "NOW" badge with blinking dot
- Bus marker animated movement using Tween
- Confetti on exam success screen
- Shimmer loading skeleton on all data-loading states
- Badge pop animation on achievements unlock

---

## 15. SECURITY & AUTH

```dart
// Supabase Auth supports:
// - Email/Password login
// - OTP via phone/email
// - Biometric (local_auth package — device-level)
// - QR code scan (mobile_scanner → validate token)
// Row Level Security (RLS) on ALL tables — students only see their own data
// JWT tokens managed by Supabase Flutter SDK
```

---

## 16. ENVIRONMENT CONFIGURATION

```
# .env file (flutter_dotenv)
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_ANON_KEY=xxxx
OPENAI_API_KEY=xxxx
OSM_ATTRIBUTION=OpenStreetMap contributors
```

---

## 17. KEY IMPLEMENTATION NOTES

1. **Never break layout** — always use `LayoutBuilder` + `MediaQuery` + `SafeArea` + `SingleChildScrollView` where needed
2. **All cards** use `BorderRadius.circular(16-20)` matching HTML's 16-20px border-radius
3. **All gradients** must exactly match the HTML color stops
4. **Bottom nav** switches to `NavigationRail` on tablet, `NavigationDrawer` on desktop
5. **Supabase Realtime** used for: bus location, live class comments, notifications, messages
6. **All modals** animate up as bottom sheets on mobile, centered dialogs on desktop
7. **Exam screen** must lock device rotation to portrait and disable screenshot (platform channels)
8. **PDF report card** generated client-side using `pdf` package then uploaded to Supabase Storage
9. **AI Chat** stores full conversation in `ai_chat_history` table with session_id for context continuity
10. **OSM map** — no API key needed; attribution must always be shown as per OSM license
