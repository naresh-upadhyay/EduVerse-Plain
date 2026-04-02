# EduSHAMIIT AI Academic App — Complete Flutter Development Guide

> **PURPOSE**: This document is a SELF-CONTAINED BLUEPRINT. Any AI model or developer reading ONLY this file should be able to build the exact EduSHAMIIT AI Academic App from scratch — a UNIFIED Flutter application for BOTH Students and Teachers, matching the provided HTML mockups pixel-perfectly across all 6 platforms (Web, Android, iOS, macOS, Linux, Windows).

> **PROJECT NAME**: EduSHAMIIT AI
> **PACKAGE NAME**: com.shamiit.edu
> **AI AGENT NAME**: Shami
> **BACKEND**: LOCAL Supabase (running on developer's machine) + Python FastAPI AI Backend
> **ARCHITECTURE**: Single app with role-based routing (Student vs Teacher)

---

## 📋 TABLE OF CONTENTS

1. [Project Overview and Architecture](#1-project-overview-and-architecture)
2. [Step-by-Step Setup Guide (LOCAL SUPABASE)](#2-step-by-step-setup-guide)
3. [Design System — Dual Theme](#3-design-system--dual-theme)
4. [Complete Database Schema (SQL)](#4-complete-database-schema-sql)
5. [Python AI Backend Integration](#5-python-ai-backend-integration)
6. [Complete Folder Structure](#6-complete-folder-structure)
7. [Core Dart Files (Templates)](#7-core-dart-files-templates)
8. [Student Screens — Complete Specifications (29 Screens)](#8-student-screens--complete-specifications)
9. [Teacher Screens — Complete Specifications (29 Screens)](#9-teacher-screens--complete-specifications)
10. [Shared Reusable Components](#10-shared-reusable-components)
11. [Responsive Design Strategy](#11-responsive-design-strategy)
12. [Navigation and Routing](#12-navigation-and-routing)
13. [Dependencies](#13-dependencies)
14. [Development Phases](#14-development-phases)
15. [Critical Implementation Notes](#15-critical-implementation-notes)

---

## 1. PROJECT OVERVIEW AND ARCHITECTURE

### What Is EduSHAMIIT AI?

EduSHAMIIT AI is a unified AI-powered Academic App built with Flutter. It serves BOTH students and teachers through a single codebase. After login, the app detects the user's role and routes them to their respective portal. The shared screens (Splash, Login, Chat Detail, Settings) use a unified design with role-specific theming. The AI assistant agent is named **Shami** and provides intelligent assistance via a **Python FastAPI backend** with 40 tools total (20 for students, 20 for teachers).

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    EduSHAMIIT AI App                            │
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────────┐          │
│  │  Splash   │→ │   Login  │→ │  Role Detection      │          │
│  │ (Shared)  │  │ (Shared) │  │  (Student/Teacher)   │          │
│  └──────────┘  └──────────┘  └──────────┬───────────┘          │
│                                          │                      │
│                    ┌─────────────────────┴──────────────┐       │
│                    ▼                                    ▼       │
│           ┌──────────────────┐            ┌──────────────────┐  │
│           │  Student Portal  │            │  Teacher Portal  │  │
│           │  (29 screens)    │            │  (29 screens)    │  │
│           │                  │            │                  │  │
│           │ Dashboard        │            │ Dashboard        │  │
│           │ Timetable        │            │ My Classes       │  │
│           │ Results          │            │ Attendance       │  │
│           │ Exams (Online)   │            │ Homework Mgr     │  │
│           │ Fees & Payments  │            │ Paper Builder    │  │
│           │ Homework         │            │ Gradebook        │  │
│           │ Transport (OSM)  │            │ Exams            │  │
│           │ Shami AI Chat    │            │ Shami AI Chat    │  │
│           │ Achievements     │            │ Live Classes     │  │
│           │ Leaderboard      │            │ Student Directory│  │
│           │ Library          │            │ Salary           │  │
│           │ Live Classes     │            │ Notices          │  │
│           │ ... (29 total)   │            │ ... (29 total)   │  │
│           └──────────────────┘            └──────────────────┘  │
│                    │                                    │       │
│                    └─────────────┬──────────────────────┘       │
│                                  ▼                              │
│                    ┌──────────────────────────┐                 │
│                    │    Shared Services        │                 │
│                    │    Supabase (LOCAL)       │                 │
│                    │    Python FastAPI (Shami) │                 │
│                    │    OSM Maps               │                 │
│                    │    Realtime Subscriptions  │                 │
│                    │    File Storage            │                 │
│                    └──────────────────────────┘                 │
└─────────────────────────────────────────────────────────────────┘
```

### Screen Count Summary

| Portal      | Screen Count | Primary Color    | Secondary Color |
| ----------- | ------------ | ---------------- | --------------- |
| **Shared**  | 7 screens    | Varies by role   | Varies          |
| **Student** | 29 screens   | Purple (#4F46E5) | Cyan (#06B6D4)  |
| **Teacher** | 29 screens   | Cyan (#0EA5E9)   | Blue (#0369A1)  |
| **Total**   | ~65 screens  |                  |                 |

### Shared Screens (7)

1. Splash Screen (dual theme)
2. Login Screen (role selector)
3. AI Chat Screen (Shami - dual theme)
4. Settings Screen (dual theme)
5. Messaging Screen (dual theme)
6. Chat Detail Screen (dual theme)
7. Create Group Screen (dual theme)

### Student Screens (29)

1. Splash, 2. Login, 3. Dashboard, 4. Timetable, 5. Results, 6. Exams, 7. Exam Instructions, 8. Online Exam, 9. Exam Success, 10. Fees, 11. Notices, 12. Homework, 13. Transport, 14. Events, 15. Achievements, 16. Attendance, 17. Leave, 18. Profile, 19. AI Chat (Shami), 20. Library, 21. Courses, 22. Notifications, 23. Settings, 24. Messaging, 25. Chat Detail, 26. Create Group, 27. Live Classes, 28. Live Class Detail, 29. Leaderboard

### Teacher Screens (29)

1. Splash, 2. Login, 3. Dashboard, 4. My Classes, 5. Class Detail, 6. Attendance, 7. Timetable, 8. Homework Manager, 9. Review Submissions, 10. Grading Screen, 11. Exams, 12. Paper Builder, 13. Gradebook, 14. Grading Configuration, 15. Student Directory, 16. Leave, 17. Notices, 18. Live Classes Dashboard, 19. Teacher Live Session, 20. Live Playback, 21. Study Materials, 22. Salary, 23. Profile, 24. Settings, 25. AI Chat (Shami), 26. Notifications, 27. Exam Attendance, 28. Grade Student Response, 29. Messaging / Chat Detail

### Target Platforms

- Web (Chrome, Firefox, Safari, Edge)
- Android (phone and tablet)
- iOS (iPhone and iPad)
- macOS (desktop)
- Linux (desktop)
- Windows (desktop)

---

## 2. STEP-BY-STEP SETUP GUIDE

### Step 1: Install LOCAL Supabase

```bash
# Install Docker Desktop if not already installed
# Then run Supabase locally using Docker:

# Install Supabase CLI
npm install -g supabase

# Initialize Supabase in your project directory
cd your-project-directory
supabase init

# Start local Supabase (uses Docker)
supabase start

# This will output:
# API URL: http://127.0.0.1:54321
# GraphQL URL: http://127.0.0.1:54321/graphql/v1
# DB URL: postgresql://postgres:postgres@127.0.0.1:54322/postgres
# Studio URL: http://127.0.0.1:54323
# Inbucket URL: http://127.0.0.1:54324
# JWT secret: super-secret-jwt-token-with-at-least-32-characters-long
# anon key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
# service_role key: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Step 2: Create Flutter Project

```bash
flutter create --org com.shamiit edu_shamiit_ai
cd edu_shamiit_ai
```

### Step 3: Run SQL Migration

```bash
# Create migration file
supabase migration new initial_schema

# Copy the SQL from Section 4 into:
# supabase/migrations/[timestamp]_initial_schema.sql

# Apply migration
supabase db reset
```

### Step 4: Add Dependencies

Replace pubspec.yaml dependencies section with Section 13.

### Step 5: Create Folder Structure

Run the complete folder creation commands from Section 6.

### Step 6: Add Fonts

Download from Google Fonts and place in assets/fonts/:
- Outfit-Light.ttf (weight 300)
- Outfit-Regular.ttf (weight 400)
- Outfit-Medium.ttf (weight 500)
- Outfit-SemiBold.ttf (weight 600)
- Outfit-Bold.ttf (weight 700)
- Outfit-ExtraBold.ttf (weight 800)
- Outfit-Black.ttf (weight 900)
- DMSans-Light.ttf (weight 300)
- DMSans-Regular.ttf (weight 400)
- DMSans-Medium.ttf (weight 500)
- DMSans-SemiBold.ttf (weight 600)
- DMSans-Bold.ttf (weight 700)

### Step 7: Configure LOCAL Supabase

```dart
// lib/core/services/supabase_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';

class SupabaseService {
  static const String supabaseUrl = 'http://127.0.0.1:54321';
  static const String supabaseAnonKey = 'YOUR_ANON_KEY_FROM_SUPABASE_START';

  static Future<void> initialize() async {
    await Supabase.initialize(
      url: supabaseUrl,
      anonKey: supabaseAnonKey,
      debug: true,
    );
  }

  static SupabaseClient get client => Supabase.instance.client;
  static GoTrueClient get auth => Supabase.instance.client.auth;
}
```

### Step 8: Run and Verify

```bash
# Terminal 1: Keep Supabase running
supabase start

# Terminal 2: Run Flutter
flutter pub get
flutter run -d chrome    # For web
flutter run              # For mobile/emulator
```

### Step 9: Supabase Studio

Open http://127.0.0.1:54323 in your browser to access Supabase Studio for viewing tables, data, running SQL queries, managing authentication, and testing realtime subscriptions.

---

## 3. DESIGN SYSTEM — DUAL THEME

### 3.1 Student Theme Colors

```dart
// lib/core/constants/student_colors.dart
import 'package:flutter/material.dart';

class StudentColors {
  // Primary
  static const Color primary = Color(0xFF4F46E5);
  static const Color primaryDeep = Color(0xFF3730A3);
  static const Color primaryLight = Color(0xFFEEF2FF);

  // Accents
  static const Color accent = Color(0xFF06B6D4);
  static const Color accent2 = Color(0xFFF59E0B);
  static const Color accent3 = Color(0xFF10B981);
  static const Color accent4 = Color(0xFFEF4444);
  static const Color accent5 = Color(0xFF8B5CF6);

  // Backgrounds
  static const Color background = Color(0xFFF0F2FF);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color card = Color(0xFFFAFAFA);
  static const Color scaffoldBackground = Color(0xFFF5F6FF);

  // Text
  static const Color text = Color(0xFF0F172A);
  static const Color text2 = Color(0xFF475569);
  static const Color text3 = Color(0xFF94A3B8);

  // Borders
  static const Color border = Color(0xFFE2E8F0);
  static const Color divider = Color(0xFFF1F5F9);

  // Status
  static const Color success = Color(0xFF059669);
  static const Color successBg = Color(0xFFECFDF5);
  static const Color warning = Color(0xFFD97706);
  static const Color warningBg = Color(0xFFFEF3C7);
  static const Color error = Color(0xFFEF4444);
  static const Color errorBg = Color(0xFFFEF2F2);
  static const Color info = Color(0xFF3B82F6);
  static const Color infoBg = Color(0xFFEFF6FF);

  // Screen-specific gradients (from HTML mockup)
  static const Color dashboardGradient1 = Color(0xFF4F46E5);
  static const Color dashboardGradient2 = Color(0xFF302B63);
  static const Color timetableGradient1 = Color(0xFF1E40AF);
  static const Color timetableGradient2 = Color(0xFF1D4ED8);
  static const Color resultsGradient1 = Color(0xFF4F46E5);
  static const Color resultsGradient2 = Color(0xFF6366F1);
  static const Color examsGradient1 = Color(0xFF134E4A);
  static const Color examsGradient2 = Color(0xFF0F766E);
  static const Color feesGradient1 = Color(0xFF065F46);
  static const Color feesGradient2 = Color(0xFF059669);
  static const Color noticesGradient1 = Color(0xFF92400E);
  static const Color noticesGradient2 = Color(0xFFD97706);
  static const Color homeworkGradient1 = Color(0xFFBE185D);
  static const Color homeworkGradient2 = Color(0xFFDB2777);
  static const Color transportGradient1 = Color(0xFF0C4A6E);
  static const Color transportGradient2 = Color(0xFF0369A1);
  static const Color eventsGradient1 = Color(0xFF831843);
  static const Color eventsGradient2 = Color(0xFFBE185D);
  static const Color achievementsGradient1 = Color(0xFF78350F);
  static const Color achievementsGradient2 = Color(0xFFB45309);
  static const Color attendanceGradient1 = Color(0xFF1D4ED8);
  static const Color attendanceGradient2 = Color(0xFF3B82F6);
  static const Color profileGradient1 = Color(0xFF0F172A);
  static const Color profileGradient2 = Color(0xFF1E293B);
  static const Color chatGradient1 = Color(0xFF0F0C29);
  static const Color chatGradient2 = Color(0xFF302B63);
  static const Color libraryGradient1 = Color(0xFF3B0764);
  static const Color libraryGradient2 = Color(0xFF6D28D9);
  static const Color leaderboardGradient1 = Color(0xFF4F46E5);
  static const Color leaderboardGradient2 = Color(0xFF7C3AED);
  static const Color liveClassGradient1 = Color(0xFFEF4444);
  static const Color liveClassGradient2 = Color(0xFFDC2626);
  static const Color settingsGradient1 = Color(0xFF1E293B);
  static const Color settingsGradient2 = Color(0xFF0F172A);
  static const Color messagingGradient1 = Color(0xFF4F46E5);
  static const Color messagingGradient2 = Color(0xFF7C3AED);
  static const Color leaveGradient1 = Color(0xFF4F46E5);
  static const Color leaveGradient2 = Color(0xFF06B6D4);
  static const Color coursesGradient1 = Color(0xFF4F46E5);
  static const Color coursesGradient2 = Color(0xFF302B63);
  static const Color splashGradient1 = Color(0xFF0F0C29);
  static const Color splashGradient2 = Color(0xFF302B63);
  static const Color splashGradient3 = Color(0xFF24243E);

  // Subject colors
  static const Color mathColor = Color(0xFF4F46E5);
  static const Color physicsColor = Color(0xFF3B82F6);
  static const Color chemistryColor = Color(0xFFF59E0B);
  static const Color englishColor = Color(0xFF10B981);
  static const Color historyColor = Color(0xFF8B5CF6);
  static const Color computerColor = Color(0xFF06B6D4);

  // Quick access grid colors (from HTML)
  static const Color qaTimetable = Color(0xFFEEF2FF);
  static const Color qaResults = Color(0xFFFDF4FF);
  static const Color qaFees = Color(0xFFECFDF5);
  static const Color qaNotices = Color(0xFFF0FDF4);
  static const Color qaHomework = Color(0xFFFDF2F8);
  static const Color qaTransport = Color(0xFFEFF6FF);
  static const Color qaEvents = Color(0xFFFEF3C7);
  static const Color qaAchievements = Color(0xFFF0FDF4);
  static const Color qaAttendance = Color(0xFFEFF6FF);
  static const Color qaLibrary = Color(0xFFFAF5FF);
  static const Color qaCourses = Color(0xFFECFDF5);
  static const Color qaLeave = Color(0xFFFEF2F2);
  static const Color qaExams = Color(0xFFEEF2FF);
  static const Color qaLiveClass = Color(0xFFFFE4E6);
  static const Color qaMessages = Color(0xFFE0E7FF);
  static const Color qaCertificates = Color(0xFFFEF3C7);
}
```

### 3.2 Teacher Theme Colors

```dart
// lib/core/constants/teacher_colors.dart
import 'package:flutter/material.dart';

class TeacherColors {
  static const Color primary = Color(0xFF0EA5E9);
  static const Color primaryDeep = Color(0xFF0369A1);
  static const Color primaryLight = Color(0xFFE0F2FE);
  static const Color accent = Color(0xFF06B6D4);
  static const Color background = Color(0xFFF0F7FF);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color text = Color(0xFF0F172A);
  static const Color text2 = Color(0xFF475569);
  static const Color border = Color(0xFFE2E8F0);
  // Add remaining colors following student_colors pattern
}
```

### 3.3 Typography

```dart
// lib/core/constants/app_fonts.dart
class AppFonts {
  static const String heading = 'Outfit';
  static const String body = 'DM Sans';
}
```

### 3.4 Shared Gradients

```dart
// lib/core/constants/app_gradients.dart
import 'package:flutter/material.dart';

class AppGradients {
  static const LinearGradient studentPrimary = LinearGradient(
    colors: [Color(0xFF4F46E5), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  static const LinearGradient teacherPrimary = LinearGradient(
    colors: [Color(0xFF0EA5E9), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
}
```

---

## 4. COMPLETE DATABASE SCHEMA (SQL)

Run these SQL scripts in Supabase SQL Editor (http://127.0.0.1:54323):

### 4.1 Enable Extensions

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
```

### 4.2 Schools Table (Multi-School Support)

```sql
CREATE TABLE schools (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  address TEXT,
  phone TEXT,
  logo_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.3 Core Tables (All with school_id)

```sql
-- PROFILES
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  school_id UUID REFERENCES schools(id),
  user_id TEXT NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('student', 'parent', 'teacher')),
  class TEXT,
  department TEXT,
  designation TEXT,
  roll_number INT,
  employee_id TEXT,
  gender TEXT CHECK (gender IN ('Male', 'Female', 'Other')),
  date_of_birth DATE,
  blood_group TEXT,
  email TEXT,
  phone TEXT,
  avatar_url TEXT,
  xp_points INT DEFAULT 0,
  learning_streak INT DEFAULT 0,
  best_streak INT DEFAULT 0,
  last_login DATE DEFAULT CURRENT_DATE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- SUBJECTS
CREATE TABLE subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  name TEXT NOT NULL,
  icon TEXT,
  color TEXT,
  teacher_id UUID REFERENCES profiles(id),
  total_chapters INT DEFAULT 0,
  class TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- TIMETABLE
CREATE TABLE timetable (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  day_of_week INT NOT NULL CHECK (day_of_week BETWEEN 0 AND 5),
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  room TEXT,
  class TEXT NOT NULL,
  is_break BOOLEAN DEFAULT FALSE,
  break_name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- COURSES
CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  syllabus_coverage DECIMAL(5,2) DEFAULT 0,
  class TEXT NOT NULL,
  academic_year TEXT,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RESULTS
CREATE TABLE results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  exam_category TEXT NOT NULL CHECK (exam_category IN ('Class Test','Lab Test','Assignment','Mid-Term','End-Term','Other')),
  academic_year TEXT NOT NULL,
  marks_obtained DECIMAL(5,2),
  max_marks DECIMAL(5,2),
  grade TEXT,
  remarks TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAMS
CREATE TABLE exams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  exam_type TEXT NOT NULL CHECK (exam_type IN ('offline','online','hybrid')),
  exam_date DATE,
  start_time TIME,
  duration_minutes INT DEFAULT 90,
  total_marks INT DEFAULT 100,
  venue TEXT,
  status TEXT DEFAULT 'upcoming' CHECK (status IN ('draft','upcoming','ongoing','completed','graded')),
  target_classes JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM QUESTIONS
CREATE TABLE exam_questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  question_number INT NOT NULL,
  section TEXT NOT NULL CHECK (section IN ('objective','subjective')),
  question_text TEXT NOT NULL,
  question_type TEXT NOT NULL CHECK (question_type IN ('mcq','true_false','short_answer','long_answer')),
  options JSONB,
  correct_answer INT,
  positive_marks INT DEFAULT 4,
  negative_marks INT DEFAULT 1,
  max_marks INT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM SUBMISSIONS
CREATE TABLE exam_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  question_id UUID REFERENCES exam_questions(id) ON DELETE CASCADE,
  selected_answer INT,
  subjective_answer TEXT,
  is_correct BOOLEAN,
  marks_awarded DECIMAL(5,2),
  is_reviewed BOOLEAN DEFAULT FALSE,
  submitted_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM SESSIONS
CREATE TABLE exam_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  started_at TIMESTAMPTZ,
  submitted_at TIMESTAMPTZ,
  total_marks DECIMAL(5,2) DEFAULT 0,
  xp_earned INT DEFAULT 0,
  status TEXT DEFAULT 'in_progress' CHECK (status IN ('in_progress','submitted','auto_submitted'))
);

-- ATTENDANCE
CREATE TABLE attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  date DATE NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('present','absent','late','excused')),
  marked_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(student_id, subject_id, date)
);

-- FEES
CREATE TABLE fees (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  fee_type TEXT NOT NULL,
  amount DECIMAL(10,2) NOT NULL,
  due_date DATE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','partial','paid','overdue')),
  amount_paid DECIMAL(10,2) DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- PAYMENTS
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  fee_id UUID REFERENCES fees(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  amount DECIMAL(10,2) NOT NULL,
  payment_method TEXT CHECK (payment_method IN ('upi','card','netbanking')),
  transaction_id TEXT UNIQUE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','success','failed','refunded')),
  paid_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- SALARY
CREATE TABLE salary (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  month TEXT NOT NULL,
  gross_salary DECIMAL(10,2),
  net_salary DECIMAL(10,2),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','credited')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- HOMEWORK
CREATE TABLE homework (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  due_date TIMESTAMPTZ NOT NULL,
  max_marks INT,
  target_class TEXT NOT NULL,
  status TEXT DEFAULT 'active' CHECK (status IN ('active','closed')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- HOMEWORK SUBMISSIONS
CREATE TABLE homework_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  homework_id UUID REFERENCES homework(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  submission_text TEXT,
  attachment_url TEXT,
  status TEXT DEFAULT 'submitted' CHECK (status IN ('submitted','graded','late')),
  grade TEXT,
  marks DECIMAL(5,2),
  teacher_remarks TEXT,
  submitted_at TIMESTAMPTZ DEFAULT NOW()
);

-- NOTICES
CREATE TABLE notices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN ('Urgent','General','Event','Academic')),
  author_id UUID REFERENCES profiles(id),
  is_pinned BOOLEAN DEFAULT FALSE,
  is_urgent BOOLEAN DEFAULT FALSE,
  status TEXT DEFAULT 'published',
  published_at TIMESTAMPTZ DEFAULT NOW()
);

-- EVENTS
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  description TEXT,
  event_date DATE,
  event_time TIME,
  venue TEXT,
  max_participants INT,
  current_participants INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- MESSAGES
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  sender_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES profiles(id),
  group_id UUID,
  content TEXT NOT NULL,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- NOTIFICATIONS
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  type TEXT NOT NULL,
  is_read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- LEAVE APPLICATIONS
CREATE TABLE leave_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  applicant_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  applicant_role TEXT CHECK (applicant_role IN ('student','teacher')),
  leave_type TEXT NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  reason TEXT NOT NULL,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ACHIEVEMENTS
CREATE TABLE achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  xp_reward INT DEFAULT 0,
  rarity TEXT DEFAULT 'common',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDENT ACHIEVEMENTS
CREATE TABLE student_achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  achievement_id UUID REFERENCES achievements(id) ON DELETE CASCADE,
  earned_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(student_id, achievement_id)
);

-- LIBRARY
CREATE TABLE library_books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  title TEXT NOT NULL,
  author TEXT,
  isbn TEXT,
  total_copies INT DEFAULT 1,
  available_copies INT DEFAULT 1,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- BUS ROUTES
CREATE TABLE bus_routes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_name TEXT NOT NULL,
  bus_number TEXT,
  driver_name TEXT,
  driver_phone TEXT,
  status TEXT DEFAULT 'active'
);

-- BUS STOPS
CREATE TABLE bus_stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  stop_name TEXT NOT NULL,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  stop_order INT NOT NULL
);

-- BUS LOCATIONS
CREATE TABLE bus_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  speed DECIMAL(5,2),
  eta_minutes INT,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIVE CLASSES
CREATE TABLE live_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  scheduled_at TIMESTAMPTZ NOT NULL,
  duration_minutes INT DEFAULT 60,
  is_live BOOLEAN DEFAULT FALSE,
  viewer_count INT DEFAULT 0,
  status TEXT DEFAULT 'scheduled',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDY MATERIALS
CREATE TABLE study_materials (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  material_type TEXT CHECK (material_type IN ('Notes','PPTs','Videos','Worksheets')),
  target_class TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- USER SETTINGS
CREATE TABLE user_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE UNIQUE,
  push_notifications BOOLEAN DEFAULT TRUE,
  dark_mode BOOLEAN DEFAULT FALSE,
  language TEXT DEFAULT 'en',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- GRADING POLICIES
CREATE TABLE grading_policies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  teacher_id UUID REFERENCES profiles(id),
  class TEXT NOT NULL,
  subject_id UUID REFERENCES subjects(id),
  mid_term_weight INT DEFAULT 30,
  final_term_weight INT DEFAULT 40,
  attendance_weight INT DEFAULT 5,
  assignment_weight INT DEFAULT 10,
  class_test_weight INT DEFAULT 10,
  lab_weight INT DEFAULT 5,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI CHAT HISTORY
CREATE TABLE ai_chat_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  school_id UUID REFERENCES schools(id),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  session_id UUID,
  role TEXT CHECK (role IN ('user','assistant')),
  content TEXT NOT NULL,
  tool_calls JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.4 RLS Policies

```sql
-- Enable RLS on all tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE results ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance ENABLE ROW LEVEL SECURITY;
ALTER TABLE fees ENABLE ROW LEVEL SECURITY;
ALTER TABLE homework_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE leave_applications ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE ai_chat_history ENABLE ROW LEVEL SECURITY;

-- Profile policies
CREATE POLICY "Users can view own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile" ON profiles FOR UPDATE USING (auth.uid() = id);

-- Student data policies
CREATE POLICY "Students can view own results" ON results FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Students can view own attendance" ON attendance FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Students can view own fees" ON fees FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Students can submit homework" ON homework_submissions FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Students can view own submissions" ON homework_submissions FOR SELECT USING (auth.uid() = student_id);

-- Teacher policies
CREATE POLICY "Teachers can view all results" ON results FOR SELECT USING (true);
CREATE POLICY "Teachers can insert results" ON results FOR INSERT WITH CHECK (true);
CREATE POLICY "Teachers can view all attendance" ON attendance FOR SELECT USING (true);
CREATE POLICY "Teachers can insert attendance" ON attendance FOR INSERT WITH CHECK (true);
CREATE POLICY "Teachers can view all submissions" ON homework_submissions FOR SELECT USING (true);
CREATE POLICY "Teachers can grade submissions" ON homework_submissions FOR UPDATE USING (true);
CREATE POLICY "Teachers can view exam submissions" ON exam_submissions FOR SELECT USING (true);
CREATE POLICY "Teachers can grade exams" ON exam_submissions FOR UPDATE USING (true);

-- Leave policies
CREATE POLICY "Users can apply leave" ON leave_applications FOR INSERT WITH CHECK (auth.uid() = applicant_id);
CREATE POLICY "Users can view own leaves" ON leave_applications FOR SELECT USING (auth.uid() = applicant_id);

-- Message policies
CREATE POLICY "Users can send messages" ON messages FOR INSERT WITH CHECK (auth.uid() = sender_id);
CREATE POLICY "Users can view own messages" ON messages FOR SELECT USING (auth.uid() = sender_id OR auth.uid() = receiver_id);

-- Notification policies
CREATE POLICY "Users can view own notifications" ON notifications FOR SELECT USING (auth.uid() = user_id);

-- AI chat history
CREATE POLICY "Users can view own AI chats" ON ai_chat_history FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users can insert AI chats" ON ai_chat_history FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Public read policies
CREATE POLICY "Anyone can read subjects" ON subjects FOR SELECT USING (true);
CREATE POLICY "Anyone can read timetable" ON timetable FOR SELECT USING (true);
CREATE POLICY "Anyone can read notices" ON notices FOR SELECT USING (true);
CREATE POLICY "Anyone can read events" ON events FOR SELECT USING (true);
CREATE POLICY "Anyone can read homework" ON homework FOR SELECT USING (true);
CREATE POLICY "Anyone can read exams" ON exams FOR SELECT USING (true);
CREATE POLICY "Anyone can read achievements" ON achievements FOR SELECT USING (true);
CREATE POLICY "Anyone can read live classes" ON live_classes FOR SELECT USING (true);
CREATE POLICY "Teachers can manage materials" ON study_materials FOR ALL USING (auth.uid() = teacher_id);
```

### 4.5 Enable Realtime

```sql
ALTER PUBLICATION supabase_realtime ADD TABLE bus_locations;
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE live_classes;
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;
ALTER PUBLICATION supabase_realtime ADD TABLE exam_sessions;
ALTER PUBLICATION supabase_realtime ADD TABLE attendance;
```

### 4.6 Seed Data

```sql
-- Sample school
INSERT INTO schools (name, address, phone) VALUES
  ('SHAMIIT Academy', '123 Education Street, Delhi', '+91-11-12345678');

-- Sample subjects
INSERT INTO subjects (name, icon, color, total_chapters, class) VALUES
  ('Mathematics', '📐', '#4F46E5', 42, 'X-A'),
  ('Physics', '⚛️', '#3B82F6', 38, 'X-A'),
  ('Chemistry', '⚗️', '#F59E0B', 35, 'X-A'),
  ('English', '📖', '#10B981', 28, 'X-A');

-- Sample achievements
INSERT INTO achievements (name, description, icon, xp_reward, rarity) VALUES
  ('Academic Excellence', 'Scored 90%+ in term exams', '🏆', 500, 'rare'),
  ('18-Day Streak', 'Logged in 18 consecutive days', '🔥', 300, 'uncommon'),
  ('Zero Late Submissions', 'All homework on time this term', '✅', 200, 'common');
```

---

## 5. PYTHON AI BACKEND INTEGRATION

### 5.1 Overview

The EduSHAMIIT AI Flutter app connects to a **Python FastAPI backend** for AI-powered assistance. The AI agent is named **Shami** and provides intelligent assistance to both students and teachers. The Flutter app sends HTTP requests to the Python backend, which processes them using LangChain Python with 40 tools total (20 for students, 20 for teachers).

**For the complete Python AI backend specification, see: `edushamiit_ai.md`**

### 5.2 Flutter → Python Backend Architecture

```
Flutter App (Dart)          Python Backend (FastAPI)
┌─────────────────┐         ┌─────────────────────────┐
│  AI Chat Screen │  HTTP   │  /chat/message (SSE)    │
│  AI Service     │ ───────→│  /chat/voice (Whisper)  │
│  Riverpod State │  SSE    │  /chat/image (Gemini)   │
│  Chat Messages  │ ←────── │  /iot/control (MQTT)    │
│  Tool Cards     │         │  /rag/ingest (pgvector)  │
└─────────────────┘         └─────────────────────────┘
```

### 5.3 Flutter AI Service (HTTP Client)

```dart
// lib/core/services/ai_service.dart
import 'dart:convert';
import 'package:http/http.dart' as http;

class AiService {
  static const String _baseUrl = 'http://127.0.0.1:8000';

  Stream<AiChunk> streamMessage({
    required String message,
    required String sessionId,
    required String token,
  }) async* {
    final request = http.Request('POST', Uri.parse('$_baseUrl/chat/message'))
      ..headers = {
        'Authorization': 'Bearer $token',
        'Content-Type': 'application/json',
      }
      ..body = jsonEncode({
        'message': message,
        'session_id': sessionId,
      });

    final response = await http.Client().send(request);
    await for (final line in response.stream
        .transform(utf8.decoder)
        .transform(const LineSplitter())) {
      if (line.startsWith('data: ')) {
        final data = jsonDecode(line.substring(6));
        yield AiChunk.fromJson(data);
      }
    }
  }

  Future<AiVoiceResponse> sendVoice(String filePath, String token) async {
    final request = http.MultipartRequest(
      'POST', Uri.parse('$_baseUrl/chat/voice'),
    )
      ..headers['Authorization'] = 'Bearer $token'
      ..files.add(await http.MultipartFile.fromPath('audio', filePath));
    final streamedResponse = await request.send();
    final body = await streamedResponse.stream.bytesToString();
    return AiVoiceResponse.fromJson(jsonDecode(body));
  }

  Future<String> sendImage({
    required String filePath,
    required String question,
    required String token,
  }) async {
    final request = http.MultipartRequest(
      'POST', Uri.parse('$_baseUrl/chat/image'),
    )
      ..headers['Authorization'] = 'Bearer $token'
      ..fields['question'] = question
      ..files.add(await http.MultipartFile.fromPath('image', filePath));
    final streamedResponse = await request.send();
    final body = await streamedResponse.stream.bytesToString();
    return jsonDecode(body)['output'];
  }

  Future<List<ChatMessage>> loadHistory({
    required String sessionId,
    required String token,
  }) async {
    final response = await http.get(
      Uri.parse('$_baseUrl/chat/history/$sessionId'),
      headers: {'Authorization': 'Bearer $token'},
    );
    final data = jsonDecode(response.body);
    return (data['messages'] as List)
        .map((m) => ChatMessage.fromJson(m))
        .toList();
  }
}

class AiChunk {
  final String type;
  final String content;
  final Map<String, dynamic>? toolData;

  AiChunk({required this.type, required this.content, this.toolData});

  factory AiChunk.fromJson(Map<String, dynamic> json) {
    return AiChunk(
      type: json['type'],
      content: json['content'] ?? '',
      toolData: json['tool_result'] != null
          ? Map<String, dynamic>.from(json['tool_result'])
          : null,
    );
  }
}
```

### 5.4 Student AI Tools Reference (Python Backend)

All 20 student tools are implemented in the Python backend. See `edushamiit_ai.md` Section 7 for complete Python implementations.

| Tool Name | Flutter Call Example | Python Endpoint |
|---|---|---|
| get_timetable | `sendText("What classes do I have today?")` | POST /chat/message |
| get_homework | `sendText("What homework is due?")` | POST /chat/message |
| get_fee_status | `sendText("How much fees do I owe?")` | POST /chat/message |
| get_attendance | `sendText("What is my attendance?")` | POST /chat/message |
| get_exam_info | `sendText("When is my next exam?")` | POST /chat/message |
| get_bus_location | `sendText("Where is my bus?")` | POST /chat/message |
| get_performance | `sendText("How am I doing?")` | POST /chat/message |
| generate_study_plan | `sendText("Help me prepare for Math exam")` | POST /chat/message |
| explain_concept | `sendText("Explain integration by parts")` | POST /chat/message |
| get_notifications | `sendText("Any new notifications?")` | POST /chat/message |
| get_library_status | `sendText("What books do I have?")` | POST /chat/message |
| get_achievements | `sendText("What badges have I earned?")` | POST /chat/message |
| get_events | `sendText("What events are coming up?")` | POST /chat/message |
| get_leaderboard | `sendText("What is my class rank?")` | POST /chat/message |
| submit_homework | `sendText("Submit my Math homework")` | POST /chat/message |
| apply_leave | `sendText("I need sick leave tomorrow")` | POST /chat/message |
| get_notices | `sendText("Any new announcements?")` | POST /chat/message |
| answer_general | `sendText("What is the school timing?")` | POST /chat/message |
| generate_practice | `sendText("Give me Physics practice problems")` | POST /chat/message |
| get_live_class | `sendText("Is there a live class now?")` | POST /chat/message |

### 5.5 Teacher AI Tools Reference (Python Backend)

All 20 teacher tools are implemented in the Python backend. See `edushamiit_ai.md` Section 8 for complete Python implementations.

| Tool Name | Flutter Call Example | Python Endpoint |
|---|---|---|
| get_class_students | `sendText("Show me X-A students")` | POST /chat/message |
| get_class_performance | `sendText("How is X-A performing?")` | POST /chat/message |
| get_at_risk_students | `sendText("Who needs extra help?")` | POST /chat/message |
| generate_questions | `sendText("Create 5 MCQs on Calculus")` | POST /chat/message |
| generate_lesson_plan | `sendText("Make lesson plan for Integration")` | POST /chat/message |
| auto_grade_homework | `sendText("Grade the pending homework")` | POST /chat/message |
| get_submission_status | `sendText("How many students submitted?")` | POST /chat/message |
| generate_remedial_plan | `sendText("Make a plan for weak students")` | POST /chat/message |
| create_notice | `sendText("Draft notice about exam dates")` | POST /chat/message |
| get_attendance_stats | `sendText("Show attendance trends for X-A")` | POST /chat/message |
| get_teacher_schedule | `sendText("What is my schedule today?")` | POST /chat/message |
| get_pending_tasks | `sendText("What needs my attention?")` | POST /chat/message |
| generate_report | `sendText("Create X-A performance report")` | POST /chat/message |
| explain_pedagogy | `sendText("How to teach fractions?")` | POST /chat/message |
| get_leave_balance | `sendText("How many leaves remaining?")` | POST /chat/message |
| get_salary_info | `sendText("Show my salary breakup")` | POST /chat/message |
| get_parent_communications | `sendText("Any parent messages?")` | POST /chat/message |
| upload_material | `sendText("Upload notes for X-A")` | POST /chat/message |
| get_exam_analytics | `sendText("Show exam pass rates for X-A")` | POST /chat/message |
| answer_general | `sendText("What is the attendance policy?")` | POST /chat/message |

---

## 6. COMPLETE FOLDER STRUCTURE

```
lib/
├── main.dart
├── app.dart
├── app_router.dart
│
├── core/
│   ├── constants/
│   │   ├── student_colors.dart
│   │   ├── teacher_colors.dart
│   │   ├── app_fonts.dart
│   │   ├── app_dimensions.dart
│   │   ├── app_strings.dart
│   │   ├── app_gradients.dart
│   │   └── app_shadows.dart
│   ├── theme/
│   │   ├── student_theme.dart
│   │   └── teacher_theme.dart
│   ├── utils/
│   │   ├── date_formatter.dart
│   │   ├── currency_formatter.dart
│   │   ├── validators.dart
│   │   ├── responsive_utils.dart
│   │   └── role_utils.dart
│   ├── services/
│   │   ├── supabase_service.dart
│   │   ├── auth_service.dart
│   │   ├── storage_service.dart
│   │   ├── notification_service.dart
│   │   ├── realtime_service.dart
│   │   ├── ai_service.dart          # HTTP client for Python backend
│   │   └── maps/
│   │       ├── osm_service.dart
│   │       └── bus_tracking_service.dart
│   ├── providers/
│   │   ├── auth_provider.dart
│   │   ├── user_provider.dart
│   │   ├── role_provider.dart
│   │   ├── theme_provider.dart
│   │   └── ai_provider.dart
│   └── models/ (20 model files)
│
├── shared/
│   ├── widgets/ (20 widget files)
│   └── animations/ (3 animation files)
│
├── features/
│   ├── shared/ (splash, login, settings, messaging, ai_chat)
│   ├── student/ (14 feature modules, 29 screens)
│   └── teacher/ (17 feature modules, 29 screens)
│
└── l10n/
    ├── app_en.arb
    └── app_hi.arb
```

**Total: approximately 250+ Dart files**

---

## 7. CORE DART FILES (TEMPLATES)

### 7.1 main.dart

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:edu_shamiit_ai/app.dart';
import 'package:edu_shamiit_ai/core/services/supabase_service.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await SupabaseService.initialize();
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  runApp(const EduShamiitApp());
}
```

### 7.2 app.dart

```dart
import 'package:flutter/material.dart';
import 'package:edu_shamiit_ai/app_router.dart';

class EduShamiitApp extends StatelessWidget {
  const EduShamiitApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'EduSHAMIIT AI',
      debugShowCheckedModeBanner: false,
      theme: ThemeData.light(),
      routerConfig: goRouter,
    );
  }
}
```

### 7.3 app_router.dart

```dart
import 'package:go_router/go_router.dart';

final goRouter = GoRouter(
  initialLocation: '/splash',
  routes: [
    GoRoute(path: '/splash', builder: (_, __) => const SplashScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/settings', builder: (_, __) => const SettingsScreen()),
    GoRoute(path: '/ai-chat', builder: (_, __) => const AIChatScreen()),
    GoRoute(path: '/student/dashboard', builder: (_, __) => const StudentDashboard()),
    GoRoute(path: '/teacher/dashboard', builder: (_, __) => const TeacherDashboard()),
    // Add all other routes...
  ],
);
```

---

## 8-12. SCREEN SPECIFICATIONS, COMPONENTS, RESPONSIVE DESIGN, NAVIGATION

See HTML mockups for pixel-perfect specifications. Each screen includes route, background, header gradient, body sections, modals, bottom nav, AI FAB.

---

## 13. DEPENDENCIES

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  flutter_riverpod: ^2.4.9
  go_router: ^13.0.0
  supabase_flutter: ^2.3.2
  http: ^1.1.0
  dio: ^5.4.0
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  flutter_map: ^6.1.0
  latlong2: ^0.9.1
  flutter_animate: ^4.3.0
  lottie: ^3.0.0
  shimmer: ^3.0.0
  cached_network_image: ^3.3.1
  fl_chart: ^0.66.0
  flutter_markdown: ^0.6.18
  intl: ^0.19.0
  uuid: ^4.2.2
```

---

## 14. DEVELOPMENT PHASES

### Phase 1: Foundation (Weeks 1-2)
Project setup, LOCAL Supabase, fonts, dual theme, shared widgets, SQL migration, GoRouter

### Phase 2: Shared Screens (Weeks 3-4)
Splash, Login with role selector, role detection, Settings, AI Chat

### Phase 3: Student Core (Weeks 5-6)
Dashboard, Timetable, Results, Courses, Attendance

### Phase 4-10: Feature Implementation
Follow screen specifications from HTML mockups

### Phase 11: AI Integration (Weeks 21-22)
Python FastAPI backend setup, 40 tools implementation, Shami system prompts, chat history

### Phase 12: Polish (Weeks 23-26)
Responsive testing, performance, animations, error handling, offline support, QA

---

## 15. CRITICAL IMPLEMENTATION NOTES

1. LOCAL Supabase uses `http://127.0.0.1:54321` for development
2. Python AI backend uses `http://127.0.0.1:8000` for development
3. Role detection from `profiles.role` determines theme, nav, tools
4. Every shared component accepts a `role` parameter for color adaptation
5. Shami AI uses Python FastAPI backend with 40 tools across 2 agents
6. AI responses streamed via SSE from Python backend to Flutter
7. Paper Builder AI generates questions from subject/difficulty/marks input
8. Auto-grading handles MCQs automatically, subjective gets AI-suggested scores
9. Exam integrity: screen lock, focus tracking, auto-submit on timer expiry
10. Realtime for bus tracking, chat, live classes, notifications, exam timer
11. Offline caching with Hive for timetable, homework, profile
12. Supabase Storage buckets: homework-files, documents, exam-answers, chat-attachments, study-materials
13. Gamification: homework +50XP, login streak +10XP/day, 90% exam +500XP, events +100XP, streak bonuses at 7/14/21/30 days
14. Web: URL strategy, deep linking, HtmlElementView for video
15. Animations: transitions 400ms, modals 300ms, FAB pulse 3000ms, typing 1200ms, confetti 3000ms