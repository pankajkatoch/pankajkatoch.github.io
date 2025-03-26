## Architectural Overview: SMS Forwarder App

This document provides an architectural overview of the Android "SMS Forwarder" application. The app allows users to define rules for forwarding incoming SMS messages to different phone numbers based on the content of the message. It demonstrates best practices for supporting Dark Mode on Android.

**1\. Core Functionality:**

* **SMS Receiving:**  The app listens for incoming SMS messages using a `BroadcastReceiver` (`SMSReceiver`).  
* **Rule-Based Forwarding:**  Users define rules consisting of text fragments to match within the SMS body and a target phone number.  If an incoming SMS matches a rule's criteria, the message (potentially modified) is forwarded to the specified target number.  
* **Rule Management:** Users can add, edit, delete, and enable/disable forwarding rules. The rules are stored persistently.  
* **App-Level Enable/Disable:**  A global switch controls whether SMS forwarding is active.  
* **Dark Mode Support:**  The application supports dark mode using the Material Design Components library and provides backward compatibility.

**2\. Key Components and Classes:**

* **`MainActivity`:** The main entry point of the application. Handles permission requests (SMS sending and receiving), sets up the toolbar, and manages the display of fragments.  
* **`SMSReceiver`:**  A `BroadcastReceiver` that triggers when an SMS is received. It extracts the message body, iterates through the defined rules, and forwards the message if a rule matches and is enabled (and the app-level forwarding is also enabled).  The message forwarding uses `android.telephony.SmsManager`.  
* **`PreferencesFragment`:**  Displays the list of forwarding rules.  Allows users to add new rules, delete existing rules, and toggle individual rules on/off (if enabled in settings).  This fragment handles the UI for rule management.  
* **`SettingsFragment`:** Provides application-level settings. Currently, it includes a switch to enable/disable individual rule toggling.  
* **`WelcomeFragment`:** Shown on the first launch of the app.  Explains the app's purpose and guides the user to the rules screen.  
* **`AddRuleDialog`:** A `DialogFragment` used to create new forwarding rules. It prompts the user for the rule fragments (text to match) and the target phone number.  
* **`RuleDef`:** A data class representing a single forwarding rule.  It stores the list of text fragments to match, the target phone number, and a boolean indicating whether the rule is enabled.  
* **`Util`:** A utility class providing helper methods for:  
  * Saving and retrieving data using `SharedPreferences`.  This includes storing the forwarding rules, the app-level enable/disable state, and a flag to indicate whether the welcome screen has been shown.  
  * Converting objects to and from JSON strings using the Jackson library (`ObjectMapper`). This is crucial for persisting the list of `RuleDef` objects.  
  * Helper function to format text of the rules  
  * Helper function to generate message for delete confirmation.  
* **`DarkThemeApplication`:**  Extends `Application`.  Its `onCreate` method reads the user's theme preference (from `SharedPreferences`) and applies the selected theme using `ThemeHelper`.  
* **`ThemeHelper`:**  Provides a static method `applyTheme` to set the application's theme (light, dark, or system default/battery saver) using `AppCompatDelegate.setDefaultNightMode()`.

**3\. Data Persistence:**

* **`SharedPreferences`:** Used to store:  
    
  * The list of forwarding rules (`Util.SAVED_RULES`), serialized as a JSON string.  
  * A boolean flag (`Util.IS_APP_ENABLED`) to control whether the app-level forwarding is enabled.  
  * A boolean flag (`Util.IS_SPLASH_SHOWN`) to track whether the welcome screen has been displayed.  
  * A boolean flag (`Util.ALLOW_INDIVIDUAL_RULES`) controlling if each rule can be individually toggled on/off.  
  * Theme preference


* **JSON Serialization (Jackson):** The `Util` class uses the Jackson library (`jackson-databind`, `jackson-core`, `jackson-annotations`) to convert the list of `RuleDef` objects into a JSON string for storage in `SharedPreferences` and to deserialize the JSON string back into a list of objects when the app starts.

**4\. UI Structure:**

