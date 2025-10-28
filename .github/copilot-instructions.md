# Kdenlive Copilot Instructions

This file provides guidance to GitHub Copilot for working with the Kdenlive codebase.

## Project Overview

Kdenlive is a professional open-source non-linear video editor built with C++, Qt 6, and KDE Frameworks 6. It uses the MLT (Media Lovin' Toolkit) framework as its core video processing engine.

### Key Technologies
- **Language**: C++17/20
- **Build System**: CMake
- **GUI Framework**: Qt 6.5.0+ and KDE Frameworks 6.3.0+
- **Video Engine**: MLT 7.28.0+
- **Multimedia**: FFmpeg (libavformat, libavcodec, libswresample)
- **Timeline Format**: OpenTimelineIO (OTIO)

## Architecture

### Core Design Pattern
Kdenlive follows a strict MVC (Model-View-Controller) architecture:

- **Models**: Located in `src/*/model/` directories
  - `TimelineModel`: Manages the entire timeline (tracks, clips, transitions)
  - `ProjectItemModel`: Manages project bin (media assets)
  - `EffectStackModel`: Manages effects applied to clips/tracks
  
- **Views**: Qt Widgets and QML/QtQuick
  - Located in `src/*/view/` directories
  - QML files in `src/qml/`
  
- **Controllers**: 
  - `Core` (src/core.{h,cpp}): Main singleton coordinating all components
  - `ProjectManager`: Handles project operations
  - `MainWindow`: Main UI coordinator

### Key Components

1. **Core Singleton** (`src/core.{h,cpp}`)
   - Access via `pCore` macro
   - Coordinates all subsystems
   - Manages MLT integration

2. **Timeline System** (`src/timeline2/`)
   - `TimelineModel`: Main timeline controller
   - `TrackModel`: Individual track management
   - `ClipModel`: Clip on timeline
   - `GroupsModel`: Clip grouping
   - Each Kdenlive track = 2 MLT playlists (for mixes)

3. **Bin System** (`src/bin/`)
   - `Bin`: Main UI and coordinator
   - `ProjectClip`: Individual media asset
   - `ProjectFolder`: Asset organization
   - `SequenceClip`: Nested timeline sequences

4. **Effects & Transitions** (`src/effects/`, `src/transitions/`)
   - `EffectStackModel`: Effect stack management
   - `AssetParameterModel`: Effect parameters
   - `KeyframeModel`: Keyframe animation

## Coding Guidelines

### Code Style

- **Format**: Use `.clang-format` configuration (run `clang-format` before commits)
- **Naming**:
  - Classes: `PascalCase` (e.g., `TimelineModel`)
  - Methods: `camelCase` (e.g., `requestClipMove()`)
  - Member variables: `m_variableName` (e.g., `m_clipId`)
  - Constants: `UPPER_SNAKE_CASE`
- **Header guards**: `#pragma once`
- **Indentation**: 4 spaces (no tabs)

### Best Practices

1. **Smart Pointers**
   ```cpp
   // Use appropriate smart pointers
   std::shared_ptr<TimelineModel>  // Shared ownership
   std::unique_ptr<EffectStack>    // Exclusive ownership
   std::weak_ptr<ClipModel>        // Non-owning reference
   ```

2. **Thread Safety**
   ```cpp
   // Use Qt synchronization primitives
   QReadWriteLock m_lock;
   QMutexLocker locker(&m_mutex);
   ```

3. **Undo/Redo System**
   ```cpp
   // Always provide undo/redo lambdas for operations
   Fun undo = []() { /* revert operation */ return true; };
   Fun redo = []() { /* perform operation */ return true; };
   UPDATE_UNDO_REDO(redo, undo, undoStack);
   ```

4. **Signal/Slot Connections**
   ```cpp
   // Prefer new-style connections
   connect(model, &TimelineModel::dataChanged, 
           this, &MyClass::onDataChanged);
   ```

5. **MLT Integration**
   ```cpp
   // Always check MLT object validity
   if (!producer.is_valid()) {
       qWarning() << "Invalid MLT producer";
       return false;
   }
   ```

### Common Patterns

#### Request Pattern for Timeline Operations

All timeline modifications follow the `request*` pattern:

```cpp
bool TimelineModel::requestClipMove(int clipId, int trackId, int position,
                                   bool updateView, Fun &undo, Fun &redo) {
    // 1. Validate constraints
    if (!isValidMove(clipId, trackId, position)) {
        return false;
    }
    
    // 2. Perform operation
    // ... modify internal state and MLT objects
    
    // 3. Generate undo/redo
    undo = [this, oldState]() { /* revert */ return true; };
    redo = [this, newState]() { /* apply */ return true; };
    
    // 4. Update view if requested
    if (updateView) {
        emit dataChanged(/* ... */);
    }
    
    return true;
}
```

#### MLT Object Access Pattern

```cpp
// Get MLT service from clip
Mlt::Producer *producer = clip->getProducer();

// Set properties
producer->set("property_name", value);

// Get properties
QString value = producer->get("property_name");
```

## Project File Format

Kdenlive uses XML-based project files (`.kdenlive`) based on MLT XML format:

- Projects store **references** to media files, not the files themselves
- Kdenlive-specific properties use `kdenlive:` prefix
- Current format version: 1.1 (Generation 5, since Kdenlive 23.04.0)
- Format details in `dev-docs/fileformat.md`

### Important Properties

- `kdenlive:id`: Unique identifier for clips in bin
- `kdenlive:clipname`: Display name in bin
- `kdenlive:proxy`: Proxy file path or "-"
- `kdenlive:uuid`: Unique identifier for sequences
- `kdenlive:trackheight`: Track height in UI

## Building and Testing

### Build Commands

```bash
# Configure
cmake -B build -S . \
  -DCMAKE_BUILD_TYPE=Debug \
  -DBUILD_TESTING=ON

# Build
cmake --build build -j$(nproc)

# Install
cmake --install build --prefix /usr/local
```

### Running Tests

```bash
cd build
ctest --output-on-failure
```

### Code Quality Tools

```bash
# Format code
clang-format -i src/**/*.{cpp,h}

# Static analysis
clang-tidy src/main.cpp -- -Isrc

# Run pre-commit hooks
pre-commit run --all-files
```

## Common Tasks

### Adding a New Effect

1. Create XML definition in `data/effects/`
2. Effect will be automatically loaded by asset system
3. UI parameters defined in XML using MLT parameter types
4. No C++ code needed for simple effects

### Adding a New Timeline Feature

1. Implement in `TimelineModel` or appropriate model class
2. Add undo/redo support
3. Update QML view in `src/timeline2/view/`
4. Add unit tests in `tests/`

### Working with MLT

```cpp
// Initialize MLT (done in Core::build())
Mlt::Factory::init();

// Create producer from file
Mlt::Profile profile(profilePath);
Mlt::Producer producer(profile, "avformat", filepath.toUtf8().constData());

// Apply filter (effect)
Mlt::Filter filter(profile, "brightness");
filter.set("level", 150);
producer.attach(filter);

// Create consumer (output)
Mlt::Consumer consumer(profile, "sdl2");
consumer.connect(producer);
consumer.start();
```

### Debugging Tips

1. **Enable Debug Logging**
   ```bash
   export QT_LOGGING_RULES="kdenlive.*=true"
   export KDENLIVE_DEBUG=1
   ```

2. **MLT Debug**
   ```bash
   export MLT_LOG_LEVEL=debug
   ```

3. **Check MLT XML**
   ```bash
   # Test project file with melt
   melt project.kdenlive
   ```

## File Organization

```
kdenlive/
├── src/
│   ├── core.{h,cpp}           # Core singleton
│   ├── main.cpp               # Application entry point
│   ├── mainwindow.{h,cpp}     # Main window
│   ├── timeline2/             # Timeline system (most active development)
│   │   ├── model/            # Timeline models
│   │   └── view/             # Timeline QML views
│   ├── bin/                   # Media bin
│   ├── effects/               # Effects system
│   ├── project/               # Project management
│   ├── monitor/               # Preview monitors
│   └── render/                # Rendering system
├── renderer/                   # CLI render tool (kdenlive_render)
├── data/                      # Application data
│   ├── effects/              # Effect definitions (XML)
│   └── transitions/          # Transition definitions (XML)
├── dev-docs/                  # Developer documentation
├── tests/                     # Unit tests
└── CMakeLists.txt            # Main build configuration
```

## Important Notes

### Do's ✓

- **Always** use the undo/redo system for user-facing operations
- **Always** validate MLT objects before use (`is_valid()`)
- **Always** check return values from `request*` functions
- Use `pCore` to access global resources
- Add Qt logging categories for new modules
- Follow the existing MVC separation
- Write unit tests for new features
- Use MLT documentation: https://www.mltframework.org/docs/

### Don'ts ✗

- Don't bypass the `TimelineModel` for timeline modifications
- Don't directly modify MLT objects without updating models
- Don't create circular dependencies with smart pointers
- Don't use raw pointers for ownership (use smart pointers)
- Don't assume MLT objects are thread-safe (they're not)
- Don't modify project XML structure without updating version handling
- Don't add UI code to model classes
- Don't forget to handle locale issues (decimal separator: comma vs. point)

