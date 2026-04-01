# EduSHAMIIT AI Academic App — Complete Flutter Development Guide

> **PURPOSE**: This document is a SELF-CONTAINED BLUEPRINT. Any AI model or developer reading ONLY this file should be able to build the exact EduSHAMIIT AI Academic App from scratch — a UNIFIED Flutter application for BOTH Students and Teachers, matching the provided HTML mockups pixel-perfectly across all 6 platforms (Web, Android, iOS, macOS, Linux, Windows).

> **PROJECT NAME**: EduSHAMIIT AI
> **PACKAGE NAME**: com.shamiit.edu
> **AI AGENT NAME**: Shami
> **BACKEND**: LOCAL Supabase (running on developer's machine)
> **ARCHITECTURE**: Single app with role-based routing (Student vs Teacher)

---

## TABLE OF CONTENTS

1. Project Overview and Architecture
2. Step-by-Step Setup Guide
3. Design System — Dual Theme
4. Complete Database Schema (SQL)
5. AI Functions (LangChain Tool Calling) — Deep Detail
6. Complete Folder Structure
7. Core Dart Files (Templates)
8. Student Screens — Complete Specifications (29 Screens)
9. Teacher Screens — Complete Specifications (29 Screens)
10. Shared Reusable Components
11. Responsive Design Strategy
12. Navigation and Routing
13. Dependencies
14. Development Phases
15. Critical Implementation Notes

---

## 1. PROJECT OVERVIEW AND ARCHITECTURE

### What Is EduSHAMIIT AI?

EduSHAMIIT AI is a unified AI-powered Academic App built with Flutter. It serves BOTH students and teachers through a single codebase. After login, the app detects the user's role and routes them to their respective portal. The shared screens (Splash, Login, Chat Detail, Settings) use a unified design with role-specific theming. The AI assistant agent is named **Shami** and provides intelligent assistance using LangChain tool calling with 40 tools total (20 for students, 20 for teachers).

### Architecture Diagram

The app follows a hub-and-spoke architecture where Splash and Login are shared entry points. After role detection, users are routed to either the Student Portal (29 screens) or Teacher Portal (29 screens). Both portals share common services including LOCAL Supabase for database and authentication, LangChain AI for the Shami assistant, OpenStreetMap via flutter_map for transport tracking, Supabase Realtime for live updates, and Supabase Storage for file uploads.

### Screen Count Summary

Shared screens: 7 (Splash, Login, AI Chat, Settings, Messaging, Chat Detail, Create Group). Student screens: 29. Teacher screens: 29. Total: approximately 65 screens.

### Target Platforms

Web (Chrome, Firefox, Safari, Edge), Android (phone and tablet), iOS (iPhone and iPad), macOS (desktop), Linux (desktop), Windows (desktop).

---

## 2. STEP-BY-STEP SETUP GUIDE

### Step 1: Install LOCAL Supabase

Install Docker Desktop. Install Supabase CLI with npm install -g supabase. Initialize with supabase init. Start with supabase start. This outputs API URL http://127.0.0.1:54321, DB URL postgresql://postgres:postgres@127.0.0.1:54322/postgres, Studio URL http://127.0.0.1:54323, and anon key.

### Step 2: Create Flutter Project

Run flutter create with --org com.shamiit and project name edu_shamiit_ai.

### Step 3: Run SQL Migration

Create migration with supabase migration new initial_schema. Copy SQL from Section 4 into the migration file. Apply with supabase db reset.

### Step 4: Add Dependencies

Replace pubspec.yaml dependencies section with Section 13.

### Step 5: Create Folder Structure

Create all directories listed in Section 6.

### Step 6: Add Fonts

Download Outfit (weights 300-900) and DM Sans (weights 300-700) from Google Fonts. Place in assets/fonts/. Add font declarations to pubspec.yaml.

### Step 7: Configure LOCAL Supabase

Create supabase_service.dart with LOCAL URLs http://127.0.0.1:54321 and anon key from supabase start output. Initialize in main.dart.

### Step 8: Run and Verify

Terminal 1: supabase start. Terminal 2: flutter pub get then flutter run -d chrome for web or flutter run for mobile.

### Step 9: Supabase Studio

Open http://127.0.0.1:54323 to view tables, data, run queries, manage auth, and test realtime.

---

## 3. DESIGN SYSTEM — DUAL THEME

### 3.1 Student Theme Colors

Primary #4F46E5, primaryDeep #3730A3, primaryLight #EEF2FF. Accent #06B6D4, accent2 #F59E0B, accent3 #10B981, accent4 #EF4444, accent5 #8B5CF6. Background #F0F2FF, surface #FFFFFF, card #FAFAFA. Text #0F172A, text2 #475569, text3 #94A3B8. Border #E2E8F0. Success #059669, successBg #ECFDF5, warning #D97706, warningBg #FEF3C7, error #EF4444, errorBg #FEF2F2. Screen gradients: dashboard #4F46E5 to #302B63, timetable #1E40AF to #1D4ED8, results #4F46E5 to #6366F1, exams #134E4A to #0F766E, fees #065F46 to #059669, notices #92400E to #D97706, homework #BE185D to #DB2777, transport #0C4A6E to #0369A1, events #831843 to #BE185D, achievements #78350F to #B45309, attendance #1D4ED8 to #3B82F6, profile #0F172A to #1E293B, chat #0F0C29 to #302B63, library #3B0764 to #6D28D9, leaderboard #4F46E5 to #7C3AED, liveClass #EF4444 to #DC2626, settings #1E293B to #0F172A, messaging #4F46E5 to #7C3AED, leave #4F46E5 to #06B6D4, courses #4F46E5 to #302B63, splash #0F0C29 to #302B63 to #24243E.

### 3.2 Teacher Theme Colors

Primary #0EA5E9, primaryDeep #0369A1, primaryLight #E0F2FE. Background #F0F7FF. Screen gradients: dashboard #0C4A6E to #0EA5E9, myClasses #0C4A6E to #0EA5E9, attendance #065F46 to #059669, timetable #1E40AF to #3B82F6, homework #BE185D to #DB2777, reviewSub #9F1239 to #E11D48, exams #134E4A to #0F766E, gradebook #4F46E5 to #6366F1, studentDir #92400E to #D97706, leave #14532D to #16A34A, notices #92400E to #D97706, liveClasses #9F1239 to #E11D48, materials #065F46 to #059669, salary #065F46 to #059669, profile #0C4A6E to #0EA5E9, settings #1E293B to #0F172A, messaging #4F46E5 to #7C3AED, notifications #0C4A6E to #0EA5E9, splash #0C4A6E to #0369A1 to #0EA5E9.

### 3.3 Spacing System

Padding: XS 4, S 8, M 12, L 14, XL 18, XXL 24. Border radius: S 6, M 8, L 10, XL 12, XXL 14, Card 16, CardL 18, CardXL 20, Modal 24, Chip 20, Avatar 50. Component sizes: buttonHeight 48, inputHeight 44, fabSize 46, bottomNavHeight 60.

### 3.4 Typography

Heading font: Outfit. Body font: DM Sans.

### 3.5-3.7 Gradients, Shadows, and Themes

Student primary gradient #4F46E5 to #06B6D4. Teacher primary gradient #0EA5E9 to #06B6D4. Card shadow rgba black 0.05 blur 12 offset 0 2. Elevated shadow rgba primary 0.4 blur 20 offset 0 6. Complete ThemeData configurations for both student and teacher with Material 3, input decoration, elevated button, card, and app bar themes.

---

## 4. COMPLETE DATABASE SCHEMA (SQL)

33 tables total including profiles, subjects, timetable, courses, results, exams, exam_questions, exam_submissions, exam_sessions, attendance, fees, payments, salary, homework, homework_submissions, notices, events, event_registrations, chat_groups, chat_group_members, messages, notifications, leave_applications, achievements, student_achievements, library_books, library_borrows, bus_routes, bus_stops, bus_locations, live_classes, live_class_comments, study_materials, user_settings, documents, grading_policies, ai_chat_history. Full CREATE TABLE statements with all columns, constraints, foreign keys, and indexes. Complete RLS policies for all tables. Realtime enabled for bus_locations, messages, live_classes, notifications, exam_sessions, live_class_comments, attendance. Seed data for subjects and achievements.

---

## 5. AI FUNCTIONS (LANGCHAIN TOOL CALLING) — DEEP DETAIL

### 5.1 Overview

40 tools total: 20 for students, 20 for teachers. Each tool is a Dart class extending Tool from langchain package. Tools query Supabase for real-time data and return formatted strings to the AI agent.

### 5.2 Student AI Tools (20 tools)

1. get_timetable: Fetch class schedule for specific day or whole week. 2. get_homework: Fetch pending, submitted, or graded homework. 3. get_fee_status: Check fee balance and payment history. 4. get_attendance: Get attendance percentage overall and per subject. 5. get_exam_info: Get upcoming exam schedule and details. 6. get_bus_location: Real-time bus location and ETA. 7. get_performance: Academic performance summary. 8. generate_study_plan: Create personalized study plan. 9. explain_concept: Explain an academic concept. 10. get_notifications: Recent alerts and notifications. 11. get_library_status: Borrowed books and due dates. 12. get_achievements: Badges, XP, and rankings. 13. get_events: Upcoming school events. 14. get_leaderboard: Class and school rankings. 15. submit_homework: Submit homework assignment. 16. apply_leave: Apply for leave. 17. get_notices: School notices and announcements. 18. answer_general: General school information Q and A. 19. generate_practice: Generate practice questions. 20. get_live_class: Live class status and schedule.

### 5.3 Teacher AI Tools (20 tools)

1. get_class_students: List all students in a class. 2. get_class_performance: Class performance statistics. 3. get_at_risk_students: Identify struggling students. 4. generate_questions: Generate exam questions. 5. generate_lesson_plan: Create a lesson plan. 6. auto_grade_homework: Auto-grade homework submissions. 7. get_submission_status: Check homework submission stats. 8. generate_remedial_plan: Create remedial sessions for weak students. 9. create_notice: Draft and create a notice. 10. get_attendance_stats: Attendance analytics and trends. 11. get_teacher_schedule: Teacher daily timetable. 12. get_pending_tasks: Pending reviews and tasks. 13. generate_report: Generate class performance report. 14. explain_pedagogy: Teaching strategies and methods. 15. get_leave_balance: Leave status and balance. 16. get_salary_info: Salary details and breakup. 17. get_parent_communications: Parent messages and concerns. 18. upload_material: Upload study material. 19. get_exam_analytics: Exam statistics and pass rates. 20. answer_general: General school information Q and A.

### 5.4 Complete Dart Implementation for All 20 Student Tools

Each tool class extends Tool with name and description. The runInternal method queries Supabase, processes data, and returns formatted strings. Full implementations provided for all 20 tools including get_timetable, get_homework, get_fee_status, get_attendance, get_exam_info, get_bus_location, get_performance, generate_study_plan, explain_concept, get_notifications, get_library_status, get_achievements, get_events, get_leaderboard, submit_homework, apply_leave, get_notices, answer_general, generate_practice, get_live_class.

### 5.5 Complete Dart Implementation for All 20 Teacher Tools

Full implementations for all 20 teacher tools including get_class_students, get_class_performance, get_at_risk_students, generate_questions, generate_lesson_plan, auto_grade_homework, get_submission_status, generate_remedial_plan, create_notice, get_attendance_stats, get_teacher_schedule, get_pending_tasks, generate_report, explain_pedagogy, get_leave_balance, get_salary_info, get_parent_communications, upload_material, get_exam_analytics, answer_general.

### 5.6 _getStudentTools() Method

Returns List of Tool containing all 20 student tool instances.

### 5.7 _getTeacherTools() Method

Returns List of Tool containing all 20 teacher tool instances.

### 5.8 AI Service Setup

AIService class with ChatOpenAI, two AgentExecutors (student and teacher). Initialize method creates LLM with gpt-4o model, creates both agents with their respective tools. sendMessage method selects agent based on role, invokes with system prompt and user message, saves to ai_chat_history table. System prompts include student info (name, class, streak, XP) and teacher info (name, department, designation).

---

## 6. COMPLETE FOLDER STRUCTURE

lib directory contains main.dart, app.dart, app_router.dart. core directory with constants (student_colors, teacher_colors, app_fonts, app_dimensions, app_strings, app_gradients, app_shadows), theme (student_theme, teacher_theme), utils (date_formatter, currency_formatter, validators, responsive_utils, role_utils), services (supabase, auth, storage, notification, realtime, ai with tools subdirectory containing student and teacher tool files, prompts, models, maps with osm and bus_tracking), providers (auth, user, role, theme, ai), models (21 model files for all database entities). shared directory with widgets (20 shared widget files) and animations (fade_slide, scale, confetti). features directory with shared (splash, login, settings, messaging, ai_chat), student (15 feature modules each with screens and widgets subdirectories), teacher (16 feature modules each with screens and widgets subdirectories). l10n directory with app_en.arb and app_hi.arb.

---

## 7. CORE DART FILES

### 7.1 main.dart

Initialize WidgetsFlutterBinding, SupabaseService, SystemChrome orientation, run EduShamiitApp.

### 7.2 app.dart

MaterialApp.router with title EduSHAMIIT AI, debug banner off, theme, router config.

### 7.3 app_router.dart

GoRouter with initial location splash. Shared routes: splash, login, settings, ai-chat, messaging, chat-detail, create-group. Student routes 22 additional: dashboard, timetable, results, exams, exam-instructions, online-exam, exam-success, fees, notices, homework, transport, events, achievements, attendance, leave, profile, library, courses, notifications, live-classes, live-class-detail, leaderboard. Teacher routes 22 additional: dashboard, my-classes, class-detail, attendance, timetable, homework, review-submissions, grading, exams, paper-builder, gradebook, grading-config, student-directory, leave, notices, live-classes, live-session, live-playback, materials, salary, profile, notifications, exam-attendance, grade-student.

---

## 8. STUDENT SCREENS (29 SCREENS)

Each screen includes route, background color, header gradient with exact hex values, body sections top to bottom with pixel values, interactive elements, modals, bottom nav state, AI FAB presence.

8.1 Splash: Route /splash. Background #0F0C29 to #302B63 to #24243E. Logo 90x90 rgba white 0.1 border radius 28 spinning ring. Title EduSHAMIIT Outfit 36 weight 900 white with AI gradient. Subtitle AI-Powered School ERP 11px rgba white 0.4. Loading bar 120x3 fill 1.5s. Auto navigate 2s to login.

8.2 Login: Route /login. Dark gradient. Role selector 3 cards Student Parent Teacher. Active card rgba(79,70,229,0.25). Input fields dark theme. Login button gradient. AI tag Shami monitors. Alt login Face ID OTP QR. Modals Biometric OTP QR.

8.3 Dashboard: Route /student/dashboard. Background #F5F6FF. Header gradient #4F46E5 to #302B63 with greeting notification bell badge 3 avatar streak 18 days stats 94% attendance 91.4 score 3rd rank 2450 XP. Quick Access 4x4 grid 16 items with specific bg colors. Today Schedule 3 class rows. AI Insight card purple bordered. Pending Homework 2 items. AI FAB purple gradient pulse animation. Bottom nav Home active.

8.4 Timetable: Route /student/timetable. Background #F0F4FF. Header gradient #1E40AF to #1D4ED8. Day chips horizontal scroll. Month navigator. Class cards left 4px color strip time subject teacher room. Break cards amber. NOW badge green.

8.5 Results: Route /student/results. Background #F8FAFC. Header gradient #4F46E5 to #6366F1. Category chips 7 options. Overall score 52px gradient text grade medal stats. Subject analytics. Download button. Modals Year picker Download Share.

8.6 Exams: Route /student/exams. Background #F0FDF9. Header gradient #134E4A to #0F766E. Countdown days hours mins. AI Exam Prep Tip. Exam schedule list. Online Exam Portal card.

8.7 Exam Instructions: Route /student/exam-instructions. Background #F8FAFC. Dark header. Exam icon title. Instructions 5 bullet points. AI Integrity Check. Sticky Start button.

8.8 Online Exam: Route /student/online-exam. Locked mode. Dark top bar timer. Section switcher. Question palette 8 columns color states. MCQ options. Subjective textarea. Bottom bar Exit Flag Prev Next Submit.

8.9 Exam Success: Route /student/exam-success. Background #F0FDF4. Confetti 50 pieces. Success check animation. Result card submission ID stats AI verdict.

8.10 Fees: Route /student/fees. Background #F0FDF9. Header gradient #065F46 to #059669. Outstanding balance Pay Now. Filter chips. Fee items status badges. AI Reminder.

8.11 Notices: Route /student/notices. Background #FFFBEB. Header gradient #92400E to #D97706. Filter chips. Notice cards category badge title content date.

8.12 Homework: Route /student/homework. Background #FFF0F8. Header gradient #BE185D to #DB2777. Tabs Pending Submitted Graded. Homework cards subject badge title due date. AI Helper card.

8.13 Transport: Route /student/transport. Background #F0F8FF. Header gradient #0C4A6E to #0369A1 LIVE badge. Route chips. SVG mock map animated bus. Bus info card ETA students stops. Route stops timeline.

8.14 Events: Route /student/events. Background #FFF1F5. Header gradient #831843 to #BE185D. Filter chips. Event cards banner gradient title info Register button.

8.15 Achievements: Route /student/achievements. Background #FFFBEB. Header gradient #78350F to #B45309. XP summary card. Badges earned locked.

8.16 Attendance: Route /student/attendance. Background #EFF6FF. Header gradient #1D4ED8 to #3B82F6. SVG ring chart. Stats row. Subject attendance progress bars. Alert card.

8.17 Leave: Route /student/leave. Background #F0F4FF. Header gradient #4F46E5 to #06B6D4. Leave dashboard stats. Tabs Upcoming Past. Apply form. Leave cards status badges.

8.18 Profile: Route /student/profile. Background #F5F6FF. Header gradient #0F172A to #1E293B. Avatar name stats. Info sections Personal Father Mother Documents. Edit Profile modal.

8.19 AI Chat: Route /ai-chat. Background #020617. Dark theme Shami avatar. Chat bubbles AI white user gradient. Typing indicator. Suggestion chips. Input bar.

8.20 Library: Route /student/library. Background #FAF5FF. Header gradient #3B0764 to #6D28D9. Search filter chips. Book cards cover title author due date Renew button. AI Picks card.

8.21 Courses: Route /student/courses. Background #F0F4FF. Header gradient #4F46E5 to #302B63. Course cards icon name teacher score progress bar Play Recording button.

8.22 Notifications: Route /student/notifications. Background #F5F6FF. Header gradient #4F46E5 to #302B63. Notification cards left border icon title body time unread dot.

8.23 Settings: Route /settings. Background #F1F5F9. Header gradient #1E293B to #0F172A. Notifications AI Privacy Appearance Account sections with toggles Logout.

8.24 Messaging: Route /messaging. Background #F8FAFC. Header gradient #4F46E5 to #7C3AED. Search chips. Chat threads avatar name last message time unread badge.

8.25 Chat Detail: Route /chat-detail. Header #4F46E5 with avatar name status. Messages area. Compose bar.

8.26 Create Group: Route /create-group. Background #F8FAFC. Header gradient #4F46E5 to #7C3AED. Group icon name description participant selection.

8.27 Live Classes: Route /student/live-classes. Background #F8FAFC. Header gradient #EF4444 to #DC2626. LIVE NOW pulsing badge. Upcoming Today. Recorded Classes.

8.28 Live Class Detail: Route /student/live-class-detail. Background #0F172A. Video player 16:9 LIVE badge viewer count. Video info. Comments scrollable. Comment input.

8.29 Leaderboard: Route /student/leaderboard. Background #F0F4FF. Header gradient #4F46E5 to #7C3AED. Tabs Class School. Your Position card. Top 3 podium. Comparison chart. Rank list. AI Motivation card.

---

## 9. TEACHER SCREENS (29 SCREENS)

9.1 Dashboard: Route /teacher/dashboard. Background #F0F7FF. Header gradient #0C4A6E to #0EA5E9. Greeting Mrs. Priya Sharma Mathematics HOD. Stats 156 Students 92% Avg Attend 6 Classes 8 Pending. Quick Access 12 items 4x3 grid. Today Schedule 4 classes NOW badge. AI Teaching Insight cyan card. Pending Reviews 2 items. AI FAB cyan gradient.

9.2 My Classes: Route /teacher/my-classes. Background #E0F2FE. Header gradient #0C4A6E to #0EA5E9. Class cards left border color cyan purple amber student count stats. AI Class Insight card.

9.3 Class Detail: Route /teacher/class-detail. Background #F0F7FF. Header gradient #0C4A6E to #0EA5E9. Tabs Students Top Perf At Risk. Students list avatar name roll attendance. Top Performers podium. At Risk warning cards. Bottom buttons Mark Attendance Assign HW.

9.4 Attendance: Route /teacher/attendance. Background #F0FDF4. Header gradient #065F46 to #059669. Date class selector. All Present button. Student list P A L toggle. Submit button count.

9.5 Timetable: Route /teacher/timetable. Background #F0F4FF. Header gradient #1E40AF to #3B82F6. Date scroller month navigator calendar picker. Class cards room badge NOW badge.

9.6 Homework Manager: Route /teacher/homework. Background #FFF0F8. Header gradient #BE185D to #DB2777 Create button. Tabs Active Submissions Graded. Active cards class badge submission count Review Edit Remind.

9.7 Review Submissions: Route /teacher/review-submissions. Background #FFF5F7. Header gradient #9F1239 to #E11D48. Filter chips. Student tiles status badges. AI Grading Assist card.

9.8 Grading: Route /teacher/grading. Background white. Header student name progress. Submission viewer. AI Feedback suggested score. Score input remarks. Save Next Message.

9.9 Exams: Route /teacher/exams. Background #F0FDF9. Header gradient #134E4A to #0F766E Create button. Tabs Upcoming Past Results. Draft published exams AI Question Generator.

9.10 Paper Builder: Route /teacher/paper-builder. Background #F8FAFC. Header title Draft Publish. Progress card dark bg. AI Magic Generator textarea. Question blocks options. Add Objective Subjective.

9.11 Gradebook: Route /teacher/gradebook. Background #F8FAFC. Header gradient #4F46E5 to #6366F1 year picker settings. Category chips. Stats tiles. Performance Matrix 5 progress bars. Academic Ledger table. AI Performance Report. Export button.

9.12 Grading Configuration: Route /teacher/grading-config. Background #F8FAFC. Header bg #1E293B. Circular gauge 100%. Weightage cards sliders Mid-Term 30% Final-Term 40% Attendance 5% Assignments 10% Class Tests 10% Labs 5%. Apply button disabled until 100%.

9.13 Student Directory: Route /teacher/student-directory. Background #FFF7ED. Header gradient #92400E to #D97706. Search filter chips. Student cards avatar name call message. At-risk highlighted.

9.14 Leave: Route /teacher/leave. Background #F0FDF4. Header gradient #14532D to #16A34A Apply button. Quota dashboard. Leave history cards.

9.15 Notices: Route /teacher/notices. Background #FFFBEB. Header gradient #92400E to #D97706 Create button. Search filters. Notice cards category badge urgent markers.

9.16 Live Classes Dashboard: Route /teacher/live-classes. Background #FFF1F2. Header gradient #9F1239 to #E11D48. Tabs Live Now Upcoming Drafts Archive.

9.17 Teacher Live Session: Route /teacher/live-session. Background #0F172A. Full screen camera preview. LIVE badge viewer count REC indicator. Chat overlay. Floating toolbar Mute Video Screen Share Chat END.

9.18 Live Playback: Route /teacher/live-playback. Background #F8FAFC. Video player 16:9 progress bar controls. Video info. Live Chat Replay.

9.19 Study Materials: Route /teacher/materials. Background #F0FDFA. Header gradient #065F46 to #059669 Upload button. Filter chips. Material cards.

9.20 Salary: Route /teacher/salary. Background #F0FDF4. Header gradient #065F46 to #059669. Net salary card. Breakup table. Download Payslip.

9.21 Profile: Route /teacher/profile. Background #F0F7FF. Header gradient #0C4A6E to #0EA5E9. Avatar name designation. Stats card. Info sections. Quick links.

9.22 Settings: Route /settings. Background #F1F5F9. Header gradient #1E293B to #0F172A. Account Notifications AI Privacy Appearance More sections.

9.23 AI Chat: Route /ai-chat. Background #0F172A. Teacher-specific chips Generate Questions Lesson Plan Performance Report Remedial Plan. Same chat structure.

9.24 Notifications: Route /teacher/notifications. Background #F0F7FF. Header gradient #0C4A6E to #0EA5E9. Notification cards.

9.25 Exam Attendance: Route /teacher/exam-attendance. Background #F8FAFC. Header gradient #1E293B to #334155. Search bar. Student list.

9.26 Grade Student Response: Route /teacher/grade-student. Background white. Header gradient #1E293B to #334155. MCQ review. Subjective review marks input feedback. Submit button.

9.27-9.29 Messaging, Notices Create, Create Group: Same as shared screens with teacher-specific contacts and options.

---

## 10. SHARED REUSABLE COMPONENTS

AppBottomNav: Role-aware with student items Home Courses Results Achieve Profile and teacher items Home Classes Exams Messages Profile. AppHeader: Gradient header with role-specific colors back button trailing widget. AIFab: 46x46 circle gradient student purple or teacher cyan pulse animation 3s. AppCard: White bg border radius 16 shadow. AppButton: 4 variants primary gradient secondary light success green danger red. AppInputField: Border radius 12 focus states. AppChip: Active inactive states border radius 20. AppChipRow: Horizontal scroll. AppModal: Bottom sheet handle bar slide up 300ms border radius 24. AppToggle: iOS style 36x20 knob 16. AppBadge: Status badges PAID green DUE red PENDING amber. AppSectionHeader: Title with View All link. AITag: Robot emoji pill badge. AITypingIndicator: 3 bouncing dots. LiveDot: Blinking 6x6. SuccessCheck: Pop animation. CountdownTimer: Days hours mins display. CircularProgress: SVG circle. GradientContainer: Reusable wrapper. ResponsiveLayout: Breakpoint builder.

---

## 11. RESPONSIVE DESIGN STRATEGY

### 11.1 Breakpoints

Mobile less than 600dp. Tablet 600 to 900dp. Desktop greater than 900dp.

### 11.2 Responsive Rules

Navigation: Bottom bar on mobile, Navigation rail on tablet, Full sidebar on desktop. Layout: Single column mobile, Two columns tablet, Three to four columns desktop. Quick Access Grid: 4 columns mobile, 6 columns tablet, 8 columns desktop. Cards: Full width mobile, 50 percent tablet, 33 percent desktop. Modals: Bottom sheet mobile, Centered dialog tablet and desktop. Chat: Full screen mobile, Split view tablet and desktop. Exam Palette: Bottom grid mobile, Side panel tablet and desktop. Map: Embedded mobile, Larger tablet, Full panel desktop. Padding: 14px mobile, 24px tablet, 32px desktop.

### 11.3 Platform-Specific

Web: URL strategy, deep linking, keyboard navigation, hover effects, HtmlElementView for video. Android: Portrait lock for exams, platform channels for screenshot prevention, back button support. iOS: Safe area insets, Cupertino animations, iPad multitasking, notch handling. macOS: Window resizing, sidebar, keyboard shortcuts, menu bar. Linux: Window manager support, system theme, keyboard navigation. Windows: Window snapping, system theme, keyboard shortcuts, touch support.

---

## 12. NAVIGATION AND ROUTING

Use GoRouter with role-based routes. After login student navigates to /student/dashboard teacher to /teacher/dashboard. Navigation: context.push for forward, context.go for replace, context.pop for back. Deep linking supported for web with URLs.

---

## 13. DEPENDENCIES

flutter sdk, cupertino_icons, provider, flutter_riverpod, go_router, supabase_flutter, dio, shared_preferences, hive, hive_flutter, flutter_map, latlong2, langchain, langchain_openai, flutter_animate, lottie, shimmer, cached_network_image, fl_chart, pdf, printing, file_picker, image_picker, path_provider, url_launcher, flutter_local_notifications, local_auth, mobile_scanner, intl, uuid, equatable, json_annotation. Dev: flutter_test, flutter_lints, build_runner, json_serializable, hive_generator, mockito.

---

## 14. DEVELOPMENT PHASES

Phase 1 Foundation weeks 1-2: Project setup, LOCAL Supabase, fonts, dual theme, shared widgets, SQL migration, GoRouter.

Phase 2 Auth weeks 3-4: Splash, Login with role selector and 3 modals, role detection, Settings, AI Chat.

Phase 3 Student Core weeks 5-6: Dashboard, Timetable, Results, Courses, Attendance.

Phase 4 Student Exams weeks 7-8: Exams, Exam Instructions, Online Exam engine with 50 MCQ plus 10 Subjective, Exam Success.

Phase 5 Student Finance weeks 9-10: Fees with payment flow, Notices, Events.

Phase 6 Student Communication weeks 11-12: Messaging, Chat Detail, Create Group, Notifications, Live Classes.

Phase 7 Student Additional weeks 13-14: Transport with OSM, Library, Achievements, Leaderboard, Leave, Profile.

Phase 8 Teacher Core weeks 15-16: Dashboard, My Classes, Class Detail, Attendance, Timetable.

Phase 9 Teacher Grading weeks 17-18: Homework Manager, Review Submissions, Grading, Paper Builder, Gradebook, Grading Configuration.

Phase 10 Teacher Additional weeks 19-20: Exams, Student Directory, Leave, Notices, Live Classes, Materials, Salary, Profile.

Phase 11 AI weeks 21-22: LangChain setup, 40 tools implementation, Shami AI Chat for both roles, system prompts, chat history persistence.

Phase 12 Polish weeks 23-26: Responsive testing on all 6 platforms, performance optimization, animation polish, error handling, offline support with Hive caching, final QA and bug fixes.

---

## 15. CRITICAL IMPLEMENTATION NOTES

1. LOCAL Supabase: Use http://127.0.0.1:54321 for development. Switch to hosted Supabase URL for production deployment.

2. Role Detection: After login, query profiles.role from Supabase. Store in RoleProvider for global access. Role determines colors, nav items, dashboard layout, and which AI tools are loaded.

3. Dual Theming: Every shared component accepts a role parameter. Use StudentColors or TeacherColors based on current role. AI FAB, headers, buttons, and accents all change based on role.

4. Shami AI: Named AI agent using LangChain Dart with 40 tools total. Student agent has 20 tools for academic queries. Teacher agent has 20 tools for teaching management. Each tool queries Supabase for real-time data and returns formatted results.

5. Paper Builder: AI generates questions based on subject, difficulty level, and marks distribution using generate_questions tool. Teacher can edit generated questions and publish the paper.

6. Auto-Grading: Shami can auto-grade MCQ questions by comparing selected_answer with correct_answer in the database. For subjective questions, Shami provides a suggested score for teacher review before finalizing the grade.

7. Exam Integrity: During online exams, lock screen to portrait orientation, disable system back button, track WidgetsBindingObserver for app lifecycle changes, count screen focus switches for integrity score, and auto-submit when the timer hits zero.

8. Real-time: Use Supabase Realtime for bus_locations table for live bus tracking, messages table for real-time chat, live_classes table for live status updates, notifications table for instant alerts, exam_sessions table for timer synchronization, live_class_comments table for live comments, attendance table for real-time attendance updates.

9. Offline Support: Use Hive to cache timetable, homework list, profile data, and recent results. Show cached data when offline, sync when connection is restored.

10. File Uploads: Use Supabase Storage buckets: homework-files for homework submissions, documents for profile documents, exam-answers for exam answer sheets, chat-attachments for message files, study-materials for teacher uploads.

11. Gamification Rules: Homework submitted on time gives plus 50 XP. Daily login streak gives plus 10 XP per day. Exam score 90 percent or above gives plus 500 XP. Event participation gives plus 100 XP. 7-day streak bonus plus 50 XP. 14-day streak bonus plus 150 XP. 21-day streak bonus plus 300 XP plus badge. 30-day streak bonus plus 500 XP plus gold badge.

12. Grading Policy: Store weightage in grading_policies table. Recalculate all student totals when the policy changes. The circular gauge must show exactly 100 percent before the Apply button becomes enabled.

13. Web-Specific: Use HtmlElementView for video player on web. Enable URL strategy with GoRouter.optionURLReflectsImperativeAPIs equals true. Support deep linking for all routes.

14. Animation Durations: Screen transitions 400ms with cubic-bezier(0.4, 0, 0.2, 1). Modal slide up 300ms. Button press 150ms. AI FAB pulse 3000ms. Typing dots 1200ms. Confetti 3000ms. Splash bar fill 1500ms.

15. Image Assets: Use emojis as placeholders throughout development. For production, replace with SVG icons or custom image assets from a design team.

16. Status Bar Colors: Each screen has a specific status bar gradient matching its header. Use AnnotatedRegion with SystemUiOverlayStyle to set status bar color per screen to match the header gradient exactly.

17. Teacher Dashboard Stats: The values 156 Students, 92 percent Average Attendance, 6 Classes, 8 Pending tasks must come from real database queries, not hardcoded values. Query profiles table for student count, attendance table for attendance percentage, timetable table for class count, and homework_submissions plus leave_applications for pending count.

18. Live Classes: Teacher can go live early before scheduled time, schedule for later, save as draft, or watch past recordings. Students see live classes with pulsing LIVE badge and viewer count, upcoming classes with time countdown, and recorded classes with play button. Comments are real-time via Supabase Realtime on live_class_comments table.

19. Notice System: Teachers can create notices with categories (Urgent, General, Event, Academic), target audiences (specific classes or all students), urgent markers with red left border and URGENT badge, scheduled publishing for future dates, and file attachments. Students see filtered notice lists with category badges and can tap to view full notice details.

20. Salary Module: Teachers can view monthly salary with gross salary breakdown (Basic Pay, HRA, DA, Special Allowance) and deductions (PF, TDS, Professional Tax). Net salary is calculated and displayed prominently. Download payslip as PDF using pdf and printing packages.