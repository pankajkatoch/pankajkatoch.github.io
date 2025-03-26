# Design Doc: Android Dark Theme Sample (SMS Forwarder)

**Version:** 1.0
**Date:** 2023-10-27
**Author:** AI Assistant (based on provided codebase)

## 1. Overview

This document outlines the design for an Android application primarily demonstrating various techniques for implementing Dark Theme (Dark Mode) support, specifically using the Material Components for Android library. The sample application also includes core functionality as an SMS Forwarder, allowing users to define rules for automatically forwarding incoming SMS messages based on content to a specified phone number.

The primary focus of this document is the Dark Theme implementation strategy and related components. The SMS Forwarding features are described as part of the application's functional context.

**Key Technologies:**

*   Android SDK
*   Kotlin & Java
*   Material Components for Android
*   AndroidX Libraries (AppCompat, Fragment, Preference, Navigation, ConstraintLayout, RecyclerView, Core-KTX, Lifecycle)
*   Gradle Build System

## 2. Goals

*   Demonstrate best practices for supporting Light, Dark, and System-default themes on Android.
*   Showcase the use of AppCompat's `DayNight` theme functionality for backward compatibility (API 14+).
*   Illustrate the use of resource qualifiers (`-night`) for theme-specific resources (colors, drawables).
*   Explain and demonstrate the use of theme attributes (`?attr/...`) for themable UI elements (colors, tints).
*   Demonstrate the "Smart Dark" (Force Dark) feature available on Android Q (API 29) and later for themes/views that don't explicitly support dark mode.
*   Provide a user setting to choose the preferred theme (Light, Dark, System Default/Battery Saver).
*   Persist the user's theme preference across app launches.
*   Implement basic SMS forwarding functionality based on user-defined rules.
*   Use Material Design components throughout the application.

## 3. Non-Goals

*   A production-ready, robust SMS forwarding service with comprehensive error handling and security features.
*   Complex UI patterns beyond demonstrating basic theming concepts.
*   Support for themes other than standard Light and Dark.
*   Dynamic theme switching without Activity recreation (though `AppCompatDelegate` handles this).
*   Extensive unit or integration tests (beyond basic structure provided).

## 4. Architecture

The application follows a standard Android architecture pattern, primarily using a Single-Activity (`MainActivity`) approach with multiple Fragments managed via basic Fragment transactions.

