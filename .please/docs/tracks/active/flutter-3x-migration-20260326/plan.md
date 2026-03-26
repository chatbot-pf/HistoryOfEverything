# Plan: Flutter 3.x Migration & Modernization

> Track: flutter-3x-migration-20260326
> Spec: [spec.md](./spec.md)

## Overview

- **Source**: .please/docs/tracks/active/flutter-3x-migration-20260326/spec.md
- **Issue**: TBD
- **Created**: 2026-03-26
- **Approach**: Incremental (bottom-up: dependencies → app code → platform configs → verification)

## Purpose

After this change, developers will be able to build and run the History of Everything app using Flutter 3.x (stable 3.41.5) with full Dart 3 null safety. They can verify it works by running `flutter analyze`, `flutter test`, and `flutter build apk` without errors.

## Context

### Problem

The History of Everything app was built with Flutter 1.x / Dart 2.0-dev in 2018. It cannot be compiled with modern Flutter/Dart SDKs. It uses two local animation libraries (Flare-Flutter and Nima-Flutter from the defunct 2d-inc/2Dimensions) as git submodules, both pre-null-safety and incompatible with Dart 3.

### Requirements Summary

The migration must address four layers: (1) local submodule libraries (Flare/Nima) need minimal null-safety migration to compile with Dart 3, (2) pub dependencies need major version bumps with API changes, (3) app source code (26 Dart files) needs null safety annotations and deprecated API replacements, and (4) Android/iOS platform configs need complete updates (AGP 3.1→8.x, Gradle 4.4→8.x, SDK levels, AndroidX, iOS deployment target 8.0→12.0).

### Constraints

- Flare/Nima libraries are unmaintained — minimal changes only to compile
- All existing .flr/.nma animation assets must be preserved as-is
- FVM is used to pin the Flutter SDK version (3.41.5 stable) for reproducible builds

### Non-Goals

- New features, UI/UX changes, or content changes
- Replacing Flare/Nima with Rive (deferred to a future track)
- Refactoring Flare/Nima library internals beyond what's needed to compile

## Architecture Decision

**Decision**: Incremental bottom-up migration with FVM for SDK management.

**Rationale**: The dependency graph flows bottom-up: `flare_dart` → `flare_flutter` → app, and `nima` → app. Migrating bottom-up ensures each layer compiles before the next begins. Using FVM pins the Flutter SDK version (3.41.5 stable) for reproducible builds.

**Migration order**:
1. Set up FVM and Flutter 3.x SDK
2. Migrate local dependencies first (flare_dart → flare_flutter → nima) — minimum null-safety changes only
3. Update pubspec.yaml with new SDK constraints and pub dependency versions
4. Migrate app source code (null safety + deprecated API replacement)
5. Update Android platform configs (Gradle, AGP, AndroidX, SDK versions)
6. Update iOS platform configs (deployment target, Podfile)
7. Verify builds and tests

**Alternatives considered**:
- Big-bang migration: rejected because debugging failures across all layers simultaneously is impractical
- Replace Flare/Nima with Rive: rejected for this track due to unknown asset availability and high effort; tracked as future work

## Tasks

### Phase 1: SDK & Build Infrastructure

- [ ] T001 Set up FVM with Flutter 3.41.5 stable (file: .fvmrc)
- [ ] T002 Update Android Gradle and build configuration (file: app/android/build.gradle) (depends on T001)
- [ ] T003 Update Android gradle wrapper and properties (file: app/android/gradle/wrapper/gradle-wrapper.properties) (depends on T001)
- [ ] T004 Update Android manifest and settings (file: app/android/app/src/main/AndroidManifest.xml) (depends on T002, T003)
- [ ] T005 Update iOS platform configuration (file: app/ios/Podfile) (depends on T001)

### Phase 2: Local Dependency Migration

- [ ] T006 Migrate flare_dart to Dart 3 null safety (file: dependencies/Flare-Flutter/flare_dart/pubspec.yaml) (depends on T001)
- [ ] T007 Migrate flare_flutter to Dart 3 null safety (file: dependencies/Flare-Flutter/flare_flutter/pubspec.yaml) (depends on T006)
- [ ] T008 Migrate nima to Dart 3 null safety (file: dependencies/Nima-Flutter/pubspec.yaml) (depends on T001)

### Phase 3: App Dependencies & Source Migration

