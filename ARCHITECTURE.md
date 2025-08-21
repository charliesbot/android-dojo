# Android Dojo - Simple Multi-Platform Architecture

## Overview

Multi-platform Android app architecture with **2 shared modules** and **multiple platform modules**.

**Shared modules:**

- `:core` - Business logic, data, shared UI
- `:features` - All app features as packages

**Platform modules:**

- `:app` - Mobile Android
- `:wearos` - WearOS
- `:tv` - Android TV (optional)
- `:auto` - Android Auto (optional)

## What is a Feature?

A **feature** is a complete user journey or business capability, not just a single screen. This distinction is crucial for creating a clean and maintainable structure.

### ✅ Good Features (Business Capabilities)

- **auth**: The entire authentication flow (login, register, forgot password).
- **profile**: User profile management (view, edit, settings).
- **cart**: The complete shopping cart and checkout process.

### ❌ Poor Features (Just Screens)

- **login-screen**: Too granular. This should be part of the `auth` feature.
- **settings-screen**: Should be part of the `profile` feature.

By grouping related screens and logic into feature packages within the `:features` module, you create cohesive and self-contained units that are easier to manage and test.

## Simplified Module Structure

```
android-dojo/
├── app/                      # Mobile Android app module
│   └── src/main/kotlin/com/yourpackage/
│       ├── MainActivity.kt
│       ├── di/
│       │   └── AppModule.kt          # Loads all DI modules (core + features)
│       └── navigation/           # Navigation 3 setup
│           ├── AppNavigation.kt      # NavDisplay, entryProvider, sceneStrategy
│           ├── NavigationRoutes.kt   # Defines all serializable NavKey objects
│           └── scenes/
│               └── DashboardScene.kt
│
├── wearos/                   # WearOS app module
│   └── src/main/kotlin/com/yourpackage/wear/
│       ├── MainActivity.kt
│       ├── di/
│       │   └── WearAppModule.kt      # Loads DI modules for wear platform
│       └── navigation/           # WearOS-specific navigation (Wear Compose Navigation)
│           ├── WearNavigation.kt     # Contains SwipeDismissableNavHost
│           └── WearRoutes.kt         # Sealed class routes for type safety
│
├── core/                     # SINGLE unified core module
│   └── src/main/kotlin/com/yourpackage/core/
│       ├── common/           # Pure Kotlin utilities, Result class, extensions
│       ├── data/             # ALL repositories, DAOs, network APIs, Room/Retrofit setup
│       │   ├── local/        # Room database, DAOs, entities
│       │   ├── remote/       # Retrofit APIs, DTOs, network layer
│       │   └── repository/   # Repository implementations
│       ├── domain/           # ALL domain models and use cases
│       │   ├── model/        # Business models (User, etc.)
│       │   ├── repository/   # Repository interfaces
│       │   └── usecase/      # Use cases
│       ├── ui/               # Shared design system
│       │   ├── theme/        # App theme, colors, typography
│       │   ├── component/    # Reusable UI components
│       │   └── util/         # UI utilities
│       └── di/               # Core infrastructure DI (repositories, network, database)
│
└── features/                 # SINGLE module containing all features as packages
    └── src/main/kotlin/com/yourpackage/features/
        ├── auth/             # Auth feature package (presentation only)
        │   ├── di/
        │   │   └── AuthModule.kt     # DI for the ViewModel
        │   │
        │   ├── AuthViewModel.kt      # SHARED ViewModel for mobile & wear
        │   │
        │   ├── LoginScreen.kt        # Mobile or shared UI screen
        │   │
        │   └── wear/                 # Sub-package for WearOS-specific UI
        │       └── WearLoginScreen.kt
        │
        ├── dashboard/        # Dashboard feature package
        │   ├── di/
        │   │   └── DashboardModule.kt # DI for dashboard ViewModels
        │   ├── DashboardViewModel.kt
        │   └── DashboardScreen.kt
        │
        └── profile/          # Additional features as packages...
```

## Platform-Specific Navigation

A key strength of this architecture is how it isolates platform-specific implementations. Navigation is a perfect example of this.

**`:app` Module**: Uses the Navigation 3 library (`androidx.navigation3`) to handle adaptive layouts with scenes, a savable back stack with keys, and a central `NavDisplay`.

**`:wearos` Module**: Uses the specialized Wear Compose Navigation library (`androidx.wear.compose:compose-navigation`), which provides components tailored for watches, like the `SwipeDismissableNavHost`.

The `:features` module simply provides the `@Composable` screens. The `:app` and `:wearos` modules are independently responsible for calling those screens using the correct navigation library for their platform.

## Why This is a Good Starting Point

**Minimal Overhead**: You only manage a few core modules instead of many. Adding new features doesn't require creating new modules - just new packages within `:features`.

**Multi-platform Ready**: All platform modules (`:app`, `:wearos`, `:tv`, `:auto`, etc.) can share feature code from day one without any additional setup.

**Keeps Clean Dependencies**: Features packages cannot depend on each other, only on `:core`. This prevents your project from becoming a "ball of mud."

**Easy to Evolve**: If your project grows, you can easily extract individual feature packages into separate modules (e.g., `:features:auth`) or split the `:core` module into sub-modules.

## The Dependency Flow Remains the Same

The fundamental principle is unchanged. The dependency direction is still strictly enforced:

```
app/wearos → features → core
```

## Example Module Dependencies

```kotlin
// In app/build.gradle.kts and wearos/build.gradle.kts
dependencies {
    // Both platforms depend on the same modules
    implementation(project(":core"))
    implementation(project(":features"))
}

// In features/build.gradle.kts
dependencies {
    // Features module depends ONLY on the core module
    implementation(project(":core"))

    // NO dependency on app/wearos modules allowed!
    // ❌ implementation(project(":app")) // This would cause circular dependency
}

// In core/build.gradle.kts
dependencies {
    // Core has no dependency on other project modules
    // Only external libraries (Retrofit, Room, etc.)
}
```

This simplified structure maintains all the critical architectural benefits while reducing initial setup complexity, making it ideal for personal projects.

## Tech Stack

- **Dependency Injection**: Koin
- **Networking**: Retrofit
- **Database**: Room
- **Navigation**:
  - Mobile: Navigation 3 (`androidx.navigation3`)
  - WearOS: Wear Compose Navigation (`androidx.wear.compose:compose-navigation`)
- **Platforms**: Mobile Android + WearOS

## Benefits

- **Solo Development Optimized**: Minimal modules reduces overhead while maintaining benefits
- **Multi-platform Code Sharing**: Features work across all platforms (mobile, wear, TV, auto) from day one
- **Faster Builds**: Gradle caches unchanged modules, fewer modules to compile
- **Platform Flexibility**: Each platform uses optimal navigation solution
- **Easy Evolution**: Can extract individual features or split core later if needed
- **Feature Organization**: Features are organized as packages within single module

## Getting Started

1. **Create unified core**: Build single `:core` module with all shared logic
2. **Create features module**: Build single `:features` module
3. **Add first feature**: Create `auth` package within `:features` with ViewModel and screens
4. **Platform setup**: Implement platform modules (`:app`, `:wearos`, `:tv`, etc.) with appropriate navigation, all depending on `:features`
5. **Iterate**: Add more feature packages as needed, each containing only presentation layer
