# First-run and recovery

## 47. Show a friendly view when a widget breaks

**Link:** https://api.flutter.dev/flutter/widgets/ErrorWidget-class.html

**Why:** when a widget fails to build, release users see an empty grey box. Show a short "Something went wrong" in the app's colors instead.

**Detect:** the app does not set `ErrorWidget.builder`. This is the framework build-error widget, not a failed HTTP load (rule 2).

**Hunt:** grep `ErrorWidget.builder`. Handled if set. Match if missing.

**Fix:** set `ErrorWidget.builder` on `MaterialApp` / `WidgetsApp` to a small branded "Something went wrong" widget. Do not use this for HTTP failures (rule 2).

---

## 48. Show loading progress while Flutter web boots

**Link:** https://docs.flutter.dev/platform-integration/web/initialization

**Why:** Flutter web takes a few seconds to boot. A blank white tab looks broken. Show a splash or progress so the user knows the app is coming.

**Detect:** `web/index.html` exists and its `<body>` has no visible splash/logo/progress while Flutter loads. No `web/` folder is N/A.

**Hunt:** check `web/index.html`. N/A if it does not exist. Match if `<body>` is only script tags / `flutter_bootstrap.js` with no visible markup.

**Fix:** in `web/index.html`, put a visible splash/logo/progress in `<body>` before the Flutter bootstrap script. Match the app's background color so the handoff does not flash.
