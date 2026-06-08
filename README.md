# Contribution 1: [Feature] Make the top bar of the desktop client black in darkmode (Windows)

**Contribution Number:** 1  
**Student:** Danny Abraham  
**Issue:** [https://github.com/session-foundation/session-desktop/issues/501](https://github.com/session-foundation/session-desktop/issues/501)  
**Status:** Phase II In Progress

---

## Why I Chose This Issue

I chose issue #501, "Make the top bar of the desktop client black in darkmode (Windows)," because it allows me to work directly with desktop application integration using TypeScript and Electron configuration patterns. The issue was recently re-marked as a "good first issue" following a major Electron dependency update, indicating that the maintainers are active, the previous architectural blocks are cleared, and they are seeking a contributor to resolve it. 

This issue aligns closely with my frontend and JavaScript/TypeScript goals. It provides a unique engineering challenge: managing cross-platform operating system behaviors and native window frames rather than standard web-app layout code. By implementing this feature, I hope to master Electron's theme state management, window customization APIs (such as `titleBarOverlay` or `titleBarStyle`), and clean codebase navigation.

---

## Understanding the Issue

### Problem Description
On Windows operating systems, when the Session application is switched into Dark Mode, the native OS top window bar (title frame/window chrome) remains white or a light default system color. This creates a harsh visual contrast against the app's clean, dark theme.

### Expected Behavior
When the desktop application is running in dark mode on a Windows machine, the title bar/top window frame should automatically change its background color to black or an ultra-dark grey to match the internal visual theme.

### Current Behavior
The internal layout of the app changes themes correctly, but the top frame/native title bar on Windows stays light-colored, disregarding the dark mode setting.

### Affected Components
* `src/main.ts` (or the core file initialization script handling `new BrowserWindow()`)
* Native Window configuration and theme initialization modules.
* Settings/Theme listeners tracking dark mode changes.

---

## Reproduction Process

### Environment Setup
I set up the repository locally running Node.js and npm according to the instructions in the project root. Node dependencies installed smoothly via `npm install`.

**Working branch:** [https://github.com/DannyAbraham/session-desktop/tree/fix-issue-501](https://github.com/DannyAbraham/session-desktop/tree/fix-issue-501)

### Steps to Reproduce
1. Run the application locally in a Windows environment using `npm start`.
2. Open the application settings dashboard.
3. Toggle the internal user interface theme to "Dark Mode".
4. **Observed Result:** The app UI darkens completely, but the top operating system title window frame remains highly contrasted bright white.

### Reproduction Evidence
- **Commit showing reproduction:** [Placeholder: Link to your branch commit once environment runs]
- **My findings:** Traced the issue to the Main Process configuration file where the Electron app initializes its primary window context. The settings lack explicit instructions on how to handle the `titleBarOverlay` colors when drawing window boundaries on Win32 systems.

---

## Solution Approach

### Analysis
The root cause is that the `BrowserWindow` instance does not explicitly map theme preferences to Windows’ native title bar overlays. Since Electron was recently upgraded in this project, we can now use the modern `titleBarOverlay` properties or standard background color definitions within the initialization constructor to update native borders seamlessly.

### Proposed Solution
Modify the window instantiation process to listen to theme configuration events, passing dynamic color preferences directly to Electron's title bar controller logic on Windows instances.

### Implementation Plan

Using UMPIRE framework:

**Understand:** Adjust the native app frame bar on Windows instances to match the user's active application theme programmatically.

**Match:** Look at how configuration variables or user settings are imported during the bootstrap initialization lifecycle in `src/main.ts`.

**Plan:**
1. Locate the core `BrowserWindow` instantiation inside the application main process directory.
2. In the configuration block, configure `titleBarStyle: 'hidden'` or implement `titleBarOverlay` settings specifically for Windows platforms.
3. Hook a theme changes event listener to trigger color adaptations dynamically (`titleBarOverlay.color = '#000000'`) when the application dark mode state updates.
4. Run cross-platform checks to verify that modifications do not interfere with macOS or Linux window configurations.

**Implement:** [Link to branch commits will go here during Phase III]

**Review:** Ensure that naming parameters match the established project standard and that no architectural linter exceptions are thrown.

**Evaluate:** Manually verify changing themes on a Windows machine updates the title bar colors instantly, and ensure existing automated build tests pass.

---
