# Architecture: The History of Everything

> A Flutter mobile app that presents a beautifully animated vertical timeline spanning from the Big Bang to the Internet age. Events are illustrated with 2D vector animations (Nima/Flare) and described with Wikipedia-sourced articles.

## System Context

This is a **single-platform mobile app** (Android + iOS) built with Flutter (Dart SDK >=2.0). It has no backend, no network calls, and no database — all data is bundled as local assets (JSON, animation files, markdown articles). User state (favorites) is persisted via `shared_preferences` on-device.

The app was originally built by [2Dimensions](https://www.2dimensions.com) as a Flutter showcase demo. The animation libraries (Nima and Flare) are included as git submodules.

## Entry Points

| # | File | Purpose |
|---|------|---------|
| 1 | `app/lib/main.dart` | **App bootstrap** — `main()` launches `TimelineApp`, which wraps the widget tree in a `BlocProvider` and renders `MainMenuWidget` as the home screen. Start here. |
| 2 | `app/lib/bloc_provider.dart` | **Dependency injection** — `InheritedWidget` that holds the `Timeline` and `FavoritesBloc` singletons. Loads `timeline.json` at boot, initializes search, and exposes static accessors. |
| 3 | `app/assets/timeline.json` | **Data source** — JSON file defining all timeline entries (eras, incidents), their dates, labels, animation assets, colors, and visual properties. The entire app content comes from here. |
| 4 | `app/pubspec.yaml` | **Project manifest** — declares dependencies, asset bundles, fonts, and Flutter configuration. |

## Module Structure

```
HistoryOfEverything/
├── app/                          # Flutter application root
│   ├── lib/                      # Dart source code
│   │   ├── main.dart             # App entry point
│   │   ├── bloc_provider.dart    # InheritedWidget DI container
│   │   ├── search_manager.dart   # SplayTreeMap-based substring search
│   │   ├── colors.dart           # Shared color constants
│   │   ├── main_menu/            # Main menu screen
│   │   ├── timeline/             # Timeline rendering engine
│   │   ├── article/              # Article detail page + animation controllers
│   │   └── blocs/                # State management (favorites)
│   ├── assets/                   # Bundled resources (animations, articles, JSON data)
│   ├── android/                  # Android platform project
│   ├── ios/                      # iOS platform project
│   └── test/                     # Test directory (minimal)
├── dependencies/                 # Git submodules
│   ├── Nima-Flutter/             # Nima animation runtime (2D skeletal)
│   └── Flare-Flutter/            # Flare animation runtime (2D vector)
└── tools/                        # Asset processing utilities
    ├── find_large_assets.dart    # Find oversized asset files
    └── resize_large_assets.dart  # Resize assets to target dimensions
```

### `app/lib/main_menu/` — Main Menu Screen

The landing screen with search, three era-section cards, favorites access, share, and about page.

| File | Role |
|------|------|
| `main_menu.dart` | `MainMenuWidget` — top-level stateful widget orchestrating search, section cards, and navigation |
| `menu_data.dart` | `MenuData` — loads `menu.json` to populate section labels, colors, and Flare background animations |
| `main_menu_section.dart` | `MenuSection` — expandable card for each era (e.g., "Universe", "Life on Earth", "Humans") |
| `search_widget.dart` | Search bar UI with focus/text controller integration |
| `thumbnail.dart` | Thumbnail rendering for timeline entries in search results |
| `thumbnail_detail_widget.dart` | Detailed thumbnail with metadata for search result items |
| `collapsible.dart` | Animated collapse/expand widget for the header during search |
| `menu_vignette.dart` | Flare animation vignette displayed behind menu section cards |
| `favorites_page.dart` | Full-screen favorites list page |
| `about_page.dart` | About/credits page |

### `app/lib/timeline/` — Timeline Rendering Engine

The core rendering engine: a custom-painted vertical timeline with zoom, pan, scroll physics, and animated entries.

| File | Role |
|------|------|
| `timeline.dart` | `Timeline` class — the engine. Manages viewport (start/end time), scroll physics, entry hierarchy, asset loading from `timeline.json`, frame scheduling, animation advancement, and hit testing. ~1400 lines, the largest and most complex file. |
| `timeline_entry.dart` | `TimelineEntry` — data model for a single event/era. Also defines `TimelineAsset`, `TimelineNima`, `TimelineFlare`, `TimelineImage` asset types. |
| `timeline_widget.dart` | `TimelineWidget` — Flutter widget wrapping the timeline with gesture detection (pan, scale, tap) and app bar with era label. |
| `timeline_render_widget.dart` | `TimelineRenderWidget` — `LeafRenderObjectWidget` + `RenderBox` that paints the timeline entries, lines, bubbles, ticks, and background gradient directly on canvas. |
| `timeline_utils.dart` | Utility types: `TickColors`, `HeaderColors`, `TimelineBackgroundColor`, and color interpolation helpers. |
| `ticks.dart` | `Ticks` — paints year/date tick marks along the timeline gutter. |

### `app/lib/article/` — Article Detail Page

Displays a selected event's animation and Wikipedia article content.

| File | Role |
|------|------|
| `article_widget.dart` | `ArticleWidget` — loads markdown from bundled asset, renders Flare animation + article text, manages favorite toggle. |
| `timeline_entry_widget.dart` | `TimelineEntryWidget` — custom `LeafRenderObjectWidget` that renders Nima/Flare animations with interactive touch control. `VignetteRenderObject` handles the actual canvas painting. |
| `controllers/` | Animation interaction controllers for specific entries: `amelia_controller.dart` (Amelia Earhart — eyes follow touch), `newton_controller.dart` (Newton — apple interaction), `flare_interaction_controller.dart`, `nima_interaction_controller.dart`. |

### `app/lib/blocs/` — State Management

| File | Role |
|------|------|
| `favorites_bloc.dart` | `FavoritesBloc` — manages user favorites list, persists to `SharedPreferences`, provides add/remove/list operations. |

### `dependencies/` — Animation Libraries (Git Submodules)

| Submodule | Source | Purpose |
|-----------|--------|---------|
| `Nima-Flutter` | `github.com/2d-inc/Nima-Flutter` | Nima skeletal animation runtime — `.nma` files |
| `Flare-Flutter` | `github.com/2d-inc/Flare-Flutter` | Flare vector animation runtime — `.flr` files |

## Data Flow

```
Boot Sequence:
  main() → TimelineApp → BlocProvider
    ├── Timeline.loadFromBundle("timeline.json")
    │     ├── Parse JSON → List<TimelineEntry> (sorted by date)
    │     ├── Load Nima/Flare actors from asset bundle
    │     └── Build parent/child hierarchy (eras contain incidents)
    ├── FavoritesBloc.init(entries)
    │     └── Load favorites from SharedPreferences, match to entries
    └── SearchManager.init(entries)
          └── Build SplayTreeMap of all substrings → O(n²) at boot, O(log n) queries

Navigation:
  MainMenu → [tap section item] → TimelineWidget (scrolls to entry)
  Timeline → [tap bubble] → ArticleWidget (loads markdown + animation)
  MainMenu → [search] → SearchManager.performSearch() → result list
  MainMenu → [favorites] → FavoritesPage

Render Loop (Timeline):
  SchedulerBinding.scheduleFrameCallback → beginFrame()
    → advance(elapsed) → _advanceItems() + _advanceAssets()
    → onNeedPaint() → RenderBox.markNeedsPaint() → paint()
```

## Architecture Invariants

1. **No network access** — All content is bundled locally. The app works entirely offline.
2. **Single InheritedWidget DI** — `BlocProvider` is the sole dependency injection mechanism. All shared state (Timeline, FavoritesBloc) is accessed via `BlocProvider.favorites(context)` and `BlocProvider.getTimeline(context)`.
3. **Custom rendering for timeline** — The timeline does NOT use standard Flutter widgets for entries. It uses a `LeafRenderObjectWidget` → `RenderBox` pattern with direct canvas painting for performance. Do not attempt to replace this with widget-based rendering.
4. **Nima/Flare are vendored** — Animation libraries are git submodules in `dependencies/`, referenced as path dependencies in `pubspec.yaml`. They are not pub packages.
5. **Asset format contract** — `timeline.json` defines all content. Changing the JSON schema requires updating `Timeline.loadFromBundle()`. Animation assets (`.nma`, `.flr`) must match the filenames and animation names declared in the JSON.
6. **Time representation** — Dates are stored as doubles (negative = BCE, positive = CE, scaled by 10x for recent dates). This convention is used throughout `Timeline`, `TimelineEntry`, and the rendering code.
7. **Platform-aware scroll physics** — `Timeline` initializes `BouncingScrollPhysics` (iOS) or `ClampingScrollPhysics` (Android) based on the detected platform.

## Cross-Cutting Concerns

### Animation System
Two animation runtimes coexist: **Nima** (`.nma`, skeletal) and **Flare** (`.flr`, vector). Both follow the same pattern: load actor from bundle → create instance → apply animation frames in the render loop. Some Flare assets have intro + idle animations; some have multiple idle animations. The `loop` flag controls whether animation replays or freezes at the end.

### State Persistence
Only favorites are persisted, using `shared_preferences` (key: `"Favorites"`, value: list of entry labels as strings). No other user state survives app restart.

### Search
`SearchManager` uses a `SplayTreeMap` pre-populated with every possible substring of every entry label. This is O(n²) memory but provides O(log n) lookup. It is a singleton initialized once at boot.

### Testing
The `app/test/` directory exists but contains minimal tests. The app was built as a demo/showcase and does not have comprehensive test coverage.

### Asset Pipeline
The `tools/` directory contains Dart scripts for finding and resizing oversized animation assets. The `app/full_quality/` directory stores original full-resolution assets.