## Key Resources

- **MLT Documentation**: https://www.mltframework.org/docs/
- **Qt Documentation**: https://doc.qt.io/qt-6/
- **KDE Frameworks**: https://api.kde.org/frameworks/
- **Kdenlive User Manual**: https://docs.kdenlive.org/
- **Developer Docs**: `dev-docs/` directory
  - `architecture.md`: Architecture overview
  - `fileformat.md`: Project file format details
  - `mlt-intro.md`: MLT concepts introduction
  - `build.md`: Build instructions
  - `coding.md`: Coding guidelines

## Specific Contexts

### When Working on Timeline
- Focus on `src/timeline2/model/` for logic
- Update `src/timeline2/view/` for UI changes
- Remember: each track has 2 MLT playlists for mix support
- Test with undo/redo
- Consider group constraints when moving clips

### When Working on Bin
- Media files are **referenced**, not embedded
- Proxy files are optional low-res versions
- `ProjectClip` in bin can have multiple `ClipModel` instances in timeline
- Audio waveforms and thumbnails are cached

### When Working on Effects
- Effects are MLT filters
- Effect definitions are XML files in `data/effects/`
- Effect stack is ordered (top-to-bottom application)
- Keyframes use `KeyframeModel` for animation
- Some effects support GPU acceleration

