# Android Dojo - Simplified Multi-Module Architecture (For Personal Projects)

## Project Overview

Android application supporting both **Mobile** and **WearOS** platforms using a pragmatic multi-module architecture with a single, unified core module for all shared business logic and presentation-only features.

## Key Simplification

We are consolidating `core/data`, `core/domain`, `core/ui`, and `core/di` into a single `:core` module. This reduces the initial setup complexity while still providing the most critical benefit: **compile-time separation between feature modules**.

## What is a Feature?

A **feature** is a complete user journey or business capability, not just a single screen. This distinction is crucial for creating a clean and maintainable structure.

### ✅ Good Features (Business Capabilities)

- **auth**: The entire authentication flow (login, register, forgot password).
- **profile**: User profile management (view, edit, settings).
- **cart**: The complete shopping cart and checkout process.

### ❌ Poor Features (Just Screens)

- **login-screen**: Too granular. This should be part of the `auth` feature.
- **settings-screen**: Should be part of the `profile` feature.

By grouping related screens and logic into a single feature module, you create a cohesive and self-contained unit that is easier to manage and test.

## Simplified Module Structure

```
android-dojo/
├── app/                      # Mobile Android app module
│   └── src/main/kotlin/com/yourpackage/
│       ├── MainActivity.kt
│       └── navigation/           # Navigation 3 setup
│           ├── AppNavigation.kt      # NavDisplay, entryProvider, sceneStrategy
│           ├── NavigationRoutes.kt   # Defines all serializable NavKey objects
│           └── scenes/
│               └── DashboardScene.kt
│
├── wearos/                   # WearOS app module
│   └── src/main/kotlin/com/yourpackage/wear/
│       ├── MainActivity.kt
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
│       └── di/               # Centralized Koin DI modules
│
└── features/                 # Presentation-only feature modules
    ├── auth/                 # Auth feature (presentation only)
    │   └── src/main/kotlin/com/yourpackage/auth/
    │       ├── di/
    │       │   └── AuthModule.kt     # DI for the ViewModel
    │       │
    │       ├── AuthViewModel.kt      # SHARED ViewModel for mobile & wear
    │       │
    │       ├── LoginScreen.kt        # Mobile or shared UI screen
    │       │
    │       └── wear/                 # Sub-package for WearOS-specific UI
    │           └── WearLoginScreen.kt
    │
    └── dashboard/            # Additional features follow the same pattern...
```

## Platform-Specific Navigation

A key strength of this architecture is how it isolates platform-specific implementations. Navigation is a perfect example of this.

**`:app` Module**: Uses the Navigation 3 library (`androidx.navigation3`) to handle adaptive layouts with scenes, a savable back stack with keys, and a central `NavDisplay`.

**`:wearos` Module**: Uses the specialized Wear Compose Navigation library (`androidx.wear.compose:compose-navigation`), which provides components tailored for watches, like the `SwipeDismissableNavHost`.

The feature modules simply provide the `@Composable` screens. The `:app` and `:wearos` modules are independently responsible for calling those screens using the correct navigation library for their platform.

## Why This is a Good Starting Point

**Low Friction**: You only have to manage one `:core` module instead of five. Adding a new repository or use case doesn't require thinking about which sub-module it belongs to.

**Keeps the Golden Rule**: The most important architectural guarantee is preserved. Features cannot depend on other features, only on `:core`. This prevents your project from becoming a "ball of mud."

**Easy to Evolve**: If your project grows and the single `:core` module becomes too large, you can easily refactor it later by splitting it back into `:core:data`, `:core:domain`, etc. The module boundaries make this a safe and predictable change.

## The Dependency Flow Remains the Same

The fundamental principle is unchanged. The dependency direction is still strictly enforced:

```
app/wearos → features → core
```

## Example Module Dependencies

```kotlin
// In app/build.gradle.kts
dependencies {
    // App depends on the single core module and the features it needs
    implementation(project(":core"))
    implementation(project(":features:auth"))
    implementation(project(":features:dashboard"))
}

// In features:auth/build.gradle.kts
dependencies {
    // Feature depends ONLY on the core module
    implementation(project(":core"))

    // NO dependency on other features is allowed!
    // ❌ implementation(project(":features:dashboard")) // This would cause a build error
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

- **Solo Development Optimized**: Single `:core` module reduces overhead while maintaining benefits
- **Compiler-Enforced Independence**: Features CANNOT depend on each other (build error)
- **Faster Builds**: Gradle caches unchanged feature modules
- **Platform Flexibility**: Each platform uses optimal navigation solution
- **Easy Evolution**: Can split core later if project actually grows
- **True Feature Isolation**: Each feature is self-contained and removable

## Getting Started

1. **Create unified core**: Build single `:core` module with all shared logic
2. **Add first feature**: Build `:features:auth` with ViewModel and screens
3. **Platform setup**: Implement `:app` and `:wearos` with appropriate navigation
4. **Iterate**: Add more features as needed, each containing only presentation layer

This architecture balances simplicity with maintainability, giving you a solid foundation that can grow with your project while keeping the complexity manageable for solo development.

