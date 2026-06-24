# Contribution 1: [Feature] Make the top bar of the desktop client black in darkmode (Windows)

**Contribution Number:** 1  
**Student:** Danny Abraham  
**Issue:** [#501](https://github.com/session-foundation/session-desktop/issues/501)  
**Status:** Phase IV — Pull Request Close to Submission (Double Checking)  
**Fork branch:** [AbrahamDann/session-desktop @ feat/dark-titlebar-windows-501](https://github.com/AbrahamDann/session-desktop/tree/feat/dark-titlebar-windows-501)  
**Pull Request:** _[paste your PR URL here once opened against session-foundation/session-desktop]_

---

## Why I Chose This Issue

I chose issue #501, "Make the top bar of the desktop client black in darkmode (Windows)," because it let me work directly with desktop application integration using TypeScript and Electron's main-process APIs. The issue was re-marked as a "good first issue" after a major Electron upgrade, signalling that the earlier architectural blockers were cleared and the maintainers were open to a contributor resolving it.

It aligned well with my TypeScript/JavaScript goals while offering something a normal web-app layout task wouldn't: reconciling an Electron app's *own* theme with the host operating system's native window chrome on Windows.

---

## Understanding the Issue

### Problem Description
On Windows, when Session uses a dark theme, the native OS title bar (window chrome) stays white/light. It only follows the *operating system's* theme, so a user running Windows in light mode sees a bright title bar clashing with Session's dark UI.

### Expected Behavior
When a dark Session theme is active, the native Windows title bar should be dark to match — and switch back to light for light themes.

### Current Behavior
The in-app UI themes correctly, but the native title bar ignores the in-app theme and tracks the OS theme instead.

### Affected Components (actual)
* `ts/mains/main_node.ts` — Electron main process; creates the `BrowserWindow` and already handles `nativeTheme`.
* `ts/themes/switchTheme.tsx` — renderer-side theme switching logic.

---

## Reproduction Process

### Environment Setup
Built locally on **Windows 11** with the project's required toolchain:
- Node.js **24.12.0** (via nvm-windows), **pnpm 10.28.1**, CMake, and Visual Studio 2022 C++ Build Tools.

This was *not* a smooth `pnpm install`. The native module `libsession_util_nodejs` failed to compile because of the Windows 260-character `MAX_PATH` limit — MSBuild's FileTracker threw `FTK1011` on deeply nested build paths. I resolved it by:
1. Enabling Git + Windows long paths (`core.longpaths`, `LongPathsEnabled`),
2. Disabling MSBuild's FileTracker (`TrackFileAccess=false`), and
3. Relocating pnpm's package store to a short path (`npm_config_virtual_store_dir=C:\pn`).

### Steps to Reproduce
1. Build and launch the app on Windows: `pnpm build` then `pnpm start-prod`.
2. Set **Windows itself to Light mode** (Settings → Personalization → Colors).
3. In Session, open Appearance settings and select a **dark theme** (e.g. Classic Dark).
4. **Observed Result:** the app UI is dark, but the native title bar stays light/white.

### Reproduction Evidence
- _[Insert screenshot: dark Session theme with a white Windows title bar]_

### Root-Cause Findings
Tracing the main process, the app **reads** `nativeTheme.shouldUseDarkColors` (the OS theme) and forwards it to renderers, but it **never sets** `nativeTheme.themeSource`. That property is exactly what tells Windows whether to paint the native frame dark or light. With it unset, Windows defaults to the OS theme regardless of the in-app selection.

---

## Solution Approach

### Analysis
The minimal, idiomatic fix is to set `nativeTheme.themeSource` to `'dark'` or `'light'` based on the selected Session theme. I deliberately avoided `titleBarStyle: 'hidden'` + `titleBarOverlay`, because that replaces the OS title bar with a custom-drawn one — a much larger, cross-platform-risky change. Setting `themeSource` keeps the native title bar and matches the file's existing `nativeTheme` usage.

### Implementation (UMPIRE)

**Understand:** Make the native Windows title bar follow the active Session theme, not the OS theme.

**Match:** Followed the existing `nativeTheme` and `ipcMain`/`ipcRenderer` patterns already in `main_node.ts` (e.g. the `get-native-theme` / `native-theme-update` handlers).

**Plan:**
1. Add a helper in the main process mapping a theme name to `nativeTheme.themeSource` (`*-dark` → `'dark'`, else `'light'`).
2. Call it on startup from the saved theme (`getThemeFromDb()`), before the `BrowserWindow` is created, so the title bar is correct on first paint.
3. Add a `set-native-theme` IPC channel so the renderer can update it live.
4. Send that IPC from `switchThemeTo` whenever the theme changes.

**Implement:** Commit `103b61320` on `feat/dark-titlebar-windows-501`. Changes (21 insertions across 2 files):

*Main process — `ts/mains/main_node.ts`:*
```ts
function setNativeThemeSource(theme: string) {
  // 'dark'/'light' tells the OS which colors to use for the native window
  // frame and title bar. Our theme names end with -dark or -light.
  nativeTheme.themeSource = theme.endsWith('-dark') ? 'dark' : 'light';
}

ipc.on('set-native-theme', (_event, theme: string) => {
  setNativeThemeSource(theme);
});
```
…and at window creation: `setNativeThemeSource(getThemeFromDb());` before `new BrowserWindow(...)`.

*Renderer — `ts/themes/switchTheme.tsx`:*
```ts
import { ipcRenderer } from 'electron';
// inside switchThemeTo, when a new theme is applied:
ipcRenderer.send('set-native-theme', newTheme);
```

**Review:** Naming and IPC style match existing handlers; change is isolated to theme/window initialization and touches no unrelated logic.


## Reflection / Next Steps
- Possible follow-up: apply the same `themeSource` handling to the password prompt window so it matches on launch when a password is set.
- Awaiting maintainer review; will respond to feedback in Phase IV iterations.