### When Working on Rendering
- Use `kdenlive_render` CLI tool for batch rendering
- Two modes: delivery (final) and preview-chunks (timeline preview)
- Render presets in `src/renderpresets/`
- Consumer parameters are FFmpeg/libavformat options

## Performance Considerations

1. **Proxy Files**: Enable for 4K+ footage to improve editing performance
2. **Timeline Preview**: Uses incremental rendering (only changed sections)
3. **Threading**: Use `QThreadPool` for background tasks via `TaskManager`
4. **Caching**: Thumbnails and waveforms use `KSharedDataCache`
5. **MLT Optimization**: MLT supports hardware acceleration (configure via profiles)

## Getting Help

- **Matrix Chat**: `#kdenlive-dev:kde.org`
- **Mailing List**: kdenlive@kde.org
- **Bug Tracker**: https://bugs.kde.org (product: kdenlive)
- **Code Review**: https://invent.kde.org/multimedia/kdenlive (KDE GitLab)

## License

Kdenlive is licensed under GPL-3.0-only OR LicenseRef-KDE-Accepted-GPL. Ensure all contributions include proper SPDX headers:

```cpp
// SPDX-FileCopyrightText: 2024 Your Name <your.email@example.com>
// SPDX-License-Identifier: GPL-3.0-only OR LicenseRef-KDE-Accepted-GPL
```

---

*This instruction file is designed to help GitHub Copilot provide better suggestions when working on Kdenlive.*
*Last updated: 2024-10-24*