* **Fragments:** The app uses fragments for the main UI sections (Welcome, Rules, Settings).  This promotes modularity and reusability.  
* **`activity_main.xml`:**  The main activity layout. Contains a `Toolbar`, a `FrameLayout` to hold the currently displayed fragment, and a `BottomNavigationView` (although it's hidden in the current implementation).  
* **`fragment_rules.xml`:** The layout for the `PreferencesFragment`. Includes:  
  * A `ScrollView` containing a `LinearLayout` (`rulesList`) to display the list of rules.  
  * A switch (`enableDisableApp`) to toggle the app-level forwarding on/off.  
  * A `Button` to add a new rule.  
  * A `LinearLayout` to disable/enable the Scrollview.  
* **`fragment_settings.xml`:** The layout for the `SettingsFragment`. It contains:  
  * Switch to allow enable/disable of individual rules.  
* **`fragment_welcome.xml`:** The layout for the `WelcomeFragment`. Includes an image, introductory text, and a button to proceed to the rules screen.  
* **`add_rule_dialog.xml`:**  The layout for the `AddRuleDialog`.  Contains `TextInputLayout` and `TextInputEditText` views for entering the rule fragments and target phone number, and "Save" and "Cancel" buttons.  
* **`fragment_rule.xml`:**  Intended to show single rule, but currently unused.

**5\. Dark Mode Implementation:**

* **`Theme.MaterialComponents.DayNight.NoActionBar`:**  The base theme for the application (`DarkThemeApp` style in `styles.xml`). This theme provides built-in support for dark mode.  
* **Resource Qualifiers:** The app uses resource qualifiers (`values-night`) to provide alternative resource values for dark mode:  
  * `values-night/colors.xml`: Defines different color values for the primary, secondary, and error colors in dark mode.  
* **Theme Attributes:** The app uses theme attributes (e.g., `?attr/colorPrimary`, `?attr/colorOnBackground`) to refer to colors defined in the current theme.  This allows the colors to automatically switch between light and dark mode.  
* **`ThemeHelper`:**  Provides the `applyTheme` method to programmatically set the theme based on user preferences.  
* **Vector Drawables:**  Uses `android:tint` with theme attributes in vector drawables (e.g., `ic_brightness_2.xml`) to automatically adjust the icon colors for dark mode.

**6\. Build System and Dependencies:**

* **Gradle:** The project uses the Gradle build system.  
* **Dependencies:**  
  * **Kotlin:**  The app is written in Kotlin.  
  * **AndroidX Libraries:**  AppCompat, ConstraintLayout, RecyclerView, Core, Lifecycle, Navigation, Preference.  
  * **Material Components:**  Provides Material Design UI components and theming support.  
  * **Glide:** (Included but not actively used in the provided code) An image loading and caching library.  
  * **Jackson:** For JSON serialization/deserialization.

**7\. Permissions:**

* **`android.permission.RECEIVE_SMS`:** Required to receive SMS messages.  
* **`android.permission.READ_SMS`:** Required to read the content of SMS messages.  
* **`android.permission.SEND_SMS`:** Required to forward SMS messages.  
* The `MainActivity` checks and requests permission during `onCreate`.

**8\. Flow of Execution (SMS Forwarding):**

1. An SMS message is received by the device.  
2. The `SMSReceiver`'s `onReceive` method is triggered.  
3. The message body is extracted from the received SMS.  
4. The `SMSReceiver` retrieves the list of forwarding rules from `SharedPreferences` (via `Util`).  
5. It iterates through each rule:  
   * Checks if the app-level forwarding is enabled (`Util.IS_APP_ENABLED`).  
   * Checks if the individual rule is enabled (`rule.isOnOffButton()`).  
   * If both are enabled, it checks if the message body contains *all* of the rule's text fragments.  
   * If the message matches the rule, the `sendSms` method is called to forward the message to the rule's target phone number.  
6. The `sendSms` method uses `android.telephony.SmsManager` to send the SMS.

**9\. Areas for Improvement/Potential Enhancements:**

* **UI/UX Improvements:**  
  * Improve display and formatting of the rule list.  
  * Add feedback to the user when a message is forwarded (e.g., a Toast message or notification).  
  * Consider using a `RecyclerView` for better performance with a large number of rules.  
  * Implement better error handling and user feedback for permission issues.  
* **Rule Editing:**  Currently, the app only supports adding and deleting rules.  Adding an edit functionality would improve usability.  
* **More Complex Rule Logic:**  The current rule matching is simple (checking for the presence of text fragments).  More complex rules could be supported, such as:  
  * Regular expression matching.  
  * Matching based on the sender's phone number.  
  * Combining multiple conditions (AND/OR).  
* **Message Customisation:**  Allow the user to add custom text to the message before forwarding.  
* **Testing:**  Add unit and UI tests to ensure the app's functionality and stability.

This overview provides a comprehensive understanding of the SMS Forwarder application's architecture, components, and functionality.  It highlights the key design decisions and provides a starting point for further development and improvements.
