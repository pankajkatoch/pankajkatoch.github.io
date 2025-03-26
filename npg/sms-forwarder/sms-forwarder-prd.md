# PRD: SMS Forwarder Android App

**Author:** [Your Name/Senior PM Name]
**Version:** 1.0
**Status:** Draft
**Date:** 2023-10-27

---

## 1. Introduction

### 1.1. Overview
This document outlines the product requirements for the **SMS Forwarder**, an Android application designed to automatically forward incoming SMS messages to a specified phone number based on user-defined rules. The core value proposition is simplicity, reliability, and user privacy – the app operates entirely offline (no network permissions required) and contains no advertisements or user tracking.

### 1.2. Goals
*   **Primary:** Provide users with a simple and reliable mechanism to forward specific SMS messages (e.g., OTPs, alerts) from their device to another phone number.
*   **Secondary:** Ensure user privacy and security by performing all operations locally on the device without requiring network access.
*   **Tertiary:** Offer a clean, intuitive user interface for managing forwarding rules.
*   **Minor:** Provide basic visual customization through Light and Dark theme options.

### 1.3. Context
This PRD is based on an analysis of the existing codebase (provided documents) which includes core forwarding logic (`SMSReceiver`, `RuleDef`, `Util`), rule management UI (`PreferencesFragment`, `AddRuleDialog`, related XML layouts), and a foundational implementation of Dark Theme switching (`ThemeHelper`, `DarkThemeApplication`, styles/colors).

## 2. Target Audience

*   **Primary Persona: The Multi-Device User:** Individuals who use multiple phones (e.g., personal and work, primary and secondary) and need to receive critical SMS messages (like OTPs or bank alerts) sent to one device while primarily using another.
*   **Secondary Persona: The Privacy-Conscious User:** Users who require SMS forwarding functionality but are wary of apps that demand excessive permissions (especially network access) or contain ads/tracking.
*   **Tertiary Persona: The Automation Enthusiast:** Users looking for simple, local automation tools for specific tasks like SMS handling.

## 3. Requirements

### 3.1. Functional Requirements - Core SMS Forwarding

| ID    | Requirement                         | Description                                                                                                                                                                                              | Priority | Notes                                                                                                  |
| :---- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | :----------------------------------------------------------------------------------------------------- |
| F-001 | **SMS Permission Request**          | On first launch or when necessary, the app must request `RECEIVE_SMS` and `SEND_SMS` permissions from the user. Operation is blocked without these permissions.                                              | Must     | Implemented in `MainActivity`.                                                                         |
| F-002 | **SMS Reception**                   | The app must listen for incoming SMS messages using a `BroadcastReceiver`.                                                                                                                               | Must     | Implemented via `SMSReceiver` triggered by `android.provider.Telephony.SMS_RECEIVED`.                    |
| F-003 | **Rule Definition**                 | Users must be able to define forwarding rules. Each rule consists of: <br> 1. **Pattern:** A text string. Incoming SMS body must contain this string (case-insensitive). <br> 2. **Target Phone Number:** The destination phone number for forwarding. | Must     | Based on `RuleDef`, `AddRuleDialog`. Pattern supports '*' as a potential wildcard/separator.             |
| F-004 | **Rule Matching Logic**             | When an SMS is received, the app iterates through enabled rules. If the SMS body (case-insensitive) contains the pattern defined in an *enabled* rule, it's considered a match.                             | Must     | Implemented in `SMSReceiver`. `String.contains()` logic. Code filters non-GSM chars from SMS body. |
| F-005 | **SMS Forwarding Action**           | If a matching, enabled rule is found AND the app's global forwarding is enabled, the app must forward the original SMS message body (potentially prefixed, e.g., "OTP: ") to the rule's Target Phone Number using the device's native SMS sending capability. | Must     | Implemented in `SMSReceiver` using `android.telephony.SmsManager`. Prefix observed in code.         |
| F-006 | **Rule Persistence**                | Defined rules must be saved locally on the device and persist across app restarts.                                                                                                                       | Must     | Implemented via `SharedPreferences` and Jackson JSON serialization in `Util.java`.                     |
| F-007 | **Global Forwarding Toggle**        | Provide a master switch to enable or disable all SMS forwarding functionality globally.                                                                                                                  | Must     | Implemented in `PreferencesFragment` (`enableDisableApp` switch), checked in `SMSReceiver`.         |
| F-008 | **Welcome Screen**                  | On the very first launch, display a welcome screen explaining the app's purpose, privacy features (no network, no ads), and prompt the user to proceed.                                                 | Must     | Implemented in `WelcomeFragment`, controlled by `IS_SPLASH_SHOWN` flag in `Util`.                    |

