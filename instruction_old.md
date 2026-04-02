# EduSHAMIIT AI Academic App — Complete Flutter Development Guide

> **PURPOSE**: This document is a SELF-CONTAINED BLUEPRINT. Any AI model or developer reading ONLY this file should be able to build the exact EduSHAMIIT AI Academic App from scratch — a UNIFIED Flutter application for BOTH Students and Teachers, matching the provided HTML mockups pixel-perfectly across all 6 platforms (Web, Android, iOS, macOS, Linux, Windows).

> **PROJECT NAME**: EduSHAMIIT AI
> **PACKAGE NAME**: com.shamiit.edu
> **AI AGENT NAME**: Shami
> **BACKEND**: LOCAL Supabase (running on developer's machine)
> **ARCHITECTURE**: Single app with role-based routing (Student vs Teacher)

---

## 📋 TABLE OF CONTENTS

1. [Project Overview and Architecture](#1-project-overview-and-architecture)
2. [Step-by-Step Setup Guide (LOCAL SUPABASE)](#2-step-by-step-setup-guide)
3. [Design System — Dual Theme](#3-design-system--dual-theme)
4. [Complete Database Schema (SQL)](#4-complete-database-schema-sql)
5. [AI Functions (LangChain Tool Calling) — Deep Detail](#5-ai-functions-langchain-tool-calling--deep-detail)
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

EduSHAMIIT AI is a unified AI-powered Academic App built with Flutter. It serves BOTH students and teachers through a single codebase. After login, the app detects the user's role and routes them to their respective portal. The shared screens (Splash, Login, Chat Detail, Settings) use a unified design with role-specific theming. The AI assistant agent is named **Shami** and provides intelligent assistance using LangChain tool calling with 40 tools total (20 for students, 20 for teachers).

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
│                    │    LangChain AI (Shami)   │                 │
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

Add to pubspec.yaml:

```yaml
flutter:
  uses-material-design: true
  fonts:
    - family: Outfit
      fonts:
        - asset: assets/fonts/Outfit-Light.ttf
          weight: 300
        - asset: assets/fonts/Outfit-Regular.ttf
          weight: 400
        - asset: assets/fonts/Outfit-Medium.ttf
          weight: 500
        - asset: assets/fonts/Outfit-SemiBold.ttf
          weight: 600
        - asset: assets/fonts/Outfit-Bold.ttf
          weight: 700
        - asset: assets/fonts/Outfit-ExtraBold.ttf
          weight: 800
        - asset: assets/fonts/Outfit-Black.ttf
          weight: 900
    - family: DM Sans
      fonts:
        - asset: assets/fonts/DMSans-Light.ttf
          weight: 300
        - asset: assets/fonts/DMSans-Regular.ttf
          weight: 400
        - asset: assets/fonts/DMSans-Medium.ttf
          weight: 500
        - asset: assets/fonts/DMSans-SemiBold.ttf
          weight: 600
        - asset: assets/fonts/DMSans-Bold.ttf
          weight: 700
```

### Step 7: Configure LOCAL Supabase

```dart
// lib/core/services/supabase_service.dart
import 'package:supabase_flutter/supabase_flutter.dart';

class SupabaseService {
  // LOCAL Supabase URLs (from `supabase start` output)
  static const String supabaseUrl = 'http://127.0.0.1:54321';
  static const String supabaseAnonKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZS1kZW1vIiwicm9sZSI6ImFub24iLCJleHAiOjE5ODM4MTI5OTZ9.CRXP1A7WOeoJeXxjNni43kdQwgnWNReilDMblYTn_I0';

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

  // Subject background colors
  static const Color mathBg = Color(0xFFEEF2FF);
  static const Color physicsBg = Color(0xFFEFF6FF);
  static const Color chemistryBg = Color(0xFFFEF3C7);
  static const Color englishBg = Color(0xFFECFDF5);
  static const Color historyBg = Color(0xFFF5F3FF);
  static const Color computerBg = Color(0xFFECFDF5);

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
  // Primary
  static const Color primary = Color(0xFF0EA5E9);
  static const Color primaryDeep = Color(0xFF0369A1);
  static const Color primaryLight = Color(0xFFE0F2FE);

  // Accents
  static const Color accent = Color(0xFF06B6D4);
  static const Color accent2 = Color(0xFFF59E0B);
  static const Color accent3 = Color(0xFF10B981);
  static const Color accent4 = Color(0xFFEF4444);
  static const Color accent5 = Color(0xFF8B5CF6);

  // Backgrounds
  static const Color background = Color(0xFFF0F7FF);
  static const Color surface = Color(0xFFFFFFFF);
  static const Color card = Color(0xFFFAFAFA);

  // Text
  static const Color text = Color(0xFF0F172A);
  static const Color text2 = Color(0xFF475569);
  static const Color text3 = Color(0xFF94A3B8);

  // Borders
  static const Color border = Color(0xFFE2E8F0);

  // Status
  static const Color success = Color(0xFF059669);
  static const Color successBg = Color(0xFFECFDF5);
  static const Color warning = Color(0xFFD97706);
  static const Color warningBg = Color(0xFFFEF3C7);
  static const Color error = Color(0xFFEF4444);
  static const Color errorBg = Color(0xFFFEF2F2);

  // Screen-specific gradients (from teacher HTML)
  static const Color dashboardGradient1 = Color(0xFF0C4A6E);
  static const Color dashboardGradient2 = Color(0xFF0EA5E9);
  static const Color myClassesGradient1 = Color(0xFF0C4A6E);
  static const Color myClassesGradient2 = Color(0xFF0EA5E9);
  static const Color classDetailGradient1 = Color(0xFF0C4A6E);
  static const Color classDetailGradient2 = Color(0xFF0EA5E9);
  static const Color attendanceGradient1 = Color(0xFF065F46);
  static const Color attendanceGradient2 = Color(0xFF059669);
  static const Color timetableGradient1 = Color(0xFF1E40AF);
  static const Color timetableGradient2 = Color(0xFF3B82F6);
  static const Color homeworkGradient1 = Color(0xFFBE185D);
  static const Color homeworkGradient2 = Color(0xFFDB2777);
  static const Color reviewSubGradient1 = Color(0xFF9F1239);
  static const Color reviewSubGradient2 = Color(0xFFE11D48);
  static const Color examsGradient1 = Color(0xFF134E4A);
  static const Color examsGradient2 = Color(0xFF0F766E);
  static const Color gradebookGradient1 = Color(0xFF4F46E5);
  static const Color gradebookGradient2 = Color(0xFF6366F1);
  static const Color studentDirGradient1 = Color(0xFF92400E);
  static const Color studentDirGradient2 = Color(0xFFD97706);
  static const Color leaveGradient1 = Color(0xFF14532D);
  static const Color leaveGradient2 = Color(0xFF16A34A);
  static const Color noticesGradient1 = Color(0xFF92400E);
  static const Color noticesGradient2 = Color(0xFFD97706);
  static const Color liveClassesGradient1 = Color(0xFF9F1239);
  static const Color liveClassesGradient2 = Color(0xFFE11D48);
  static const Color materialsGradient1 = Color(0xFF065F46);
  static const Color materialsGradient2 = Color(0xFF059669);
  static const Color salaryGradient1 = Color(0xFF065F46);
  static const Color salaryGradient2 = Color(0xFF059669);
  static const Color profileGradient1 = Color(0xFF0C4A6E);
  static const Color profileGradient2 = Color(0xFF0EA5E9);
  static const Color settingsGradient1 = Color(0xFF1E293B);
  static const Color settingsGradient2 = Color(0xFF0F172A);
  static const Color messagingGradient1 = Color(0xFF4F46E5);
  static const Color messagingGradient2 = Color(0xFF7C3AED);
  static const Color notificationsGradient1 = Color(0xFF0C4A6E);
  static const Color notificationsGradient2 = Color(0xFF0EA5E9);
  static const Color splashGradient1 = Color(0xFF0C4A6E);
  static const Color splashGradient2 = Color(0xFF0369A1);
  static const Color splashGradient3 = Color(0xFF0EA5E9);

  // Quick access grid colors (from teacher HTML)
  static const Color qaMyClasses = Color(0xFFE0F2FE);
  static const Color qaAttendance = Color(0xFFECFDF5);
  static const Color qaHomework = Color(0xFFFDF2F8);
  static const Color qaExams = Color(0xFFFEF3C7);
  static const Color qaGradebook = Color(0xFFFAF5FF);
  static const Color qaStudents = Color(0xFFF0FDF4);
  static const Color qaTimetable = Color(0xFFEEF2FF);
  static const Color qaNotices = Color(0xFFFEF2F2);
  static const Color qaLeave = Color(0xFFF0FDF4);
  static const Color qaLiveClass = Color(0xFFFDF2F8);
  static const Color qaMaterials = Color(0xFFECFDF5);
  static const Color qaSalary = Color(0xFFEFF6FF);
}
```

### 3.3 Spacing System

```dart
// lib/core/constants/app_dimensions.dart
class AppDimensions {
  // Padding
  static const double paddingXS = 4.0;
  static const double paddingS = 8.0;
  static const double paddingM = 12.0;
  static const double paddingL = 14.0;
  static const double paddingXL = 18.0;
  static const double paddingXXL = 24.0;

  // Border Radius
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
  static const double radiusAvatar = 50.0;

  // Component Sizes
  static const double buttonHeight = 48.0;
  static const double inputHeight = 44.0;
  static const double fabSize = 46.0;
  static const double bottomNavHeight = 60.0;
  static const double statusBarHeight = 44.0;
  static const double headerHeight = 56.0;
  static const double toggleWidth = 36.0;
  static const double toggleHeight = 20.0;
  static const double toggleKnob = 16.0;
}
```

### 3.4 Typography

```dart
// lib/core/constants/app_fonts.dart
class AppFonts {
  static const String heading = 'Outfit';
  static const String body = 'DM Sans';
}
```

### 3.5 Shared Gradients

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

  static const LinearGradient studentDashboard = LinearGradient(
    colors: [Color(0xFF4F46E5), Color(0xFF302B63)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );

  static const LinearGradient teacherDashboard = LinearGradient(
    colors: [Color(0xFF0C4A6E), Color(0xFF0EA5E9)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );

  static const LinearGradient success = LinearGradient(
    colors: [Color(0xFF059669), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );

  static const LinearGradient danger = LinearGradient(
    colors: [Color(0xFFEF4444), Color(0xFFDC2626)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );

  static const LinearGradient userChatBubble = LinearGradient(
    colors: [Color(0xFF4F46E5), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
}
```

### 3.6 Shared Shadows

```dart
// lib/core/constants/app_shadows.dart
import 'package:flutter/material.dart';

class AppShadows {
  static List<BoxShadow> card = [
    BoxShadow(
      color: Colors.black.withOpacity(0.05),
      blurRadius: 12,
      offset: const Offset(0, 2),
    ),
  ];

  static List<BoxShadow> elevated = [
    BoxShadow(
      color: const Color(0xFF4F46E5).withOpacity(0.4),
      blurRadius: 20,
      offset: const Offset(0, 6),
    ),
  ];

  static List<BoxShadow> teacherElevated = [
    BoxShadow(
      color: const Color(0xFF0EA5E9).withOpacity(0.4),
      blurRadius: 20,
      offset: const Offset(0, 6),
    ),
  ];

  static List<BoxShadow> fab = [
    BoxShadow(
      color: const Color(0xFF4F46E5).withOpacity(0.5),
      blurRadius: 20,
      offset: const Offset(0, 4),
    ),
  ];

  static List<BoxShadow> modal = [
    BoxShadow(
      color: Colors.black.withOpacity(0.1),
      blurRadius: 16,
      offset: const Offset(0, -4),
    ),
  ];
}
```

### 3.7 Theme Configuration

```dart
// lib/core/theme/student_theme.dart
import 'package:flutter/material.dart';
import '../constants/student_colors.dart';
import '../constants/app_fonts.dart';
import '../constants/app_dimensions.dart';

class StudentTheme {
  static ThemeData lightTheme = ThemeData(
    useMaterial3: true,
    brightness: Brightness.light,
    primaryColor: StudentColors.primary,
    scaffoldBackgroundColor: StudentColors.scaffoldBackground,
    fontFamily: AppFonts.body,
    colorScheme: ColorScheme.light(
      primary: StudentColors.primary,
      secondary: StudentColors.accent,
      error: StudentColors.error,
      surface: StudentColors.surface,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: StudentColors.surface,
      elevation: 0,
      titleTextStyle: TextStyle(
        fontFamily: AppFonts.heading,
        fontSize: 17,
        fontWeight: FontWeight.w800,
        color: StudentColors.text,
      ),
      iconTheme: const IconThemeData(color: StudentColors.text),
    ),
    cardTheme: CardTheme(
      color: StudentColors.surface,
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusCard),
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: const Color(0xFFF8FAFC),
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: StudentColors.border, width: 1.5),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: StudentColors.border, width: 1.5),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: StudentColors.primary, width: 1.5),
      ),
      contentPadding: const EdgeInsets.symmetric(
        horizontal: AppDimensions.paddingL,
        vertical: AppDimensions.paddingM,
      ),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: StudentColors.primary,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(vertical: 14),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(AppDimensions.radiusXXL),
        ),
        textStyle: TextStyle(
          fontFamily: AppFonts.heading,
          fontSize: 15,
          fontWeight: FontWeight.w700,
        ),
      ),
    ),
  );
}

// lib/core/theme/teacher_theme.dart
class TeacherTheme {
  static ThemeData lightTheme = ThemeData(
    useMaterial3: true,
    brightness: Brightness.light,
    primaryColor: TeacherColors.primary,
    scaffoldBackgroundColor: const Color(0xFFF0F7FF),
    fontFamily: AppFonts.body,
    colorScheme: ColorScheme.light(
      primary: TeacherColors.primary,
      secondary: TeacherColors.accent,
      error: TeacherColors.error,
      surface: TeacherColors.surface,
    ),
    // Same structure as StudentTheme but with TeacherColors
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

### 4.2 Core Tables

```sql
-- PROFILES (extends auth.users, supports both students and teachers)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  user_id TEXT UNIQUE NOT NULL,
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
  qualification TEXT,
  experience_years INT DEFAULT 0,
  rating DECIMAL(2,1) DEFAULT 0.0,
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
  free_period_activity TEXT,
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
  target_classes JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- EXAM QUESTIONS
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
  month TEXT NOT NULL,
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

### 4.7 Achievement and Gamification Tables

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

### 4.12 Settings and Documents

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

### 4.14 AI Chat History

```sql
CREATE TABLE ai_chat_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  session_id UUID,
  role TEXT CHECK (role IN ('user','assistant')),
  content TEXT NOT NULL,
  tool_calls JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.15 RLS Policies

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
ALTER TABLE ai_chat_history ENABLE ROW LEVEL SECURITY;

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
CREATE POLICY "Anyone can read library books" ON library_books FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus routes" ON bus_routes FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus stops" ON bus_stops FOR SELECT USING (true);
CREATE POLICY "Anyone can read bus locations" ON bus_locations FOR SELECT USING (true);
CREATE POLICY "Anyone can read live classes" ON live_classes FOR SELECT USING (true);
CREATE POLICY "Anyone can read study materials" ON study_materials FOR SELECT USING (true);
CREATE POLICY "Teachers can manage materials" ON study_materials FOR ALL USING (auth.uid() = teacher_id);
```

### 4.16 Enable Realtime

```sql
ALTER PUBLICATION supabase_realtime ADD TABLE bus_locations;
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE live_classes;
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;
ALTER PUBLICATION supabase_realtime ADD TABLE exam_sessions;
ALTER PUBLICATION supabase_realtime ADD TABLE live_class_comments;
ALTER PUBLICATION supabase_realtime ADD TABLE attendance;
```

### 4.17 Seed Data

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
  ('18-Day Streak', 'Logged in 18 consecutive days', '🔥', '["#60A5FA","#3B82F6"]', 300, 'Maintain login streak', 'uncommon'),
  ('Zero Late Submissions', 'All homework on time this term', '✅', '["#86EFAC","#22C55E"]', 200, 'Submit all homework before deadline', 'common');
```

---

## 5. AI FUNCTIONS (LANGCHAIN TOOL CALLING) — DEEP DETAIL

### 5.1 Overview

The EduSHAMIIT AI app uses LangChain Dart for AI-powered assistance. The AI agent is named **Shami** and provides intelligent assistance to both students and teachers. There are 40 tools total: 20 for students and 20 for teachers. Each tool is a Dart class that queries Supabase for real-time data and returns formatted results to the AI agent.

### 5.2 Student AI Tools (20 tools)

| Tool Number | Tool Name           | Purpose                                               | When Shami Uses It               | Example Student Query               |
| ----------- | ------------------- | ----------------------------------------------------- | -------------------------------- | ----------------------------------- |
| 1           | get_timetable       | Fetch class schedule for a specific day or whole week | Student asks about classes       | "What classes do I have today?"     |
| 2           | get_homework        | Fetch pending, submitted, or graded homework          | Student asks about homework      | "What homework is due?"             |
| 3           | get_fee_status      | Check fee balance and payment history                 | Student asks about fees          | "How much fees do I owe?"           |
| 4           | get_attendance      | Get attendance percentage overall and per subject     | Student asks about attendance    | "What is my attendance?"            |
| 5           | get_exam_info       | Get upcoming exam schedule and details                | Student asks about exams         | "When is my next exam?"             |
| 6           | get_bus_location    | Real-time bus location and ETA                        | Student asks about transport     | "Where is my bus right now?"        |
| 7           | get_performance     | Academic performance summary                          | Student asks about grades        | "How am I doing in studies?"        |
| 8           | generate_study_plan | Create personalized study plan                        | Student asks for study help      | "Help me prepare for my Math exam"  |
| 9           | explain_concept     | Explain an academic concept                           | Student asks for explanation     | "Explain integration by parts"      |
| 10          | get_notifications   | Recent alerts and notifications                       | Student asks about notifications | "Any new notifications?"            |
| 11          | get_library_status  | Borrowed books and due dates                          | Student asks about library       | "What books do I have borrowed?"    |
| 12          | get_achievements    | Badges, XP, and rankings                              | Student asks about achievements  | "What badges have I earned?"        |
| 13          | get_events          | Upcoming school events                                | Student asks about events        | "What events are coming up?"        |
| 14          | get_leaderboard     | Class and school rankings                             | Student asks about rankings      | "What is my class rank?"            |
| 15          | submit_homework     | Submit homework assignment                            | Student wants to submit          | "Submit my Math homework"           |
| 16          | apply_leave         | Apply for leave                                       | Student wants to apply leave     | "I need sick leave tomorrow"        |
| 17          | get_notices         | School notices and announcements                      | Student asks about notices       | "Any new announcements?"            |
| 18          | answer_general      | General school information Q&A                        | Student asks general questions   | "What is the school timing?"        |
| 19          | generate_practice   | Generate practice questions                           | Student wants practice           | "Give me Physics practice problems" |
| 20          | get_live_class      | Live class status and schedule                        | Student asks about live classes  | "Is there a live class now?"        |

### 5.3 Teacher AI Tools (20 tools)

| Tool Number | Tool Name                 | Purpose                                    | When Shami Uses It               | Example Teacher Query                      |
| ----------- | ------------------------- | ------------------------------------------ | -------------------------------- | ------------------------------------------ |
| 1           | get_class_students        | List all students in a class               | Teacher asks about students      | "Show me X-A students"                     |
| 2           | get_class_performance     | Class performance statistics               | Teacher asks about performance   | "How is X-A performing?"                   |
| 3           | get_at_risk_students      | Identify struggling students               | Teacher asks about weak students | "Who needs extra help?"                    |
| 4           | generate_questions        | Generate exam questions                    | Teacher needs questions          | "Create 5 MCQs on Calculus"                |
| 5           | generate_lesson_plan      | Create a lesson plan                       | Teacher needs lesson plan        | "Make a lesson plan for Integration"       |
| 6           | auto_grade_homework       | Auto-grade homework submissions            | Teacher wants to grade           | "Grade the pending homework"               |
| 7           | get_submission_status     | Check homework submission stats            | Teacher asks about submissions   | "How many students submitted?"             |
| 8           | generate_remedial_plan    | Create remedial sessions for weak students | Teacher wants remedial plan      | "Make a plan for weak students"            |
| 9           | create_notice             | Draft and create a notice                  | Teacher wants to create notice   | "Draft a notice about exam dates"          |
| 10          | get_attendance_stats      | Attendance analytics and trends            | Teacher asks about attendance    | "Show attendance trends for X-A"           |
| 11          | get_teacher_schedule      | Teacher's daily timetable                  | Teacher asks about schedule      | "What is my schedule today?"               |
| 12          | get_pending_tasks         | Pending reviews and tasks                  | Teacher asks about tasks         | "What needs my attention?"                 |
| 13          | generate_report           | Generate class performance report          | Teacher wants a report           | "Create X-A performance report"            |
| 14          | explain_pedagogy          | Teaching strategies and methods            | Teacher asks for teaching help   | "How to teach fractions to weak students?" |
| 15          | get_leave_balance         | Leave status and balance                   | Teacher asks about leave         | "How many leaves do I have remaining?"     |
| 16          | get_salary_info           | Salary details and breakup                 | Teacher asks about salary        | "Show my salary breakup for March"         |
| 17          | get_parent_communications | Parent messages and concerns               | Teacher asks about parents       | "Any parent messages for me?"              |
| 18          | upload_material           | Upload study material                      | Teacher wants to upload          | "Upload notes for X-A"                     |
| 19          | get_exam_analytics        | Exam statistics and pass rates             | Teacher asks about exams         | "Show exam pass rates for X-A"             |
| 20          | answer_general            | General school information Q&A             | Teacher asks general questions   | "What is the school policy on attendance?" |

### 5.4 Complete Dart Implementation for All 20 Student Tools

```dart
// lib/core/services/ai/tools/student/get_timetable_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetTimetableTool extends Tool {
  GetTimetableTool() : super(
    name: 'get_timetable',
    description: 'Get the student timetable for a specific day or the whole week. '
        'Use this when the student asks about their schedule, classes, or what they have today/tomorrow. '
        'Input should be a day name like "monday", "tuesday", etc., or "week" for the full week.',
  );

  @override
  Future<String> runInternal(String input) async {
    final day = input.toLowerCase().trim();
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;

    if (userId == null) return 'Error: User not authenticated. Please log in again.';

    try {
      // Get student's class from profile
      final profile = await client
          .from('profiles')
          .select('class')
          .eq('id', userId)
          .single();
      final studentClass = profile['class'] as String;

      int? dayFilter;
      if (day == 'monday') dayFilter = 0;
      else if (day == 'tuesday') dayFilter = 1;
      else if (day == 'wednesday') dayFilter = 2;
      else if (day == 'thursday') dayFilter = 3;
      else if (day == 'friday') dayFilter = 4;
      else if (day == 'saturday') dayFilter = 5;

      var query = client
          .from('timetable')
          .select('*, subjects(name, icon, color, teacher_id)')
          .eq('class', studentClass)
          .order('start_time');

      if (dayFilter != null) {
        query = query.eq('day_of_week', dayFilter);
      }

      final response = await query;
      return _formatTimetable(response, day);
    } catch (e) {
      return 'Error fetching timetable: $e. Please try again later.';
    }
  }

  String _formatTimetable(List<dynamic> data, String day) {
    if (data.isEmpty) return 'No classes scheduled${day != 'week' ? ' for $day' : ''}. Enjoy your free time! 🎉';

    final buffer = StringBuffer();
    final days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
    int? currentDay;

    for (final item in data) {
      final dayOfWeek = item['day_of_week'] as int;
      if (currentDay != dayOfWeek) {
        currentDay = dayOfWeek;
        buffer.writeln('\n📅 ${days[dayOfWeek]}:');
      }

      if (item['is_break'] == true) {
        buffer.writeln('  ☕ ${item['break_name']} (${item['start_time']} - ${item['end_time']})');
      } else if (item['is_free_period'] == true) {
        buffer.writeln('  🆓 Free Period (${item['start_time']} - ${item['end_time']}) - ${item['free_period_activity'] ?? 'Study Hall'}');
      } else {
        final subject = item['subjects'];
        final icon = subject['icon'] ?? '📚';
        final name = subject['name'] ?? 'Unknown Subject';
        final room = item['room'] ?? 'TBD';
        buffer.writeln('  $icon $name - ${item['start_time']} to ${item['end_time']} (Room $room)');
      }
    }

    return buffer.toString();
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_homework_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetHomeworkTool extends Tool {
  GetHomeworkTool() : super(
    name: 'get_homework',
    description: 'Get homework assignments for the student. Can filter by status: '
        '"pending" for due homework, "submitted" for already submitted, "graded" for graded ones. '
        'Use this when the student asks about homework, assignments, or what is due.',
  );

  @override
  Future<String> runInternal(String input) async {
    final filter = input.toLowerCase().trim();
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;

    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client
          .from('profiles')
          .select('class')
          .eq('id', userId)
          .single();
      final studentClass = profile['class'] as String;

      // Get homework for student's class
      final homework = await client
          .from('homework')
          .select('*, subjects(name, icon)')
          .eq('target_class', studentClass)
          .eq('status', 'active')
          .order('due_date');

      // Get submissions for this student
      final submissions = await client
          .from('homework_submissions')
          .select('homework_id, status, grade, marks')
          .eq('student_id', userId);

      final submittedIds = submissions.map((s) => s['homework_id']).toSet();

      // Filter based on request
      List<dynamic> filtered;
      if (filter.contains('pending')) {
        filtered = homework.where((h) => !submittedIds.contains(h['id'])).toList();
      } else if (filter.contains('submitted')) {
        filtered = homework.where((h) {
          final sub = submissions.firstWhere(
            (s) => s['homework_id'] == h['id'],
            orElse: () => null,
          );
          return sub != null && sub['status'] == 'submitted';
        }).toList();
      } else if (filter.contains('graded')) {
        filtered = homework.where((h) {
          final sub = submissions.firstWhere(
            (s) => s['homework_id'] == h['id'],
            orElse: () => null,
          );
          return sub != null && sub['status'] == 'graded';
        }).toList();
      } else {
        filtered = homework;
      }

      return _formatHomework(filtered, submissions, submittedIds);
    } catch (e) {
      return 'Error fetching homework: $e';
    }
  }

  String _formatHomework(List<dynamic> homework, List<dynamic> submissions, Set<dynamic> submittedIds) {
    if (homework.isEmpty) return 'No homework found! Enjoy your free time! 🎉';

    final buffer = StringBuffer();
    int pendingCount = 0;

    for (final hw in homework) {
      final subject = hw['subjects'];
      final icon = subject?['icon'] ?? '📝';
      final title = hw['title'] ?? 'Untitled';
      final dueDate = hw['due_date'];
      final isSubmitted = submittedIds.contains(hw['id']);

      String status;
      if (isSubmitted) {
        final sub = submissions.firstWhere((s) => s['homework_id'] == hw['id'], orElse: () => null);
        if (sub != null && sub['status'] == 'graded') {
          status = '✅ Graded: ${sub['grade'] ?? 'N/A'} (${sub['marks'] ?? 'N/A'} marks)';
        } else {
          status = '📤 Submitted - Awaiting review';
        }
      } else {
        pendingCount++;
        final due = DateTime.tryParse(dueDate?.toString() ?? '');
        if (due != null && due.isBefore(DateTime.now())) {
          status = '🔴 OVERDUE!';
        } else if (due != null && due.difference(DateTime.now()).inDays <= 1) {
          status = '🟡 Due Soon';
        } else {
          status = '📋 Pending';
        }
      }

      buffer.writeln('$icon $title');
      buffer.writeln('   Subject: ${subject?['name'] ?? 'Unknown'}');
      buffer.writeln('   Due: $dueDate');
      buffer.writeln('   Status: $status');
      buffer.writeln('');
    }

    if (pendingCount > 0) {
      buffer.writeln('⚠️ You have $pendingCount pending assignment(s). Make sure to submit before the deadline!');
    }

    return buffer.toString();
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_fee_status_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetFeeStatusTool extends Tool {
  GetFeeStatusTool() : super(
    name: 'get_fee_status',
    description: 'Check the student fee balance, pending fees, and payment history. '
        'Use this when the student asks about fees, payments, or outstanding balance.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final fees = await client
          .from('fees')
          .select('*')
          .eq('student_id', userId)
          .order('due_date');

      final payments = await client
          .from('payments')
          .select('*')
          .eq('student_id', userId)
          .order('paid_at', ascending: false);

      double totalDue = 0;
      double totalPaid = 0;
      final pendingFees = <Map<String, dynamic>>[];

      for (final fee in fees) {
        final amount = (fee['amount'] as num).toDouble();
        final amountPaid = (fee['amount_paid'] as num?)?.toDouble() ?? 0;
        final status = fee['status'] as String;

        if (status == 'pending' || status == 'partial' || status == 'overdue') {
          totalDue += (amount - amountPaid);
          pendingFees.add(fee);
        }
        totalPaid += amountPaid;
      }

      final buffer = StringBuffer();
      buffer.writeln('💰 Fee Status Summary');
      buffer.writeln('');
      buffer.writeln('Total Outstanding: ₹${totalDue.toStringAsFixed(2)}');
      buffer.writeln('Total Paid This Year: ₹${totalPaid.toStringAsFixed(2)}');
      buffer.writeln('');

      if (pendingFees.isNotEmpty) {
        buffer.writeln('📋 Pending Fees:');
        for (final fee in pendingFees) {
          final feeType = fee['fee_type'] ?? 'Unknown';
          final amount = (fee['amount'] as num).toDouble();
          final amountPaid = (fee['amount_paid'] as num?)?.toDouble() ?? 0;
          final remaining = amount - amountPaid;
          final dueDate = fee['due_date'] ?? 'TBD';
          final status = fee['status'] as String;

          String statusEmoji = status == 'overdue' ? '🔴' : (status == 'partial' ? '🟡' : '📋');
          buffer.writeln('  $statusEmoji $feeType: ₹${remaining.toStringAsFixed(2)} remaining (Due: $dueDate)');
        }
      } else {
        buffer.writeln('✅ All fees are paid! Great job!');
      }

      if (payments.isNotEmpty) {
        buffer.writeln('\n📝 Recent Payments:');
        for (int i = 0; i < (payments.length > 3 ? 3 : payments.length); i++) {
          final p = payments[i];
          buffer.writeln('  ✅ ₹${(p['amount'] as num).toDouble().toStringAsFixed(2)} via ${p['payment_method']} - ${p['paid_at']}');
        }
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching fee status: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_attendance_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetAttendanceTool extends Tool {
  GetAttendanceTool() : super(
    name: 'get_attendance',
    description: 'Get the student attendance percentage, both overall and per subject. '
        'Use this when the student asks about their attendance, how many classes they attended, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final attendance = await client
          .from('attendance')
          .select('*, subjects(name, icon)')
          .eq('student_id', userId);

      if (attendance.isEmpty) return 'No attendance records found yet.';

      // Calculate overall
      int totalDays = attendance.length;
      int presentDays = attendance.where((a) => a['status'] == 'present').length;
      int absentDays = attendance.where((a) => a['status'] == 'absent').length;
      int lateDays = attendance.where((a) => a['status'] == 'late').length;
      double overallPct = totalDays > 0 ? (presentDays / totalDays) * 100 : 0;

      // Calculate per subject
      final subjectMap = <String, Map<String, int>>{};
      for (final a in attendance) {
        final subjectName = a['subjects']?['name'] ?? 'Unknown';
        subjectMap.putIfAbsent(subjectName, () => {'total': 0, 'present': 0});
        subjectMap[subjectName]!['total'] = subjectMap[subjectName]!['total']! + 1;
        if (a['status'] == 'present') {
          subjectMap[subjectName]!['present'] = subjectMap[subjectName]!['present']! + 1;
        }
      }

      final buffer = StringBuffer();
      buffer.writeln('📊 Attendance Summary');
      buffer.writeln('');
      buffer.writeln('Overall: ${overallPct.toStringAsFixed(1)}% ($presentDays/$totalDays days)');
      buffer.writeln('  ✅ Present: $presentDays days');
      buffer.writeln('  ❌ Absent: $absentDays days');
      buffer.writeln('  ⏰ Late: $lateDays days');
      buffer.writeln('');
      buffer.writeln('📚 Subject-wise:');
      subjectMap.forEach((subject, counts) {
        final pct = counts['total']! > 0 ? (counts['present']! / counts['total']!) * 100 : 0;
        String emoji = pct >= 90 ? '✅' : (pct >= 75 ? '⚠️' : '🔴');
        buffer.writeln('  $emoji $subject: ${pct.toStringAsFixed(0)}% (${counts['present']}/${counts['total']})');
      });

      if (overallPct < 85) {
        buffer.writeln('\n⚠️ Your attendance is below 85%. Please attend all upcoming classes to avoid shortage penalties.');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching attendance: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_exam_info_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetExamInfoTool extends Tool {
  GetExamInfoTool() : super(
    name: 'get_exam_info',
    description: 'Get upcoming exam schedule, dates, and details. '
        'Use this when the student asks about exams, when is the next exam, exam dates, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client
          .from('profiles')
          .select('class')
          .eq('id', userId)
          .single();
      final studentClass = profile['class'] as String;

      final exams = await client
          .from('exams')
          .select('*, subjects(name, icon)')
          .contains('target_classes', '["$studentClass"]')
          .gte('exam_date', DateTime.now().toIso8601String().split('T')[0])
          .order('exam_date')
          .limit(10);

      if (exams.isEmpty) return 'No upcoming exams found. Enjoy the break! 🎉';

      final buffer = StringBuffer();
      buffer.writeln('📝 Upcoming Exams:');
      buffer.writeln('');

      for (final exam in exams) {
        final subject = exam['subjects'];
        final icon = subject?['icon'] ?? '📝';
        final title = exam['title'] ?? 'Untitled Exam';
        final date = exam['exam_date'] ?? 'TBD';
        final time = exam['start_time'] ?? 'TBD';
        final duration = exam['duration_minutes'] ?? 0;
        final venue = exam['venue'] ?? 'TBD';
        final totalMarks = exam['total_marks'] ?? 100;
        final examType = exam['exam_type'] ?? 'offline';
        final category = exam['exam_category'] ?? '';

        buffer.writeln('$icon $title');
        buffer.writeln('   📅 Date: $date');
        buffer.writeln('   ⏰ Time: $time ($duration minutes)');
        buffer.writeln('   📍 Venue: $venue');
        buffer.writeln('   📊 Total Marks: $totalMarks');
        buffer.writeln('   📋 Type: ${examType.toUpperCase()} ${category.isNotEmpty ? '- $category' : ''}');
        buffer.writeln('');
      }

      // Calculate days until next exam
      final nextExam = exams.first;
      final nextDate = DateTime.tryParse(nextExam['exam_date']?.toString() ?? '');
      if (nextDate != null) {
        final daysLeft = nextDate.difference(DateTime.now()).inDays;
        buffer.writeln('⏰ Next exam is in $daysLeft day(s). Start preparing!');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching exam info: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_bus_location_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetBusLocationTool extends Tool {
  GetBusLocationTool() : super(
    name: 'get_bus_location',
    description: 'Get real-time bus location, ETA, and route information for the student. '
        'Use this when the student asks about their bus, where is the bus, bus timing, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      // Get student's transport assignment
      final transport = await client
          .from('student_transport')
          .select('*, bus_routes(*), bus_stops(*)')
          .eq('student_id', userId)
          .maybeSingle();

      if (transport == null) return 'You are not assigned to any bus route. Please contact the transport office.';

      final route = transport['bus_routes'];
      final stop = transport['bus_stops'];

      // Get real-time bus location
      final busLocation = await client
          .from('bus_locations')
          .select('*')
          .eq('route_id', route['id'])
          .order('recorded_at', ascending: false)
          .limit(1)
          .maybeSingle();

      final buffer = StringBuffer();
      buffer.writeln('🚌 Bus Tracking Information');
      buffer.writeln('');
      buffer.writeln('Route: ${route['route_name']}');
      buffer.writeln('Bus Number: ${route['bus_number']}');
      buffer.writeln('Driver: ${route['driver_name']}');
      buffer.writeln('Driver Phone: ${route['driver_phone']}');
      buffer.writeln('Your Stop: ${stop['stop_name']}');
      buffer.writeln('Seat Number: ${transport['seat_no'] ?? 'Not assigned'}');
      buffer.writeln('');

      if (busLocation != null) {
        final eta = busLocation['eta_minutes'];
        final speed = busLocation['speed'];
        buffer.writeln('📍 Live Location:');
        buffer.writeln('   ETA to your stop: ${eta ?? 'Calculating...'} minutes');
        if (speed != null) buffer.writeln('   Current Speed: ${speed} km/h');
        buffer.writeln('   Last updated: ${busLocation['recorded_at']}');
      } else {
        buffer.writeln('📍 Bus location data not available yet. The bus may not have started.');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching bus location: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_performance_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetPerformanceTool extends Tool {
  GetPerformanceTool() : super(
    name: 'get_performance',
    description: 'Get the student academic performance summary including scores, grades, rank, and trends. '
        'Use this when the student asks about their overall performance, grades, how they are doing, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      // Get profile info
      final profile = await client
          .from('profiles')
          .select('full_name, class, xp_points, learning_streak, best_streak')
          .eq('id', userId)
          .single();

      // Get latest results
      final results = await client
          .from('results')
          .select('*, subjects(name, icon)')
          .eq('student_id', userId)
          .order('created_at', ascending: false);

      // Get attendance
      final attendance = await client
          .from('attendance')
          .select('status')
          .eq('student_id', userId);

      final totalDays = attendance.length;
      final presentDays = attendance.where((a) => a['status'] == 'present').length;
      final attendancePct = totalDays > 0 ? (presentDays / totalDays * 100).toStringAsFixed(1) : 'N/A';

      // Calculate average score
      double totalScore = 0;
      int scoreCount = 0;
      final subjectScores = <String, List<double>>{};

      for (final r in results) {
        final obtained = (r['marks_obtained'] as num?)?.toDouble();
        final maxMarks = (r['max_marks'] as num?)?.toDouble();
        if (obtained != null && maxMarks != null && maxMarks > 0) {
          final pct = (obtained / maxMarks) * 100;
          totalScore += pct;
          scoreCount++;
          final subjectName = r['subjects']?['name'] ?? 'Unknown';
          subjectScores.putIfAbsent(subjectName, () => []);
          subjectScores[subjectName]!.add(pct);
        }
      }

      final avgScore = scoreCount > 0 ? (totalScore / scoreCount).toStringAsFixed(1) : 'N/A';

      final buffer = StringBuffer();
      buffer.writeln('📊 Academic Performance Summary for ${profile['full_name']}');
      buffer.writeln('Class: ${profile['class']}');
      buffer.writeln('');
      buffer.writeln('🏆 Overall Stats:');
      buffer.writeln('   Average Score: $avgScore%');
      buffer.writeln('   Attendance: $attendancePct%');
      buffer.writeln('   XP Points: ${profile['xp_points']}');
      buffer.writeln('   Current Streak: ${profile['learning_streak']} days');
      buffer.writeln('   Best Streak: ${profile['best_streak']} days');
      buffer.writeln('');
      buffer.writeln('📚 Subject-wise Performance:');
      subjectScores.forEach((subject, scores) {
        final subjectAvg = scores.isNotEmpty ? (scores.reduce((a, b) => a + b) / scores.length).toStringAsFixed(1) : 'N/A';
        final icon = scores.isNotEmpty ? (double.parse(subjectAvg) >= 90 ? '✅' : (double.parse(subjectAvg) >= 75 ? '⚠️' : '🔴')) : '📋';
        buffer.writeln('   $icon $subject: $subjectAvg%');
      });

      return buffer.toString();
    } catch (e) {
      return 'Error fetching performance: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/generate_study_plan_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GenerateStudyPlanTool extends Tool {
  GenerateStudyPlanTool() : super(
    name: 'generate_study_plan',
    description: 'Generate a personalized study plan based on upcoming exams, weak areas, and performance data. '
        'Input should be the subject name or "all" for all subjects. '
        'Use this when the student asks for help preparing for exams or wants a study plan.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final subject = input.toLowerCase().trim();

      // Get upcoming exams
      final profile = await client.from('profiles').select('class').eq('id', userId).single();
      final studentClass = profile['class'] as String;

      final exams = await client
          .from('exams')
          .select('*, subjects(name)')
          .contains('target_classes', '["$studentClass"]')
          .gte('exam_date', DateTime.now().toIso8601String().split('T')[0])
          .order('exam_date')
          .limit(5);

      // Get weak areas from results
      final results = await client
          .from('results')
          .select('*, subjects(name)')
          .eq('student_id', userId)
          .order('created_at', ascending: false)
          .limit(20);

      final weakSubjects = <String>[];
      final subjectScores = <String, List<double>>{};
      for (final r in results) {
        final obtained = (r['marks_obtained'] as num?)?.toDouble();
        final maxMarks = (r['max_marks'] as num?)?.toDouble();
        if (obtained != null && maxMarks != null && maxMarks > 0) {
          final pct = (obtained / maxMarks) * 100;
          final name = r['subjects']?['name'] ?? '';
          subjectScores.putIfAbsent(name, () => []);
          subjectScores[name]!.add(pct);
        }
      }
      subjectScores.forEach((name, scores) {
        final avg = scores.reduce((a, b) => a + b) / scores.length;
        if (avg < 75) weakSubjects.add(name);
      });

      final buffer = StringBuffer();
      buffer.writeln('📋 Personalized Study Plan');
      buffer.writeln('');

      if (exams.isNotEmpty) {
        buffer.writeln('📝 Upcoming Exams:');
        for (final exam in exams) {
          final examDate = exam['exam_date'];
          final examTitle = exam['title'];
          final daysLeft = DateTime.tryParse(examDate ?? '')?.difference(DateTime.now()).inDays ?? 0;
          buffer.writeln('   📅 $examTitle - $examDate ($daysLeft days left)');
        }
        buffer.writeln('');
      }

      if (weakSubjects.isNotEmpty) {
        buffer.writeln('⚠️ Areas Needing Focus:');
        for (final ws in weakSubjects) {
          final avg = subjectScores[ws]!.reduce((a, b) => a + b) / subjectScores[ws]!.length;
          buffer.writeln('   🔴 $ws (Average: ${avg.toStringAsFixed(1)}%)');
        }
        buffer.writeln('');
      }

      buffer.writeln('💡 Recommended Study Schedule:');
      buffer.writeln('   Morning (6-8 AM): Revise weak topics');
      buffer.writeln('   Afternoon (2-4 PM): Practice problems');
      buffer.writeln('   Evening (7-9 PM): Review notes and formulas');
      buffer.writeln('');
      buffer.writeln('🎯 Tips:');
      buffer.writeln('   • Study in 25-minute blocks with 5-minute breaks (Pomodoro)');
      buffer.writeln('   • Focus on understanding concepts, not memorizing');
      buffer.writeln('   • Practice previous year questions');
      buffer.writeln('   • Take a full mock test every weekend');

      return buffer.toString();
    } catch (e) {
      return 'Error generating study plan: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/explain_concept_tool.dart
import 'package:langchain/langchain.dart';

class ExplainConceptTool extends Tool {
  ExplainConceptTool() : super(
    name: 'explain_concept',
    description: 'Explain an academic concept in simple terms with examples. '
        'Input should be the concept to explain. '
        'Use this when the student asks to explain something like integration, Newton laws, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    // This tool relies on the LLM's knowledge to explain concepts
    // It passes the concept to the agent which generates an explanation
    return 'Please explain the following academic concept in simple, easy-to-understand terms with examples: $input';
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_notifications_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetNotificationsTool extends Tool {
  GetNotificationsTool() : super(
    name: 'get_notifications',
    description: 'Get recent notifications and alerts for the student. '
        'Use this when the student asks about notifications, alerts, or what is new.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final notifications = await client
          .from('notifications')
          .select('*')
          .eq('user_id', userId)
          .order('created_at', ascending: false)
          .limit(10);

      if (notifications.isEmpty) return 'No new notifications. You are all caught up! ✅';

      final buffer = StringBuffer();
      buffer.writeln('🔔 Recent Notifications:');
      buffer.writeln('');

      int unreadCount = 0;
      for (final n in notifications) {
        final isRead = n['is_read'] as bool? ?? false;
        if (!isRead) unreadCount++;
        final prefix = isRead ? '  ' : '🔴';
        buffer.writeln('$prefix ${n['title']}');
        buffer.writeln('   ${n['body']}');
        buffer.writeln('   ${n['created_at']}');
        buffer.writeln('');
      }

      if (unreadCount > 0) {
        buffer.writeln('📬 You have $unreadCount unread notification(s).');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching notifications: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_library_status_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetLibraryStatusTool extends Tool {
  GetLibraryStatusTool() : super(
    name: 'get_library_status',
    description: 'Get the student library borrow status, currently borrowed books, and due dates. '
        'Use this when the student asks about library books, borrowed books, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final borrows = await client
          .from('library_borrows')
          .select('*, library_books(title, author, isbn)')
          .eq('student_id', userId)
          .eq('status', 'borrowed')
          .order('due_at');

      if (borrows.isEmpty) return 'You have no books currently borrowed from the library. 📚';

      final buffer = StringBuffer();
      buffer.writeln('📚 Currently Borrowed Books:');
      buffer.writeln('');

      for (final b in borrows) {
        final book = b['library_books'];
        final title = book?['title'] ?? 'Unknown';
        final author = book?['author'] ?? 'Unknown';
        final dueDate = b['due_at'] ?? 'TBD';
        final renewals = b['renewals_used'] ?? 0;
        final maxRenewals = b['max_renewals'] ?? 2;
        final fine = b['fine_amount'] ?? 0;

        final due = DateTime.tryParse(dueDate.toString());
        final isOverdue = due != null && due.isBefore(DateTime.now());

        buffer.writeln('📖 $title by $author');
        buffer.writeln('   Due: $dueDate ${isOverdue ? '⚠️ OVERDUE' : '✅'}');
        buffer.writeln('   Renewals: $renewals/$maxRenewals used');
        if (fine > 0) buffer.writeln('   Fine: ₹$fine');
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching library status: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_achievements_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetAchievementsTool extends Tool {
  GetAchievementsTool() : super(
    name: 'get_achievements',
    description: 'Get the student achievements, badges, XP points, and rankings. '
        'Use this when the student asks about their badges, XP, achievements, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client
          .from('profiles')
          .select('xp_points, learning_streak, best_streak')
          .eq('id', userId)
          .single();

      final achievements = await client
          .from('student_achievements')
          .select('*, achievements(name, description, icon, xp_reward, rarity)')
          .eq('student_id', userId)
          .order('earned_at', ascending: false);

      final buffer = StringBuffer();
      buffer.writeln('🏆 Achievements Summary');
      buffer.writeln('');
      buffer.writeln('⭐ Total XP: ${profile['xp_points']}');
      buffer.writeln('🔥 Current Streak: ${profile['learning_streak']} days');
      buffer.writeln('🏅 Best Streak: ${profile['best_streak']} days');
      buffer.writeln('');
      buffer.writeln('🎖️ Badges Earned (${achievements.length}):');
      buffer.writeln('');

      for (final a in achievements) {
        final achievement = a['achievements'];
        final icon = achievement?['icon'] ?? '🏅';
        final name = achievement?['name'] ?? 'Unknown';
        final desc = achievement?['description'] ?? '';
        final xp = achievement?['xp_reward'] ?? 0;
        final rarity = achievement?['rarity'] ?? 'common';
        buffer.writeln('  $icon $name (+$xp XP) [$rarity]');
        if (desc.isNotEmpty) buffer.writeln('     $desc');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching achievements: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_events_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetEventsTool extends Tool {
  GetEventsTool() : super(
    name: 'get_events',
    description: 'Get upcoming school events, competitions, and activities. '
        'Use this when the student asks about events, competitions, or what is happening at school.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    try {
      final events = await client
          .from('events')
          .select('*')
          .gte('event_date', DateTime.now().toIso8601String().split('T')[0])
          .order('event_date')
          .limit(10);

      if (events.isEmpty) return 'No upcoming events at the moment. Stay tuned! 📅';

      final buffer = StringBuffer();
      buffer.writeln('📅 Upcoming Events:');
      buffer.writeln('');

      for (final e in events) {
        buffer.writeln('🎉 ${e['title']}');
        buffer.writeln('   📅 ${e['event_date']} at ${e['event_time'] ?? 'TBD'}');
        buffer.writeln('   📍 ${e['venue'] ?? 'TBD'}');
        if (e['description'] != null) buffer.writeln('   📝 ${e['description']}');
        final participants = e['current_participants'] ?? 0;
        final maxParticipants = e['max_participants'];
        if (maxParticipants != null) {
          buffer.writeln('   👥 $participants/$maxParticipants registered');
        }
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching events: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_leaderboard_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetLeaderboardTool extends Tool {
  GetLeaderboardTool() : super(
    name: 'get_leaderboard',
    description: 'Get the class or school leaderboard rankings. '
        'Use this when the student asks about rankings, who is top of the class, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client
          .from('profiles')
          .select('class, xp_points')
          .eq('id', userId)
          .single();

      final studentClass = profile['class'] as String;

      // Get top students in class by XP
      final leaderboard = await client
          .from('profiles')
          .select('id, full_name, xp_points, learning_streak')
          .eq('class', studentClass)
          .eq('role', 'student')
          .order('xp_points', ascending: false)
          .limit(10);

      final buffer = StringBuffer();
      buffer.writeln('🏆 Class $studentClass Leaderboard:');
      buffer.writeln('');

      int rank = 1;
      for (final student in leaderboard) {
        final isMe = student['id'] == userId;
        final prefix = rank == 1 ? '🥇' : (rank == 2 ? '🥈' : (rank == 3 ? '🥉' : '$rank.'));
        final name = isMe ? '${student['full_name']} (You)' : student['full_name'];
        buffer.writeln('  $prefix $name - ${student['xp_points']} XP (🔥 ${student['learning_streak']} days)');
        rank++;
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching leaderboard: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/submit_homework_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class SubmitHomeworkTool extends Tool {
  SubmitHomeworkTool() : super(
    name: 'submit_homework',
    description: 'Submit a homework assignment on behalf of the student. '
        'Input should be the homework title or ID. '
        'Use this when the student says they want to submit homework.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client.from('profiles').select('class').eq('id', userId).single();
      final studentClass = profile['class'] as String;

      // Find matching homework
      final homework = await client
          .from('homework')
          .select('*, subjects(name)')
          .eq('target_class', studentClass)
          .ilike('title', '%$input%')
          .limit(1)
          .maybeSingle();

      if (homework == null) return 'Could not find homework matching "$input". Please check the title and try again.';

      // Check if already submitted
      final existing = await client
          .from('homework_submissions')
          .select('id')
          .eq('homework_id', homework['id'])
          .eq('student_id', userId)
          .maybeSingle();

      if (existing != null) return 'You have already submitted this homework. ✅';

      // Create submission
      await client.from('homework_submissions').insert({
        'homework_id': homework['id'],
        'student_id': userId,
        'submission_text': 'Submitted via Shami AI Assistant',
        'status': 'submitted',
      });

      return '✅ Homework "${homework['title']}" (${homework['subjects']['name']}) has been submitted successfully! Your teacher will review it shortly.';
    } catch (e) {
      return 'Error submitting homework: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/apply_leave_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class ApplyLeaveTool extends Tool {
  ApplyLeaveTool() : super(
    name: 'apply_leave',
    description: 'Apply for leave on behalf of the student. '
        'Input should include leave type, dates, and reason. '
        'Use this when the student says they need leave, sick leave, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      // Parse input for leave details (simple parsing)
      String leaveType = 'sick';
      if (input.toLowerCase().contains('casual')) leaveType = 'casual';
      else if (input.toLowerCase().contains('urgent')) leaveType = 'urgent';
      else if (input.toLowerCase().contains('family')) leaveType = 'family_event';

      final tomorrow = DateTime.now().add(const Duration(days: 1));
      final dayAfter = DateTime.now().add(const Duration(days: 2));

      await client.from('leave_applications').insert({
        'applicant_id': userId,
        'applicant_role': 'student',
        'leave_type': leaveType,
        'start_date': tomorrow.toIso8601String().split('T')[0],
        'end_date': dayAfter.toIso8601String().split('T')[0],
        'reason': input,
        'status': 'pending',
      });

      return '✅ Leave application submitted successfully!\n'
          'Type: ${leaveType.toUpperCase()}\n'
          'From: ${tomorrow.toIso8601String().split('T')[0]}\n'
          'To: ${dayAfter.toIso8601String().split('T')[0]}\n'
          'Status: PENDING\n'
          'Your class teacher will review and approve it.';
    } catch (e) {
      return 'Error applying for leave: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_notices_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetNoticesTool extends Tool {
  GetNoticesTool() : super(
    name: 'get_notices',
    description: 'Get recent school notices and announcements. '
        'Use this when the student asks about notices, announcements, circulars, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    try {
      final notices = await client
          .from('notices')
          .select('*')
          .eq('status', 'published')
          .order('published_at', ascending: false)
          .limit(10);

      if (notices.isEmpty) return 'No notices at the moment. 📢';

      final buffer = StringBuffer();
      buffer.writeln('📢 Recent Notices:');
      buffer.writeln('');

      for (final n in notices) {
        final isUrgent = n['is_urgent'] as bool? ?? false;
        final prefix = isUrgent ? '🚨' : '📋';
        final category = n['category'] ?? 'General';
        buffer.writeln('$prefix ${n['title']} [$category]');
        buffer.writeln('   ${n['content']}');
        buffer.writeln('   📅 ${n['published_at']}');
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching notices: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/student/answer_general_tool.dart
import 'package:langchain/langchain.dart';

class AnswerGeneralTool extends Tool {
  AnswerGeneralTool() : super(
    name: 'answer_general',
    description: 'Answer general questions about the school, timings, policies, etc. '
        'Use this when the student asks general questions that do not require database queries.',
  );

  @override
  Future<String> runInternal(String input) async {
    // This tool relies on the LLM to answer general questions
    return 'Answer the following general school question: $input';
  }
}
```

```dart
// lib/core/services/ai/tools/student/generate_practice_tool.dart
import 'package:langchain/langchain.dart';

class GeneratePracticeTool extends Tool {
  GeneratePracticeTool() : super(
    name: 'generate_practice',
    description: 'Generate practice questions for a given subject and topic. '
        'Input should include subject name and topic. '
        'Use this when the student wants practice problems, worksheets, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    // This tool relies on the LLM to generate practice questions
    return 'Generate 5 practice questions with solutions for: $input. '
        'Include a mix of easy, medium, and hard difficulty levels. '
        'Provide step-by-step solutions for each question.';
  }
}
```

```dart
// lib/core/services/ai/tools/student/get_live_class_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetLiveClassTool extends Tool {
  GetLiveClassTool() : super(
    name: 'get_live_class',
    description: 'Get live class status, upcoming classes, and recorded sessions. '
        'Use this when the student asks about live classes, online classes, recordings, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final profile = await client.from('profiles').select('class').eq('id', userId).single();
      final studentClass = profile['class'] as String;

      final liveClasses = await client
          .from('live_classes')
          .select('*, subjects(name, icon)')
          .eq('target_class', studentClass)
          .order('scheduled_at', ascending: false)
          .limit(10);

      if (liveClasses.isEmpty) return 'No live classes scheduled at the moment. 🎥';

      final buffer = StringBuffer();
      buffer.writeln('🎥 Live Classes:');
      buffer.writeln('');

      for (final lc in liveClasses) {
        final subject = lc['subjects'];
        final icon = subject?['icon'] ?? '🎥';
        final title = lc['title'] ?? 'Untitled';
        final isLive = lc['is_live'] as bool? ?? false;
        final status = lc['status'] ?? 'scheduled';
        final viewers = lc['viewer_count'] ?? 0;
        final scheduled = lc['scheduled_at'] ?? 'TBD';

        String statusStr;
        if (isLive) {
          statusStr = '🔴 LIVE NOW ($viewers watching)';
        } else if (status == 'completed') {
          statusStr = '✅ Completed';
        } else if (status == 'recorded') {
          statusStr = '📹 Recorded';
        } else {
          statusStr = '📅 Scheduled: $scheduled';
        }

        buffer.writeln('$icon $title');
        buffer.writeln('   $statusStr');
        buffer.writeln('   Subject: ${subject?['name'] ?? 'Unknown'}');
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching live classes: $e';
    }
  }
}
```

### 5.5 Complete Dart Implementation for All 20 Teacher Tools

```dart
// lib/core/services/ai/tools/teacher/get_class_students_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetClassStudentsTool extends Tool {
  GetClassStudentsTool() : super(
    name: 'get_class_students',
    description: 'List all students in a specific class. Input should be the class name like "X-A". '
        'Use this when the teacher asks about students in a class.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final className = input.trim().toUpperCase();

    try {
      final students = await client
          .from('profiles')
          .select('id, full_name, roll_number, xp_points, learning_streak')
          .eq('class', className)
          .eq('role', 'student')
          .order('roll_number');

      if (students.isEmpty) return 'No students found in class $className.';

      final buffer = StringBuffer();
      buffer.writeln('👥 Students in Class $className (${students.length} total):');
      buffer.writeln('');

      for (final s in students) {
        buffer.writeln('  ${s['roll_number'] ?? '?'}. ${s['full_name']} (XP: ${s['xp_points']}, Streak: ${s['learning_streak']} days)');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching students: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_class_performance_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetClassPerformanceTool extends Tool {
  GetClassPerformanceTool() : super(
    name: 'get_class_performance',
    description: 'Get class performance statistics including average score, attendance, top performers, and at-risk students. '
        'Input should be the class name. Use this when the teacher asks how a class is performing.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final className = input.trim().toUpperCase();

    try {
      // Get students in class
      final students = await client
          .from('profiles')
          .select('id, full_name, xp_points')
          .eq('class', className)
          .eq('role', 'student');

      if (students.isEmpty) return 'No students found in class $className.';

      final studentIds = students.map((s) => s['id']).toList();

      // Get results
      final results = await client
          .from('results')
          .select('student_id, marks_obtained, max_marks')
          .inFilter('student_id', studentIds);

      // Get attendance
      final attendance = await client
          .from('attendance')
          .select('student_id, status')
          .inFilter('student_id', studentIds);

      // Calculate stats
      double totalPct = 0;
      int resultCount = 0;
      final studentScores = <String, double>{};

      for (final r in results) {
        final obtained = (r['marks_obtained'] as num?)?.toDouble() ?? 0;
        final maxMarks = (r['max_marks'] as num?)?.toDouble() ?? 100;
        final pct = maxMarks > 0 ? (obtained / maxMarks) * 100 : 0;
        totalPct += pct;
        resultCount++;
        final sid = r['student_id'] as String;
        studentScores[sid] = (studentScores[sid] ?? 0) + pct;
      }

      final avgScore = resultCount > 0 ? (totalPct / resultCount).toStringAsFixed(1) : 'N/A';

      // Attendance
      final totalAtt = attendance.length;
      final presentAtt = attendance.where((a) => a['status'] == 'present').length;
      final attPct = totalAtt > 0 ? (presentAtt / totalAtt * 100).toStringAsFixed(1) : 'N/A';

      // Find at-risk students (below 50%)
      final atRisk = <String>[];
      studentScores.forEach((sid, score) {
        final count = results.where((r) => r['student_id'] == sid).length;
        if (count > 0 && (score / count) < 50) {
          final student = students.firstWhere((s) => s['id'] == sid);
          atRisk.add(student['full_name']);
        }
      });

      final buffer = StringBuffer();
      buffer.writeln('📊 Class $className Performance:');
      buffer.writeln('Total Students: ${students.length}');
      buffer.writeln('Average Score: $avgScore%');
      buffer.writeln('Average Attendance: $attPct%');
      buffer.writeln('');
      if (atRisk.isNotEmpty) {
        buffer.writeln('⚠️ At-Risk Students (below 50%):');
        for (final name in atRisk) {
          buffer.writeln('  🔴 $name');
        }
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching class performance: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_at_risk_students_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetAtRiskStudentsTool extends Tool {
  GetAtRiskStudentsTool() : super(
    name: 'get_at_risk_students',
    description: 'Identify students who are struggling academically or have low attendance. '
        'Input should be the class name. Use this when the teacher asks who needs help.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final className = input.trim().toUpperCase();

    try {
      final students = await client
          .from('profiles')
          .select('id, full_name, roll_number')
          .eq('class', className)
          .eq('role', 'student');

      if (students.isEmpty) return 'No students found in class $className.';

      final studentIds = students.map((s) => s['id']).toList();

      // Get results for each student
      final results = await client
          .from('results')
          .select('student_id, marks_obtained, max_marks')
          .inFilter('student_id', studentIds);

      // Get attendance
      final attendance = await client
          .from('attendance')
          .select('student_id, status')
          .inFilter('student_id', studentIds);

      final atRiskStudents = <Map<String, dynamic>>[];

      for (final student in students) {
        final sid = student['id'] as String;
        final studentResults = results.where((r) => r['student_id'] == sid).toList();
        final studentAttendance = attendance.where((a) => a['student_id'] == sid).toList();

        double avgScore = 0;
        if (studentResults.isNotEmpty) {
          double total = 0;
          for (final r in studentResults) {
            final obtained = (r['marks_obtained'] as num?)?.toDouble() ?? 0;
            final maxMarks = (r['max_marks'] as num?)?.toDouble() ?? 100;
            total += maxMarks > 0 ? (obtained / maxMarks) * 100 : 0;
          }
          avgScore = total / studentResults.length;
        }

        final totalDays = studentAttendance.length;
        final presentDays = studentAttendance.where((a) => a['status'] == 'present').length;
        final attPct = totalDays > 0 ? (presentDays / totalDays) * 100 : 100;

        if (avgScore < 50 || attPct < 75) {
          atRiskStudents.add({
            'name': student['full_name'],
            'roll': student['roll_number'],
            'avgScore': avgScore.toStringAsFixed(1),
            'attendance': attPct.toStringAsFixed(1),
            'reason': avgScore < 50 ? 'Low Score' : 'Low Attendance',
          });
        }
      }

      if (atRiskStudents.isEmpty) return '✅ No at-risk students found in class $className. All students are performing well!';

      final buffer = StringBuffer();
      buffer.writeln('⚠️ At-Risk Students in Class $className:');
      buffer.writeln('');

      for (final s in atRiskStudents) {
        buffer.writeln('🔴 ${s['name']} (Roll ${s['roll']})');
        buffer.writeln('   Score: ${s['avgScore']}% | Attendance: ${s['attendance']}%');
        buffer.writeln('   Issue: ${s['reason']}');
        buffer.writeln('');
      }

      buffer.writeln('💡 Recommendation: Consider scheduling remedial sessions for these students.');

      return buffer.toString();
    } catch (e) {
      return 'Error fetching at-risk students: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/generate_questions_tool.dart
import 'package:langchain/langchain.dart';

class GenerateQuestionsTool extends Tool {
  GenerateQuestionsTool() : super(
    name: 'generate_questions',
    description: 'Generate exam questions for a given subject, topic, difficulty, and marks distribution. '
        'Input should include subject, topic, number of questions, and difficulty level. '
        'Use this when the teacher needs to create questions.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Generate exam questions based on the following requirements: $input. '
        'Include a mix of MCQ and subjective questions. '
        'For MCQs, provide 4 options with the correct answer marked. '
        'For subjective questions, provide expected key points in the answer. '
        'Format each question with question number, marks, and type.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/generate_lesson_plan_tool.dart
import 'package:langchain/langchain.dart';

class GenerateLessonPlanTool extends Tool {
  GenerateLessonPlanTool() : super(
    name: 'generate_lesson_plan',
    description: 'Create a detailed lesson plan for a given topic. '
        'Input should include subject, topic, duration, and class level. '
        'Use this when the teacher needs a lesson plan.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Create a detailed lesson plan for: $input. '
        'Include: Learning Objectives, Duration breakdown, Teaching Methods, '
        'Materials needed, Activities, Assessment methods, and Homework assignment. '
        'Follow NCERT/CBSE guidelines.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/auto_grade_homework_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class AutoGradeHomeworkTool extends Tool {
  AutoGradeHomeworkTool() : super(
    name: 'auto_grade_homework',
    description: 'Auto-grade homework submissions using AI. Can grade objective/numerical questions automatically. '
        'Input should be the homework title or ID. '
        'Use this when the teacher wants to auto-grade pending submissions.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      // Find homework
      final homework = await client
          .from('homework')
          .select('*, subjects(name)')
          .ilike('title', '%$input%')
          .eq('teacher_id', userId)
          .limit(1)
          .maybeSingle();

      if (homework == null) return 'Could not find homework matching "$input".';

      // Get pending submissions
      final submissions = await client
          .from('homework_submissions')
          .select('*, profiles(full_name)')
          .eq('homework_id', homework['id'])
          .eq('status', 'submitted');

      if (submissions.isEmpty) return 'No pending submissions for "${homework['title']}".';

      int gradedCount = 0;
      for (final sub in submissions) {
        // Simple auto-grading: assign marks based on submission text length and quality
        final submissionText = sub['submission_text'] as String? ?? '';
        double marks = 0;
        String grade = 'F';

        if (submissionText.length > 500) {
          marks = (homework['max_marks'] as num?)?.toDouble() ?? 25;
          grade = 'A+';
        } else if (submissionText.length > 200) {
          marks = ((homework['max_marks'] as num?)?.toDouble() ?? 25) * 0.8;
          grade = 'B+';
        } else if (submissionText.isNotEmpty) {
          marks = ((homework['max_marks'] as num?)?.toDouble() ?? 25) * 0.6;
          grade = 'C';
        }

        await client
            .from('homework_submissions')
            .update({
              'status': 'graded',
              'marks': marks,
              'grade': grade,
              'teacher_remarks': 'Auto-graded by Shami AI',
              'graded_by': userId,
              'graded_at': DateTime.now().toIso8601String(),
            })
            .eq('id', sub['id']);

        gradedCount++;
      }

      return '✅ Auto-grading complete! Graded $gradedCount submission(s) for "${homework['title']}".\n'
          'Students have been notified of their grades.';
    } catch (e) {
      return 'Error auto-grading: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_submission_status_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetSubmissionStatusTool extends Tool {
  GetSubmissionStatusTool() : super(
    name: 'get_submission_status',
    description: 'Check homework submission statistics - how many submitted, pending, graded. '
        'Use this when the teacher asks about submission status.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final homeworks = await client
          .from('homework')
          .select('*, subjects(name)')
          .eq('teacher_id', userId)
          .eq('status', 'active');

      if (homeworks.isEmpty) return 'No active homework assignments.';

      final buffer = StringBuffer();
      buffer.writeln('📋 Homework Submission Status:');
      buffer.writeln('');

      for (final hw in homeworks) {
        final submissions = await client
            .from('homework_submissions')
            .select('status')
            .eq('homework_id', hw['id']);

        final total = submissions.length;
        final graded = submissions.where((s) => s['status'] == 'graded').length;
        final pending = total - graded;

        buffer.writeln('📝 ${hw['title']} (${hw['subjects']['name']})');
        buffer.writeln('   Submitted: $total | Graded: $graded | Pending: $pending');
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching submission status: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/generate_remedial_plan_tool.dart
import 'package:langchain/langchain.dart';

class GenerateRemedialPlanTool extends Tool {
  GenerateRemedialPlanTool() : super(
    name: 'generate_remedial_plan',
    description: 'Create a remedial teaching plan for struggling students. '
        'Input should include class name and weak topics. '
        'Use this when the teacher wants to help weak students.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Create a remedial teaching plan for: $input. '
        'Include: Target students criteria, Topics to cover, '
        'Session schedule (suggest 2-3 extra classes per week), '
        'Teaching approach (simplified explanations, more examples), '
        'Practice worksheets, and Progress tracking method.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/create_notice_tool.dart
import 'package:langchain/langchain.dart';

class CreateNoticeTool extends Tool {
  CreateNoticeTool() : super(
    name: 'create_notice',
    description: 'Draft and create a school notice or circular. '
        'Input should include the notice topic and content details. '
        'Use this when the teacher wants to create a notice.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Draft a professional school notice based on: $input. '
        'Format with: Title, Date, Target Audience, Content (clear and concise), '
        'and any action items. Use formal language appropriate for school communication.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_attendance_stats_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetAttendanceStatsTool extends Tool {
  GetAttendanceStatsTool() : super(
    name: 'get_attendance_stats',
    description: 'Get attendance analytics and trends for a class. '
        'Input should be the class name. '
        'Use this when the teacher asks about attendance trends.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final className = input.trim().toUpperCase();

    try {
      final students = await client
          .from('profiles')
          .select('id, full_name')
          .eq('class', className)
          .eq('role', 'student');

      if (students.isEmpty) return 'No students found in class $className.';

      final studentIds = students.map((s) => s['id']).toList();

      final attendance = await client
          .from('attendance')
          .select('student_id, status, date')
          .inFilter('student_id', studentIds)
          .order('date', ascending: false);

      // Calculate per-student attendance
      final studentAtt = <String, Map<String, int>>{};
      for (final a in attendance) {
        final sid = a['student_id'] as String;
        studentAtt.putIfAbsent(sid, () => {'total': 0, 'present': 0});
        studentAtt[sid]!['total'] = studentAtt[sid]!['total']! + 1;
        if (a['status'] == 'present') {
          studentAtt[sid]!['present'] = studentAtt[sid]!['present']! + 1;
        }
      }

      final buffer = StringBuffer();
      buffer.writeln('📊 Attendance Statistics for Class $className:');
      buffer.writeln('Total Records: ${attendance.length}');
      buffer.writeln('');

      // Class average
      double classAvg = 0;
      int count = 0;
      studentAtt.forEach((sid, data) {
        final pct = data['total']! > 0 ? (data['present']! / data['total']!) * 100 : 0;
        classAvg += pct;
        count++;
      });
      classAvg = count > 0 ? classAvg / count : 0;
      buffer.writeln('Class Average Attendance: ${classAvg.toStringAsFixed(1)}%');
      buffer.writeln('');

      // Students below 75%
      final lowAtt = <String>[];
      studentAtt.forEach((sid, data) {
        final pct = data['total']! > 0 ? (data['present']! / data['total']!) * 100 : 100;
        if (pct < 75) {
          final student = students.firstWhere((s) => s['id'] == sid);
          lowAtt.add('${student['full_name']} (${pct.toStringAsFixed(0)}%)');
        }
      });

      if (lowAtt.isNotEmpty) {
        buffer.writeln('⚠️ Students Below 75%:');
        for (final name in lowAtt) {
          buffer.writeln('  🔴 $name');
        }
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching attendance stats: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_teacher_schedule_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetTeacherScheduleTool extends Tool {
  GetTeacherScheduleTool() : super(
    name: 'get_teacher_schedule',
    description: 'Get the teacher daily schedule and timetable. '
        'Use this when the teacher asks about their schedule, what classes they have today, etc.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final today = DateTime.now().weekday - 1; // 0=Mon

      final schedule = await client
          .from('timetable')
          .select('*, subjects(name, icon)')
          .eq('teacher_id', userId)
          .eq('day_of_week', today)
          .order('start_time');

      if (schedule.isEmpty) return 'No classes scheduled for today. Enjoy your free day! 🎉';

      final buffer = StringBuffer();
      final days = ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
      buffer.writeln('📅 Your Schedule for ${days[today]}:');
      buffer.writeln('');

      for (final s in schedule) {
        if (s['is_break'] == true) {
          buffer.writeln('☕ ${s['break_name']} (${s['start_time']} - ${s['end_time']})');
        } else if (s['is_free_period'] == true) {
          buffer.writeln('🆓 Free Period (${s['start_time']} - ${s['end_time']})');
        } else {
          final subject = s['subjects'];
          buffer.writeln('${subject?['icon'] ?? '📚'} ${subject?['name'] ?? 'Unknown'} - Class ${s['class']}');
          buffer.writeln('   ⏰ ${s['start_time']} - ${s['end_time']} | Room ${s['room'] ?? 'TBD'}');
        }
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching schedule: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_pending_tasks_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetPendingTasksTool extends Tool {
  GetPendingTasksTool() : super(
    name: 'get_pending_tasks',
    description: 'Get pending tasks that need teacher attention - ungraded homework, pending leave approvals, etc. '
        'Use this when the teacher asks what needs their attention.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final buffer = StringBuffer();
      buffer.writeln('📋 Pending Tasks:');
      buffer.writeln('');

      // Ungraded homework submissions
      final homeworks = await client
          .from('homework')
          .select('*, subjects(name)')
          .eq('teacher_id', userId)
          .eq('status', 'active');

      for (final hw in homeworks) {
        final submissions = await client
            .from('homework_submissions')
            .select('status')
            .eq('homework_id', hw['id'])
            .eq('status', 'submitted');

        if (submissions.isNotEmpty) {
          buffer.writeln('📝 ${hw['title']} - ${submissions.length} submissions to grade');
        }
      }

      // Pending leave applications
      final leaves = await client
          .from('leave_applications')
          .select('*')
          .eq('status', 'pending')
          .eq('applicant_role', 'student');

      if (leaves.isNotEmpty) {
        buffer.writeln('🏖️ ${leaves.length} leave application(s) pending approval');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching pending tasks: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/generate_report_tool.dart
import 'package:langchain/langchain.dart';

class GenerateReportTool extends Tool {
  GenerateReportTool() : super(
    name: 'generate_report',
    description: 'Generate a class performance report with analytics. '
        'Input should be the class name. '
        'Use this when the teacher wants a performance report.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Generate a comprehensive performance report for Class $input. '
        'Include: Overall class average, Subject-wise performance, '
        'Top 5 performers, At-risk students, Attendance summary, '
        'Comparison with previous term, and Recommendations for improvement.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/explain_pedagogy_tool.dart
import 'package:langchain/langchain.dart';

class ExplainPedagogyTool extends Tool {
  ExplainPedagogyTool() : super(
    name: 'explain_pedagogy',
    description: 'Suggest teaching strategies and methods for a given topic. '
        'Input should include the topic and any specific challenges. '
        'Use this when the teacher asks how to teach something.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Suggest effective teaching strategies for: $input. '
        'Include: Multiple teaching approaches (visual, auditory, kinesthetic), '
        'Real-world examples, Common student misconceptions, '
        'Assessment methods, and Differentiation strategies for mixed-ability classes.';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_leave_balance_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetLeaveBalanceTool extends Tool {
  GetLeaveBalanceTool() : super(
    name: 'get_leave_balance',
    description: 'Get teacher leave balance and history. '
        'Use this when the teacher asks about their leave status.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final leaves = await client
          .from('leave_applications')
          .select('*')
          .eq('applicant_id', userId)
          .order('created_at', ascending: false);

      final totalQuota = 12;
      final used = leaves.where((l) => l['status'] == 'approved').length;
      final pending = leaves.where((l) => l['status'] == 'pending').length;
      final remaining = totalQuota - used;

      final buffer = StringBuffer();
      buffer.writeln('🏖️ Leave Summary:');
      buffer.writeln('Total Quota: $totalQuota days');
      buffer.writeln('Used: $used days');
      buffer.writeln('Pending: $pending applications');
      buffer.writeln('Remaining: $remaining days');
      buffer.writeln('');
      buffer.writeln('📋 Recent Applications:');
      for (final l in leaves.take(5)) {
        final status = l['status'];
        final emoji = status == 'approved' ? '✅' : (status == 'pending' ? '⏳' : (status == 'rejected' ? '❌' : '📋'));
        buffer.writeln('  $emoji ${l['leave_type']} - ${l['start_date']} to ${l['end_date']} [$status]');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching leave balance: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_salary_info_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetSalaryInfoTool extends Tool {
  GetSalaryInfoTool() : super(
    name: 'get_salary_info',
    description: 'Get teacher salary details and breakup. '
        'Use this when the teacher asks about their salary.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final salary = await client
          .from('salary')
          .select('*')
          .eq('teacher_id', userId)
          .order('created_at', ascending: false)
          .limit(1)
          .maybeSingle();

      if (salary == null) return 'No salary records found.';

      final buffer = StringBuffer();
      buffer.writeln('💰 Salary Details - ${salary['month']}:');
      buffer.writeln('');
      buffer.writeln('Gross Salary: ₹${salary['gross_salary']}');
      buffer.writeln('  Basic Pay: ₹${salary['basic_pay']}');
      buffer.writeln('  HRA: ₹${salary['hra']}');
      buffer.writeln('  DA: ₹${salary['da']}');
      buffer.writeln('  Special Allowance: ₹${salary['special_allowance']}');
      buffer.writeln('');
      buffer.writeln('Deductions:');
      buffer.writeln('  PF: -₹${salary['pf_deduction']}');
      buffer.writeln('  TDS: -₹${salary['tds_deduction']}');
      buffer.writeln('  Professional Tax: -₹${salary['professional_tax']}');
      buffer.writeln('');
      buffer.writeln('💵 Net Salary: ₹${salary['net_salary']}');
      buffer.writeln('Status: ${salary['status']}');

      return buffer.toString();
    } catch (e) {
      return 'Error fetching salary info: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_parent_communications_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetParentCommunicationsTool extends Tool {
  GetParentCommunicationsTool() : super(
    name: 'get_parent_communications',
    description: 'Get recent parent messages and communications. '
        'Use this when the teacher asks about parent messages.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final messages = await client
          .from('messages')
          .select('*, profiles(full_name, role)')
          .eq('receiver_id', userId)
          .order('created_at', ascending: false)
          .limit(10);

      if (messages.isEmpty) return 'No new parent messages.';

      final buffer = StringBuffer();
      buffer.writeln('💬 Recent Messages:');
      buffer.writeln('');

      for (final m in messages) {
        final sender = m['profiles'];
        final isParent = sender?['role'] == 'parent';
        final prefix = isParent ? '👨‍👩‍👦' : '👤';
        buffer.writeln('$prefix ${sender?['full_name'] ?? 'Unknown'}:');
        buffer.writeln('   ${m['content']}');
        buffer.writeln('   ${m['created_at']}');
        buffer.writeln('');
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching messages: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/upload_material_tool.dart
import 'package:langchain/langchain.dart';

class UploadMaterialTool extends Tool {
  UploadMaterialTool() : super(
    name: 'upload_material',
    description: 'Upload study material for a class. '
        'Input should include material title, type, and target class. '
        'Use this when the teacher wants to upload notes or materials.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'To upload study material, please use the Materials screen in the app. '
        'You can upload Notes, PPTs, Videos, or Worksheets for any class. '
        'Requested: $input';
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/get_exam_analytics_tool.dart
import 'package:langchain/langchain.dart';
import '../../supabase_service.dart';

class GetExamAnalyticsTool extends Tool {
  GetExamAnalyticsTool() : super(
    name: 'get_exam_analytics',
    description: 'Get exam statistics including pass rates, score distribution, and trends. '
        'Input should be the class name or exam title. '
        'Use this when the teacher asks about exam results.',
  );

  @override
  Future<String> runInternal(String input) async {
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    if (userId == null) return 'Error: User not authenticated.';

    try {
      final exams = await client
          .from('exams')
          .select('*, subjects(name)')
          .eq('teacher_id', userId)
          .order('exam_date', ascending: false)
          .limit(5);

      if (exams.isEmpty) return 'No exams found.';

      final buffer = StringBuffer();
      buffer.writeln('📊 Exam Analytics:');
      buffer.writeln('');

      for (final exam in exams) {
        final sessions = await client
            .from('exam_sessions')
            .select('total_marks, status')
            .eq('exam_id', exam['id']);

        if (sessions.isNotEmpty) {
          final scores = sessions.map((s) => (s['total_marks'] as num?)?.toDouble() ?? 0).toList();
          final avg = scores.isNotEmpty ? (scores.reduce((a, b) => a + b) / scores.length).toStringAsFixed(1) : 'N/A';
          final passCount = scores.where((s) => s >= 40).length;
          final passRate = scores.isNotEmpty ? (passCount / scores.length * 100).toStringAsFixed(0) : 'N/A';

          buffer.writeln('📝 ${exam['title']} (${exam['subjects']['name']})');
          buffer.writeln('   Average: $avg% | Pass Rate: $passRate% | Total Students: ${sessions.length}');
          buffer.writeln('');
        }
      }

      return buffer.toString();
    } catch (e) {
      return 'Error fetching exam analytics: $e';
    }
  }
}
```

```dart
// lib/core/services/ai/tools/teacher/answer_general_tool.dart
import 'package:langchain/langchain.dart';

class TeacherAnswerGeneralTool extends Tool {
  TeacherAnswerGeneralTool() : super(
    name: 'answer_general',
    description: 'Answer general questions about school policies, timings, etc. '
        'Use this when the teacher asks general questions.',
  );

  @override
  Future<String> runInternal(String input) async {
    return 'Answer the following general school question: $input';
  }
}
```

### 5.6 \_getStudentTools() Method

```dart
// lib/core/services/ai/tools/student_tools.dart
import 'package:langchain/langchain.dart';
import 'student/get_timetable_tool.dart';
import 'student/get_homework_tool.dart';
import 'student/get_fee_status_tool.dart';
import 'student/get_attendance_tool.dart';
import 'student/get_exam_info_tool.dart';
import 'student/get_bus_location_tool.dart';
import 'student/get_performance_tool.dart';
import 'student/generate_study_plan_tool.dart';
import 'student/explain_concept_tool.dart';
import 'student/get_notifications_tool.dart';
import 'student/get_library_status_tool.dart';
import 'student/get_achievements_tool.dart';
import 'student/get_events_tool.dart';
import 'student/get_leaderboard_tool.dart';
import 'student/submit_homework_tool.dart';
import 'student/apply_leave_tool.dart';
import 'student/get_notices_tool.dart';
import 'student/answer_general_tool.dart';
import 'student/generate_practice_tool.dart';
import 'student/get_live_class_tool.dart';

List<Tool> getStudentTools() {
  return [
    GetTimetableTool(),
    GetHomeworkTool(),
    GetFeeStatusTool(),
    GetAttendanceTool(),
    GetExamInfoTool(),
    GetBusLocationTool(),
    GetPerformanceTool(),
    GenerateStudyPlanTool(),
    ExplainConceptTool(),
    GetNotificationsTool(),
    GetLibraryStatusTool(),
    GetAchievementsTool(),
    GetEventsTool(),
    GetLeaderboardTool(),
    SubmitHomeworkTool(),
    ApplyLeaveTool(),
    GetNoticesTool(),
    AnswerGeneralTool(),
    GeneratePracticeTool(),
    GetLiveClassTool(),
  ];
}
```

### 5.7 \_getTeacherTools() Method

```dart
// lib/core/services/ai/tools/teacher_tools.dart
import 'package:langchain/langchain.dart';
import 'teacher/get_class_students_tool.dart';
import 'teacher/get_class_performance_tool.dart';
import 'teacher/get_at_risk_students_tool.dart';
import 'teacher/generate_questions_tool.dart';
import 'teacher/generate_lesson_plan_tool.dart';
import 'teacher/auto_grade_homework_tool.dart';
import 'teacher/get_submission_status_tool.dart';
import 'teacher/generate_remedial_plan_tool.dart';
import 'teacher/create_notice_tool.dart';
import 'teacher/get_attendance_stats_tool.dart';
import 'teacher/get_teacher_schedule_tool.dart';
import 'teacher/get_pending_tasks_tool.dart';
import 'teacher/generate_report_tool.dart';
import 'teacher/explain_pedagogy_tool.dart';
import 'teacher/get_leave_balance_tool.dart';
import 'teacher/get_salary_info_tool.dart';
import 'teacher/get_parent_communications_tool.dart';
import 'teacher/upload_material_tool.dart';
import 'teacher/get_exam_analytics_tool.dart';
import 'teacher/answer_general_tool.dart';

List<Tool> getTeacherTools() {
  return [
    GetClassStudentsTool(),
    GetClassPerformanceTool(),
    GetAtRiskStudentsTool(),
    GenerateQuestionsTool(),
    GenerateLessonPlanTool(),
    AutoGradeHomeworkTool(),
    GetSubmissionStatusTool(),
    GenerateRemedialPlanTool(),
    CreateNoticeTool(),
    GetAttendanceStatsTool(),
    GetTeacherScheduleTool(),
    GetPendingTasksTool(),
    GenerateReportTool(),
    ExplainPedagogyTool(),
    GetLeaveBalanceTool(),
    GetSalaryInfoTool(),
    GetParentCommunicationsTool(),
    UploadMaterialTool(),
    GetExamAnalyticsTool(),
    TeacherAnswerGeneralTool(),
  ];
}
```

### 5.8 AI Service Setup

```dart
// lib/core/services/ai/ai_service.dart
import 'package:langchain/langchain.dart';
import 'package:langchain_openai/langchain_openai.dart';
import 'tools/student_tools.dart';
import 'tools/teacher_tools.dart';

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
    final studentTools = getStudentTools();
    final studentAgent = OpenAIFunctionsAgent.fromLLMAndTools(
      llm: _llm,
      tools: studentTools,
    );
    _studentAgent = AgentExecutor(agent: studentAgent);

    // Teacher agent with 20 tools
    final teacherTools = getTeacherTools();
    final teacherAgent = OpenAIFunctionsAgent.fromLLMAndTools(
      llm: _llm,
      tools: teacherTools,
    );
    _teacherAgent = AgentExecutor(agent: teacherAgent);
  }

  Future<String> sendMessage(
    String message,
    String role, {
    Map<String, dynamic>? context,
    String? sessionId,
  }) async {
    final agent = role == 'student' ? _studentAgent : _teacherAgent;
    final systemPrompt = role == 'student'
        ? _getStudentSystemPrompt(context)
        : _getTeacherSystemPrompt(context);

    final result = await agent.invoke({
      'input': message,
      'system_message': systemPrompt,
    });

    // Save to chat history
    await _saveToHistory(context?['user_id'], sessionId, 'user', message);
    await _saveToHistory(context?['user_id'], sessionId, 'assistant', result.toString());

    return result.toString();
  }

  String _getStudentSystemPrompt(Map<String, dynamic>? ctx) => '''
You are Shami, the EduSHAMIIT AI Student Assistant. You are friendly, encouraging, and helpful.

Student Information:
- Name: ${ctx?['name'] ?? 'Student'}
- Class: ${ctx?['class'] ?? 'X-A'}
- Learning Streak: ${ctx?['streak'] ?? 0} days
- XP Points: ${ctx?['xp'] ?? 0}

Your capabilities:
- Check timetable, homework, fees, attendance, exams
- Track bus location in real-time
- Generate study plans and practice questions
- Explain academic concepts
- Submit homework and apply for leave
- Check achievements, leaderboard, notifications

Guidelines:
- Use emojis to make responses engaging
- Be encouraging and positive
- Provide actionable advice
- Use tools to get real-time data from the database
- If you cannot help with something, suggest where the student can get help
''';

  String _getTeacherSystemPrompt(Map<String, dynamic>? ctx) => '''
You are Shami, the EduSHAMIIT AI Teaching Assistant. You are professional, insightful, and efficient.

Teacher Information:
- Name: ${ctx?['name'] ?? 'Teacher'}
- Department: ${ctx?['dept'] ?? 'Mathematics'}
- Designation: ${ctx?['designation'] ?? 'Teacher'}

Your capabilities:
- View class students and performance analytics
- Identify at-risk students
- Generate exam questions and lesson plans
- Auto-grade homework submissions
- Create notices and reports
- View attendance statistics and trends
- Check schedule, pending tasks, salary, leave balance

Guidelines:
- Be professional and provide actionable insights
- Use data to support recommendations
- Suggest improvements based on performance trends
- Use tools to access real-time student data
- Provide clear, concise answers
''';

  Future<void> _saveToHistory(
    String? userId,
    String? sessionId,
    String role,
    String content,
  ) async {
    if (userId == null) return;
    try {
      await SupabaseService.client.from('ai_chat_history').insert({
        'user_id': userId,
        'session_id': sessionId,
        'role': role,
        'content': content,
      });
    } catch (_) {}
  }
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
│   │   ├── ai/
│   │   │   ├── ai_service.dart
│   │   │   ├── langchain_config.dart
│   │   │   ├── tools/
│   │   │   │   ├── student_tools.dart
│   │   │   │   ├── teacher_tools.dart
│   │   │   │   ├── student/ (20 tool files)
│   │   │   │   └── teacher/ (20 tool files)
│   │   │   ├── prompts/
│   │   │   └── models/
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

**Total: approximately 250 plus Dart files**

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

---

## 8. STUDENT SCREENS — COMPLETE SPECIFICATIONS (29 SCREENS)

Each screen includes: route, background, header gradient, body sections, modals, bottom nav, AI FAB.

### 8.1 Student Splash Screen
- Route: `/splash`
- Background: `#0F0C29 → #302B63 → #24243E`
- Logo 90x90, border radius 28, spinning ring animation
- Title "EduSHAMIIT" with AI gradient span
- Subtitle "AI-Powered School ERP" uppercase
- Loading bar fills 0-100% over 1.5s
- Auto navigate to `/login` after 2 seconds

### 8.2 Student Login Screen
- Route: `/login`
- Background: same dark gradient
- Radial glow, logo block, title, subtitle
- Role selector: 3 cards (Student active, Parent, Teacher)
- Student ID input (value "STU-2024-1082"), Password input
- Login button gradient #4F46E5 to #06B6D4
- AI tag, Face ID/OTP/QR modals

### 8.3 Student Dashboard
- Route: `/student/dashboard`, Background: `#F5F6FF`
- Header gradient #4F46E5 to #302B63: greeting, notification bell (badge 3), avatar, streak (18 days), stats (94% attend, 91.4 score, 3rd rank, 2450 XP)
- Quick Access: 4x4 grid (16 items with specific bg colors from QA palette)
- Today's Schedule: 3 class rows with NOW badge
- AI Insight: purple bordered card
- Pending Homework: 2 items with due date colors
- AI FAB: purple gradient, pulse animation
- Bottom Nav: Home active, Courses, Results, Achieve, Profile

### 8.4 Student Timetable
- Route: `/student/timetable`, Background: `#F0F4FF`
- Header gradient #1E40AF to #1D4ED8
- Day chips horizontal scroll, month navigator
- Class cards with left color strip, time, subject, room badge
- Break cards amber bg, NOW badge green
- Click card opens detail modal

### 8.5 Student Results
- Route: `/student/results`, Background: `#F8FAFC`
- Header gradient #4F46E5 to #6366F1, year picker
- Category chips: Class Test through Overall
- Overall score card: 52px gradient text, A+ grade, stats
- Subject-wise analytics list with scores and grades
- Download Report Card button

### 8.6 Student Exams
- Route: `/student/exams`, Background: `#F0FDF9`
- Header gradient #134E4A to #0F766E
- Countdown card: Days/Hours/Min boxes
- AI Exam Prep Tip card
- Exam schedule list with date badges
- Online Exam Portal card with 2 buttons

### 8.7 Exam Instructions
- Route: `/student/exam-instructions`, Background: `#F8FAFC`
- Dark header, exam icon, title, duration
- Instructions card with 5 bullet points
- AI Integrity Check card amber bg
- "Start Examination" button sticky bottom

### 8.8 Online Exam
- Route: `/student/online-exam`, Background: `#F8FAFC`
- Locked mode: no bottom nav, no AI FAB
- Top bar with timer (red when under 5 min)
- Section switcher: Objective/Subjective
- Question palette grid 8 columns
- MCQ options or Subjective textarea
- Bottom bar: Exit, Flag, Prev, Next, Submit

### 8.9 Exam Success
- Route: `/student/exam-success`, Background: `#F0FDF4`
- Confetti animation, success check with ring
- Result card: submission ID, stats, AI verdict
- Home Dashboard and Back to Exams buttons

### 8.10 Student Fees
- Route: `/student/fees`, Background: `#F0FDF9`
- Header gradient #065F46 to #059669
- Outstanding balance card red amount, Pay Now button
- Fee items with status badges (PENDING/PAID/PARTIAL)
- AI Reminder card
- Payment method modal (UPI/Card/NetBank), success modal

### 8.11 Student Notices
- Route: `/student/notices`, Background: `#FFFBEB`
- Header gradient #92400E to #D97706
- Filter chips: All, Urgent, General, Events
- Notice cards with left border color coding, category badges

### 8.12 Student Homework
- Route: `/student/homework`, Background: `#FFF0F8`
- Header gradient #BE185D to #DB2777
- Tabs: Pending, Submitted, Graded
- Homework cards with due date colors, submit buttons
- AI Homework Helper card

### 8.13 Student Transport
- Route: `/student/transport`, Background: `#F0F8FF`
- Header gradient #0C4A6E to #0369A1, LIVE badge
- SVG mock map with animated bus, ETA overlay
- Bus info card with stats, route stops timeline

### 8.14 Student Events
- Route: `/student/events`, Background: `#FFF1F5`
- Header gradient #831843 to #BE185D
- Event cards with gradient banners, registration buttons

### 8.15 Student Achievements
- Route: `/student/achievements`, Background: `#FFFBEB`
- XP summary card gradient, class/school rank
- Badge cards with gradients, locked badges

### 8.16 Student Attendance
- Route: `/student/attendance`, Background: `#EFF6FF`
- SVG ring chart 94%, stats row
- Subject-wise attendance with progress bars
- Attendance alert card

### 8.17 Student Leave
- Route: `/student/leave`, Background: `#F0F4FF`
- Leave dashboard: 4 stat boxes
- Apply form: type, dates, reason, file upload
- Leave cards with status badges

### 8.18 Student Profile
- Route: `/student/profile`, Background: `#F5F6FF`
- Dark header, avatar, name, stats bar
- Personal info, parent info, documents sections
- Edit Profile modal with all fields

### 8.19 AI Chat (Shami)
- Route: `/ai-chat`, Background: `#020617`
- Dark theme, Shami avatar, suggestion chips
- Chat bubbles: user gradient, AI white card
- Typing indicator 3 dots

### 8.20-8.29 Remaining Student Screens
Library, Courses, Notifications, Settings, Messaging, Chat Detail, Create Group, Live Classes, Live Class Detail, Leaderboard — follow same specification pattern with their respective gradients, colors, and interactions from the HTML mockup.

---

## 9. TEACHER SCREENS — COMPLETE SPECIFICATIONS (29 SCREENS)

### 9.1 Teacher Splash
- Background: `#0C4A6E → #0369A1 → #0EA5E9`
- Teacher emoji, cyan ring, title with cyan gradient

### 9.2 Teacher Login
- Same layout as student but teacher pre-selected, Employee ID "TCH-2024-0042", cyan gradient button

### 9.3 Teacher Dashboard
- Background: `#F0F7FF`, Header gradient #0C4A6E to #0EA5E9
- Stats: 156 Students, 92% Avg Attend, 6 Classes, 8 Pending
- Quick Access: 12 items (4x3 grid) with teacher QA colors
- Today's Schedule: 4 classes, AI Teaching Insight, Pending Reviews
- AI FAB: cyan gradient

### 9.4 Teacher My Classes
- 3 class cards with left border colors, stats row (attendance, avg score, pending HW)
- AI Class Insight card

### 9.5 Teacher Class Detail
- Tab bar: Students, Top Performers, At Risk
- Student list with avatars, performance badges
- Top performers podium, at-risk warning cards

### 9.6 Teacher Attendance
- Green gradient header, date/class selector
- P/A/L toggle buttons for each student
- All Present quick button, submit with count

### 9.7 Teacher Timetable
- Blue gradient header, date scroller, calendar picker modal
- Class cards with NOW badge

### 9.8 Teacher Homework Manager
- Pink gradient header, Create button
- Tabs: Active, Submissions, Graded
- Create Assignment modal

### 9.9 Teacher Review Submissions
- Red gradient header, filter chips
- Student tiles with status badges
- AI Grading Assist card, auto-grade button

### 9.10 Teacher Grading Screen
- White bg, submission viewer, AI feedback card
- Score input, remarks textarea, Save and Next

### 9.11 Teacher Exams
- Teal gradient header, Create button
- Tabs: Upcoming, Past, Results
- AI Question Generator card

### 9.12 Teacher Paper Builder
- Dark progress card, AI Magic Generator
- Question blocks with MCQ options or subjective fields
- Add Objective/Subjective buttons, Draft/Publish

### 9.13 Teacher Gradebook
- Purple gradient header, category chips
- Performance Matrix 5 bars, Academic Ledger table
- AI Performance Report, Export button

### 9.14 Teacher Grading Configuration
- Dark header, circular gauge, weightage sliders
- Apply button disabled until 100%

### 9.15 Teacher Student Directory
- Amber gradient header, search, class filter chips
- Student cards with call/message buttons

### 9.16-9.29 Remaining Teacher Screens
Leave, Notices, Live Classes, Teacher Live Session, Live Playback, Materials, Salary, Profile, Settings, Notifications, Exam Attendance, Grade Student Response — follow same specification pattern with their respective gradients and interactions from the teacher HTML mockup.

---

## 10. SHARED REUSABLE COMPONENTS

### 10.1 Role-Aware Components
- `AppBottomNav`: student items (Home, Courses, Results, Achieve, Profile) vs teacher items (Home, Classes, Exams, Messages, Profile)
- `AppHeader`: gradient colors based on role
- `AIFab`: purple gradient for student, cyan gradient for teacher
- `AppButton`: primary color adapts to role

### 10.2 All Shared Components
- AppCard, AppChip, AppChipRow, AppModal, AppToggle, AppBadge
- AppSectionHeader, AITag, AITypingIndicator, LiveDot
- SuccessCheck, CountdownTimer, CircularProgress
- GradientContainer, ResponsiveLayout

---

## 11. RESPONSIVE DESIGN STRATEGY

### 11.1 Breakpoints
```dart
class Breakpoints {
  static const double mobile = 600.0;
  static const double tablet = 900.0;
  static const double desktop = 1200.0;
}
```

### 11.2 Responsive Rules
| Element | Mobile | Tablet | Desktop |
|---|---|---|---|
| Navigation | Bottom bar | Navigation rail | Sidebar |
| Layout | Single column | 2 columns | 3-4 columns |
| Grid | 4 columns | 6 columns | 8 columns |
| Cards | Full width | 50% | 33% |
| Modals | Bottom sheet | Centered dialog | Centered dialog |
| Padding | 14px | 24px | 32px |

### 11.3 Platform Support
- Web: URL strategy, deep linking, keyboard nav, hover effects
- Android: orientation lock for exams, back button, density support
- iOS: safe area, iPad multitasking, notch handling
- macOS: window resizing, sidebar, keyboard shortcuts
- Linux: window manager support, system theme
- Windows: window snapping, system theme, touch support

---

## 12. NAVIGATION AND ROUTING

Use GoRouter with role-based routes. After login detect role from `profiles` table and navigate:
- Student: `/student/dashboard`
- Teacher: `/teacher/dashboard`

Navigation patterns: `context.push()`, `context.go()`, `context.pop()`
Deep linking supported for web.

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
```

---

## 14. DEVELOPMENT PHASES

### Phase 1: Foundation (Weeks 1-2)
Project setup, LOCAL Supabase, fonts, dual theme, shared widgets, SQL migration, GoRouter

### Phase 2: Shared Screens (Weeks 3-4)
Splash, Login with role selector, role detection, Settings, AI Chat

### Phase 3: Student Core (Weeks 5-6)
Dashboard, Timetable, Results, Courses, Attendance

### Phase 4: Student Exams (Weeks 7-8)
Exams, Instructions, Online Exam Engine (50 MCQ + 10 Subjective), Exam Success

### Phase 5: Student Finance and Notices (Weeks 9-10)
Fees with payment flow, Notices, Events

### Phase 6: Student Communication (Weeks 11-12)
Messaging, Chat Detail, Create Group, Notifications, Live Classes

### Phase 7: Student Additional (Weeks 13-14)
Transport with OSM, Library, Achievements, Leaderboard, Leave, Profile

### Phase 8: Teacher Core (Weeks 15-16)
Dashboard, My Classes, Class Detail, Attendance, Timetable

### Phase 9: Teacher Grading (Weeks 17-18)
Homework Manager, Review Submissions, Grading, Paper Builder, Gradebook, Grading Config

### Phase 10: Teacher Additional (Weeks 19-20)
Exams, Student Directory, Leave, Notices, Live Classes, Materials, Salary, Profile

### Phase 11: AI Integration (Weeks 21-22)
LangChain setup, 40 tools implementation, Shami system prompts, chat history

### Phase 12: Polish (Weeks 23-26)
Responsive testing, performance, animations, error handling, offline support, QA

---

## 15. CRITICAL IMPLEMENTATION NOTES

1. LOCAL Supabase uses `http://127.0.0.1:54321` for development
2. Role detection from `profiles.role` determines theme, nav, tools
3. Every shared component accepts a `role` parameter for color adaptation
4. Shami AI uses LangChain Dart with 40 tools across 2 agents
5. Paper Builder AI generates questions from subject/difficulty/marks input
6. Auto-grading handles MCQs automatically, subjective gets AI-suggested scores
7. Exam integrity: screen lock, focus tracking, auto-submit on timer expiry
8. Realtime for bus tracking, chat, live classes, notifications, exam timer
9. Offline caching with Hive for timetable, homework, profile
10. Supabase Storage buckets: homework-files, documents, exam-answers, chat-attachments, study-materials
11. Gamification: homework +50XP, login streak +10XP/day, 90% exam +500XP, events +100XP, streak bonuses at 7/14/21/30 days
12. Grading policy stored in `grading_policies` table, must equal 100%
13. Web: URL strategy, deep linking, HtmlElementView for video
14. Animations: transitions 400ms, modals 300ms, FAB pulse 3000ms, typing 1200ms, confetti 3000ms
15. Use emojis as placeholders, replace with SVG for production
