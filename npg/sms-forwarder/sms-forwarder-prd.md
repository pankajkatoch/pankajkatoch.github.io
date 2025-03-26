# PRD: Android Dark Theme Implementation Sample

**Version:** 1.0
**Date:** 2023-10-27
**Author:** Senior Product Manager
**Status:** Draft

## 1. Introduction

This document outlines the product requirements for the **Android Dark Theme Implementation Sample** application. This application serves as an educational tool and reference implementation for Android developers. Its primary purpose is to demonstrate various recommended techniques for effectively supporting Dark Theme (or Dark Mode) within an Android application, leveraging the capabilities provided by the Android framework, AppCompat library, and Material Design Components library. The sample focuses on clarity, best practices, and showcasing different approaches to achieve robust Dark Theme support, including backward compatibility.

*Note: While the codebase contains elements related to an "SMS Forwarder" feature (rules, SMS permissions, receiver), the primary documented goal and focus of this sample, as per the README, is the demonstration of Dark Theme techniques. This PRD will focus on the Dark Theme educational aspects.*

## 2. Goals

*   **Educate Developers:** Provide clear, concise, and runnable examples of implementing Dark Theme on Android.
*   **Showcase Best Practices:** Demonstrate recommended approaches using `AppCompatDelegate`, `DayNight` themes, resource qualifiers (`-night`), theme attributes (`?attr/...`), and Material Design Components.
*   **Illustrate Different Techniques:** Cover multiple methods for handling theming, including theme attributes for colors/drawables, `-night` qualifiers for overrides, programmatic tinting, and the `forceDarkAllowed` (Smart Dark) mechanism.
*   **Promote Material Design:** Encourage the use of Material Design Components and their built-in support for theming.
*   **Ensure Clarity:** Make it easy for developers to find and understand the code corresponding to each technique demonstrated.

## 3. Target Audience

*   **Android Developers:** Primarily intermediate to experienced developers seeking to implement or improve Dark Theme support in their new or existing applications.
*   **UI/UX Designers:** Can use the sample to understand the visual implications and options available for Dark Theme on Android.

## 4. Features

### 4.1 Core Theming Engine (AppCompat DayNight)

*   **Requirement:** The application MUST utilize a `Theme.MaterialComponents.DayNight` theme as its base theme.
*   **Rationale:** Provides robust Dark Theme support with backward compatibility (down to API 14) and integrates seamlessly with Material Components.
*   **Implementation Notes:** Defined in `values/styles.xml` and referenced in `AndroidManifest.xml`.

### 4.2 User Theme Selection

*   **Requirement:** The application MUST provide a user-facing setting to explicitly select the theme preference.
*   **Requirement:** The available options MUST include:
    *   Light Mode (`MODE_NIGHT_NO`)
    *   Dark Mode (`MODE_NIGHT_YES`)
    *   System Default (`MODE_NIGHT_FOLLOW_SYSTEM` on Android 10+) / Set by Battery Saver (`MODE_NIGHT_AUTO_BATTERY` pre-Android 10).
*   **Requirement:** The user's selected preference MUST be persisted (`SharedPreferences`) and applied upon subsequent app launches.
*   **Rationale:** Aligns with Google's recommendations for user control over app themes and ensures a consistent experience.
*   **Implementation Notes:** Implemented in `SettingsFragment` using the `androidx.preference` library (`preferences.xml`), options defined in `arrays.xml` (and `-v28`), persistence handled in `DarkThemeApplication` and `ThemeHelper`.

### 4.3 Theming Techniques Demonstration

The sample MUST demonstrate the following techniques:

*   **4.3.1 Theme Attribute Usage:**
    *   **Requirement:** Demonstrate using theme attributes (e.g., `?attr/colorPrimary`, `?attr/colorOnBackground`, `?attr/colorSurface`) for styling UI elements (backgrounds, text colors, icon tints).
    *   **Rationale:** This is the most flexible and recommended approach, allowing colors to adapt automatically based on the current theme (Light/Dark).
    *   **Implementation Notes:** Examples in `values/styles.xml`, `drawable/bottom_nav_item_background.xml`, `drawable/ic_brightness_2.xml`.

*   **4.3.2 `-night` Resource Qualifiers:**
    *   **Requirement:** Demonstrate overriding specific resource values (primarily colors) for Dark Theme using the `-night` qualifier (e.g., `values-night/colors.xml`).
    *   **Rationale:** Useful for cases where theme attributes aren't sufficient or specific overrides are needed (e.g., different primary color shades).
    *   **Implementation Notes:** `values/colors.xml` vs `values-night/colors.xml` for `@color/primary`.