### 3.2. Functional Requirements - Rule Management UI

| ID    | Requirement                         | Description                                                                                                                                                                                              | Priority | Notes                                                                                                           |
| :---- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- | :-------------------------------------------------------------------------------------------------------------- |
| F-101 | **Display Rule List**               | Display all saved rules in a clear list format, showing the pattern and target phone number for each rule.                                                                                                 | Must     | Implemented in `PreferencesFragment`, dynamically populating a `LinearLayout`. Header row included.        |
| F-102 | **Add Rule UI**                     | Provide an "Add New Rule" button that launches a dialog (`AddRuleDialog`) for entering the pattern and target phone number. Input validation should ensure fields are not empty before saving.                 | Must     | Implemented via `AddRuleDialog` and its interaction with `PreferencesFragment`.                               |
| F-103 | **Delete Rule UI**                  | Provide a mechanism (e.g., a delete icon) next to each rule to allow users to remove it. A confirmation dialog must be shown before deletion.                                                               | Must     | Implemented via delete icon added programmatically in `PreferencesFragment`, showing `AlertDialog`.          |
| F-104 | **Individual Rule Toggle (Optional)** | *If* the corresponding setting is enabled, display an enable/disable switch next to each rule in the list. Toggling this switch should enable/disable that specific rule. State must be persisted.          | Should   | Implemented in `PreferencesFragment`, visibility/functionality tied to `ALLOW_INDIVIDUAL_RULES` setting. |
| F-105 | **Visual Indication for Disabled State** | When global forwarding is disabled via the master switch (F-007), the rule list UI should be visually distinct (e.g., grayed out, overlaid) to indicate that rules are inactive and cannot be interacted with. | Must     | Implemented via `rulesListScrollerWrapperDisabler` overlay in `fragment_rules.xml`.                           |

### 3.3. Functional Requirements - Settings

| ID    | Requirement                         | Description                                                                                                                                                                                             | Priority | Notes                                                                                                                                         |
| :---- | :---------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| F-201 | **Dark Theme Setting**              | Provide options for the user to select the app's theme: Light, Dark, System Default (Android Q+) / Set by Battery Saver (Pre-Q). The choice must be persisted.                                          | Should   | Core logic in `ThemeHelper`, `DarkThemeApplication`. UI for selection seems present in `preferences.xml` & arrays, but may need wiring up in `SettingsFragment`. |
| F-202 | **Apply Theme**                     | The selected theme must be applied across the application consistently and immediately upon selection (or app restart if necessary).                                                                      | Should   | Implemented via `AppCompatDelegate.setDefaultNightMode`.                                                                                   |
| F-203 | **Individual Rule Toggle Setting** | Provide a setting (e.g., a switch in a dedicated Settings screen) to enable/disable the display and functionality of individual rule toggles (F-104). Persist the choice.                                | Should   | Implemented in `SettingsFragment.java` controlling `ALLOW_INDIVIDUAL_RULES` flag.                                                         |

### 3.4. Non-Functional Requirements

