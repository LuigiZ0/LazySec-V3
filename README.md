What's New?

* State Management: The extension now knows what changes it made and can seamlessly revert them.
* Persistence: Changes can be persisted when navigating between pages.
* Modern Chrome APIs: Uses Service Worker, `chrome.scripting`, and `chrome.storage`.
* Dynamic Interface: The popup reacts to user actions, changing the appearance and text of the buttons.
* Internationalization (i18n): The text has been prepared for multiple languages.
* Improved UX: Instead of alerts that lock the screen, it uses discreet "toast" notifications.

Change Overview: The old version consisted of simple, isolated scripts, with duplicated code and basic functionality. The new version is a cohesive, modern application.

### File Structure

Added Files:
* `background.js`
* `lazysec-script.js`
* `popup.css`

Removed Files:
* `unblock.js`
* `unveil.js`

---

### File-by-File Analysis

#### 1. manifest.json

The manifest has been completely modernized.
* Architecture: Changed from background scripts to a `service_worker` (`background.js`), which is the modern standard.
* Permissions: Added `scripting` and `storage` to support the new script injection and state/persistence storage features.
* Internationalization: The `name`, `description`, and `default_title` fields now use `__MSG_key__` to allow translations. Added `"default_locale": "en"`.

#### 2. Content Scripts (unblock.js, unveil.js vs. lazysec-script.js)

The two old, buggy scripts have been replaced with a single, robust script.
* Consolidation: The "unveil" and "unblock" logic, which was in two separate files (and contained a lot of duplicate code), has been unified in the new `lazysec-script.js`.
* State Management:
    * Old: Did not save the original state of elements. Reverting was not possible.
    * New: Creates a `window.lazySecState` object that saves the original state of each modified element. This allows for a `revertAllChanges` function that seamlessly restores the original attributes and styles.
* Element Selection:
    * Old: The selection logic was inefficient, had duplicate code, and contained invalid selectors (e.g., `querySelectorAll('input[type=td]')`).
    * New: The `getElementsToProcess` function is much cleaner, more accurate, and more comprehensive, including `textarea` selection, `select`, `p`, `h1`, `a`, etc., and uses a `Set` to avoid duplicates.
* Notifications:
    * Old: Used the `alert()` function, which interrupts user navigation.
    * New: Implements a `showToast()` function that displays elegant, non-blocking notifications in the corner of the screen. Additionally, messages are fetched from `background.js` to support pluralization ("1 element revealed" vs. "5 elements revealed").

#### 3. popup.html

The popup interface has been completely rebuilt.
* Structure and Style:
    * Old: Minimal HTML, no custom styling.
    * New: Uses divs to organize options, adds a separator line, and implements a custom checkbox design (`<span class="checkmark"></span>`). The interface is linked to the new `popup.css`.
* New Feature (Persistence): A new "Keep changes between pages" checkbox (`persist_changes`) has been added.
* Dynamic Buttons: Added an `updateBothButton` button (initially hidden) for the new update logic. The buttons are now empty, as their text is dynamically set by `popup.js`.
* Internationalization: All visible elements now have a `data-i18n-key` attribute for localization.

#### 4. popup.css (New File)

This file didn't exist. It was created from scratch to give the extension a modern and professional look, with a dark theme, custom fonts (Fira Code), and neon colors for the buttons and highlights, including `:hover` states and `.active` classes to indicate which actions are active.

#### 5. popup.js

This script is the brains of the new interface and has been completely rewritten.
* State Logic:
    * Old: Just sent a command and forgot about it.
    * New: Before doing anything, it first checks the current page state (`pageState`) to see what has already been done. This allows the UI to be rendered intelligently.
* Dynamic UI: The `renderButtons` function changes the text and style of buttons based on the `pageState`. For example, after clicking "Reveal Elements," the button displays "Elements Revealed :)" and changes color. If the user changes the selection options (checkboxes) after an action, the button changes to "Update Selection."
* Persistence Logic: Interacts with `chrome.storage.session` to save actions and options for a specific tab if persistence is enabled.
* Communication and Execution: Uses the modern `chrome.scripting.executeScript` API to inject `lazysec-script.js`.
* Improved Rollback: Clicking "Undo Changes" not only performs the rollback, but also removes the tab's persistence data and unchecks the persistence checkbox in the UI.

#### 6. background.js (New File)

This file is the backbone of the new background functionality.
* Persistence Logic: Contains the `chrome.tabs.onUpdated` listener. When the user navigates to a new page in the same tab, this script checks for saved settings for that tab in `chrome.storage.session`. If so, it automatically reapplies the changes, ensuring persistence.
* Translation Center: Contains a `chrome.runtime.onMessage` listener that acts as a hub for providing the correct translations for `lazysec-script.js`, allowing toast notifications to appear in the correct language and with proper pluralization.
