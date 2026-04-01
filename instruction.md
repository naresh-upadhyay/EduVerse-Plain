# Mindora AI Academic App — Complete Flutter Development Guide

> **PURPOSE**: This document is a SELF-CONTAINED BLUEPRINT. Any AI model or developer reading ONLY this file should be able to build the exact Mindora AI Academic App from scratch — a UNIFIED Flutter application for BOTH Students and Teachers, matching the provided HTML mockups pixel-perfectly across all 6 platforms (Web, Android, iOS, macOS, Linux, Windows).

> **PROJECT NAME**: Mindora AI (renamed from EduVerse)
> **BACKEND**: LOCAL Supabase (running on developer's machine)
> **ARCHITECTURE**: Single app with role-based routing (Student vs Teacher)

---

## 📋 TABLE OF CONTENTS

1. [Project Overview & Architecture](#1-project-overview--architecture)
2. [Step-by-Step Setup Guide (LOCAL SUPABASE)](#2-step-by-step-setup-guide)
3. [Design System — Dual Theme](#3-design-system--dual-theme)
4. [Complete Database Schema (SQL)](#4-complete-database-schema-sql)
5. [AI Functions (LangChain Tool Calling)](#5-ai-functions-langchain-tool-calling)
6. [Complete Folder Structure](#6-complete-folder-structure)
7. [Core Dart Files (Templates)](#7-core-dart-files-templates)
8. [Student Screens (29 screens)](#8-student-screens)
9. [Teacher Screens (29 screens)](#9-teacher-screens)
10. [Shared Reusable Components](#10-shared-reusable-components)
11. [Responsive Design Strategy](#11-responsive-design-strategy)
12. [Navigation & Routing](#12-navigation--routing)
13. [Dependencies](#13-dependencies)
14. [Development Phases](#14-development-phases)
15. [Critical Implementation Notes](#15-critical-implementation-notes)

---

## 1. PROJECT OVERVIEW & ARCHITECTURE

### What Is Mindora AI?
Mindora AI is a unified AI-powered Academic App built with Flutter. It serves BOTH students and teachers through a single codebase. After login, the app detects the user's role and routes them to their respective portal. The shared screens (Splash, Login, Chat Detail, Settings) use a unified design with role-specific theming.

### Architecture Diagram
```
┌─────────────────────────────────────────────────────┐
│                  Mindora AI App                     │
│                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │  Splash   │→ │   Login  │→ │  Role Detection  │  │
│  │ (Shared)  │  │ (Shared) │  │                  │  │
│  └──────────┘  └──────────┘  └────────┬─────────┘  │
│                                        │            │
│                    ┌───────────────────┴────────┐   │
│                    ▼                            ▼   │
│           ┌────────────────┐        ┌────────────┐  │
│           │ Student Portal │        │Teacher Portal│ │
│           │   (29 screens) │        │ (29 screens) │ │
│           │                │        │              │ │
│           │ Dashboard      │        │ Dashboard    │ │
│           │ Timetable      │        │ My Classes   │ │
│           │ Results        │        │ Attendance   │ │
│           │ Exams          │        │ Homework Mgr │ │
│           │ Fees           │        │ Paper Builder│ │
│           │ Homework       │        │ Gradebook    │ │
│           │ Transport      │        │ Exams        │ │
│           │ AI Chat        │        │ AI Chat      │ │
│           │ ...            │        │ ...          │ │
│           └────────────────┘        └──────────────┘ │
│                    │                            │    │
│                    └───────────┬────────────────┘    │
│                                ▼                     │
│                    ┌─────────────────────┐           │
│                    │   Shared Services   │           │
│                    │   Supabase (LOCAL)  │           │
│                    │   LangChain AI      │           │
│                    │   OSM Maps          │           │
│                    │   Realtime          │           │
│                    └─────────────────────┘           │
└─────────────────────────────────────────────────────┘
```

### Screen Count Summary
| Portal | Screens | Primary Color |
|--------|---------|---------------|
| **Shared** | Splash, Login, Chat Detail, Settings, Create Group | Varies |
| **Student** | 29 screens | Purple (#4F46E5) |
| **Teacher** | 29 screens | Cyan (#0EA5E9) |
| **Total** | ~58 screens | |

### Target Platforms
- Web (Chrome, Firefox, Safari, Edge)
- Android (phone + tablet)
- iOS (iPhone + iPad)
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
flutter create --org com.mindora mindora_ai
cd mindora_ai
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
Replace `pubspec.yaml` dependencies section with Section 13.

### Step 5: Create Folder Structure
```bash
# Run from project root (Windows PowerShell or CMD)
mkdir lib\core\constants
mkdir lib\core\theme
mkdir lib\core\utils
mkdir lib\core\services\ai\tools
mkdir lib\core\services\ai\prompts
mkdir lib\core\services\ai\models
mkdir lib\core\services\maps
mkdir lib\core\providers
mkdir lib\core\models
mkdir lib\shared\widgets
mkdir lib\shared\animations
mkdir lib\features\shared\splash\screens
mkdir lib\features\shared\login\screens
mkdir lib\features\shared\login\widgets
mkdir lib\features\shared\settings\screens
mkdir lib\features\shared\messaging\screens
mkdir lib\features\shared\messaging\widgets
mkdir lib\features\shared\ai_chat\screens
mkdir lib\features\shared\ai_chat\widgets
mkdir lib\features\student\dashboard\screens
mkdir lib\features\student\dashboard\widgets
mkdir lib\features\student\academics\timetable\screens
mkdir lib\features\student\academics\timetable\widgets
mkdir lib\features\student\academics\results\screens
mkdir lib\features\student\academics\results\widgets
mkdir lib\features\student\academics\courses\screens
mkdir lib\features\student\academics\courses\widgets
mkdir lib\features\student\academics\attendance\screens
mkdir lib\features\student\academics\attendance\widgets
mkdir lib\features\student\examination\screens
mkdir lib\features\student\examination\widgets
mkdir lib\features\student\finance\screens
mkdir lib\features\student\finance\widgets
mkdir lib\features\student\notices_events\notices\screens
mkdir lib\features\student\notices_events\notices\widgets
mkdir lib\features\student\notices_events\events\screens
mkdir lib\features\student\notices_events\events\widgets
mkdir lib\features\student\homework\screens
mkdir lib\features\student\homework\widgets
mkdir lib\features\student\transport\screens
mkdir lib\features\student\transport\widgets
mkdir lib\features\student\profile\screens
mkdir lib\features\student\profile\widgets
mkdir lib\features\student\leave\screens
mkdir lib\features\student\leave\widgets
mkdir lib\features\student\achievements\screens
mkdir lib\features\student\achievements\widgets
mkdir lib\features\student\leaderboard\screens
mkdir lib\features\student\leaderboard\widgets
mkdir lib\features\student\library\screens
mkdir lib\features\student\library\widgets
mkdir lib\features\student\notifications\screens
mkdir lib\features\student\notifications\widgets
mkdir lib\features\student\live_classes\screens
mkdir lib\features\student\live_classes\widgets
mkdir lib\features\teacher\dashboard\screens
mkdir lib\features\teacher\dashboard\widgets
mkdir lib\features\teacher\my_classes\screens
mkdir lib\features\teacher\my_classes\widgets
mkdir lib\features\teacher\attendance\screens
mkdir lib\features\teacher\attendance\widgets
mkdir lib\features\teacher\homework_manager\screens
mkdir lib\features\teacher\homework_manager\widgets
mkdir lib\features\teacher\exams\screens
mkdir lib\features\teacher\exams\widgets
mkdir lib\features\teacher\paper_builder\screens
mkdir lib\features\teacher\paper_builder\widgets
mkdir lib\features\teacher\gradebook\screens
mkdir lib\features\teacher\gradebook\widgets
mkdir lib\features\teacher\student_directory\screens
mkdir lib\features\teacher\student_directory\widgets
mkdir lib\features\teacher\leave\screens
mkdir lib\features\teacher\leave\widgets
mkdir lib\features\teacher\notices\screens
mkdir lib\features\teacher\notices\widgets
mkdir lib\features\teacher\live_classes\screens
mkdir lib\features\teacher\live_classes\widgets
mkdir lib\features\teacher\materials\screens
mkdir lib\features\teacher\materials\widgets
mkdir lib\features\teacher\salary\screens
mkdir lib\features\teacher\salary\widgets
mkdir lib\features\teacher\profile\screens
mkdir lib\features\teacher\profile\widgets
mkdir lib\features\teacher\notifications\screens
mkdir lib\features\teacher\notifications\widgets
mkdir lib\features\teacher\timetable\screens
mkdir lib\features\teacher\timetable\widgets
```

### Step 6: Add Fonts
Download from Google Fonts and place in `assets/fonts/`:
- Outfit-Light.ttf (300)
- Outfit-Regular.ttf (400)
- Outfit-Medium.ttf (500)
- Outfit-SemiBold.ttf (600)
- Outfit-Bold.ttf (700)
- Outfit-ExtraBold.ttf (800)
- Outfit-Black.ttf (900)
- DMSans-Light.ttf (300)
- DMSans-Regular.ttf (400)
- DMSans-Medium.ttf (500)
- DMSans-SemiBold.ttf (600)
- DMSans-Bold.ttf (700)

### Step 7: Configure LOCAL Supabase
```dart
// lib/core/services/supabase_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';

class SupabaseService {
  // LOCAL Supabase URLs (from `supabase start` output)
  static const String supabaseUrl = 'http://127.0.0.1:54321';
  static const String supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0';
  
  // For production, use environment variables:
  // static const String supabaseUrl = String.fromEnvironment('SUPABASE_URL', defaultValue: 'http://127.0.0.1:54321');
  // static const String supabaseAnonKey = String.fromEnvironment('SUPABASE_ANON_KEY', defaultValue: '...');

  static Future<void> initialize() async {
    await Supabase.initialize(
      url: supabaseUrl,
      anonKey: supabaseAnonKey,
      debug: true, // Enable debug mode for local development
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
Open http://127.0.0.1:54323 in your browser to access Supabase Studio for:
- Viewing tables and data
- Running SQL queries
- Managing authentication
- Testing realtime subscriptions

---

## 3. DESIGN SYSTEM — DUAL THEME

### 3.1 Student Theme Colors
```dart
class StudentColors {
  static const Color primary = Color(0xFF4F46E5);
  static const Color primaryDeep = Color(0xFF3730A3);
  static const Color accent = Color(0xFF06B6D4);
  static const Color accent2 = Color(0xFFF59E0B);
  static const Color accent3 = Color(0xFF10B981);
  static const Color accent4 = Color(0xFFEF4444);
  static const Color accent5 = Color(0xFF8B5CF6);
  static const Color background = Color(0xFFF0F2FF);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color card = Color(0xFFFAFAFA);
  static const Color text = Color(0xFF0F172A);
  static const Color text2 = Color(0xFF475569);
  static const Color text3 = Color(0xFF94A3B8);
  static const Color border = Color(0xFFE2E8F0);
  static const Color primaryLight = Color(0xFFEEF2FF);
  static const Color success = Color(0xFF059669);
  static const Color successBg = Color(0xFFECFDF5);
  static const Color warning = Color(0xFFD97706);
  static const Color warningBg = Color(0xFFFEF3C7);
  static const Color error = Color(0xFFEF4444);
  static const Color errorBg = Color(0xFFFEF2F2);
}
```

### 3.2 Teacher Theme Colors
```dart
class TeacherColors {
  static const Color primary = Color(0xFF0EA5E9);
  static const Color primaryDeep = Color(0xFF0369A1);
  static const Color accent = Color(0xFF06B6D4);
  static const Color accent2 = Color(0xFFF59E0B);
  static const Color accent3 = Color(0xFF10B981);
  static const Color accent4 = Color(0xFFEF4444);
  static const Color accent5 = Color(0xFF8B5CF6);
  static const Color background = Color(0xFFF0F7FF);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color card = Color(0xFFFAFAFA);
  static const Color text = Color(0xFF0F172A);
  static const Color text2 = Color(0xFF475569);
  static const Color text3 = Color(0xFF94A3B8);
  static const Color border = Color(0xFFE2E8F0);
  static const Color primaryLight = Color(0xFFE0F2FE);
  static const Color success = Color(0xFF059669);
  static const Color successBg = Color(0xFFECFDF5);
  static const Color warning = Color(0xFFD97706);
  static const Color warningBg = Color(0xFFFEF3C7);
  static const Color error = Color(0xFFEF4444);
  static const Color errorBg = Color(0xFFFEF2F2);
}
```

### 3.3 Shared Screen Gradients

#### Splash Screen
- Student: `#0F0C29 → #302B63 → #24243E` (dark purple)
- Teacher: `#0C4A6E → #0369A1 → #0EA5E9` (dark blue)
- Logo emoji: Student 🎓, Teacher 👩‍🏫
- Ring animation: Student rgba(79,70,229,.4), Teacher rgba(14,165,233,.4)

#### Login Screen
- Same gradient as respective splash
- Role selector: 3 cards (Student 🎒, Parent 👨‍👩‍👧, Teacher 👨‍🏫)
- Active card highlighted with role's primary color
- Input fields: dark theme (rgba white .08 bg)
- Login button: gradient of role's primary

#### Dashboard Headers
- Student: `#4F46E5 → #302B63`
- Teacher: `#0C4A6E → #0EA5E9`

### 3.4 Screen-Specific Gradients (Teacher)

| Screen | Gradient |
|--------|----------|
| My Classes | `#0C4A6E → #0EA5E9` |
| Class Detail | `#0C4A6E → #0EA5E9` |
| Attendance | `#065F46 → #059669` |
| Timetable | `#1E40AF → #3B82F6` |
| Homework | `#BE185D → #DB2777` |
| Review Submissions | `#9F1239 → #E11D48` |
| Exams | `#134E4A → #0F766E` |
| Paper Builder | White bg, dark header |
| Gradebook | `#4F46E5 → #6366F1` |
| Student Directory | `#92400E → #D97706` |
| Leave | `#14532D → #16A34A` |
| Notices | `#92400E → #D97706` |
| Messaging | `#4F46E5 → #7C3AED` |
| Live Classes | `#9F1239 → #E11D48` |
| Materials | `#065F46 → #059669` |
| Salary | `#065F46 → #059669` |
| Profile | `#0C4A6E → #0EA5E9` |
| Settings | `#1E293B → #0F172A` |
| AI Chat | `#0F172A` (dark) |
| Notifications | `#0C4A6E → #0EA5E9` |

### 3.5 Spacing System
```dart
class AppDimensions {
  static const double paddingXS = 4.0;
  static const double paddingS = 8.0;
  static const double paddingM = 12.0;
  static const double paddingL = 14.0;
  static const double paddingXL = 18.0;
  static const double paddingXXL = 24.0;
  static const double radiusS = 6.0;
  static const double radiusM = 8.0;
  static const double radiusL = 10.0;
  static const double radiusXL = 12.0;
  static const double radiusXXL = 14.0;
  static const double radiusCard = 16.0;
  static const double radiusCardL = 18.0;
  static const double radiusCardXL = 20.0;
  static const double radiusModal = 24.0;
  static const double radiusChip = 20.0;
  static const double buttonHeight = 48.0;
  static const double inputHeight = 44.0;
  static const double fabSize = 46.0;
  static const double bottomNavHeight = 60.0;
}
```

### 3.6 Typography
```dart
class AppFonts {
  static const String heading = 'Outfit';
  static const String body = 'DM Sans';
}
```

### 3.7 Shared Shadows
```dart
class AppShadows {
  static List<BoxShadow> card = [BoxShadow(color: Colors.black.withOpacity(0.05), blurRadius: 12, offset: Offset(0, 2))];
  static List<BoxShadow> elevated = [BoxShadow(color: Color(0xFF4F46E5).withOpacity(0.4), blurRadius: 20, offset: Offset(0, 6))];
  static List<BoxShadow> teacherElevated = [BoxShadow(color: Color(0xFF0EA5E9).withOpacity(0.4), blurRadius: 20, offset: Offset(0, 6))];
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

### 4.2 Core Tables
```sql
-- PROFILES (extends auth.users, supports both students and teachers)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  user_id TEXT UNIQUE NOT NULL,            -- STU-2024-1082 or TCH-2024-0042
  full_name TEXT NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('student', 'parent', 'teacher')),
  class TEXT,                              -- X-A, X-B, IX-A (for students)
  department TEXT,                          -- Mathematics, Physics (for teachers)
  designation TEXT,                         -- HOD, Teacher, Coordinator (for teachers)
  roll_number INT,                          -- For students
  employee_id TEXT,                         -- For teachers: TCH-2024-0042
  gender TEXT CHECK (gender IN ('Male', 'Female', 'Other')),
  date_of_birth DATE,
  blood_group TEXT,
  email TEXT,
  phone TEXT,
  admission_number TEXT,
  nationality TEXT DEFAULT 'Indian',
  religion TEXT,
  category TEXT CHECK (category IN ('General','OBC','SC','ST','EWS')),
  address TEXT,
  house TEXT,
  avatar_url TEXT,
  father_name TEXT,
  father_occupation TEXT,
  father_phone TEXT,
  mother_name TEXT,
  mother_occupation TEXT,
  mother_phone TEXT,
  local_guardian TEXT,
  qualification TEXT,                       -- M.Sc Maths, B.Ed (for teachers)
  experience_years INT DEFAULT 0,           -- For teachers
  rating DECIMAL(2,1) DEFAULT 0.0,          -- For teachers
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
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  day_of_week INT NOT NULL CHECK (day_of_week BETWEEN 0 AND 5),
  start_time TIME NOT NULL,
  end_time TIME NOT NULL,
  room TEXT,
  class TEXT NOT NULL,
  is_break BOOLEAN DEFAULT FALSE,
  break_name TEXT,
  is_free_period BOOLEAN DEFAULT FALSE,
  free_period_activity TEXT,               -- Staff Room, Lesson Planning
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- COURSES
CREATE TABLE courses (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  syllabus_coverage DECIMAL(5,2) DEFAULT 0,
  total_chapters INT DEFAULT 0,
  completed_chapters INT DEFAULT 0,
  class TEXT NOT NULL,
  academic_year TEXT,
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- RESULTS
CREATE TABLE results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  exam_category TEXT NOT NULL CHECK (exam_category IN ('Class Test','Lab Test','Assignment','Mid-Term','End-Term','Other')),
  academic_year TEXT NOT NULL,
  marks_obtained DECIMAL(5,2),
  max_marks DECIMAL(5,2),
  grade TEXT,
  remarks TEXT,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAMS
CREATE TABLE exams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  exam_type TEXT NOT NULL CHECK (exam_type IN ('offline','online','hybrid')),
  exam_category TEXT CHECK (exam_category IN ('Class test','Lab test','Assignment','Mid term','End term','Other')),
  exam_date DATE,
  start_time TIME,
  duration_minutes INT DEFAULT 90,
  total_marks INT DEFAULT 100,
  venue TEXT,
  instructions TEXT,
  status TEXT DEFAULT 'upcoming' CHECK (status IN ('draft','upcoming','ongoing','completed','graded')),
  target_classes JSONB,                     -- ["X-A","X-B"]
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM QUESTIONS (Teacher creates, students answer)
CREATE TABLE exam_questions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
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
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  question_id UUID REFERENCES exam_questions(id) ON DELETE CASCADE,
  selected_answer INT,
  subjective_answer TEXT,
  attachment_url TEXT,
  is_correct BOOLEAN,
  marks_awarded DECIMAL(5,2),
  is_reviewed BOOLEAN DEFAULT FALSE,
  reviewed_by UUID REFERENCES profiles(id),
  submitted_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM SESSIONS
CREATE TABLE exam_sessions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  exam_id UUID REFERENCES exams(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  started_at TIMESTAMPTZ,
  submitted_at TIMESTAMPTZ,
  time_remaining_seconds INT,
  total_attempted INT DEFAULT 0,
  total_correct INT DEFAULT 0,
  total_marks DECIMAL(5,2) DEFAULT 0,
  xp_earned INT DEFAULT 0,
  integrity_score DECIMAL(3,2) DEFAULT 1.00,
  screen_switches INT DEFAULT 0,
  status TEXT DEFAULT 'in_progress' CHECK (status IN ('in_progress','submitted','auto_submitted'))
);

-- ATTENDANCE
CREATE TABLE attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  date DATE NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('present','absent','late','excused')),
  marked_at TIMESTAMPTZ DEFAULT NOW(),
  remarks TEXT,
  UNIQUE(student_id, subject_id, date)
);
```

### 4.3 Finance Tables
```sql
-- FEES
CREATE TABLE fees (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  fee_type TEXT NOT NULL,
  fee_period TEXT,
  amount DECIMAL(10,2) NOT NULL,
  due_date DATE,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','partial','paid','overdue')),
  amount_paid DECIMAL(10,2) DEFAULT 0,
  academic_year TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- PAYMENTS
CREATE TABLE payments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  fee_id UUID REFERENCES fees(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  amount DECIMAL(10,2) NOT NULL,
  payment_method TEXT CHECK (payment_method IN ('upi','card','netbanking')),
  transaction_id TEXT UNIQUE,
  receipt_number TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','success','failed','refunded')),
  paid_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- SALARY (Teacher)
CREATE TABLE salary (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  teacher_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  month TEXT NOT NULL,                     -- March 2026
  gross_salary DECIMAL(10,2),
  basic_pay DECIMAL(10,2),
  hra DECIMAL(10,2),
  da DECIMAL(10,2),
  special_allowance DECIMAL(10,2),
  pf_deduction DECIMAL(10,2),
  tds_deduction DECIMAL(10,2),
  professional_tax DECIMAL(10,2),
  net_salary DECIMAL(10,2),
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','credited')),
  credited_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.4 Homework Tables
```sql
-- HOMEWORK
CREATE TABLE homework (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  due_date TIMESTAMPTZ NOT NULL,
  max_marks INT,
  attachment_url TEXT,
  reference_material TEXT,
  rubric TEXT,
  target_class TEXT NOT NULL,
  status TEXT DEFAULT 'active' CHECK (status IN ('active','closed')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- HOMEWORK SUBMISSIONS
CREATE TABLE homework_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  homework_id UUID REFERENCES homework(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  submission_text TEXT,
  attachment_url TEXT,
  file_name TEXT,
  submitted_at TIMESTAMPTZ DEFAULT NOW(),
  status TEXT DEFAULT 'submitted' CHECK (status IN ('submitted','graded','late')),
  grade TEXT,
  marks DECIMAL(5,2),
  teacher_remarks TEXT,
  graded_by UUID REFERENCES profiles(id),
  graded_at TIMESTAMPTZ,
  is_plagiarism_flagged BOOLEAN DEFAULT FALSE
);
```

### 4.5 Communication Tables
```sql
-- NOTICES
CREATE TABLE notices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN ('Urgent','General','Event','Academic')),
  author_id UUID REFERENCES profiles(id),
  author_name TEXT,
  target_audience TEXT DEFAULT 'all',
  attachment_url TEXT,
  is_pinned BOOLEAN DEFAULT FALSE,
  is_urgent BOOLEAN DEFAULT FALSE,
  status TEXT DEFAULT 'published' CHECK (status IN ('draft','published','scheduled')),
  scheduled_at TIMESTAMPTZ,
  published_at TIMESTAMPTZ DEFAULT NOW(),
  expires_at TIMESTAMPTZ
);

-- EVENTS
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  description TEXT,
  event_date DATE,
  event_time TIME,
  venue TEXT,
  banner_url TEXT,
  gradient_colors JSONB,
  max_participants INT,
  current_participants INT DEFAULT 0,
  registration_deadline DATE,
  is_registration_open BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EVENT REGISTRATIONS
CREATE TABLE event_registrations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id UUID REFERENCES events(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  registered_at TIMESTAMPTZ DEFAULT NOW(),
  status TEXT DEFAULT 'registered',
  UNIQUE(event_id, student_id)
);

-- CHAT GROUPS
CREATE TABLE chat_groups (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  icon_url TEXT,
  created_by UUID REFERENCES profiles(id),
  member_count INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- CHAT GROUP MEMBERS
CREATE TABLE chat_group_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  group_id UUID REFERENCES chat_groups(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'member' CHECK (role IN ('admin','member')),
  joined_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(group_id, user_id)
);

-- MESSAGES
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  receiver_id UUID REFERENCES profiles(id),
  group_id UUID REFERENCES chat_groups(id),
  content TEXT NOT NULL,
  attachment_url TEXT,
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- NOTIFICATIONS
CREATE TABLE notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  type TEXT NOT NULL,
  reference_id UUID,
  reference_type TEXT,
  is_read BOOLEAN DEFAULT FALSE,
  priority TEXT DEFAULT 'normal',
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.6 Leave Tables
```sql
-- LEAVE APPLICATIONS (both student and teacher)
CREATE TABLE leave_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  applicant_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  applicant_role TEXT CHECK (applicant_role IN ('student','teacher')),
  leave_type TEXT NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  reason TEXT NOT NULL,
  attachment_url TEXT,
  status TEXT DEFAULT 'pending' CHECK (status IN ('pending','approved','rejected','cancelled')),
  approved_by UUID REFERENCES profiles(id),
  approved_at TIMESTAMPTZ,
  remarks TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.7 Achievement & Gamification Tables
```sql
-- ACHIEVEMENTS
CREATE TABLE achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  badge_color JSONB,
  xp_reward INT DEFAULT 0,
  criteria TEXT,
  rarity TEXT DEFAULT 'common',
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- STUDENT ACHIEVEMENTS
CREATE TABLE student_achievements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  achievement_id UUID REFERENCES achievements(id) ON DELETE CASCADE,
  earned_at TIMESTAMPTZ DEFAULT NOW(),
  progress DECIMAL(5,2) DEFAULT 100,
  UNIQUE(student_id, achievement_id)
);
```

### 4.8 Library Tables
```sql
CREATE TABLE library_books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  author TEXT,
  isbn TEXT,
  category TEXT,
  shelf_location TEXT,
  total_copies INT DEFAULT 1,
  available_copies INT DEFAULT 1,
  cover_url TEXT,
  is_digital BOOLEAN DEFAULT FALSE,
  digital_url TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE library_borrows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  book_id UUID REFERENCES library_books(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  borrowed_at TIMESTAMPTZ DEFAULT NOW(),
  due_at TIMESTAMPTZ NOT NULL,
  returned_at TIMESTAMPTZ,
  renewals_used INT DEFAULT 0,
  max_renewals INT DEFAULT 2,
  status TEXT DEFAULT 'borrowed',
  fine_amount DECIMAL(10,2) DEFAULT 0
);
```

### 4.9 Transport Tables
```sql
CREATE TABLE bus_routes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_name TEXT NOT NULL,
  bus_number TEXT,
  driver_name TEXT,
  driver_phone TEXT,
  total_capacity INT DEFAULT 40,
  current_passengers INT DEFAULT 0,
  status TEXT DEFAULT 'active'
);

CREATE TABLE bus_stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  stop_name TEXT NOT NULL,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  stop_order INT NOT NULL,
  estimated_arrival TIME,
  is_student_stop BOOLEAN DEFAULT FALSE
);

CREATE TABLE bus_locations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  speed DECIMAL(5,2),
  heading DECIMAL(5,2),
  eta_minutes INT,
  recorded_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.10 Live Classes Tables
```sql
CREATE TABLE live_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  scheduled_at TIMESTAMPTZ NOT NULL,
  duration_minutes INT DEFAULT 60,
  platform TEXT DEFAULT 'EduVerse',
  stream_url TEXT,
  recording_url TEXT,
  is_live BOOLEAN DEFAULT FALSE,
  viewer_count INT DEFAULT 0,
  target_class TEXT,
  status TEXT DEFAULT 'scheduled' CHECK (status IN ('draft','scheduled','live','completed','recorded')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE live_class_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  live_class_id UUID REFERENCES live_classes(id) ON DELETE CASCADE,
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  comment TEXT NOT NULL,
  is_pinned BOOLEAN DEFAULT FALSE,
  likes INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.11 Study Materials Table
```sql
CREATE TABLE study_materials (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  material_type TEXT CHECK (material_type IN ('Notes','PPTs','Videos','Worksheets')),
  target_class TEXT,
  attachment_urls JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.12 Settings & Documents
```sql
CREATE TABLE user_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE UNIQUE,
  push_notifications BOOLEAN DEFAULT TRUE,
  sms_alerts BOOLEAN DEFAULT TRUE,
  email_reports BOOLEAN DEFAULT FALSE,
  ai_personalization BOOLEAN DEFAULT TRUE,
  biometric_login BOOLEAN DEFAULT TRUE,
  dark_mode BOOLEAN DEFAULT FALSE,
  language TEXT DEFAULT 'en',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  document_type TEXT NOT NULL,
  file_url TEXT NOT NULL,
  file_name TEXT,
  verification_status TEXT DEFAULT 'pending',
  uploaded_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.13 Grading Policy Table
```sql
CREATE TABLE grading_policies (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  teacher_id UUID REFERENCES profiles(id),
  class TEXT NOT NULL,
  subject_id UUID REFERENCES subjects(id),
  mid_term_weight INT DEFAULT 30,
  final_term_weight INT DEFAULT 40,
  attendance_weight INT DEFAULT 5,
  assignment_weight INT DEFAULT 10,
  class_test_weight INT DEFAULT 10,
  lab_weight INT DEFAULT 5,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.14 RLS Policies
```sql
-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE results ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance ENABLE ROW LEVEL SECURITY;
ALTER TABLE fees ENABLE ROW LEVEL SECURITY;
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;
ALTER TABLE salary ENABLE ROW LEVEL SECURITY;
ALTER TABLE homework_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE leave_applications ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE event_registrations ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_achievements ENABLE ROW LEVEL SECURITY;
ALTER TABLE library_borrows ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE grading_policies ENABLE ROW LEVEL SECURITY;

-- Profile policies
CREATE POLICY "Users can view own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile" ON profiles FOR UPDATE USING (auth.uid() = id);
CREATE POLICY "Teachers can view student profiles" ON profiles FOR SELECT USING (true);

-- Student data policies
CREATE POLICY "Students can view own results" ON results FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Teachers can view all results" ON results FOR SELECT USING (true);
CREATE POLICY "Teachers can insert results" ON results FOR INSERT WITH CHECK (true);
CREATE POLICY "Teachers can update results" ON results FOR UPDATE USING (true);

CREATE POLICY "Students can view own attendance" ON attendance FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Teachers can view all attendance" ON attendance FOR SELECT USING (true);
CREATE POLICY "Teachers can insert attendance" ON attendance FOR INSERT WITH CHECK (true);

-- Fee policies
CREATE POLICY "Students can view own fees" ON fees FOR SELECT USING (auth.uid() = student_id);

-- Salary policies
CREATE POLICY "Teachers can view own salary" ON salary FOR SELECT USING (auth.uid() = teacher_id);

-- Homework policies
CREATE POLICY "Students can submit homework" ON homework_submissions FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Students can view own submissions" ON homework_submissions FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Teachers can view all submissions" ON homework_submissions FOR SELECT USING (true);
CREATE POLICY "Teachers can grade submissions" ON homework_submissions FOR UPDATE USING (true);

-- Exam policies
CREATE POLICY "Students can submit exams" ON exam_submissions FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Teachers can view exam submissions" ON exam_submissions FOR SELECT USING (true);
CREATE POLICY "Teachers can grade exams" ON exam_submissions FOR UPDATE USING (true);

-- Leave policies
CREATE POLICY "Users can apply leave" ON leave_applications FOR INSERT WITH CHECK (auth.uid() = applicant_id);
CREATE POLICY "Users can view own leaves" ON leave_applications FOR SELECT USING (auth.uid() = applicant_id);
CREATE POLICY "Teachers can view student leaves" ON leave_applications FOR SELECT USING (true);
CREATE POLICY "Teachers can approve leaves" ON leave_applications FOR UPDATE USING (true);

-- Message policies
CREATE POLICY "Users can send messages" ON messages FOR INSERT WITH CHECK (auth.uid() = sender_id);
CREATE POLICY "Users can view own messages" ON messages FOR SELECT USING (auth.uid() = sender_id OR auth.uid() = receiver_id);

-- Notification policies
CREATE POLICY "Users can view own notifications" ON notifications FOR SELECT USING (auth.uid() = user_id);

-- Grading policy
CREATE POLICY "Teachers can manage grading" ON grading_policies FOR ALL USING (auth.uid() = teacher_id);

-- Public read policies
CREATE POLICY "Anyone can read subjects" ON subjects FOR SELECT USING (true);
CREATE POLICY "Anyone can read timetable" ON timetable FOR SELECT USING (true);
CREATE POLICY "Anyone can read notices" ON notices FOR SELECT USING (true);
CREATE POLICY "Anyone can read events" ON events FOR SELECT USING (true);
CREATE POLICY "Anyone can read homework" ON homework FOR SELECT USING (true);
CREATE POLICY "Anyone can read exams" ON exams FOR SELECT USING (true);
CREATE POLICY "Anyone can read achievements" ON achievements FOR SELECT USING (true);
CREATE POLICY "Anyone can read library books" ON library_books FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus routes" ON bus_routes FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus stops" ON bus_stops FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus locations" ON bus_locations FOR SELECT USING (true);
CREATE POLICY "Anyone can read live classes" ON live_classes FOR SELECT USING (true);
CREATE POLICY "Anyone can read study materials" ON study_materials FOR SELECT USING (true);
CREATE POLICY "Teachers can manage materials" ON study_materials FOR ALL USING (auth.uid() = teacher_id);
```

### 4.15 Enable Realtime
```sql
ALTER PUBLICATION supabase_realtime ADD TABLE bus_locations;
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE live_classes;
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;
ALTER PUBLICATION supabase_realtime ADD TABLE exam_sessions;
ALTER PUBLICATION supabase_realtime ADD TABLE live_class_comments;
ALTER PUBLICATION supabase_realtime ADD TABLE attendance;
```

### 4.16 Seed Data
```sql
-- Sample subjects
INSERT INTO subjects (name, icon, color, total_chapters, class) VALUES
  ('Mathematics', '📐', '#4F46E5', 42, 'X-A'),
  ('Physics', '⚛️', '#3B82F6', 38, 'X-A'),
  ('Chemistry', '⚗️', '#F59E0B', 35, 'X-A'),
  ('English', '📖', '#10B981', 28, 'X-A');

-- Sample achievements
INSERT INTO achievements (name, description, icon, badge_color, xp_reward, criteria, rarity) VALUES
  ('Academic Excellence', 'Scored 90%+ in term exams', '🏆', '["#FCD34D","#F59E0B"]', 500, 'Score 90%+ in any term', 'rare'),
  ('18-Day Streak', 'Logged in 18 consecutive days', '🔥', '["#60A5FA","#3B82F6"]', 300, 'Maintain login streak', 'uncommon');
```

---

## 5. AI FUNCTIONS (LANGCHAIN TOOL CALLING)

### 5.1 Student AI Tools (20 tools)
| # | Tool Name | Purpose | Example Query |
|---|-----------|---------|---------------|
| 1 | `get_timetable` | Fetch class schedule | "What classes do I have today?" |
| 2 | `get_homework` | Fetch homework status | "What homework is due?" |
| 3 | `get_fee_status` | Check fee balance | "How much fees do I owe?" |
| 4 | `get_attendance` | Get attendance % | "What's my attendance?" |
| 5 | `get_exam_info` | Get exam schedule | "When is my next exam?" |
| 6 | `get_bus_location` | Real-time bus ETA | "Where is my bus?" |
| 7 | `get_performance` | Academic summary | "How am I doing?" |
| 8 | `generate_study_plan` | Create study plan | "Help me prepare for exams" |
| 9 | `explain_concept` | Explain topic | "Explain integration by parts" |
| 10 | `get_notifications` | Recent alerts | "Any new notifications?" |
| 11 | `get_library_status` | Borrowed books | "What books do I have?" |
| 12 | `get_achievements` | Badges & XP | "What badges have I earned?" |
| 13 | `get_events` | School events | "What events are coming up?" |
| 14 | `get_leaderboard` | Rankings | "What's my class rank?" |
| 15 | `submit_homework` | Submit assignment | "Submit my math homework" |
| 16 | `apply_leave` | Apply for leave | "I need sick leave" |
| 17 | `get_notices` | School notices | "Any new announcements?" |
| 18 | `answer_general` | General Q&A | "What is the school timing?" |
| 19 | `generate_practice` | Practice questions | "Give me physics problems" |
| 20 | `get_live_class` | Live class status | "Is there a live class now?" |

### 5.2 Teacher AI Tools (20 tools)
| # | Tool Name | Purpose | Example Query |
|---|-----------|---------|---------------|
| 1 | `get_class_students` | List students in class | "Show me X-A students" |
| 2 | `get_class_performance` | Class performance stats | "How is X-A performing?" |
| 3 | `get_at_risk_students` | Identify struggling students | "Who needs help?" |
| 4 | `generate_questions` | Generate exam questions | "Create 5 MCQs on calculus" |
| 5 | `generate_lesson_plan` | Create lesson plan | "Make a lesson plan for integration" |
| 6 | `auto_grade_homework` | Auto-grade submissions | "Grade the pending homework" |
| 7 | `get_submission_status` | Check submission stats | "How many submitted?" |
| 8 | `generate_remedial_plan` | Create remedial sessions | "Make a plan for weak students" |
| 9 | `create_notice` | Draft a notice | "Draft a notice about exam dates" |
| 10 | `get_attendance_stats` | Attendance analytics | "Show attendance trends" |
| 11 | `get_teacher_schedule` | Teacher's timetable | "What's my schedule today?" |
| 12 | `get_pending_tasks` | Pending reviews | "What needs my attention?" |
| 13 | `generate_report` | Generate class report | "Create X-A performance report" |
| 14 | `explain_pedagogy` | Teaching strategies | "How to teach fractions?" |
| 15 | `get_leave_balance` | Leave status | "How many leaves do I have?" |
| 16 | `get_salary_info` | Salary details | "Show my salary breakup" |
| 17 | `get_parent_communications` | Parent messages | "Any parent messages?" |
| 18 | `upload_material` | Upload study material | "Upload notes for X-A" |
| 19 | `get_exam_analytics` | Exam statistics | "Show exam pass rates" |
| 20 | `answer_general` | General Q&A | "What's the school policy?" |

### 5.3 AI Service Setup
```dart
// lib/core/services/ai/ai_service.dart
class AIService {
  late final ChatOpenAI _llm;
  late final AgentExecutor _studentAgent;
  late final AgentExecutor _teacherAgent;
  
  Future<void> initialize() async {
    _llm = ChatOpenAI(
      apiKey: 'YOUR_OPENAI_API_KEY',
      defaultOptions: ChatOpenAIOptions(model: 'gpt-4o'),
    );
    
    // Student agent with 20 tools
    _studentAgent = AgentExecutor(
      agent: OpenAIFunctionsAgent.fromLLMAndTools(
        llm: _llm,
        tools: _getStudentTools(),
      ),
    );
    
    // Teacher agent with 20 tools
    _teacherAgent = AgentExecutor(
      agent: OpenAIFunctionsAgent.fromLLMAndTools(
        llm: _llm,
        tools: _getTeacherTools(),
      ),
    );
  }
  
  Future<String> sendMessage(String message, String role, {Map<String, dynamic>? context}) async {
    final agent = role == 'student' ? _studentAgent : _teacherAgent;
    final systemPrompt = role == 'student' 
        ? _getStudentPrompt(context)
        : _getTeacherPrompt(context);
    
    final result = await agent.invoke({
      'input': message,
      'system_message': systemPrompt,
    });
    
    return result.toString();
  }
  
  String _getStudentPrompt(Map<String, dynamic>? ctx) => '''
You are Mindora AI Student Assistant. Help with academic queries, school info, and study planning.
Student: ${ctx?['name'] ?? 'Student'} | Class: ${ctx?['class'] ?? 'X-A'} | Streak: ${ctx?['streak'] ?? 0} days
Use tools to get real-time data. Be encouraging and use emojis.
''';

  String _getTeacherPrompt(Map<String, dynamic>? ctx) => '''
You are Mindora AI Teaching Assistant. Help with lesson planning, grading, student analytics, and classroom management.
Teacher: ${ctx?['name'] ?? 'Teacher'} | Dept: ${ctx?['dept'] ?? 'Mathematics'}
Use tools to access student data and class analytics. Be professional and provide actionable insights.
''';
}
```

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
│   │   ├── app_colors.dart              # Both student & teacher colors
│   │   ├── app_fonts.dart
│   │   ├── app_dimensions.dart
│   │   ├── app_strings.dart
│   │   ├── app_gradients.dart
│   │   └── app_shadows.dart
│   ├── theme/
│   │   ├── app_theme.dart
│   │   ├── student_theme.dart
│   │   └── teacher_theme.dart
│   ├── utils/
│   │   ├── date_formatter.dart
│   │   ├── currency_formatter.dart
│   │   ├── validators.dart
│   │   ├── responsive_utils.dart
│   │   └── role_utils.dart              # Detect student vs teacher
│   ├── services/
│   │   ├── supabase_service.dart        # LOCAL Supabase config
│   │   ├── auth_service.dart
│   │   ├── storage_service.dart
│   │   ├── notification_service.dart
│   │   ├── realtime_service.dart
│   │   ├── ai/
│   │   │   ├── ai_service.dart
│   │   │   ├── langchain_config.dart
│   │   │   ├── tools/
│   │   │   │   ├── student/             # 20 student tools
│   │   │   │   └── teacher/             # 20 teacher tools
│   │   │   ├── prompts/
│   │   │   └── models/
│   │   └── maps/
│   │       ├── osm_service.dart
│   │       └── bus_tracking_service.dart
│   ├── providers/
│   │   ├── auth_provider.dart
│   │   ├── user_provider.dart
│   │   ├── role_provider.dart           # Student or Teacher
│   │   ├── theme_provider.dart
│   │   └── ai_provider.dart
│   └── models/
│       ├── user_model.dart
│       ├── subject_model.dart
│       ├── timetable_model.dart
│       ├── result_model.dart
│       ├── exam_model.dart
│       ├── fee_model.dart
│       ├── salary_model.dart
│       ├── homework_model.dart
│       ├── attendance_model.dart
│       ├── leave_model.dart
│       ├── event_model.dart
│       ├── notice_model.dart
│       ├── achievement_model.dart
│       ├── library_model.dart
│       ├── message_model.dart
│       ├── notification_model.dart
│       ├── bus_route_model.dart
│       ├── live_class_model.dart
│       ├── study_material_model.dart
│       ├── grading_policy_model.dart
│       └── leaderboard_model.dart
│
├── shared/
│   ├── widgets/
│   │   ├── app_bottom_nav.dart          # Role-aware nav
│   │   ├── app_header.dart              # Role-aware gradient
│   │   ├── ai_fab.dart                  # Role-aware color
│   │   ├── app_card.dart
│   │   ├── app_button.dart              # 4 variants
│   │   ├── app_input_field.dart
│   │   ├── app_chip.dart
│   │   ├── app_chip_row.dart
│   │   ├── app_modal.dart
│   │   ├── app_toggle.dart
│   │   ├── app_badge.dart
│   │   ├── app_section_header.dart
│   │   ├── app_quick_access_grid.dart
│   │   ├── ai_tag.dart
│   │   ├── ai_typing_indicator.dart
│   │   ├── live_dot.dart
│   │   ├── success_check.dart
│   │   ├── countdown_timer.dart
│   │   ├── circular_progress.dart
│   │   ├── gradient_container.dart
│   │   └── responsive_layout.dart
│   └── animations/
│       ├── fade_slide_animation.dart
│       ├── scale_animation.dart
│       └── confetti_animation.dart
│
├── features/
│   ├── shared/                          # Screens shared by both roles
│   │   ├── splash/
│   │   │   └── screens/splash_screen.dart
│   │   ├── login/
│   │   │   ├── screens/login_screen.dart
│   │   │   └── widgets/
│   │   │       ├── login_role_selector.dart
│   │   │       ├── biometric_modal.dart
│   │   │       ├── otp_modal.dart
│   │   │       └── qr_scan_modal.dart
│   │   ├── settings/
│   │   │   └── screens/settings_screen.dart
│   │   ├── messaging/
│   │   │   ├── screens/
│   │   │   │   ├── messaging_screen.dart
│   │   │   │   ├── chat_detail_screen.dart
│   │   │   │   └── create_group_screen.dart
│   │   │   └── widgets/
│   │   └── ai_chat/
│   │       ├── screens/ai_chat_screen.dart
│   │       └── widgets/
│   │
│   ├── student/                         # 29 student screens
│   │   ├── dashboard/
│   │   │   ├── screens/dashboard_screen.dart
│   │   │   └── widgets/
│   │   │       ├── dashboard_header.dart
│   │   │       ├── stats_row.dart
│   │   │       ├── quick_access_grid.dart
│   │   │       ├── today_schedule_card.dart
│   │   │       ├── ai_insight_card.dart
│   │   │       └── pending_homework_card.dart
│   │   ├── academics/
│   │   │   ├── timetable/
│   │   │   ├── results/
│   │   │   ├── courses/
│   │   │   └── attendance/
│   │   ├── examination/
│   │   ├── finance/
│   │   ├── notices_events/
│   │   ├── homework/
│   │   ├── transport/
│   │   ├── profile/
│   │   ├── leave/
│   │   ├── achievements/
│   │   ├── leaderboard/
│   │   ├── library/
│   │   ├── notifications/
│   │   └── live_classes/
│   │
│   └── teacher/                         # 29 teacher screens
│       ├── dashboard/
│       │   ├── screens/dashboard_screen.dart
│       │   └── widgets/
│       │       ├── teacher_header.dart
│       │       ├── teacher_stats_row.dart
│       │       ├── teacher_quick_access.dart
│       │       ├── today_schedule_card.dart
│       │       ├── ai_teaching_insight.dart
│       │       └── pending_reviews_card.dart
│       ├── my_classes/
│       │   ├── screens/my_classes_screen.dart
│       │   ├── screens/class_detail_screen.dart
│       │   └── widgets/
│       │       ├── class_card.dart
│       │       ├── student_tile.dart
│       │       ├── student_profile_modal.dart
│       │       ├── top_performers_tab.dart
│       │       └── at_risk_tab.dart
│       ├── attendance/
│       │   ├── screens/attendance_screen.dart
│       │   └── widgets/
│       │       ├── attendance_row.dart
│       │       ├── attendance_settings_modal.dart
│       │       └── attendance_success_modal.dart
│       ├── homework_manager/
│       │   ├── screens/homework_screen.dart
│       │   ├── screens/review_submissions_screen.dart
│       │   ├── screens/grading_screen.dart
│       │   └── widgets/
│       │       ├── homework_card.dart
│       │       ├── submission_tile.dart
│       │       ├── create_assignment_modal.dart
│       │       └── remind_modal.dart
│       ├── exams/
│       │   ├── screens/exams_screen.dart
│       │   └── widgets/
│       │       ├── exam_card.dart
│       │       ├── create_exam_modal.dart
│       │       └── edit_exam_modal.dart
│       ├── paper_builder/
│       │   ├── screens/paper_builder_screen.dart
│       │   └── widgets/
│       │       ├── question_block.dart
│       │       ├── mcq_option.dart
│       │       ├── ai_generator.dart
│       │       └── paper_stats.dart
│       ├── gradebook/
│       │   ├── screens/gradebook_screen.dart
│       │   ├── screens/grading_config_screen.dart
│       │   └── widgets/
│       │       ├── performance_matrix.dart
│       │       ├── student_grade_row.dart
│       │       ├── mark_update_modal.dart
│       │       ├── export_modal.dart
│       │       └── weightage_card.dart
│       ├── student_directory/
│       │   ├── screens/student_directory_screen.dart
│       │   └── widgets/
│       │       ├── student_card.dart
│       │       ├── class_filter_chips.dart
│       │       └── student_profile_modal.dart
│       ├── timetable/
│       │   ├── screens/timetable_screen.dart
│       │   └── widgets/
│       │       ├── day_chip_selector.dart
│       │       ├── calendar_picker.dart
│       │       └── class_slot_card.dart
│       ├── leave/
│       │   ├── screens/leave_screen.dart
│       │   └── widgets/
│       │       ├── leave_quota.dart
│       │       ├── leave_card.dart
│       │       └── apply_leave_modal.dart
│       ├── notices/
│       │   ├── screens/notices_screen.dart
│       │   └── widgets/
│       │       ├── notice_card.dart
│       │       ├── create_notice_modal.dart
│       │       └── notice_detail_modal.dart
│       ├── live_classes/
│       │   ├── screens/live_classes_screen.dart
│       │   ├── screens/teacher_live_session_screen.dart
│       │   ├── screens/live_playback_screen.dart
│       │   └── widgets/
│       │       ├── live_class_card.dart
│       │       ├── live_controls.dart
│       │       ├── chat_overlay.dart
│       │       └── schedule_class_modal.dart
│       ├── materials/
│       │   ├── screens/materials_screen.dart
│       │   └── widgets/
│       │       ├── material_card.dart
│       │       └── upload_material_modal.dart
│       ├── salary/
│       │   ├── screens/salary_screen.dart
│       │   └── widgets/
│       │       ├── salary_header.dart
│       │       ├── breakup_table.dart
│       │       └── download_button.dart
│       ├── profile/
│       │   ├── screens/profile_screen.dart
│       │   └── widgets/
│       │       ├── teacher_profile_header.dart
│       │       ├── teacher_info_section.dart
│       │       └── edit_profile_modal.dart
│       └── notifications/
│           ├── screens/notifications_screen.dart
│           └── widgets/
│               └── notification_card.dart
│
└── l10n/
    ├── app_en.arb
    └── app_hi.arb
```

---

## 7. CORE DART FILES (TEMPLATES)

### 7.1 main.dart
```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:eduverse_student/app.dart';
import 'package:eduverse_student/core/services/supabase_service.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await SupabaseService.initialize();
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  runApp(const MindoraApp());
}
```

### 7.2 app.dart
```dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/theme/app_theme.dart';
import 'package:eduverse_student/app_router.dart';

class MindoraApp extends StatelessWidget {
  const MindoraApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Mindora AI',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      themeMode: ThemeMode.light,
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
    // SHARED ROUTES
    GoRoute(path: '/splash', builder: (_, __) => const SplashScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/settings', builder: (_, __) => const SettingsScreen()),
    GoRoute(path: '/ai-chat', builder: (_, __) => const AIChatScreen()),
    GoRoute(path: '/messaging', builder: (_, __) => const MessagingScreen()),
    GoRoute(path: '/chat-detail', builder: (_, __) => const ChatDetailScreen()),
    GoRoute(path: '/create-group', builder: (_, __) => const CreateGroupScreen()),
    
    // STUDENT ROUTES (29)
    GoRoute(path: '/student/dashboard', builder: (_, __) => const StudentDashboard()),
    GoRoute(path: '/student/timetable', builder: (_, __) => const StudentTimetable()),
    GoRoute(path: '/student/results', builder: (_, __) => const StudentResults()),
    GoRoute(path: '/student/exams', builder: (_, __) => const StudentExams()),
    GoRoute(path: '/student/exam-instructions', builder: (_, __) => const ExamInstructionsScreen()),
    GoRoute(path: '/student/online-exam', builder: (_, __) => const OnlineExamScreen()),
    GoRoute(path: '/student/exam-success', builder: (_, __) => const ExamSuccessScreen()),
    GoRoute(path: '/student/fees', builder: (_, __) => const StudentFees()),
    GoRoute(path: '/student/notices', builder: (_, __) => const StudentNotices()),
    GoRoute(path: '/student/homework', builder: (_, __) => const StudentHomework()),
    GoRoute(path: '/student/transport', builder: (_, __) => const StudentTransport()),
    GoRoute(path: '/student/events', builder: (_, __) => const StudentEvents()),
    GoRoute(path: '/student/achievements', builder: (_, __) => const StudentAchievements()),
    GoRoute(path: '/student/attendance', builder: (_, __) => const StudentAttendance()),
    GoRoute(path: '/student/leave', builder: (_, __) => const StudentLeave()),
    GoRoute(path: '/student/profile', builder: (_, __) => const StudentProfile()),
    GoRoute(path: '/student/library', builder: (_, __) => const StudentLibrary()),
    GoRoute(path: '/student/courses', builder: (_, __) => const StudentCourses()),
    GoRoute(path: '/student/notifications', builder: (_, __) => const StudentNotifications()),
    GoRoute(path: '/student/live-classes', builder: (_, __) => const StudentLiveClasses()),
    GoRoute(path: '/student/live-class-detail', builder: (_, __) => const StudentLiveClassDetail()),
    GoRoute(path: '/student/leaderboard', builder: (_, __) => const StudentLeaderboard()),
    
    // TEACHER ROUTES (29)
    GoRoute(path: '/teacher/dashboard', builder: (_, __) => const TeacherDashboard()),
    GoRoute(path: '/teacher/my-classes', builder: (_, __) => const TeacherMyClasses()),
    GoRoute(path: '/teacher/class-detail', builder: (_, __) => const TeacherClassDetail()),
    GoRoute(path: '/teacher/attendance', builder: (_, __) => const TeacherAttendance()),
    GoRoute(path: '/teacher/timetable', builder: (_, __) => const TeacherTimetable()),
    GoRoute(path: '/teacher/homework', builder: (_, __) => const TeacherHomework()),
    GoRoute(path: '/teacher/review-submissions', builder: (_, __) => const TeacherReviewSubmissions()),
    GoRoute(path: '/teacher/grading', builder: (_, __) => const TeacherGrading()),
    GoRoute(path: '/teacher/exams', builder: (_, __) => const TeacherExams()),
    GoRoute(path: '/teacher/paper-builder', builder: (_, __) => const TeacherPaperBuilder()),
    GoRoute(path: '/teacher/gradebook', builder: (_, __) => const TeacherGradebook()),
    GoRoute(path: '/teacher/grading-config', builder: (_, __) => const TeacherGradingConfig()),
    GoRoute(path: '/teacher/student-directory', builder: (_, __) => const TeacherStudentDirectory()),
    GoRoute(path: '/teacher/leave', builder: (_, __) => const TeacherLeave()),
    GoRoute(path: '/teacher/notices', builder: (_, __) => const TeacherNotices()),
    GoRoute(path: '/teacher/live-classes', builder: (_, __) => const TeacherLiveClasses()),
    GoRoute(path: '/teacher/live-session', builder: (_, __) => const TeacherLiveSession()),
    GoRoute(path: '/teacher/live-playback', builder: (_, __) => const TeacherLivePlayback()),
    GoRoute(path: '/teacher/materials', builder: (_, __) => const TeacherMaterials()),
    GoRoute(path: '/teacher/salary', builder: (_, __) => const TeacherSalary()),
    GoRoute(path: '/teacher/profile', builder: (_, __) => const TeacherProfile()),
    GoRoute(path: '/teacher/notifications', builder: (_, __) => const TeacherNotifications()),
    GoRoute(path: '/teacher/exam-attendance', builder: (_, __) => const TeacherExamAttendance()),
    GoRoute(path: '/teacher/grade-student', builder: (_, __) => const TeacherGradeStudent()),
  ],
);
```

### 7.4 Role-Based Navigation
```dart
// After login, navigate based on role
void handleLoginSuccess(String role) {
  if (role == 'student') {
    context.go('/student/dashboard');
  } else if (role == 'teacher') {
    context.go('/teacher/dashboard');
  }
}

// Bottom nav items change based on role
class RoleAwareBottomNav extends StatelessWidget {
  final String role; // 'student' or 'teacher'
  final int currentIndex;
  
  // Student nav: Home, Courses, Results, Achieve, Profile
  // Teacher nav: Home, Classes, Exams, Messages, Profile
}
```

---

## 8. STUDENT SCREENS (29 SCREENS)

Each screen specification includes: route, background, header gradient, body sections, modals, bottom nav.

### 8.1 Student Dashboard
- Route: `/student/dashboard`
- Background: `#F5F6FF`
- Header: `#4F46E5 → #302B63` gradient
- Content: Greeting, notification bell (badge 3), avatar, streak (18 days), stats (94% attendance, 91.4 score, 3rd rank, 2450 XP)
- Quick Access: 4x4 grid (16 items with specific bg colors)
- Today's Schedule: 3 class rows
- AI Insight: Purple bordered card
- Pending Homework: 2 items
- Bottom Nav: Home (active), Courses, Results, Achieve, Profile
- AI FAB: Purple gradient

### 8.2-8.29 Remaining Student Screens
Follow the pattern from the previous EduVerse instruction.md (which has detailed specs for all 29 student screens).

---

## 9. TEACHER SCREENS (29 SCREENS)

### 9.1 Teacher Dashboard
- Route: `/teacher/dashboard`
- Background: `#F0F7FF`
- Header: `#0C4A6E → #0EA5E9` gradient
- Content: "Good Morning 🌤️" + "Mrs. Priya Sharma" + "Mathematics · HOD"
- Stats: 156 Students, 92% Avg Attend., 6 Classes, 8 Pending
- Quick Access: 12 items (4x3 grid)
  - My Classes 📚, Attendance 📋, Homework 📝, Exams 📝
  - Gradebook 📊, Students 👥, Timetable 🗓️, Notices 📢
  - Leave 🏖️, Live Class 🎥, Materials 📁, Salary 💰
- Today's Schedule: 4 classes (Math X-A 8AM, Math X-B 9AM NOW, Math IX-A 11:20, Free Period 1PM)
- AI Teaching Insight: "5 students in X-A scored below 40%. Suggest remedial session."
- Pending Reviews: Integration HW (32/45 submitted), Algebra Quiz (28/40 graded)
- Bottom Nav: Home (active), Classes, Exams, Messages, Profile
- AI FAB: Cyan gradient `#0EA5E9 → #06B6D4`

### 9.2 My Classes Screen
- Route: `/teacher/my-classes`
- Header: `#0C4A6E → #0EA5E9`, title "My Classes", "3 Sections"
- Class Cards (3): X-A (45 students, 92% attendance, 78.5 avg, 3 pending HW), X-B (48 students), IX-A (40 students)
- Each card: left border color, stats row, "Next Class" info
- AI Class Insight card
- Click → Class Detail Screen

### 9.3 Class Detail Screen
- Route: `/teacher/class-detail`
- Header: "Class X-A", student count badge
- Tab Bar: Students, Top Perf., At Risk
- Students Tab: List with avatar, name, roll, attendance badge
- Top Performers Tab: 🥇🥈🥉 podium with scores
- At Risk Tab: Warning cards with low scores, "Generate Remedial Plan" button
- Bottom buttons: "📋 Mark Attendance", "📝 Assign HW"
- Click student → Student Profile Modal (call, message, performance, rank)

### 9.4 Teacher Attendance Screen
- Route: `/teacher/attendance`
- Header: `#065F46 → #059669`, "Mark Attendance"
- Date & class selector bar
- "All Present" quick button
- Student list: Roll number, name, P/A/L toggle buttons
- Submit button with count (42P · 2A · 1L)
- Success modal

### 9.5 Teacher Timetable
- Route: `/teacher/timetable`
- Header: `#1E40AF → #3B82F6`
- Date scroller with calendar picker
- Class cards with subject, class name, room, NOW badge
- Break/free period indicators

### 9.6 Homework Manager
- Route: `/teacher/homework`
- Header: `#BE185D → #DB2777`, "+ Create" button
- Tab Bar: Active (3), Submissions, Graded
- Active cards: Class, title, submission count, Review/Edit/Remind buttons
- Create Assignment Modal: title, class, due date, max marks, instructions, file attach
- Submissions Tab: Overview cards
- Graded Tab: Completed assignments

### 9.7 Review Submissions
- Route: `/teacher/review-submissions`
- Header: `#9F1239 → #E11D48`
- Chip filters: All, Pending, Graded, Flagged
- Student submission tiles with status badges
- AI Grading Assist card with "Run AI Auto-grade" button
- Click → Grading Screen

### 9.8 Grading Screen
- Route: `/teacher/grading`
- Header: Student name, "1/32" progress
- Submission viewer (PDF/image)
- AI Feedback card with suggested score
- Score input field
- Teacher remarks textarea
- "Save & Next", "Message" buttons

### 9.9 Teacher Exams
- Route: `/teacher/exams`
- Header: `#134E4A → #0F766E`, "+ Create"
- Tab Bar: Upcoming, Past, Results
- Upcoming: Draft exams, published exams, edit/publish buttons
- AI Question Generator card
- Past: Historical exams with grading button
- Results: Performance analytics

### 9.10 Paper Builder
- Route: `/teacher/paper-builder`
- White background with dark header
- Progress card (total marks, question count)
- AI Magic Generator: textarea + "Magic Generate" button
- Question blocks: Objective (with options) or Subjective
- "+ Objective" / "+ Subjective" buttons
- Draft / Publish buttons

### 9.11 Gradebook
- Route: `/teacher/gradebook`
- Header: `#4F46E5 → #6366F1`, "Gradebook Summary", year picker, settings
- Category chips: Class Test, Lab Test, Assignment, Mid-Term, End-Term, Other, Overall
- Stats tiles: Class Avg, Attendance, Proficiency
- Performance Matrix: 5 progress bars with dynamic labels
- Academic Ledger: Student table with marks, grades, trends
- AI Performance Report card
- Export Report Cards button

### 9.12 Grading Configuration
- Route: `/teacher/grading-config`
- Dark header with "Grading Policy"
- Circular gauge showing total allocation (must = 100%)
- Weightage cards with sliders and +/- buttons:
  - Mid-Term (30%), Final-Term (40%), Attendance (5%), Assignments (10%), Class Tests (10%), Labs (5%)
- "Apply Grading Policy" button

### 9.13 Student Directory
- Route: `/teacher/student-directory`
- Header: `#92400E → #D97706`
- Search bar
- Class filter chips: All (133), X-A (45), X-B (48), IX-A (40)
- Student cards: avatar, name, class, call/message buttons
- At-risk students highlighted in red

### 9.14 Teacher Leave
- Route: `/teacher/leave`
- Header: `#14532D → #16A34A`
- Quota dashboard: 12 Total, 4 Used, 8 Remaining
- Leave history cards with status badges
- Apply Leave Modal: type, dates, reason

### 9.15 Teacher Notices
- Route: `/teacher/notices`
- Header: `#92400E → #D97706`, "+ Create"
- Search bar
- Tab filters: All, My Notices, School, Drafts
- Notice cards with category badges, urgent markers
- Create Notice Modal: title, category, target audience, content, urgent toggle, schedule toggle, file attach, Publish/Draft buttons

### 9.16 Teacher Live Classes
- Route: `/teacher/live-classes`
- Header: `#9F1239 → #E11D48`
- Tab: Live Now, Upcoming, Drafts, Archive
- Live Now card: pulsing badge, viewer count, Join/End buttons
- Upcoming cards: Go Live Early, Edit, Delete
- Draft cards: Publish, Delete
- Archive cards: Recording tiles with watch replay
- FAB: "🎥" → Schedule Class Modal

### 9.17 Teacher Live Session (YouTube-Style)
- Route: `/teacher/live-session`
- Full screen dark background
- Camera preview area
- Top bar: LIVE badge, viewer count, REC indicator, title, timer
- Chat overlay: student messages with Reply button
- Floating toolbar: Mute, Video, Screen Share, Chat, More, END
- Reply bar: replying to student, input, send

### 9.18 Live Playback
- Route: `/teacher/live-playback`
- YouTube-style video player
- Progress bar, play/pause controls
- Video info: title, views, date
- Live Chat Replay section

### 9.19 Study Materials
- Route: `/teacher/materials`
- Header: `#065F46 → #059669`, "📤 Upload"
- Filter chips: All, Notes, PPTs, Videos, Worksheets
- Material cards: icon, title, attachments, class, date, edit/delete
- Upload Modal: title, description, type, class, file attach, save

### 9.20 Salary & Payslips
- Route: `/teacher/salary`
- Header: `#065F46 → #059669`
- Net salary card: ₹68,450, credited date
- Breakup table: Basic, HRA, DA, Special Allowance, PF, TDS, PT
- Download Payslip button

### 9.21 Teacher Profile
- Route: `/teacher/profile`
- Header: `#0C4A6E → #0EA5E9`
- Avatar, name, designation, employee ID
- Stats: 12 Yrs Exp, 133 Students, 4.8⭐ Rating
- Info sections: email, phone, gender, parents, qualification, school, joined date
- Quick links: Salary, Leave, Settings, Logout
- Edit Profile Modal: full form with document upload

### 9.22-9.29 Remaining Teacher Screens
- Notifications, Settings, Exam Attendance, Grade Student Response — follow the same specification pattern.

---

## 10. SHARED REUSABLE COMPONENTS

### 10.1 Role-Aware Components
```dart
class AppBottomNav extends StatelessWidget {
  final String role; // 'student' or 'teacher'
  final int currentIndex;
  final Function(int) onTap;
  
  // Student items: Home, Courses, Results, Achieve, Profile
  // Teacher items: Home, Classes, Exams, Messages, Profile
}

class AppHeader extends StatelessWidget {
  final String title;
  final String role; // determines gradient colors
  final Widget? trailing;
  final bool showBackButton;
}

class AIFab extends StatelessWidget {
  final String role; // determines gradient colors
  // Student: purple gradient, Teacher: cyan gradient
}

class AppButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final AppButtonVariant variant;
  final String? role; // adjusts primary color
}
```

### 10.2 All Shared Components
Same 14+ components as before (AppCard, AppChip, AppBadge, AppModal, AppToggle, AITag, SectionHeader, CircularProgress, ResponsiveLayout, etc.) — with `role` parameter added where color needs to adapt.

---

## 11. RESPONSIVE DESIGN STRATEGY

Same as before:
- Mobile (<600dp): Single column, bottom nav, 4-col grid
- Tablet (600-900dp): 2 columns, 6-col grid
- Desktop (>900dp): Sidebar, 3-4 columns, 8-col grid

---

## 12. NAVIGATION & ROUTING

Use GoRouter with role-based routes (see Section 7.3). After login:
- Student → `/student/dashboard`
- Teacher → `/teacher/dashboard`

---

## 13. DEPENDENCIES

```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  provider: ^6.1.1
  flutter_riverpod: ^2.4.9
  go_router: ^13.0.0
  supabase_flutter: ^2.3.2
  dio: ^5.4.0
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  flutter_map: ^6.1.0
  latlong2: ^0.9.1
  langchain: ^0.5.0
  langchain_openai: ^0.5.0
  flutter_animate: ^4.3.0
  lottie: ^3.0.0
  shimmer: ^3.0.0
  cached_network_image: ^3.3.1
  fl_chart: ^0.66.0
  pdf: ^3.10.7
  printing: ^5.12.0
  file_picker: ^6.1.1
  image_picker: ^1.0.7
  path_provider: ^2.1.2
  url_launcher: ^6.2.3
  flutter_local_notifications: ^17.0.0
  local_auth: ^2.1.8
  mobile_scanner: ^4.0.0
  intl: ^0.19.0
  uuid: ^4.2.2
  equatable: ^2.0.5
  json_annotation: ^4.8.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.1
  build_runner: ^2.4.8
  json_serializable: ^6.7.1
  hive_generator: ^2.0.1
  mockito: ^5.4.4
```

---

## 14. DEVELOPMENT PHASES

### Phase 1: Foundation (Week 1-2)
- Project setup, LOCAL Supabase, fonts, theme (dual)
- All shared widgets with role awareness
- SQL migration, RLS policies
- GoRouter with role-based routes

### Phase 2: Shared + Auth (Week 3-4)
- Splash (dual theme), Login (role selector)
- Role detection and routing
- Settings, AI Chat

### Phase 3: Student Portal (Week 5-8)
- Dashboard, Timetable, Results, Courses, Attendance
- Homework, Exams (incl. exam engine), Fees
- Notices, Events, Transport, Library

### Phase 4: Teacher Portal (Week 9-12)
- Dashboard, My Classes, Class Detail
- Attendance marking, Homework Manager
- Paper Builder, Exams, Gradebook
- Student Directory, Notices, Live Classes

### Phase 5: Communication (Week 13-14)
- Messaging, Chat Detail, Create Group
- Live Classes (teacher session + student viewing)
- Notifications

### Phase 6: Advanced Features (Week 15-16)
- Leave (student + teacher)
- Salary, Materials, Grading Config
- Achievements, Leaderboard

### Phase 7: AI Integration (Week 17-18)
- LangChain setup (40 tools total)
- AI Chat for both roles
- AI insights across all screens
- Auto-grading, question generation

### Phase 8: Polish & Testing (Week 19-22)
- Responsive testing (all platforms)
- Performance optimization
- Animation polish
- Error handling, offline support
- Final QA

---

## 15. CRITICAL IMPLEMENTATION NOTES

1. **LOCAL Supabase**: Always use `http://127.0.0.1:54321` for development. For production, switch to hosted Supabase URL.

2. **Role Detection**: After login, check `profiles.role` and route accordingly. Store role in `RoleProvider`.

3. **Dual Theming**: Every shared component must accept a `role` parameter to adapt colors. Use `StudentColors` or `TeacherColors` based on role.

4. **Teacher Dashboard Stats**: 156 Students, 6 Classes, 8 Pending — these come from real database queries.

5. **Paper Builder**: The AI question generator should use LangChain to generate questions based on subject, difficulty, and marks distribution.

6. **Auto-Grading**: AI can auto-grade MCQs by comparing `selected_answer` with `correct_answer`. For subjective, provide suggested score for teacher review.

7. **Exam Integrity**: Lock screen, track focus changes, auto-submit on timer expiry.

8. **Real-time**: Use Supabase Realtime for bus tracking, chat, live classes, notifications, exam timer sync.

9. **Offline**: Cache timetable, homework, profile using Hive.

10. **File Uploads**: Use Supabase Storage buckets: `homework-files`, `documents`, `exam-answers`, `chat-attachments`, `study-materials`.

11. **Teacher-Specific Gamification**: Teachers earn ratings from students, track class performance improvements.

12. **Grading Policy**: Store weightage in `grading_policies` table. Recalculate all student totals when policy changes.

13. **Web-Specific**: Use `HtmlElementView` for video, enable URL strategy, support deep linking.

14. **Animation Durations**: Screen transitions 400ms, modals 300ms, AI FAB pulse 3000ms, typing dots 1200ms, confetti 3000ms.

15. **Image Assets**: Use emojis as placeholders. Replace with SVG icons for production.