| ID    | Requirement        | Description                                                                                                                            | Priority | Notes                                                              |
| :---- | :----------------- | :------------------------------------------------------------------------------------------------------------------------------------- | :------- | :----------------------------------------------------------------- |
| NF-01 | **Performance**    | Rule matching on incoming SMS should be performant and not introduce noticeable delay or battery drain.                                  | Must     | Current logic seems simple (String.contains).                      |
| NF-02 | **Reliability**    | SMS forwarding should be reliable, provided permissions are granted and device SMS functionality is operational.                       | Must     | Depends on OS SMS handling.                                      |
| NF-03 | **Privacy**        | The application MUST NOT use network permissions. All data (rules, SMS content during processing) must remain local to the device.     | Must     | Confirmed via `AndroidManifest.xml` and Welcome screen claims. |
| NF-04 | **Security**       | Rules (including target phone numbers) are stored locally. No specific encryption mentioned in code, relies on device security.        | Should   | Data stored in app-private `SharedPreferences`.                  |
| NF-05 | **Usability**      | The UI for managing rules should be simple, clear, and intuitive for the target audience.                                              | Must     | Strive for minimal steps to add/manage rules.                    |
| NF-06 | **Compatibility** | Support Android API Level 21+ as defined in `build.gradle`. Ensure compatibility with DayNight theme features across supported versions. | Must     | `minSdkVersion 21`. Theme uses AppCompat for compatibility.    |

## 4. Design & UI/UX Considerations

*   **Style:** Adhere to Material Design guidelines for components, layout, typography, and interaction patterns.
*   **Simplicity:** Prioritize a clean and uncluttered interface. Avoid unnecessary visual elements or complex navigation.
*   **Clarity:** Rule display must clearly show the pattern and target number. Settings options should be unambiguous. Visual feedback for disabled states (global toggle) is crucial.
*   **Dark Theme:** Ensure proper contrast ratios and readability in both Light and Dark themes. Utilize theme attributes (`?attr/...`) where possible for colors and drawables (`?attr/colorPrimary`, `?attr/colorOnSurface`, etc.). Use `-night` resource qualifiers for specific overrides where necessary.
*   **Layouts:** Utilize layouts provided (`fragment_rules.xml`, `add_rule_dialog.xml`, `fragment_welcome.xml`, `fragment_preferences.xml`) as a base. Ensure responsiveness on typical phone screen sizes.
*   **(Placeholder)** Link to Mockups/Wireframes: [Link to Figma/Sketch/Zeplin TBD]

## 5. Release Criteria

*   All "Must" priority functional requirements (F-001 to F-008, F-101 to F-103, F-105) are implemented and pass QA testing.
*   "Should" priority functional requirements (F-104, F-201 to F-203) are implemented and pass QA testing.
*   All non-functional requirements (NF-01 to NF-06) are met, with specific focus on Privacy (NF-03) verification.
*   No critical or major bugs identified in core forwarding logic or rule management UI.
*   UI is consistent with Material Design guidelines and works correctly in both Light and Dark themes.
*   App functions correctly on target Android versions (API 21+).
*   Privacy policy implications reviewed (even for offline apps, transparency is good).

## 6. Future Considerations / Open Issues

*   **Advanced Rule Conditions:** Explore options beyond simple `contains`:
    *   Matching based on Sender Number.
    *   Regular Expression (Regex) pattern matching.
    *   Combining multiple conditions (AND/OR logic).
*   **Rule Import/Export:** Allow users to backup/restore or share their rule sets.
*   **Improved Wildcard Handling:** Define and implement more robust wildcard support in patterns beyond simple '*'.
*   **Forwarding History/Log:** Optionally log forwarded messages within the app for user review (requires careful consideration of privacy).
*   **UI for Theme Selection:** Confirm implementation status and finalize the UI element (likely a `ListPreference` within `SettingsFragment`) for selecting the theme.
*   **Error Handling:** Improve resilience (e.g., notification if SMS sending fails repeatedly).
*   **Accessibility:** Ensure UI elements are properly tagged for screen readers and meet accessibility standards.

## 7. Appendix

*   **Source Code:** [Link to relevant source code repository/directory if applicable]
*   **Design Mockups:** [Link to design files TBD]

---