- [ ] T009 Update pubspec.yaml SDK constraint and pub dependencies (file: app/pubspec.yaml) (depends on T007, T008)
- [ ] T010 Migrate core data models to null safety (file: app/lib/timeline/timeline_entry.dart) (depends on T009)
- [ ] T011 Migrate BLoC provider and favorites to null safety (file: app/lib/bloc_provider.dart) (depends on T009)
- [ ] T012 Migrate timeline engine to null safety (file: app/lib/timeline/timeline.dart) (depends on T010)
- [ ] T013 Migrate timeline render widget to null safety (file: app/lib/timeline/timeline_render_widget.dart) (depends on T010, T012)
- [ ] T014 Migrate timeline widget and utilities (file: app/lib/timeline/timeline_widget.dart) (depends on T013)
- [ ] T015 Migrate article widgets and controllers (file: app/lib/article/timeline_entry_widget.dart) (depends on T010)
- [ ] T016 Migrate main menu and related widgets (file: app/lib/main_menu/main_menu.dart) (depends on T010, T011)
- [ ] T017 Migrate search, colors, and remaining files (file: app/lib/search_manager.dart) (depends on T010)
- [ ] T018 Update main.dart entry point and ThemeData (file: app/lib/main.dart) (depends on T011, T016)

### Phase 4: Verification & Cleanup

- [ ] T019 Run flutter analyze and fix all issues (depends on T004, T005, T018)
- [ ] T020 Run flutter test and fix failures (depends on T019)
- [ ] T021 Verify flutter build apk succeeds (depends on T020)
- [ ] T022 Add analysis_options.yaml with recommended lints (file: app/analysis_options.yaml) (depends on T019)

## Key Files

### Create
- `.fvmrc` — FVM Flutter version pin
- `app/analysis_options.yaml` — Dart analysis options

### Modify
- `app/pubspec.yaml` — SDK constraint, dependency versions
- `dependencies/Flare-Flutter/flare_dart/pubspec.yaml` — SDK constraint
- `dependencies/Flare-Flutter/flare_dart/lib/**/*.dart` — null safety
- `dependencies/Flare-Flutter/flare_flutter/pubspec.yaml` — SDK constraint
- `dependencies/Flare-Flutter/flare_flutter/lib/**/*.dart` — null safety
- `dependencies/Nima-Flutter/pubspec.yaml` — SDK constraint
- `dependencies/Nima-Flutter/lib/**/*.dart` — null safety
- `app/lib/**/*.dart` — null safety, deprecated API replacement (26 files)
- `app/android/app/build.gradle` — compileSdk, minSdk, targetSdk, AGP
- `app/android/build.gradle` — Gradle plugin version, repositories
- `app/android/gradle/wrapper/gradle-wrapper.properties` — Gradle version
- `app/android/gradle.properties` — AndroidX flags
- `app/android/settings.gradle` — plugin management
- `app/android/app/src/main/AndroidManifest.xml` — remove FlutterApplication
- `app/ios/Podfile` — platform version, plugin resolution
- `app/ios/Runner.xcodeproj/project.pbxproj` — deployment target

### Reuse
- `app/lib/colors.dart` — minimal changes expected
- `app/assets/**` — all animation assets unchanged

## Verification

### Automated Tests
- [ ] `flutter analyze` reports 0 errors, 0 warnings
- [ ] `flutter test` — all existing tests pass
- [ ] `flutter build apk --debug` succeeds

### Observable Outcomes
- Running `fvm flutter doctor` shows Flutter 3.41.5 stable with no issues
- Running `fvm flutter analyze` in `app/` shows no errors
- Running `fvm flutter build apk` in `app/` produces a valid APK

### Manual Testing
- [ ] App launches on Android emulator
- [ ] Timeline scrolls and zooms correctly
- [ ] Tap on event opens article page
- [ ] Search finds events
- [ ] Favorites can be added and removed
- [ ] Flare/Nima animations render correctly

### Acceptance Criteria Check
- [ ] Dart SDK constraint is `>=3.0.0 <4.0.0`
- [ ] All 26 source files compile with sound null safety
- [ ] No deprecated Flutter API usage remains
- [ ] Android minSdkVersion >= 21, compileSdk >= 34
- [ ] iOS deployment target >= 12.0
- [ ] FVM configured with .fvmrc

## Decision Log

- Decision: Use FVM for Flutter SDK version management
  Rationale: Reproducible builds, team consistency, easy version switching
  Date/Author: 2026-03-26 / Claude

- Decision: Fork & minimal migration for Flare/Nima (not replace with Rive)
  Rationale: Preserves existing animation assets, minimizes blast radius; Rive replacement deferred to future track
  Date/Author: 2026-03-26 / Claude + User

- Decision: Incremental bottom-up migration order
  Rationale: Dependency graph requires bottom-up; each layer must compile before the next
  Date/Author: 2026-03-26 / Claude

- Decision: Remove unused rxdart dependency
  Rationale: Declared in pubspec but never imported in any source file
  Date/Author: 2026-03-26 / Claude