*   **4.3.3 Vector Drawable Tinting:**
    *   **Requirement:** Demonstrate tinting vector drawables using:
        *   Theme attributes within the drawable XML (`android:tint="?attr/..."`).
        *   Setting the tint via layout XML (`app:tint` or `android:tint`).
        *   Programmatic tinting in code.
    *   **Rationale:** Shows different ways to make icons theme-aware without duplicating assets.
    *   **Implementation Notes:** `drawable/ic_brightness_2.xml`, `fragment_welcome.xml`, `MainActivity.java` for menu icon tinting.

*   **4.3.4 `ColorStateList` for Variations:**
    *   **Requirement:** Demonstrate using a `ColorStateList` resource (`color/`) to apply variations (like alpha) to a theme attribute color.
    *   **Rationale:** Allows reusing base theme colors with modifications without hardcoding new color values.
    *   **Implementation Notes:** `color/color_on_primary_mask.xml`.

*   **4.3.5 Smart Dark (`forceDarkAllowed`) Opt-In:**
    *   **Requirement:** Demonstrate how to opt-in a specific view hierarchy for Android Q's (10+) "Smart Dark" (also known as Force Dark) feature using `android:forceDarkAllowed="true"`. This should be applied to a layout that intentionally uses hardcoded light theme colors.
    *   **Requirement:** Explain that this feature is primarily for apps *without* explicit Dark Theme support and can be inconsistent.
    *   **Rationale:** Shows developers how this automatic system feature works and how to control it, while highlighting it's not the preferred method for new development.
    *   **Implementation Notes:** `layout/fragment_preferences.xml` (Note: This layout seems repurposed for SMS rules in the code, but the `forceDarkAllowed` attribute demonstrates the concept described in the README).

### 4.4 Basic Application Structure

*   **Requirement:** The application MUST have a basic structure including a main `Activity`, `Toolbar`, and `Fragment` container to host the different demonstration screens/settings.
*   **Rationale:** Provides a standard Android application context for the theme demonstrations.
*   **Implementation Notes:** `MainActivity.java`, `activity_main.xml`, various `Fragment` classes (Welcome, Preferences/Rules, Settings).

## 5. Design & UX Considerations

*   **Clarity:** The visual difference between Light and Dark modes for each demonstrated technique MUST be obvious.
*   **Material Design:** The overall UI SHOULD adhere to Material Design guidelines.
*   **Simplicity:** The UI should be simple and focused on showcasing the theming aspects, avoiding unnecessary complexity.
*   **Code Association:** It should be reasonably easy for a developer browsing the code to associate specific UI elements with the theming technique being demonstrated (aided by the README).

## 6. Technical Considerations

*   **Libraries:** Must use `androidx.appcompat` and `com.google.android.material:material`.
*   **Build:** Must be buildable using standard Android Studio and Gradle.
*   **Compatibility:** DayNight theme should target compatibility down to API 14. Smart Dark demonstration is relevant for API 29+.

## 7. Release Criteria / Definition of Done (for Sample)

*   All features listed in section 4 are implemented and functional.
*   The application builds successfully without errors.
*   The application runs correctly on target Android versions.
*   Each demonstrated theming technique is clearly visible and behaves as expected when switching themes (Light, Dark, System/Battery).
*   The README accurately describes the features and how they are implemented in the code.
*   Code is reasonably clean and understandable for its educational purpose.

## 8. Future Considerations / Potential Enhancements

*   Add examples for theming custom views.
*   Demonstrate handling of theme changes during runtime more explicitly (if not already covered).
*   Include examples related to Jetpack Compose theming.
*   Add more complex drawable state examples (pressed, disabled) with theme-aware colors.
*   **Scope Clarification:** Resolve the discrepancy between the documented "Dark Theme Sample" goal and the implemented "SMS Forwarder" functionality. Consider splitting into two separate samples or removing the unrelated SMS code for clarity.

## 9. Open Issues

*   The codebase contains significant "SMS Forwarder" functionality not directly related to the core Dark Theme demonstration described in the `README.md`. This adds complexity and potential confusion for developers using this purely as a Dark Theme sample. Recommend clarifying scope or refactoring.
*   The `BottomNavigationView` is present in `activity_main.xml` and `navigation.xml` but seems disabled/unused in `MainActivity.java`. Recommend removing unused elements.