*   **Application Class (`DarkThemeApplication`):** Initializes and applies the user's preferred theme upon application startup.
*   **Activity (`MainActivity`):** Acts as the main container for Fragments. Hosts the Toolbar. Handles navigation between core sections (though the `BottomNavigationView` is initially hidden/unused in the current flow) and manages SMS permission requests.
*   **Fragments:**
    *   `WelcomeFragment`: Initial screen shown on first launch. Basic layout demonstrating themed elements.
    *   `PreferencesFragment`: Displays and manages SMS forwarding rules. Demonstrates Smart Dark (`forceDarkAllowed="true"`) for its layout. Handles rule creation via `AddRuleDialog`.
    *   `SettingsFragment`: Provides app-level settings (currently, toggling individual rule enable/disable feature visibility). It *does not* contain the theme selection preference itself in this implementation (that's in `xml/preferences.xml` used potentially by a different, perhaps missing, PreferenceFragmentCompat, or was intended to be here). *Correction based on `MainActivity`: `SettingsFragment` is shown from the Options Menu, and `PreferencesFragment` is the main screen after `WelcomeFragment`.* *Further Correction based on `SettingsFragment.java` and `PreferencesFragment.java`: `PreferencesFragment` handles rules; `SettingsFragment` handles the "Allow individual rule enable/disable" switch.*
    *   `AddRuleDialog`: A `DialogFragment` used to add new SMS forwarding rules.
*   **BroadcastReceiver (`SMSReceiver`):** Listens for incoming SMS messages (`android.provider.Telephony.SMS_RECEIVED`), evaluates them against saved rules, and forwards them if a match is found and the app/rule is enabled.
*   **Data Persistence (`SharedPreferences`):** Used to store:
    *   User's theme preference (managed by `androidx.preference` library via `xml/preferences.xml` if a `PreferenceFragmentCompat` were used, or potentially manually if `SettingsFragment` were different). *Based on `DarkThemeApplication`, it reads from default SharedPreferences, likely populated by a Preference screen.*
    *   SMS Forwarding Rules (`Util.SAVED_RULES`, stored as JSON).
    *   App/Rule enable/disable states (`Util.IS_APP_ENABLED`, `RuleDef.onOffButton`).
    *   Other settings (`Util.ALLOW_INDIVIDUAL_RULES`, `Util.IS_SPLASH_SHOWN`).
*   **Utilities (`Util`, `ThemeHelper`, `ColorUtils`):** Helper classes for JSON serialization, SharedPreferences access, theme application logic, and retrieving theme colors.
*   **Data Model (`RuleDef`):** Simple POJO representing an SMS forwarding rule.

## 5. Detailed Design

### 5.1. Theme Implementation Strategy

The app employs two primary strategies for Dark Theme support:

#### 5.1.1. AppCompat DayNight Theme

This is the primary method used for most of the app.

*   **Base Theme:** The application theme `DarkThemeApp` (defined in `values/styles.xml#L16`) inherits from `Theme.MaterialComponents.DayNight.NoActionBar`. This base theme automatically switches between its `*.Light` and `*.Dark` variants based on the current night mode setting.
*   **Night Mode Management:**
    *   The `ThemeHelper` class (`Application/src/main/java/com/example/android/darktheme/ThemeHelper.java`) provides the `applyTheme` method.
    *   This method calls `AppCompatDelegate.setDefaultNightMode()` based on a preference string ("light", "dark", "default").
    *   Supported modes used:
        *   `MODE_NIGHT_NO`: Forces Light theme.
        *   `MODE_NIGHT_YES`: Forces Dark theme.
        *   `MODE_NIGHT_FOLLOW_SYSTEM` (Android Q+): Uses the system-wide Dark Theme setting.
        *   `MODE_NIGHT_AUTO_BATTERY` (Pre-Q): Uses Dark Theme when Battery Saver is active.
    *   The chosen theme preference is retrieved from `SharedPreferences` and applied in `DarkThemeApplication.onCreate()` (`Application/src/main/java/com/example/android/darktheme/DarkThemeApplication.java`), ensuring the correct theme is set *before* any Activities are created.
*   **Theme Attributes:** UI elements rely heavily on theme attributes for colors and drawables to ensure they adapt correctly. Examples:
    *   `colorPrimary`, `colorSecondary`, `colorError`: Defined in `values/styles.xml` referencing `@color/*` resources.
    *   `colorOnBackground`, `colorOnPrimary`, `colorSurface`, `colorControlNormal`: Standard Material theme attributes used for text, icons, and surfaces. (e.g., `values/styles.xml#L25`, `drawable/baseline_phone_black_48dp.xml#L6`, `drawable/bottom_nav_item_background.xml`)
    *   `android:textColor="?attr/colorOnBackground"` (`values/styles.xml#L25`)
    *   `android:windowBackground`: Set in the theme (`values/styles.xml`, implicitly via parent) to control the default window background color (adapts splash screen).
*   **Resource Qualifiers (`-night`):** Specific resources are overridden for dark mode using the `-night` qualifier.
    *   `values-night/colors.xml`: Defines different values for `@color/primary`, `@color/secondary`, `@color/error` compared to `values/colors.xml`.
    *   `drawable-night/`: Could be used for providing entirely different drawable assets for dark mode (though not explicitly shown in the provided file list, it's a standard technique mentioned in the README).
*   **ColorStateList:** Used to apply variations (like alpha) based on theme attributes. Example: `color/color_on_primary_mask.xml` applies an alpha to `?attr/colorOnPrimary`.

#### 5.1.2. Smart Dark (Force Dark)

This feature is specifically demonstrated in the `PreferencesFragment`.

*   **Purpose:** To provide automatic dark theming for views/layouts that were originally designed only for a light theme (using hardcoded light colors). This is intended as a fallback for apps without full dark theme support.
*   **Activation:** Enabled by the system on Android Q+ when the system dark theme is on AND the app's theme declares `android:isLightTheme="true"` (which `Theme.*.Light.*` themes do).
*   **Opt-in/Opt-out:** Developers can control this feature:
    *   **Layout Opt-in:** The `PreferencesFragment` layout (`layout/fragment_preferences.xml#L9`) uses `android:forceDarkAllowed="true"` on its root `LinearLayout`.
    *   **View Opt-out:** Individual views can opt-out using `android:forceDarkAllowed="false"` or `view.setForceDarkAllowed(false)`.
    *   **Theme Opt-out:** A theme can set `android:forceDarkAllowed="false"`.
*   **Mechanism:** The system dynamically analyzes the view hierarchy at draw time and attempts to invert most light colors (backgrounds, text, icons) to suitable dark equivalents.
*   **Demonstration:** The `PreferencesFragment` layout (`layout/fragment_preferences.xml`) is intentionally designed with hardcoded light-theme suitable colors (implicitly, as no dark-specific overrides are provided for its specific views within the layout itself, unlike other fragments that rely on theme attributes). When Force Dark is active (either system-wide on Q+ or forced via `adb shell setprop debug.hwui.force_dark true`), this fragment's appearance will be automatically inverted by the system due to the `forceDarkAllowed="true"` attribute.

### 5.2. Core Components & Functionality

*   **`DarkThemeApplication`:**
    *   Entry point for the application process.
    *   Reads the saved theme preference from default `SharedPreferences`.
    *   Calls `ThemeHelper.applyTheme()` to set the appropriate `AppCompatDelegate.setDefaultNightMode()` *before* the UI is created.
*   **`MainActivity`:**
    *   Sets the main content view (`R.layout.activity_main`).
    *   Sets up the `Toolbar` as the `ActionBar`.
    *   Manages fragment transactions to display `WelcomeFragment`, `PreferencesFragment`, or `SettingsFragment`.
    *   Handles the initial check for whether the `WelcomeFragment` needs to be shown (`Util.IS_SPLASH_SHOWN` preference).
    *   Requests necessary SMS permissions (`RECEIVE_SMS`, `SEND_SMS`) using `ActivityCompat.requestPermissions`.
    *   Includes logic for handling `BottomNavigationView` (though it's initially hidden).
    *   Sets status bar to transparent and enables fullscreen flags (this might conflict slightly with standard Material theming expectations for the status bar background, depending on the desired effect).
    *   Handles `onOptionsItemSelected` for potential menu actions (like navigating to `SettingsFragment`).
*   **`ThemeHelper`:** Encapsulates the logic for mapping a preference string ("light", "dark", "default") to the corresponding `AppCompatDelegate.setDefaultNightMode()` constant, handling SDK version differences for the "default" case.
*   **`WelcomeFragment`:**
    *   Simple introductory screen (`layout/fragment_welcome.xml`).
    *   Displays introductory text and a logo.
    *   Contains a "Proceed" button (`R.id.button_proceed`) which navigates to the `PreferencesFragment`.
    *   UI elements like `TextView`s and `Button`s will inherit themed colors/styles.
*   **`PreferencesFragment`:**
    *   Displays the list of SMS forwarding rules (`layout/fragment_rules.xml`).
    *   Uses a `ScrollView` containing a `LinearLayout` (`R.id.rulesList`) to display rules dynamically.
    *   Each rule is rendered possibly using `MaterialCardView` (`layout/fragment_rule.xml` seems intended for this, although the rendering logic in `PreferencesFragment.java` dynamically creates `LinearLayouts`, `TextViews`, `Switches`, `ImageViews` and adds them to `MaterialCardView` manually).
    *   Includes a master switch (`R.id.enableDisableApp`) to enable/disable all forwarding (`Util.IS_APP_ENABLED`). This switch overlays a semi-transparent "disabler" view (`R.id.rulesListScrollerWrapperDisabler`) over the rules list when forwarding is disabled.
    *   Includes an "Add new rule" button (`R.id.button`) which shows the `AddRuleDialog`.
    *   Implements `AddRuleDialog.AddRuleDialogListener` to receive new rules.
    *   Uses `android:forceDarkAllowed="true"` in its root layout (`layout/fragment_rules.xml`) to demonstrate Smart Dark.
    *   Dynamically renders rule lines, including delete icons and optional enable/disable switches per rule (visibility controlled by `Util.ALLOW_INDIVIDUAL_RULES`).
*   **`SettingsFragment`:**
    *   Displays application settings (`layout/fragment_preferences.xml` is reused here, which might be slightly confusing naming).
    *   Contains a `Switch` (`R.id.switchAllowIndividualRules`) to control whether individual rule enable/disable switches are shown in `PreferencesFragment`.
    *   Saves this setting to `SharedPreferences` (`Util.ALLOW_INDIVIDUAL_RULES`).
*   **`AddRuleDialog`:**
    *   A `DialogFragment` providing UI (`layout/add_rule_dialog.xml`) for inputting rule criteria (text contains) and the target phone number.
    *   Uses `TextInputLayout` and `TextInputEditText` for input fields.
    *   Includes "Save" and "Cancel" buttons.
    *   Communicates the newly created `RuleDef` back to the hosting fragment (`PreferencesFragment`) via the `AddRuleDialogListener` interface.
*   **`SMSReceiver`:**
    *   Triggered by `android.provider.Telephony.SMS_RECEIVED`.
    *   Extracts SMS message body.
    *   Retrieves the list of `RuleDef` from `SharedPreferences` via `Util.getRulesListFromSavedData`.
    *   Iterates through rules:
        *   Checks if the rule is enabled (`RuleDef.isOnOffButton()`).
        *   Checks if the app is globally enabled (`Util.IS_APP_ENABLED`).
        *   Checks if the SMS body contains the required text fragments (`RuleDef.ruleFrags`).
        *   If all conditions met, uses `android.telephony.SmsManager` to send the filtered SMS content to the `RuleDef.targetPhoneNum`.
    *   Includes basic filtering (`filterUnsupportedCharacters`) for potentially problematic characters before sending.
*   **`Util`:**
    *   Provides static methods for saving/retrieving data from `SharedPreferences`.
    *   Uses Jackson `ObjectMapper` for serializing/deserializing the `List<RuleDef>` to/from JSON string for storage in `SharedPreferences`.
    *   Defines constant keys for `SharedPreferences`.
    *   Helper methods for generating rule text/confirmation messages.
*   **`RuleDef`:** Simple data class holding rule fragments (`List<String> ruleFrags`), target number (`String targetPhoneNum`), and enabled state (`Boolean onOffButton`).

### 5.3. UI Elements & Theming Techniques Used

*   **Colors:**
    *   Defined in `values/colors.xml` (light) and `values-night/colors.xml` (dark).
    *   Applied via `DarkThemeApp` style using `@color/primary`, etc.
    *   Referenced in layouts/styles using theme attributes like `?attr/colorPrimary`, `?attr/colorOnBackground`, `?attr/colorSurface`.
*   **Drawables (Vectors):**
    *   **Theme-aware Tinting:** Icons like `ic_brightness_2.xml` use `android:tint="?attr/colorPrimary"`. Others like `baseline_phone_black_48dp.xml` use `android:tint="?attr/colorControlNormal"`. Requires `vectorDrawables.useSupportLibrary = true` in `build.gradle`.
    *   **Hardcoded Colors:** `ic_brightness.xml` has `android:fillColor="#FF000000"`. The README indicates it's tinted via `app:tint` in `fragment_welcome.xml` or programmatically.
    *   **Programmatic Tinting:** Mentioned in README for menu icons (`R.id.action_more` in `MainActivity.java`), likely using `DrawableCompat.setTint` or similar.
    *   **ColorStateList:** `color/color_on_primary_mask.xml` demonstrates applying alpha to a theme color attribute. `drawable/bottom_nav_item_background.xml` uses a selector based on `android:state_checked` to switch between `?attr/colorPrimary` and `?attr/colorOnSurface`.
*   **Layouts:**
    *   `activity_main.xml`: Uses `androidx.constraintlayout.widget.ConstraintLayout`, `Toolbar`, `FrameLayout` (for fragments), `BottomNavigationView`. Theming applied via `DarkThemeApp`.
    *   `fragment_preferences.xml` / `fragment_rules.xml`: Uses `android:forceDarkAllowed="true"` for Smart Dark demo. Relies on standard widgets (`TextView`, `Switch`, `Button`, `ScrollView`, `LinearLayout`, `RelativeLayout`).
    *   `fragment_welcome.xml`: Basic `LinearLayout` with `TextView`, `ImageView`, `Button`. Theming via attributes.
    *   `add_rule_dialog.xml`: Uses Material `TextInputLayout`, `TextInputEditText`, `Button`.
    *   `fragment_rule.xml`: Defines a `MaterialCardView` layout for displaying a single rule, intended for use within `PreferencesFragment`'s list (though current implementation builds views programmatically).
*   **Components:**
    *   `Toolbar`: Styled using `style="@style/Widget.MaterialComponents.Toolbar.Primary"`. Background comes from `?attr/colorPrimary`.
    *   `BottomNavigationView`: Theming controlled via `app:itemBackground="?attr/colorSurface"`, `app:itemIconTint="@drawable/bottom_nav_item_background"`, `app:itemTextColor="?attr/colorOnBackground"`.
    *   `Switch`: Standard switch, appearance controlled by the theme.
    *   `Button`: Standard Material button, background/text color from theme (`colorPrimary`, `colorOnPrimary` usually). Note: Some buttons manually set `android:textColor="#FFFFFF"` (`layout/add_rule_dialog.xml`, `layout/fragment_rules.xml`, `layout/fragment_welcome.xml`), which overrides theme colors and might not look ideal in all theme variations. This should ideally use `?attr/colorOnPrimary` or rely on the default Material Button styling.
    *   `MaterialCardView`: Used in `fragment_rule.xml`, background/elevation controlled by theme.
    *   `TextView`: Text color typically set via `android:textColor="?attr/colorOnBackground"` in the main theme or using specific `textAppearance` attributes.
*   **Text:** Text color generally defaults to `?attr/colorOnBackground` or `?android:attr/textColorPrimary` as defined in the theme (`values/styles.xml`).

### 5.4. Persistence

*   **Theme Preference:** Saved in default `SharedPreferences`. Accessed via `PreferenceManager.getDefaultSharedPreferences`. Key: `"themePref"`. Likely managed by an `androidx.preference.ListPreference` defined in `xml/preferences.xml` associated with a (potentially missing or adapted) `PreferenceFragmentCompat`.
*   **SMS Rules:** Saved as a JSON string in default `SharedPreferences`. Key: `Util.SAVED_RULES`. Managed by `Util` class using Jackson library.
*   **App/Rule States:** Saved as Strings ("true"/"false") in default `SharedPreferences`. Keys: `Util.IS_APP_ENABLED`, `Util.ALLOW_INDIVIDUAL_RULES`. Individual rule states stored within the JSON of `RuleDef` objects.
*   **Splash Screen Flag:** Saved as String ("true"/"false") in default `SharedPreferences`. Key: `Util.IS_SPLASH_SHOWN`.

### 5.5. Permissions

*   `android.permission.RECEIVE_SMS`: Required for `SMSReceiver` to intercept incoming SMS. Declared in `AndroidManifest.xml`.
*   `android.permission.READ_SMS`: Declared in `AndroidManifest.xml`, potentially needed though not explicitly used in the receiver logic shown (might be legacy or for future features).
*   `android.permission.SEND_SMS`: Required to forward SMS messages. Declared in `AndroidManifest.xml`.
*   Runtime Request: `MainActivity` checks and requests `RECEIVE_SMS` and `SEND_SMS` permissions at runtime if not already granted.

## 6. Dependencies (Key)

*   `androidx.appcompat:appcompat`: Core support library, provides `AppCompatActivity`, `AppCompatDelegate`.
*   `com.google.android.material:material`: Material Design Components library, provides `DayNight` themes, `MaterialCardView`, `BottomNavigationView`, `TextInputLayout`, etc.
*   `androidx.constraintlayout:constraintlayout`: For building flexible layouts.
*   `androidx.fragment:fragment-ktx`: For Fragment management.
*   `androidx.preference:preference`: For creating settings screens and managing `SharedPreferences`.
*   `org.jetbrains.kotlin:kotlin-stdlib`: Base Kotlin library.
*   `com.fasterxml.jackson.core:jackson-databind`: For JSON serialization/deserialization of rules.

## 7. Future Considerations / Improvements

*   **Theme Selection UI:** Ensure the Theme selection (`ListPreference` in `xml/preferences.xml`) is actually presented to the user, likely via a dedicated `PreferenceFragmentCompat` screen, perhaps integrated into `SettingsFragment`.
*   **Button Theming:** Remove hardcoded `android:textColor="#FFFFFF"` from buttons and rely on Material Button styles (`style="@style/Widget.MaterialComponents.Button"`) or theme attributes (`?attr/colorOnPrimary`) for better theme adaptability.
*   **Status Bar:** Re-evaluate the `FLAG_FULLSCREEN` and transparent status bar in `MainActivity`. Typically, Material apps allow the theme to color the status bar (`colorPrimaryDark` or a translucent scrim).
*   **Error Handling:** Add more robust error handling, especially in `SMSReceiver` (e.g., invalid numbers, SMS sending failures).
*   **Rule Complexity:** Allow more complex rule definitions (e.g., sender matching, AND/OR conditions).
*   **UI/UX:** Improve the rule display in `PreferencesFragment`, perhaps using `RecyclerView` with `ListAdapter` for better performance and view recycling instead of dynamic inflation into a `LinearLayout`. Use the `layout/fragment_rule.xml` with a proper adapter.
*   **Testing:** Add unit and instrumentation tests.
*   **Smart Dark Granularity:** Explore finer control over Smart Dark using `View#setForceDarkAllowed`.

## 8. Open Questions

*   Is `xml/preferences.xml` intended to be used by a standard `PreferenceFragmentCompat`, and if so, where is that fragment implemented or supposed to be? Currently, `SettingsFragment` manually creates its UI. *Assumption: The `xml/preferences.xml` is likely used implicitly by `DarkThemeApplication` to read the theme pref, assuming it was set elsewhere, or it's vestigial.*
*   The exact line `MainActivity.java#L85` mentioned in the README for programmatic tinting was not present in the provided `MainActivity.java` snippet, but the concept is understood.

