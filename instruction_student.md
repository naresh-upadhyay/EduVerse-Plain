# EduVerse Student Portal — Complete Flutter Development Guide

> **PURPOSE**: This document is a self-contained blueprint. Any AI model or developer reading ONLY this file should be able to build the exact EduVerse Student Portal Flutter application from scratch, matching the provided HTML mockup pixel-perfectly across all 6 platforms (Web, Android, iOS, macOS, Linux, Windows).

---

## 📋 TABLE OF CONTENTS

1. [Project Overview](#1-project-overview)
2. [Step-by-Step Setup Guide](#2-step-by-step-setup-guide)
3. [Design System (Exact Values)](#3-design-system-exact-values)
4. [Database Schema (Supabase SQL)](#4-database-schema-supabase-sql)
5. [AI Functions (LangChain Tool Calling)](#5-ai-functions-langchain-tool-calling)
6. [Complete Folder Structure](#6-complete-folder-structure)
7. [Core Dart Files (Templates)](#7-core-dart-files-templates)
8. [Screen-by-Screen Specifications](#8-screen-by-screen-specifications)
9. [Reusable Components (Full Code)](#9-reusable-components-full-code)
10. [Responsive Design Strategy](#10-responsive-design-strategy)
11. [Navigation & Routing](#11-navigation--routing)
12. [Dependencies](#12-dependencies)
13. [Development Phases](#13-development-phases)
14. [Critical Implementation Notes](#14-critical-implementation-notes)

---

## 1. PROJECT OVERVIEW

### What This Is
EduVerse is an AI-powered School ERP Student Portal — a Flutter application that replicates an interactive HTML mockup with 29 screens. It uses Supabase as the backend, OpenStreetMap for transport tracking, and LangChain for an AI assistant with tool-calling capabilities.

### Target Platforms
- **Web** (Chrome, Firefox, Safari, Edge)
- **Android** (phone + tablet)
- **iOS** (iPhone + iPad)
- **macOS** (desktop)
- **Linux** (desktop)
- **Windows** (desktop)

### Key Features
1. Splash → Login (with Face ID, OTP, QR)
2. Dashboard with stats, quick access grid, schedule, AI insights
3. Timetable with day/month navigation
4. Results with category switching, year picker, report download
5. Exams with countdown, online exam engine (50 MCQ + 10 Subjective)
6. Fees with UPI/Card/NetBank payment flow
7. Notices, Events, Homework (Pending/Submitted/Graded)
8. Transport with live OSM bus tracking
9. Achievements, Leaderboard, Attendance
10. Leave management with approval workflow
11. Profile with full edit and document upload
12. AI Chat with 20 tool-calling functions
13. Library with renewals and digital books
14. Messaging with groups and real-time chat
15. Live classes with YouTube-style player and comments
16. Settings with toggles

---

## 2. STEP-BY-STEP SETUP GUIDE

### Step 1: Create Flutter Project
```bash
# Open terminal in your desired directory
flutter create --org com.eduverse eduverse_student
cd eduverse_student
```

### Step 2: Configure Project Name and Description
Edit `pubspec.yaml`:
```yaml
name: eduverse_student
description: EduVerse Student Portal - AI-Powered School ERP
publish_to: 'none'
version: 1.0.0+1
```

### Step 3: Add All Dependencies
Replace the `dependencies` section in `pubspec.yaml` with:
```yaml
dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^1.0.6
  
  # State Management
  provider: ^6.1.1
  flutter_riverpod: ^2.4.9
  
  # Routing
  go_router: ^13.0.0
  
  # Backend (Supabase)
  supabase_flutter: ^2.3.2
  
  # HTTP
  dio: ^5.4.0
  
  # Local Storage
  shared_preferences: ^2.2.2
  hive: ^2.2.3
  hive_flutter: ^1.1.0
  
  # Maps (OpenStreetMap)
  flutter_map: ^6.1.0
  latlong2: ^0.9.1
  
  # AI (LangChain)
  langchain: ^0.5.0
  langchain_openai: ^0.5.0
  
  # UI
  flutter_animate: ^4.3.0
  lottie: ^3.0.0
  shimmer: ^3.0.0
  cached_network_image: ^3.3.1
  
  # Charts
  fl_chart: ^0.66.0
  
  # PDF
  pdf: ^3.10.7
  printing: ^5.12.0
  
  # Files
  file_picker: ^6.1.1
  image_picker: ^1.0.7
  path_provider: ^2.1.2
  
  # URL
  url_launcher: ^6.2.3
  
  # Notifications
  flutter_local_notifications: ^17.0.0
  
  # Auth
  local_auth: ^2.1.8
  mobile_scanner: ^4.0.0
  
  # Utilities
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

### Step 4: Create Folder Structure
Run these commands from project root:
```bash
# Core
mkdir -p lib/core/constants
mkdir -p lib/core/theme
mkdir -p lib/core/utils
mkdir -p lib/core/services/ai/tools
mkdir -p lib/core/services/ai/prompts
mkdir -p lib/core/services/ai/models
mkdir -p lib/core/services/maps
mkdir -p lib/core/providers
mkdir -p lib/core/models

# Shared
mkdir -p lib/shared/widgets
mkdir -p lib/shared/animations

# Features
mkdir -p lib/features/auth/screens
mkdir -p lib/features/auth/widgets
mkdir -p lib/features/dashboard/screens
mkdir -p lib/features/dashboard/widgets
mkdir -p lib/features/academics/timetable/screens
mkdir -p lib/features/academics/timetable/widgets
mkdir -p lib/features/academics/results/screens
mkdir -p lib/features/academics/results/widgets
mkdir -p lib/features/academics/courses/screens
mkdir -p lib/features/academics/courses/widgets
mkdir -p lib/features/academics/attendance/screens
mkdir -p lib/features/academics/attendance/widgets
mkdir -p lib/features/examination/screens
mkdir -p lib/features/examination/widgets
mkdir -p lib/features/finance/screens
mkdir -p lib/features/finance/widgets
mkdir -p lib/features/notices_events/notices/screens
mkdir -p lib/features/notices_events/notices/widgets
mkdir -p lib/features/notices_events/events/screens
mkdir -p lib/features/notices_events/events/widgets
mkdir -p lib/features/homework/screens
mkdir -p lib/features/homework/widgets
mkdir -p lib/features/transport/screens
mkdir -p lib/features/transport/widgets
mkdir -p lib/features/profile/screens
mkdir -p lib/features/profile/widgets
mkdir -p lib/features/leave/screens
mkdir -p lib/features/leave/widgets
mkdir -p lib/features/achievements/screens
mkdir -p lib/features/achievements/widgets
mkdir -p lib/features/leaderboard/screens
mkdir -p lib/features/leaderboard/widgets
mkdir -p lib/features/ai_chat/screens
mkdir -p lib/features/ai_chat/widgets
mkdir -p lib/features/library/screens
mkdir -p lib/features/library/widgets
mkdir -p lib/features/notifications/screens
mkdir -p lib/features/notifications/widgets
mkdir -p lib/features/settings/screens
mkdir -p lib/features/settings/widgets
mkdir -p lib/features/messaging/screens
mkdir -p lib/features/messaging/widgets
mkdir -p lib/features/live_classes/screens
mkdir -p lib/features/live_classes/widgets
```

### Step 5: Add Fonts
Create `assets/fonts/` directory and download:
- Outfit (all weights 300-900) from Google Fonts
- DM Sans (all weights 300-700) from Google Fonts

Add to `pubspec.yaml`:
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

### Step 6: Initialize Supabase
1. Go to https://supabase.com and create a new project
2. Copy the project URL and anon key
3. Run the SQL migration scripts (see Section 4)
4. Enable Realtime for tables listed in Section 4

### Step 7: Configure Supabase
Create `lib/core/services/supabase_service.dart`:
```dart
import 'package:supabase_flutter/supabase_flutter.dart';

class SupabaseService {
  static const String supabaseUrl = 'YOUR_SUPABASE_URL';
  static const String supabaseAnonKey = 'YOUR_SUPABASE_ANON_KEY';

  static Future<void> initialize() async {
    await Supabase.initialize(
      url: supabaseUrl,
      anonKey: supabaseAnonKey,
    );
  }

  static SupabaseClient get client => Supabase.instance.client;
  static GoTrueClient get auth => Supabase.instance.client.auth;
}
```

### Step 8: Create main.dart
```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:eduverse_student/app.dart';
import 'package:eduverse_student/core/services/supabase_service.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // Initialize Supabase
  await SupabaseService.initialize();
  
  // Lock orientation for mobile (optional)
  await SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  
  runApp(const EduVerseApp());
}
```

### Step 9: Create app.dart
```dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/theme/app_theme.dart';
import 'package:eduverse_student/app_router.dart';

class EduVerseApp extends StatelessWidget {
  const EduVerseApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'EduVerse',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      themeMode: ThemeMode.light,
      routerConfig: goRouter,
    );
  }
}
```

### Step 10: Run and Verify
```bash
flutter pub get
flutter run -d chrome    # For web
flutter run              # For mobile/emulator
```

---

## 3. DESIGN SYSTEM (EXACT VALUES)

### 3.1 Color Palette (EXACT hex values from mockup)
```dart
// lib/core/constants/app_colors.dart
import 'package:flutter/material.dart';

class AppColors {
  // Primary
  static const Color primary = Color(0xFF4F46E5);
  static const Color primaryDeep = Color(0xFF3730A3);
  static const Color primaryLight = Color(0xFFEEF2FF);
  
  // Accents
  static const Color accent = Color(0xFF06B6D4);      // Cyan
  static const Color accent2 = Color(0xFFF59E0B);     // Amber
  static const Color accent3 = Color(0xFF10B981);     // Emerald
  static const Color accent4 = Color(0xFFEF4444);     // Red
  static const Color accent5 = Color(0xFF8B5CF6);     // Purple
  
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
  
  // Specific screen colors (from mockup)
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
  
  // Subject colors (from mockup)
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
  
  // Quick access grid colors
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

### 3.2 Spacing System
```dart
// lib/core/constants/app_dimensions.dart
class AppDimensions {
  // Padding
  static const double paddingXS = 4.0;
  static const double paddingS = 8.0;
  static const double paddingM = 12.0;
  static const double paddingL = 14.0;    // Standard horizontal padding from mockup
  static const double paddingXL = 18.0;
  static const double paddingXXL = 24.0;
  
  // Margin
  static const double marginXS = 4.0;
  static const double marginS = 8.0;
  static const double marginM = 10.0;
  static const double marginL = 12.0;
  static const double marginXL = 14.0;
  
  // Border Radius
  static const double radiusS = 6.0;
  static const double radiusM = 8.0;
  static const double radiusL = 10.0;
  static const double radiusXL = 12.0;    // Input fields
  static const double radiusXXL = 14.0;   // Buttons
  static const double radiusCard = 16.0;  // Cards
  static const double radiusCardL = 18.0; // Larger cards
  static const double radiusCardXL = 20.0; // Performance cards
  static const double radiusModal = 24.0; // Modal sheets
  static const double radiusChip = 20.0;  // Chips
  static const double radiusAvatar = 50.0; // Circular
  
  // Icon Sizes
  static const double iconXS = 12.0;
  static const double iconS = 14.0;
  static const double iconM = 16.0;
  static const double iconL = 18.0;
  static const double iconXL = 20.0;
  static const double iconXXL = 24.0;
  static const double iconAvatar = 38.0;
  static const double iconLarge = 42.0;
  static const double iconHuge = 60.0;
  
  // Font Sizes
  static const double fontXS = 9.0;
  static const double fontS = 10.0;
  static const double fontM = 11.0;
  static const double fontL = 12.0;
  static const double fontXL = 13.0;
  static const double fontXXL = 14.0;
  static const double fontTitle = 15.0;
  static const double fontHeading = 17.0;
  static const double fontLarge = 18.0;
  static const double fontHuge = 22.0;
  static const double fontMassive = 28.0;
  static const double fontGiant = 36.0;
  static const double fontEnormous = 52.0;
  
  // Component Sizes
  static const double buttonHeight = 48.0;
  static const double inputHeight = 44.0;
  static const double chipHeight = 28.0;
  static const double bottomNavHeight = 60.0;
  static const double fabSize = 46.0;
  static const double statusBarHeight = 44.0;
  static const double headerHeight = 56.0;
  static const double cardMinHeight = 80.0;
  static const double toggleWidth = 36.0;
  static const double toggleHeight = 20.0;
  static const double toggleKnob = 16.0;
  
  // Shadows
  static const double shadowBlur = 12.0;
  static const double shadowOffset = 2.0;
  static const double elevatedShadowBlur = 20.0;
  static const double elevatedShadowOffset = 6.0;
}
```

### 3.3 Typography
```dart
// lib/core/constants/app_fonts.dart
import 'package:flutter/material.dart';

class AppFonts {
  static const String heading = 'Outfit';
  static const String body = 'DM Sans';
  
  static TextStyle headingStyle({
    double fontSize = 14,
    FontWeight fontWeight = FontWeight.w700,
    Color color = const Color(0xFF0F172A),
  }) => TextStyle(
    fontFamily: heading,
    fontSize: fontSize,
    fontWeight: fontWeight,
    color: color,
  );
  
  static TextStyle bodyStyle({
    double fontSize = 12,
    FontWeight fontWeight = FontWeight.w400,
    Color color = const Color(0xFF0F172A),
  }) => TextStyle(
    fontFamily: body,
    fontSize: fontSize,
    fontWeight: fontWeight,
    color: color,
  );
}
```

### 3.4 Gradient Definitions
```dart
// lib/core/constants/app_gradients.dart
import 'package:flutter/material.dart';

class AppGradients {
  static const LinearGradient primary = LinearGradient(
    colors: [Color(0xFF4F46E5), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  
  static const LinearGradient darkHeader = LinearGradient(
    colors: [Color(0xFF4F46E5), Color(0xFF302B63)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  
  static const LinearGradient success = LinearGradient(
    colors: [Color(0xFF059669), Color(0xFF06B6D4)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  
  static const LinearGradient warning = LinearGradient(
    colors: [Color(0xFF134E4A), Color(0xFF0F766E)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  
  static const LinearGradient pink = LinearGradient(
    colors: [Color(0xFFBE185D), Color(0xFFDB2777)],
    begin: Alignment.topLeft,
    end: Alignment.bottomRight,
  );
  
  static const LinearGradient purple = LinearGradient(
    colors: [Color(0xFF3B0764), Color(0xFF6D28D9)],
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

### 3.5 Theme Configuration
```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';

class AppTheme {
  static ThemeData lightTheme = ThemeData(
    useMaterial3: true,
    brightness: Brightness.light,
    primaryColor: AppColors.primary,
    scaffoldBackgroundColor: AppColors.scaffoldBackground,
    fontFamily: AppFonts.body,
    colorScheme: ColorScheme.light(
      primary: AppColors.primary,
      secondary: AppColors.accent,
      error: AppColors.error,
      surface: AppColors.surface,
    ),
    appBarTheme: AppBarTheme(
      backgroundColor: AppColors.surface,
      elevation: 0,
      titleTextStyle: AppFonts.headingStyle(fontSize: 17),
      iconTheme: const IconThemeData(color: AppColors.text),
    ),
    cardTheme: CardTheme(
      color: AppColors.surface,
      elevation: 0,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusCard),
      ),
      margin: const EdgeInsets.symmetric(
        horizontal: AppDimensions.paddingL,
        vertical: AppDimensions.marginS,
      ),
    ),
    inputDecorationTheme: InputDecorationTheme(
      filled: true,
      fillColor: const Color(0xFFF8FAFC),
      border: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: AppColors.border, width: 1.5),
      ),
      enabledBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: AppColors.border, width: 1.5),
      ),
      focusedBorder: OutlineInputBorder(
        borderRadius: BorderRadius.circular(AppDimensions.radiusXL),
        borderSide: const BorderSide(color: AppColors.primary, width: 1.5),
      ),
      contentPadding: const EdgeInsets.symmetric(
        horizontal: AppDimensions.paddingL,
        vertical: AppDimensions.paddingM,
      ),
      hintStyle: AppFonts.bodyStyle(
        fontSize: 13,
        color: AppColors.text3,
      ),
    ),
    elevatedButtonTheme: ElevatedButtonThemeData(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.primary,
        foregroundColor: Colors.white,
        elevation: 0,
        padding: const EdgeInsets.symmetric(vertical: 14),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(AppDimensions.radiusXXL),
        ),
        textStyle: AppFonts.headingStyle(
          fontSize: 15,
          fontWeight: FontWeight.w700,
          color: Colors.white,
        ),
      ),
    ),
  );
  
  static ThemeData darkTheme = ThemeData(
    useMaterial3: true,
    brightness: Brightness.dark,
    primaryColor: AppColors.primary,
    fontFamily: AppFonts.body,
  );
}
```

### 3.6 Box Shadows
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

---

## 4. DATABASE SCHEMA (SUPABASE SQL)

Run these SQL scripts in Supabase SQL Editor in order:

### 4.1 Enable Extensions
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
```

### 4.2 Core Tables
```sql
-- PROFILES (extends auth.users)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  student_id TEXT UNIQUE NOT NULL,
  full_name TEXT NOT NULL,
  role TEXT NOT NULL DEFAULT 'student' CHECK (role IN ('student', 'parent', 'teacher')),
  class TEXT,
  roll_number INT,
  gender TEXT CHECK (gender IN ('Male', 'Female', 'Other')),
  date_of_birth DATE,
  blood_group TEXT CHECK (blood_group IN ('A+','A-','B+','B-','O+','O-','AB+','AB-')),
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
  teacher_name TEXT,
  teacher_id UUID REFERENCES profiles(id),
  total_chapters INT DEFAULT 0,
  class TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- TIMETABLE
CREATE TABLE timetable (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
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
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  syllabus_coverage DECIMAL(5,2) DEFAULT 0 CHECK (syllabus_coverage BETWEEN 0 AND 100),
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
  title TEXT NOT NULL,
  exam_type TEXT NOT NULL CHECK (exam_type IN ('offline','online','hybrid')),
  exam_date DATE,
  start_time TIME,
  duration_minutes INT DEFAULT 90,
  total_marks INT DEFAULT 100,
  venue TEXT,
  instructions TEXT,
  status TEXT DEFAULT 'upcoming' CHECK (status IN ('upcoming','ongoing','completed','graded')),
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
  date DATE NOT NULL,
  status TEXT NOT NULL CHECK (status IN ('present','absent','late','excused')),
  marked_by UUID REFERENCES profiles(id),
  remarks TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
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
  payment_gateway TEXT,
  gateway_order_id TEXT,
  gateway_payment_id TEXT,
  paid_at TIMESTAMPTZ,
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
  attachment_url TEXT,
  reference_material TEXT,
  rubric TEXT,
  class TEXT NOT NULL,
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
  max_marks DECIMAL(5,2),
  teacher_remarks TEXT,
  graded_at TIMESTAMPTZ
);
```

### 4.5 Communication Tables
```sql
-- NOTICES
CREATE TABLE notices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN ('urgent','general','event')),
  author_name TEXT,
  author_role TEXT,
  target_audience TEXT DEFAULT 'all',
  attachment_url TEXT,
  is_pinned BOOLEAN DEFAULT FALSE,
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
  status TEXT DEFAULT 'registered' CHECK (status IN ('registered','attended','cancelled')),
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
  priority TEXT DEFAULT 'normal' CHECK (priority IN ('low','normal','high','urgent')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.6 Leave Tables
```sql
-- LEAVE APPLICATIONS
CREATE TABLE leave_applications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  leave_type TEXT NOT NULL CHECK (leave_type IN ('sick','casual','urgent','family_event','other')),
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

### 4.7 Achievement Tables
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
  rarity TEXT DEFAULT 'common' CHECK (rarity IN ('common','uncommon','rare','legendary')),
  is_locked_default BOOLEAN DEFAULT FALSE,
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
-- LIBRARY BOOKS
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
  class TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIBRARY BORROWS
CREATE TABLE library_borrows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  book_id UUID REFERENCES library_books(id) ON DELETE CASCADE,
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  borrowed_at TIMESTAMPTZ DEFAULT NOW(),
  due_at TIMESTAMPTZ NOT NULL,
  returned_at TIMESTAMPTZ,
  renewals_used INT DEFAULT 0,
  max_renewals INT DEFAULT 2,
  status TEXT DEFAULT 'borrowed' CHECK (status IN ('borrowed','returned','overdue')),
  fine_amount DECIMAL(10,2) DEFAULT 0
);
```

### 4.9 Transport Tables
```sql
-- BUS ROUTES
CREATE TABLE bus_routes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_name TEXT NOT NULL,
  bus_number TEXT,
  driver_name TEXT,
  driver_phone TEXT,
  total_capacity INT DEFAULT 40,
  current_passengers INT DEFAULT 0,
  status TEXT DEFAULT 'active' CHECK (status IN ('active','inactive','maintenance')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- BUS STOPS
CREATE TABLE bus_stops (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  route_id UUID REFERENCES bus_routes(id) ON DELETE CASCADE,
  stop_name TEXT NOT NULL,
  latitude DECIMAL(10,8) NOT NULL,
  longitude DECIMAL(11,8) NOT NULL,
  stop_order INT NOT NULL,
  estimated_arrival TIME,
  is_student_stop BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- BUS LOCATIONS (Realtime)
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
-- LIVE CLASSES
CREATE TABLE live_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID REFERENCES subjects(id) ON DELETE CASCADE,
  teacher_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT,
  scheduled_at TIMESTAMPTZ NOT NULL,
  duration_minutes INT DEFAULT 60,
  stream_url TEXT,
  recording_url TEXT,
  is_live BOOLEAN DEFAULT FALSE,
  viewer_count INT DEFAULT 0,
  status TEXT DEFAULT 'scheduled' CHECK (status IN ('scheduled','live','completed','recorded')),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- LIVE CLASS COMMENTS
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

### 4.11 Settings & Documents Tables
```sql
-- STUDENT SETTINGS
CREATE TABLE student_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE UNIQUE,
  push_notifications BOOLEAN DEFAULT TRUE,
  sms_alerts BOOLEAN DEFAULT TRUE,
  email_reports BOOLEAN DEFAULT FALSE,
  ai_personalization BOOLEAN DEFAULT TRUE,
  biometric_login BOOLEAN DEFAULT TRUE,
  dark_mode BOOLEAN DEFAULT FALSE,
  language TEXT DEFAULT 'en',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- DOCUMENTS
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  document_type TEXT NOT NULL,
  file_url TEXT NOT NULL,
  file_name TEXT,
  verification_status TEXT DEFAULT 'pending' CHECK (verification_status IN ('pending','verified','rejected')),
  uploaded_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 4.12 Row Level Security (RLS)
```sql
-- Enable RLS on all tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE results ENABLE ROW LEVEL SECURITY;
ALTER TABLE attendance ENABLE ROW LEVEL SECURITY;
ALTER TABLE fees ENABLE ROW LEVEL SECURITY;
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;
ALTER TABLE homework_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_submissions ENABLE ROW LEVEL SECURITY;
ALTER TABLE exam_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE leave_applications ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notifications ENABLE ROW LEVEL SECURITY;
ALTER TABLE event_registrations ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_achievements ENABLE ROW LEVEL SECURITY;
ALTER TABLE library_borrows ENABLE ROW LEVEL SECURITY;
ALTER TABLE student_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Profile policies
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Results policies
CREATE POLICY "Students can view own results" ON results
  FOR SELECT USING (auth.uid() = student_id);

-- Attendance policies
CREATE POLICY "Students can view own attendance" ON attendance
  FOR SELECT USING (auth.uid() = student_id);

-- Fee policies
CREATE POLICY "Students can view own fees" ON fees
  FOR SELECT USING (auth.uid() = student_id);
CREATE POLICY "Students can view own payments" ON payments
  FOR SELECT USING (auth.uid() = student_id);

-- Homework policies
CREATE POLICY "Students can submit homework" ON homework_submissions
  FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Students can view own submissions" ON homework_submissions
  FOR SELECT USING (auth.uid() = student_id);

-- Exam policies
CREATE POLICY "Students can submit exams" ON exam_submissions
  FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Students can view own exam sessions" ON exam_sessions
  FOR SELECT USING (auth.uid() = student_id);

-- Leave policies
CREATE POLICY "Students can apply leave" ON leave_applications
  FOR INSERT WITH CHECK (auth.uid() = student_id);
CREATE POLICY "Students can view own leaves" ON leave_applications
  FOR SELECT USING (auth.uid() = student_id);

-- Message policies
CREATE POLICY "Users can send messages" ON messages
  FOR INSERT WITH CHECK (auth.uid() = sender_id);
CREATE POLICY "Users can view own messages" ON messages
  FOR SELECT USING (auth.uid() = sender_id OR auth.uid() = receiver_id);

-- Notification policies
CREATE POLICY "Users can view own notifications" ON notifications
  FOR SELECT USING (auth.uid() = user_id);

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
```

### 4.13 Enable Realtime
```sql
-- Enable realtime for specific tables
ALTER PUBLICATION supabase_realtime ADD TABLE bus_locations;
ALTER PUBLICATION supabase_realtime ADD TABLE messages;
ALTER PUBLICATION supabase_realtime ADD TABLE live_classes;
ALTER PUBLICATION supabase_realtime ADD TABLE notifications;
ALTER PUBLICATION supabase_realtime ADD TABLE exam_sessions;
ALTER PUBLICATION supabase_realtime ADD TABLE live_class_comments;
```

### 4.14 Seed Data
```sql
-- Insert sample subjects
INSERT INTO subjects (name, icon, color, teacher_name, total_chapters, class) VALUES
  ('Mathematics', '📐', '#4F46E5', 'Mr. R. Sharma', 42, 'X-A'),
  ('Physics', '⚛️', '#3B82F6', 'Dr. A. Verma', 38, 'X-A'),
  ('Chemistry', '⚗️', '#F59E0B', 'Dr. S. Mehta', 35, 'X-A'),
  ('English', '📖', '#10B981', 'Ms. P. Gupta', 28, 'X-A'),
  ('History', '🗺️', '#8B5CF6', 'Mrs. K. Rao', 24, 'X-A'),
  ('Computer Sci.', '💻', '#06B6D4', 'Mr. V. Jain', 30, 'X-A');

-- Insert sample achievements
INSERT INTO achievements (name, description, icon, badge_color, xp_reward, criteria, rarity) VALUES
  ('Academic Excellence', 'Scored 90%+ in term exams', '🏆', '["#FCD34D","#F59E0B"]', 500, 'Score 90%+ in any term examination', 'rare'),
  ('18-Day Streak', 'Logged in 18 consecutive days', '🔥', '["#60A5FA","#3B82F6"]', 300, 'Maintain login streak', 'uncommon'),
  ('Zero Late Submissions', 'All homework on time this term', '✅', '["#86EFAC","#22C55E"]', 200, 'Submit all homework before deadline', 'common');
```

---

## 5. AI FUNCTIONS (LANGCHAIN TOOL CALLING)

### 5.1 Tool Definitions

Each tool is a Dart class with `@Tool` annotation. The AI assistant calls these tools to fetch real-time data.

```dart
// lib/core/services/ai/tools/timetable_tool.dart
import 'package:langchain/langchain.dart';
import 'package:eduverse_student/core/services/supabase_service.dart';

class TimetableTool extends Tool {
  TimetableTool() : super(
    name: 'get_timetable',
    description: 'Get the student timetable for a specific day or the whole week. '
        'Use this when the student asks about their schedule, classes, or what they have today/tomorrow.',
  );

  @override
  Future<String> runInternal(String input) async {
    final day = input.toLowerCase().trim();
    final client = SupabaseService.client;
    final userId = client.auth.currentUser?.id;
    
    if (userId == null) return 'Error: User not authenticated';
    
    try {
      int? dayFilter;
      if (day == 'monday') dayFilter = 0;
      else if (day == 'tuesday') dayFilter = 1;
      else if (day == 'wednesday') dayFilter = 2;
      else if (day == 'thursday') dayFilter = 3;
      else if (day == 'friday') dayFilter = 4;
      else if (day == 'saturday') dayFilter = 5;
      
      var query = client
          .from('timetable')
          .select('*, subjects(name, icon, teacher_name)')
          .eq('class', 'X-A')
          .order('start_time');
      
      if (dayFilter != null) {
        query = query.eq('day_of_week', dayFilter);
      }
      
      final response = await query;
      return _formatTimetable(response);
    } catch (e) {
      return 'Error fetching timetable: $e';
    }
  }
  
  String _formatTimetable(List<dynamic> data) {
    if (data.isEmpty) return 'No classes scheduled.';
    
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
      } else {
        final subject = item['subjects'];
        buffer.writeln('  ${subject['icon']} ${subject['name']} - ${item['start_time']} to ${item['end_time']} (${item['room']}) - ${subject['teacher_name']}');
      }
    }
    
    return buffer.toString();
  }
}
```

### 5.2 All 20 Tools List

| # | Tool Name | Purpose | When AI Uses It |
|---|-----------|---------|-----------------|
| 1 | `get_timetable` | Fetch class schedule | "What classes do I have today?" |
| 2 | `get_homework` | Fetch homework status | "What homework is due?" |
| 3 | `get_fee_status` | Check fee balance | "How much fees do I owe?" |
| 4 | `get_attendance` | Get attendance % | "What's my attendance?" |
| 5 | `get_exam_info` | Get exam schedule | "When is my next exam?" |
| 6 | `get_bus_location` | Real-time bus ETA | "Where is my bus?" |
| 7 | `get_performance` | Academic summary | "How am I doing in studies?" |
| 8 | `generate_study_plan` | Create study plan | "Help me prepare for exams" |
| 9 | `explain_concept` | Explain topic | "Explain integration by parts" |
| 10 | `get_notifications` | Recent alerts | "Any new notifications?" |
| 11 | `get_library_status` | Borrowed books | "What books do I have?" |
| 12 | `get_achievements` | Badges & XP | "What badges have I earned?" |
| 13 | `get_events` | School events | "What events are coming up?" |
| 14 | `get_leaderboard` | Rankings | "What's my class rank?" |
| 15 | `submit_homework` | Submit assignment | "Submit my math homework" |
| 16 | `apply_leave` | Apply for leave | "I need sick leave tomorrow" |
| 17 | `get_notices` | School notices | "Any new announcements?" |
| 18 | `answer_general` | General Q&A | "What is the school timing?" |
| 19 | `generate_practice` | Practice questions | "Give me physics practice problems" |
| 20 | `get_live_class` | Live class status | "Is there a live class now?" |

### 5.3 AI Service Setup
```dart
// lib/core/services/ai/ai_service.dart
import 'package:langchain/langchain.dart';
import 'package:langchain_openai/langchain_openai.dart';
import 'timetable_tool.dart';
import 'homework_tool.dart';
// ... import all 20 tools

class AIService {
  late final ChatOpenAI _llm;
  late final AgentExecutor _agent;
  
  Future<void> initialize() async {
    _llm = ChatOpenAI(
      apiKey: 'YOUR_OPENAI_API_KEY',
      defaultOptions: const ChatOpenAIOptions(model: 'gpt-4o'),
    );
    
    final tools = [
      TimetableTool(),
      HomeworkTool(),
      FeeStatusTool(),
      AttendanceTool(),
      ExamInfoTool(),
      BusLocationTool(),
      PerformanceTool(),
      StudyPlanTool(),
      ConceptTool(),
      NotificationTool(),
      LibraryStatusTool(),
      AchievementTool(),
      EventTool(),
      LeaderboardTool(),
      SubmissionTool(),
      LeaveTool(),
      NoticeTool(),
      GeneralQATool(),
      PracticeTool(),
      LiveClassTool(),
    ];
    
    final agent = OpenAIFunctionsAgent.fromLLMAndTools(
      llm: _llm,
      tools: tools,
    );
    
    _agent = AgentExecutor(agent: agent);
  }
  
  Future<String> sendMessage(String message, {Map<String, dynamic>? context}) async {
    final systemPrompt = '''
You are EduVerse AI, an intelligent school assistant. Help students with academic queries, school information, and study planning. Use tools to get real-time data. Be friendly and use emojis.

Student: ${context?['name'] ?? 'Student'}
Class: ${context?['class'] ?? 'X-A'}
Streak: ${context?['streak'] ?? 0} days
XP: ${context?['xp'] ?? 0}
''';
    
    final result = await _agent.invoke({
      'input': message,
      'system_message': systemPrompt,
    });
    
    return result.toString();
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
│   │   ├── app_colors.dart          # All colors (Section 3.1)
│   │   ├── app_fonts.dart           # Typography (Section 3.3)
│   │   ├── app_dimensions.dart      # Spacing (Section 3.2)
│   │   ├── app_strings.dart         # All text strings
│   │   ├── app_gradients.dart       # Gradients (Section 3.4)
│   │   └── app_shadows.dart         # Shadows (Section 3.6)
│   ├── theme/
│   │   └── app_theme.dart           # ThemeData (Section 3.5)
│   ├── utils/
│   │   ├── date_formatter.dart      # DateFormat helpers
│   │   ├── currency_formatter.dart  # ₹ formatting
│   │   ├── validators.dart          # Form validators
│   │   └── responsive_utils.dart    # Screen size helpers
│   ├── services/
│   │   ├── supabase_service.dart    # Supabase client
│   │   ├── auth_service.dart        # Authentication
│   │   ├── storage_service.dart     # File uploads
│   │   ├── notification_service.dart# Push notifications
│   │   ├── realtime_service.dart    # Realtime subscriptions
│   │   ├── ai/
│   │   │   ├── ai_service.dart      # Main AI orchestrator
│   │   │   ├── langchain_config.dart# LangChain setup
│   │   │   ├── tools/               # 20 tool files
│   │   │   ├── prompts/
│   │   │   │   └── system_prompt.dart
│   │   │   └── models/
│   │   │       ├── ai_message.dart
│   │   │       ├── ai_tool_call.dart
│   │   │       └── ai_response.dart
│   │   └── maps/
│   │       ├── osm_service.dart     # OpenStreetMap config
│   │       └── bus_tracking_service.dart
│   ├── providers/
│   │   ├── auth_provider.dart       # Auth state
│   │   ├── user_provider.dart       # User data
│   │   ├── theme_provider.dart      # Dark/light mode
│   │   └── ai_provider.dart         # AI chat state
│   └── models/
│       ├── user_model.dart
│       ├── subject_model.dart
│       ├── timetable_model.dart
│       ├── result_model.dart
│       ├── exam_model.dart
│       ├── fee_model.dart
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
│       └── leaderboard_model.dart
│
├── shared/
│   ├── widgets/
│   │   ├── app_bottom_nav.dart
│   │   ├── app_header.dart
│   │   ├── ai_fab.dart
│   │   ├── app_card.dart
│   │   ├── app_button.dart
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
│   ├── auth/screens/{splash,login}_screen.dart + widgets/
│   ├── dashboard/screens/dashboard_screen.dart + widgets/
│   ├── academics/timetable/ + results/ + courses/ + attendance/
│   ├── examination/screens/ + widgets/
│   ├── finance/screens/ + widgets/
│   ├── notices_events/notices/ + events/
│   ├── homework/screens/ + widgets/
│   ├── transport/screens/ + widgets/
│   ├── profile/screens/ + widgets/
│   ├── leave/screens/ + widgets/
│   ├── achievements/screens/ + widgets/
│   ├── leaderboard/screens/ + widgets/
│   ├── ai_chat/screens/ + widgets/
│   ├── library/screens/ + widgets/
│   ├── notifications/screens/ + widgets/
│   ├── settings/screens/ + widgets/
│   ├── messaging/screens/ + widgets/
│   └── live_classes/screens/ + widgets/
│
└── l10n/
    ├── app_en.arb
    └── app_hi.arb
```

**Total files: ~200+ Dart files**

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
  runApp(const EduVerseApp());
}
```

### 7.2 app.dart
```dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/theme/app_theme.dart';
import 'package:eduverse_student/app_router.dart';

class EduVerseApp extends StatelessWidget {
  const EduVerseApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'EduVerse',
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
import 'package:eduverse_student/features/auth/screens/splash_screen.dart';
import 'package:eduverse_student/features/auth/screens/login_screen.dart';
import 'package:eduverse_student/features/dashboard/screens/dashboard_screen.dart';
// ... import all 29 screens

final goRouter = GoRouter(
  initialLocation: '/splash',
  routes: [
    GoRoute(path: '/splash', builder: (_, __) => const SplashScreen()),
    GoRoute(path: '/login', builder: (_, __) => const LoginScreen()),
    GoRoute(path: '/dashboard', builder: (_, __) => const DashboardScreen()),
    GoRoute(path: '/timetable', builder: (_, __) => const TimetableScreen()),
    GoRoute(path: '/results', builder: (_, __) => const ResultsScreen()),
    GoRoute(path: '/exams', builder: (_, __) => const ExamsScreen()),
    GoRoute(path: '/exam-instructions', builder: (_, __) => const ExamInstructionsScreen()),
    GoRoute(path: '/online-exam', builder: (_, __) => const OnlineExamScreen()),
    GoRoute(path: '/exam-success', builder: (_, __) => const ExamSuccessScreen()),
    GoRoute(path: '/fees', builder: (_, __) => const FeesScreen()),
    GoRoute(path: '/notices', builder: (_, __) => const NoticesScreen()),
    GoRoute(path: '/homework', builder: (_, __) => const HomeworkScreen()),
    GoRoute(path: '/transport', builder: (_, __) => const TransportScreen()),
    GoRoute(path: '/events', builder: (_, __) => const EventsScreen()),
    GoRoute(path: '/achievements', builder: (_, __) => const AchievementsScreen()),
    GoRoute(path: '/attendance', builder: (_, __) => const AttendanceScreen()),
    GoRoute(path: '/leave', builder: (_, __) => const LeaveScreen()),
    GoRoute(path: '/profile', builder: (_, __) => const ProfileScreen()),
    GoRoute(path: '/ai-chat', builder: (_, __) => const AIChatScreen()),
    GoRoute(path: '/library', builder: (_, __) => const LibraryScreen()),
    GoRoute(path: '/courses', builder: (_, __) => const CoursesScreen()),
    GoRoute(path: '/notifications', builder: (_, __) => const NotificationsScreen()),
    GoRoute(path: '/settings', builder: (_, __) => const SettingsScreen()),
    GoRoute(path: '/messaging', builder: (_, __) => const MessagingScreen()),
    GoRoute(path: '/chat-detail', builder: (_, __) => const ChatDetailScreen()),
    GoRoute(path: '/create-group', builder: (_, __) => const CreateGroupScreen()),
    GoRoute(path: '/live-classes', builder: (_, __) => const LiveClassesScreen()),
    GoRoute(path: '/live-class-detail', builder: (_, __) => const LiveClassDetailScreen()),
    GoRoute(path: '/leaderboard', builder: (_, __) => const LeaderboardScreen()),
  ],
);
```

### 7.4 Sample Screen Template (Dashboard)
```dart
// lib/features/dashboard/screens/dashboard_screen.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';
import 'package:eduverse_student/shared/widgets/app_bottom_nav.dart';
import 'package:eduverse_student/shared/widgets/ai_fab.dart';
import '../widgets/dashboard_header.dart';
import '../widgets/quick_access_grid.dart';
import '../widgets/today_schedule_card.dart';
import '../widgets/ai_insight_card.dart';
import '../widgets/pending_homework_card.dart';

class DashboardScreen extends StatefulWidget {
  const DashboardScreen({super.key});

  @override
  State<DashboardScreen> createState() => _DashboardScreenState();
}

class _DashboardScreenState extends State<DashboardScreen> {
  int _currentNavIndex = 0;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: AppColors.scaffoldBackground,
      body: Stack(
        children: [
          SingleChildScrollView(
            child: Column(
              children: [
                const DashboardHeader(),
                const SizedBox(height: AppDimensions.marginL),
                const SectionHeader(title: 'Quick Access'),
                const QuickAccessGrid(),
                const TodayScheduleCard(),
                const AIInsightCard(),
                const PendingHomeworkCard(),
                const SizedBox(height: 100), // Space for FAB
              ],
            ),
          ),
          const AIFab(),
        ],
      ),
      bottomNavigationBar: AppBottomNav(
        currentIndex: _currentNavIndex,
        onTap: (index) => setState(() => _currentNavIndex = index),
      ),
    );
  }
}
```

---

## 8. SCREEN-BY-SCREEN SPECIFICATIONS

### For each of the 29 screens, use this template:

#### Screen: [Name]
- **Route**: `/route-path`
- **Background Color**: `Color(0xHEX)`
- **Header**: Gradient colors, title, back button (Y/N), trailing widget
- **Body**: List all sections top to bottom with exact spacing
- **Bottom Nav**: Which index is active
- **AI FAB**: Present (Y/N)
- **Modals**: List all modals triggered from this screen
- **Key Widgets**: List widget file names

### 8.1 Splash Screen
- Route: `/splash`
- Background: `#0F0C29 → #302B63 → #24243E` (dark gradient)
- Content: Center → logo 🎓 (90x90, border radius 28, rgba bg), title "EduVerse" (Outfit, 36, weight 900, white, gradient span), subtitle "AI-Powered School ERP" (11px, rgba white .4, uppercase, letter spacing .1em)
- Loading bar: 120x3px, rgba white .1 bg, inner bar fills 0→100% over 1.5s with primary gradient
- Auto-navigate: 2 seconds → `/login`
- No bottom nav, no AI FAB

### 8.2 Login Screen
- Route: `/login`
- Background: Same dark gradient as splash
- Header: Status bar (9:41, 📶, 🔋)
- Body top to bottom:
  1. Radial glow (200px circle, primary .4 opacity, positioned top center)
  2. Logo block (70x70, rgba bg, border 1px, spinning ring animation)
  3. Title "EduVerse" with gradient
  4. Subtitle "AI-Powered School ERP"
  5. "Login as" label (11px, rgba white .5, align left)
  6. Role selector: 3 cards grid (Student 🎒, Parent 👨‍👩‍👧, Teacher 👨‍🏫) — active card has rgba(79,70,229,.25) bg, border rgba(79,70,229,.6)
  7. Student ID input (dark theme, value "STU-2024-1082")
  8. Password input (dark theme, value "••••••••")
  9. "✨ Login with AI Assist" button (gradient, weight 700, shadow)
  10. AI tag "🤖 AI monitors your learning journey every second" (10px, rgba white .3)
  11. Alt login row: 3 cards (Face ID, OTP, QR) — rgba white .06 bg, rounded 10px
- Modals: Biometric (face scan), OTP (4-digit input), QR (code display)
- On login → navigate to `/dashboard`

### 8.3 Dashboard Screen
- Route: `/dashboard`
- Background: `#F5F6FF`
- Status bar: gradient primary→deep, white text
- Body sections:
  1. **Header** (gradient primary→deep, padding 8px 18px 20px):
     - Row: greeting "Good Morning 🌤️" (11px, rgba white .6) + name "Arjun Kumar" (Outfit, 17, weight 700, white)
     - Right: notification bell (32x32, rgba bg, badge "3" red) + avatar (38px circle, gradient, border 2px rgba white .3)
     - Streak card: "🔥 Learning Streak" (12px, rgba white .8) + "18 days" (Outfit, 22, weight 800, #F59E0B)
     - Stats row: 4 boxes (Attend 94%, Score 91.4, Rank 3rd, XP 2,450) — rgba white .1 bg, rounded 12px
  2. **Quick Access** section header "Quick Access" (14px, Outfit, weight 700)
  3. **Grid**: 4x4 (16 items), each item: icon (46x46, rounded 15, specific bg color) + label (9.5px, Outfit, weight 600, #475569)
  4. **Today's Schedule** card: "Today's Schedule" title with "View All →" link, 3 class rows (time, dot, subject, teacher/room)
  5. **AI Insight** card: purple bordered, "🤖 AI INSIGHT" tag, tip text (12px, #3730A3)
  6. **Pending Homework** card: 2 items with subject icon, title, due date color-coded
- AI FAB: 46x46, gradient, shadow, pulse animation, positioned bottom-right 16px, above bottom nav
- Bottom nav: Home (active), Courses, Results, Achieve, Profile

### 8.4 Timetable Screen
- Route: `/timetable`
- Background: `#F0F4FF`
- Header: gradient #1E40AF→#1D4ED8, back button, title "Timetable", class "Class X-A"
- Body:
  1. Day chips: horizontal scroll, each chip has day label (9px) + date (15px, Outfit, weight 800), active chip has white bg
  2. Month navigator: ◀ March 2025 ▶ (arrows + label)
  3. Class cards: vertical list, each has left 4px color strip, time (42px wide), subject dot, subject name, teacher, room badge
  4. Break cards: orange bg, "☕ Break 10:00 – 10:20"
  5. "NOW" badge on current class (green bg, 9px, weight 700)
- Click card → detail modal (teacher, topic, material, homework, notes)
- Bottom nav: Timetable (active)

### 8.5-8.29 (Remaining Screens)
Follow the same pattern as 8.1-8.4 above. Each screen specification should include:
- Route path
- Exact background color
- Header gradient colors and content
- Body sections listed top-to-bottom with exact pixel values
- Card layouts with exact padding, border radius, colors
- Interactive elements and their behaviors
- Modals triggered
- Bottom nav active index
- AI FAB presence

---

## 9. REUSABLE COMPONENTS (FULL CODE)

### 9.1 AppBottomNav
```dart
// lib/shared/widgets/app_bottom_nav.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';
import 'package:go_router/go_router.dart';

class AppBottomNav extends StatelessWidget {
  final int currentIndex;
  final Function(int) onTap;

  const AppBottomNav({
    super.key,
    required this.currentIndex,
    required this.onTap,
  });

  static const List<_NavItem> _items = [
    _NavItem(icon: '🏠', label: 'Home', route: '/dashboard'),
    _NavItem(icon: '📚', label: 'Courses', route: '/courses'),
    _NavItem(icon: '📊', label: 'Results', route: '/results'),
    _NavItem(icon: '🏆', label: 'Achieve', route: '/achievements'),
    _NavItem(icon: '👤', label: 'Profile', route: '/profile'),
  ];

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: const BoxDecoration(
        color: Colors.white,
        border: Border(top: BorderSide(color: Color(0xFFF1F5F9))),
      ),
      padding: const EdgeInsets.only(top: 8, bottom: 14),
      child: Row(
        children: _items.asMap().entries.map((entry) {
          final index = entry.key;
          final item = entry.value;
          final isActive = index == currentIndex;
          
          return Expanded(
            child: GestureDetector(
              onTap: () {
                onTap(index);
                context.go(item.route);
              },
              behavior: HitTestBehavior.opaque,
              child: Column(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Container(
                    width: 36,
                    height: 28,
                    decoration: BoxDecoration(
                      color: isActive ? AppColors.primaryLight : Colors.transparent,
                      borderRadius: BorderRadius.circular(10),
                    ),
                    child: Center(
                      child: Text(item.icon, style: const TextStyle(fontSize: 16)),
                    ),
                  ),
                  const SizedBox(height: 3),
                  Text(
                    item.label,
                    style: TextStyle(
                      fontFamily: AppFonts.heading,
                      fontSize: 9,
                      fontWeight: isActive ? FontWeight.w700 : FontWeight.w500,
                      color: isActive ? AppColors.primary : AppColors.text3,
                    ),
                  ),
                ],
              ),
            ),
          );
        }).toList(),
      ),
    );
  }
}

class _NavItem {
  final String icon;
  final String label;
  final String route;
  const _NavItem({required this.icon, required this.label, required this.route});
}
```

### 9.2 AppHeader
```dart
// lib/shared/widgets/app_header.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';

class AppHeader extends StatelessWidget {
  final String title;
  final List<Color> gradientColors;
  final Widget? trailing;
  final Widget? subtitle;
  final bool showBackButton;
  final VoidCallback? onBackPressed;

  const AppHeader({
    super.key,
    required this.title,
    required this.gradientColors,
    this.trailing,
    this.subtitle,
    this.showBackButton = true,
    this.onBackPressed,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(
        gradient: LinearGradient(
          colors: gradientColors,
          begin: Alignment.topLeft,
          end: Alignment.bottomRight,
        ),
      ),
      padding: const EdgeInsets.fromLTRB(
        AppDimensions.paddingL,
        AppDimensions.statusBarHeight + 8,
        AppDimensions.paddingL,
        16,
      ),
      child: Row(
        children: [
          if (showBackButton) ...[
            GestureDetector(
              onTap: onBackPressed ?? () => Navigator.pop(context),
              child: Container(
                width: 28,
                height: 28,
                decoration: BoxDecoration(
                  color: Colors.white.withOpacity(0.15),
                  borderRadius: BorderRadius.circular(9),
                ),
                child: const Center(
                  child: Text('←', style: TextStyle(color: Colors.white, fontSize: 12)),
                ),
              ),
            ),
            const SizedBox(width: 10),
          ],
          Expanded(
            child: Text(
              title,
              style: TextStyle(
                fontFamily: AppFonts.heading,
                fontSize: 17,
                fontWeight: FontWeight.w800,
                color: Colors.white,
              ),
            ),
          ),
          if (subtitle != null) subtitle!,
          if (trailing != null) trailing!,
        ],
      ),
    );
  }
}
```

### 9.3 AIFab
```dart
// lib/shared/widgets/ai_fab.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:go_router/go_router.dart';

class AIFab extends StatefulWidget {
  const AIFab({super.key});

  @override
  State<AIFab> createState() => _AIFabState();
}

class _AIFabState extends State<AIFab> with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(seconds: 3),
      vsync: this,
    )..repeat(reverse: true);
    _animation = Tween<double>(begin: 0.5, end: 0.8).animate(_controller);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Positioned(
      bottom: 130,
      right: 16,
      child: AnimatedBuilder(
        animation: _animation,
        builder: (context, child) {
          return Container(
            width: 46,
            height: 46,
            decoration: BoxDecoration(
              shape: BoxShape.circle,
              gradient: const LinearGradient(
                colors: [AppColors.primary, AppColors.accent],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
              boxShadow: [
                BoxShadow(
                  color: AppColors.primary.withOpacity(_animation.value),
                  blurRadius: 20 + (_animation.value * 8),
                  spreadRadius: _animation.value * 6,
                ),
              ],
            ),
            child: Material(
              color: Colors.transparent,
              child: InkWell(
                borderRadius: BorderRadius.circular(23),
                onTap: () => context.go('/ai-chat'),
                child: const Center(
                  child: Text('🤖', style: TextStyle(fontSize: 20)),
                ),
              ),
            ),
          );
        },
      ),
    );
  }
}
```

### 9.4 AppCard
```dart
// lib/shared/widgets/app_card.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';
import 'package:eduverse_student/core/constants/app_shadows.dart';

class AppCard extends StatelessWidget {
  final Widget child;
  final EdgeInsetsGeometry? padding;
  final EdgeInsetsGeometry? margin;
  final Color? backgroundColor;
  final Color? borderColor;
  final double borderRadius;
  final VoidCallback? onTap;

  const AppCard({
    super.key,
    required this.child,
    this.padding = const EdgeInsets.all(14),
    this.margin = const EdgeInsets.symmetric(horizontal: 14, vertical: 6),
    this.backgroundColor = Colors.white,
    this.borderColor,
    this.borderRadius = 16,
    this.onTap,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: margin,
      decoration: BoxDecoration(
        color: backgroundColor,
        borderRadius: BorderRadius.circular(borderRadius),
        boxShadow: AppShadows.card,
        border: borderColor != null
            ? Border.all(color: borderColor!)
            : null,
      ),
      child: Material(
        color: Colors.transparent,
        child: onTap != null
            ? InkWell(
                borderRadius: BorderRadius.circular(borderRadius),
                onTap: onTap,
                child: Padding(padding: padding!, child: child),
              )
            : Padding(padding: padding!, child: child),
      ),
    );
  }
}
```

### 9.5 AppButton
```dart
// lib/shared/widgets/app_button.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';

enum AppButtonVariant { primary, secondary, success, danger }

class AppButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;
  final AppButtonVariant variant;
  final bool isLoading;
  final double? width;

  const AppButton({
    super.key,
    required this.text,
    required this.onPressed,
    this.variant = AppButtonVariant.primary,
    this.isLoading = false,
    this.width = double.infinity,
  });

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: width,
      height: AppDimensions.buttonHeight,
      child: DecoratedBox(
        decoration: BoxDecoration(
          gradient: variant == AppButtonVariant.primary
              ? const LinearGradient(colors: [Color(0xFF4F46E5), Color(0xFF06B6D4)])
              : variant == AppButtonVariant.success
                  ? const LinearGradient(colors: [Color(0xFF059669), Color(0xFF06B6D4)])
                  : null,
          color: variant == AppButtonVariant.secondary
              ? AppColors.primaryLight
              : variant == AppButtonVariant.danger
                  ? AppColors.errorBg
                  : null,
          borderRadius: BorderRadius.circular(AppDimensions.radiusXXL),
          boxShadow: variant == AppButtonVariant.primary
              ? [BoxShadow(color: const Color(0xFF4F46E5).withOpacity(0.4), blurRadius: 20, offset: const Offset(0, 6))]
              : null,
        ),
        child: ElevatedButton(
          onPressed: isLoading ? null : onPressed,
          style: ElevatedButton.styleFrom(
            backgroundColor: Colors.transparent,
            shadowColor: Colors.transparent,
            shape: RoundedRectangleBorder(
              borderRadius: BorderRadius.circular(AppDimensions.radiusXXL),
            ),
          ),
          child: isLoading
              ? const SizedBox(
                  width: 20,
                  height: 20,
                  child: CircularProgressIndicator(strokeWidth: 2, color: Colors.white),
                )
              : Text(
                  text,
                  style: TextStyle(
                    fontFamily: AppFonts.heading,
                    fontSize: 15,
                    fontWeight: FontWeight.w700,
                    color: variant == AppButtonVariant.primary || variant == AppButtonVariant.success
                        ? Colors.white
                        : variant == AppButtonVariant.danger
                            ? AppColors.error
                            : AppColors.primary,
                  ),
                ),
        ),
      ),
    );
  }
}
```

### 9.6 AppChip
```dart
// lib/shared/widgets/app_chip.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';

class AppChip extends StatelessWidget {
  final String label;
  final bool isSelected;
  final Color activeColor;
  final Color inactiveColor;
  final Color activeTextColor;
  final Color inactiveTextColor;
  final VoidCallback onTap;
  final double fontSize;

  const AppChip({
    super.key,
    required this.label,
    required this.isSelected,
    required this.activeColor,
    required this.inactiveColor,
    this.activeTextColor = Colors.white,
    this.inactiveTextColor = const Color(0xFF475569),
    required this.onTap,
    this.fontSize = 11,
  });

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: Container(
        padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 5),
        decoration: BoxDecoration(
          color: isSelected ? activeColor : inactiveColor,
          borderRadius: BorderRadius.circular(20),
        ),
        child: Text(
          label,
          style: TextStyle(
            fontFamily: AppFonts.heading,
            fontSize: fontSize,
            fontWeight: FontWeight.w600,
            color: isSelected ? activeTextColor : inactiveTextColor,
          ),
        ),
      ),
    );
  }
}
```

### 9.7 AppBadge
```dart
// lib/shared/widgets/app_badge.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';

enum BadgeVariant { paid, due, partial, pending, approved, rejected, cancelled }

class AppBadge extends StatelessWidget {
  final String text;
  final BadgeVariant variant;

  const AppBadge({
    super.key,
    required this.text,
    required this.variant,
  });

  @override
  Widget build(BuildContext context) {
    final (Color bg, Color fg) = switch (variant) {
      BadgeVariant.paid => (AppColors.successBg, AppColors.success),
      BadgeVariant.due => (AppColors.errorBg, AppColors.error),
      BadgeVariant.partial => (AppColors.warningBg, AppColors.warning),
      BadgeVariant.pending => (const Color(0xFFFDF4FF), const Color(0xFFD97706)),
      BadgeVariant.approved => (AppColors.successBg, AppColors.success),
      BadgeVariant.rejected => (AppColors.errorBg, AppColors.error),
      BadgeVariant.cancelled => (AppColors.errorBg, AppColors.error),
    };

    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 3),
      decoration: BoxDecoration(
        color: bg,
        borderRadius: BorderRadius.circular(6),
      ),
      child: Text(
        text,
        style: TextStyle(
          fontFamily: AppFonts.heading,
          fontSize: 9,
          fontWeight: FontWeight.w700,
          color: fg,
        ),
      ),
    );
  }
}
```

### 9.8 AppModal
```dart
// lib/shared/widgets/app_modal.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_dimensions.dart';

class AppModal extends StatelessWidget {
  final String? title;
  final Widget content;
  final List<Widget>? actions;

  const AppModal({
    super.key,
    this.title,
    required this.content,
    this.actions,
  });

  static Future<T?> show<T>({
    required BuildContext context,
    String? title,
    required Widget content,
    List<Widget>? actions,
  }) {
    return showModalBottomSheet<T>(
      context: context,
      isScrollControlled: true,
      backgroundColor: Colors.white,
      shape: const RoundedRectangleBorder(
        borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
      ),
      builder: (context) => AppModal(
        title: title,
        content: content,
        actions: actions,
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.only(
        left: 20,
        right: 20,
        top: 20,
        bottom: MediaQuery.of(context).viewInsets.bottom + 20,
      ),
      child: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          // Handle bar
          Center(
            child: Container(
              width: 40,
              height: 4,
              decoration: BoxDecoration(
                color: const Color(0xFFE2E8F0),
                borderRadius: BorderRadius.circular(2),
              ),
            ),
          ),
          const SizedBox(height: 16),
          // Title
          if (title != null) ...[
            Text(
              title!,
              style: const TextStyle(
                fontFamily: 'Outfit',
                fontSize: 16,
                fontWeight: FontWeight.w800,
                color: Color(0xFF0F172A),
              ),
              textAlign: TextAlign.center,
            ),
            const SizedBox(height: 12),
          ],
          // Content
          content,
          // Actions
          if (actions != null) ...[
            const SizedBox(height: 16),
            ...actions!,
          ],
        ],
      ),
    );
  }
}
```

### 9.9 AITag
```dart
// lib/shared/widgets/ai_tag.dart
import 'package:flutter/material.dart';
import 'package:eduverse_student/core/constants/app_colors.dart';
import 'package:eduverse_student/core/constants/app_fonts.dart';

class AITag extends StatelessWidget {
  final String text;
  final Color? backgroundColor;

  const AITag({
    super.key,
    this.text = '🤖 AI INSIGHT',
    this.backgroundColor,
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 9, vertical: 3),
      decoration: BoxDecoration(
        color: backgroundColor ?? AppColors.primary,
        borderRadius: BorderRadius.circular(20),
      ),
      child: Text(
        text,
        style: TextStyle(
          fontFamily: AppFonts.heading,
          fontSize: 9,
          fontWeight: FontWeight.w700,
          color: Colors.white,
          letterSpacing: 0.05,
        ),
      ),
    );
  }
}
```

### 9.10 CircularProgress
```dart
// lib/shared/widgets/circular_progress.dart
import 'package:flutter/material.dart';
import 'dart:math' as math;

class CircularProgress extends StatelessWidget {
  final double percentage;
  final Color strokeColor;
  final Color trackColor;
  final double size;
  final double strokeWidth;
  final Widget? centerWidget;

  const CircularProgress({
    super.key,
    required this.percentage,
    this.strokeColor = const Color(0xFF4F46E5),
    this.trackColor = const Color(0xFFF1F5F9),
    this.size = 120,
    this.strokeWidth = 8,
    this.centerWidget,
  });

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: size,
      height: size,
      child: CustomPaint(
        painter: _CircularProgressPainter(
          percentage: percentage,
          strokeColor: strokeColor,
          trackColor: trackColor,
          strokeWidth: strokeWidth,
        ),
        child: Center(child: centerWidget),
      ),
    );
  }
}

class _CircularProgressPainter extends CustomPainter {
  final double percentage;
  final Color strokeColor;
  final Color trackColor;
  final double strokeWidth;

  _CircularProgressPainter({
    required this.percentage,
    required this.strokeColor,
    required this.trackColor,
    required this.strokeWidth,
  });

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = (size.width - strokeWidth) / 2;

    // Track
    final trackPaint = Paint()
      ..color = trackColor
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth;
    canvas.drawCircle(center, radius, trackPaint);

    // Progress
    final progressPaint = Paint()
      ..color = strokeColor
      ..style = PaintingStyle.stroke
      ..strokeWidth = strokeWidth
      ..strokeCap = StrokeCap.round;
    
    final sweepAngle = 2 * math.pi * (percentage / 100);
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2,
      sweepAngle,
      false,
      progressPaint,
    );
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}
```

---

## 10. RESPONSIVE DESIGN STRATEGY

### Breakpoints
```dart
class Breakpoints {
  static const double mobile = 600;
  static const double tablet = 900;
  static const double desktop = 1200;
  static const double largeDesktop = 1600;
}
```

### Responsive Rules Per Screen Element

| Element | Mobile (<600) | Tablet (600-900) | Desktop (>900) |
|---------|--------------|-----------------|----------------|
| Layout | Single column | 2 columns | 3-4 columns |
| Nav | Bottom bar | Bottom bar | Sidebar |
| Quick Access Grid | 4 columns | 6 columns | 8 columns |
| Cards | Full width | 50% width | 33% width |
| Modals | Bottom sheet | Centered dialog | Centered dialog |
| Chat | Full screen | Split view | Split view |
| Exam Palette | Bottom grid | Side panel | Side panel |
| Map | Embedded | Larger | Full panel |
| Padding | 14px | 24px | 32px |

### ResponsiveWidget Template
```dart
class ResponsiveBuilder extends StatelessWidget {
  final Widget Function(BuildContext context, BoxConstraints constraints) builder;

  const ResponsiveBuilder({super.key, required this.builder});

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) => builder(context, constraints),
    );
  }
}
```

---

## 11. NAVIGATION & ROUTING

Use GoRouter (already defined in Section 7.3). Key patterns:

```dart
// Navigate forward (push)
context.push('/timetable');

// Navigate and replace (login → dashboard)
context.go('/dashboard');

// Go back
context.pop();

// Navigate with parameters
context.push('/chat-detail?name=Mr.Sharma&avatar=👨‍🏫');

// Deep linking supported for web
// URL: https://eduverse.com/dashboard → DashboardScreen
```

---

## 12. DEPENDENCIES

See Section 2 Step 3 for complete pubspec.yaml.

---

## 13. DEVELOPMENT PHASES

### Phase 1: Foundation (Week 1-2)
- Project setup, fonts, theme, constants
- All shared widgets (14+ components)
- Supabase setup, SQL migration
- GoRouter configuration

### Phase 2: Auth + Dashboard (Week 3-4)
- Splash, Login (with 3 modals)
- Dashboard (all sections)
- Profile, Settings

### Phase 3: Academics (Week 5-6)
- Timetable, Results, Courses, Attendance, Homework

### Phase 4: Exams (Week 7-8)
- Exams, Instructions, Online Exam Engine (50 MCQ + 10 Subjective), Success

### Phase 5: Finance + Notices (Week 9-10)
- Fees, Payment flow, Notices, Events

### Phase 6: Communication (Week 11-12)
- Messaging, Chat Detail, Create Group, Notifications, Live Classes

### Phase 7: Transport + Library (Week 13-14)
- OSM Map, Bus Tracking, Library

### Phase 8: Gamification (Week 15)
- Achievements, Leaderboard, Leave

### Phase 9: AI (Week 16-17)
- LangChain setup, 20 tools, AI Chat

### Phase 10: Polish (Week 18-20)
- Responsive testing, platform testing, animations, performance

---

## 14. CRITICAL IMPLEMENTATION NOTES

1. **Pixel-Perfect UI**: Every color, font size, border radius, padding, and shadow must match the HTML mockup exactly. Reference Section 3 for exact values.

2. **All 6 Platforms**: Test on Web (Chrome), Android (emulator + device), iOS (simulator + device), macOS, Linux, Windows. Use `Platform.isX` checks where needed.

3. **Real-time**: Use Supabase Realtime channels for:
   - `bus_locations` → live bus tracking on OSM map
   - `messages` → real-time chat
   - `live_classes` → live status updates
   - `notifications` → instant push
   - `exam_sessions` → timer sync

4. **Exam Integrity**: During online exam:
   - Lock to portrait orientation
   - Disable system back button
   - Track `WidgetsBindingObserver` for app lifecycle changes
   - Count screen focus switches for integrity score
   - Auto-submit when timer hits zero

5. **AI Context**: Pass student's full context (name, class, results, attendance, streak, XP) to the AI system prompt for personalized responses.

6. **Offline**: Use Hive to cache: timetable, homework list, profile data, recent results. Show cached data when offline, sync when online.

7. **File Uploads**: Use Supabase Storage buckets:
   - `homework-files` → homework submissions
   - `documents` → profile documents
   - `exam-answers` → exam answer sheets
   - `chat-attachments` → message files

8. **Gamification Rules**:
   - Homework on time: +50 XP
   - Daily login streak: +10 XP per day
   - Exam score 90%+: +500 XP
   - Event participation: +100 XP
   - 7-day streak: +50 XP bonus
   - 14-day streak: +150 XP bonus
   - 21-day streak: +300 XP + badge
   - 30-day streak: +500 XP + gold badge

9. **Status Bar Colors**: Each screen has a specific status bar gradient. Use `AnnotatedRegion<SystemUiOverlayStyle>` to set status bar color per screen.

10. **Animation Durations**:
    - Screen transitions: 400ms, cubic-bezier(0.4, 0, 0.2, 1)
    - Modal slide up: 300ms
    - Button press: 150ms
    - AI FAB pulse: 3000ms
    - Typing dots: 1200ms
    - Confetti: 3000ms
    - Splash bar: 1500ms

11. **Image Assets**: Use emojis as placeholders. For production, replace with SVG icons or custom image assets.

12. **Web-Specific**:
    - Use `HtmlElementView` for video player on web
    - Enable URL strategy: `GoRouter.optionURLReflectsImperativeAPIs = true`
    - Set `usePathUrlStrategy()` for clean URLs