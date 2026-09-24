# Platform

## 39. Keep iOS swipe-back when routes are Material

**Link:** https://api.flutter.dev/flutter/material/PageTransitionsTheme-class.html

**Why:** a custom `PageRouteBuilder` or a `pageTransitionsTheme` that uses zoom/fade on every platform kills the iOS edge swipe. The user feels trapped.

**Detect:** `ios/` exists and routes on iOS do not use `CupertinoPageTransitionsBuilder` / `CupertinoPageRoute` (or an equivalent that still supports the back swipe). Default `MaterialApp` with no transition override is handled — Flutter already uses Cupertino on iOS. N/A if there is no `ios/` folder.

**Hunt:** grep `pageTransitionsTheme|PageRouteBuilder|CupertinoPageTransitionsBuilder|CupertinoPageRoute`. N/A if no `ios/`. Handled if iOS still gets `CupertinoPageTransitionsBuilder` (or `CupertinoPageRoute`) and the back swipe works. Match a theme/builder that applies zoom/fade/slide-up to iOS, or a `PageRouteBuilder` with no iOS back gesture.

**Fix:** set `pageTransitionsTheme` so `TargetPlatform.iOS` uses `CupertinoPageTransitionsBuilder`, or push `CupertinoPageRoute` on iOS. Do not use one `PageRouteBuilder` for every platform unless it restores the iOS back swipe.

---

## 40. Keep Android system back and predictive back

**Link:** https://api.flutter.dev/flutter/material/PredictiveBackPageTransitionsBuilder-class.html

**Why:** Android users hit the system back button or gesture. A route, sheet, or `WillPopScope` that swallows it traps them. On Android 14+ the previous screen should peek during the gesture (predictive back).

**Detect:** `android/` exists and any of these: (1) `WillPopScope` is still in the tree, or `PopScope(canPop: false)` with no `onPopInvokedWithResult` that eventually pops; (2) a modal/sheet uses `isDismissible: false` with no other close that pops; (3) Android page transitions were customized (`pageTransitionsTheme` / `PageRouteBuilder`) and `PredictiveBackPageTransitionsBuilder` is missing. Default `MaterialApp` with no transition override and no back-swallow is handled. Dirty-form intercepts are rule 33 — handled if they use `PopScope`. N/A if there is no `android/` folder.

**Hunt:** grep `WillPopScope|PopScope|isDismissible:\s*false|pageTransitionsTheme|PageRouteBuilder|PredictiveBackPageTransitionsBuilder|enableOnBackInvokedCallback`. N/A if no `android/`. Handled if there is no `WillPopScope`, routes/sheets pop on back, and any custom Android transition uses `PredictiveBackPageTransitionsBuilder`. Match a back-swallowing `WillPopScope` / `PopScope`, a non-dismissible sheet with no close, or a custom Android transition that is not predictive-back.

**Fix:** replace `WillPopScope` with `PopScope`. Let routes and sheets pop on system back (`isDismissible: true`, unless a close control also pops). If `pageTransitionsTheme` is customized, set `TargetPlatform.android` to `PredictiveBackPageTransitionsBuilder`. On `<application>` in `AndroidManifest.xml`, set `android:enableOnBackInvokedCallback="true"`. Dirty forms stay on rule 33.

---

## 41. Use start/end insets when the app supports RTL

**Link:** https://docs.flutter.dev/ui/internationalization

**Why:** `EdgeInsets.only(left: 16)` and `Alignment.centerLeft` put padding and icons on the wrong side in Arabic, Hebrew, and other RTL locales.

**Detect:** the app ships an RTL locale (`ar`, `he`, `fa`, `ur`, or equivalent) or sets `textDirection`, and user-visible layout still uses physical `left`/`right` instead of `start`/`end`. N/A if there is no RTL locale. Brand marks that must stay on a physical side are N/A.

**Hunt:** grep l10n/locale for `ar|he|fa|ur`. N/A if none. Then grep `EdgeInsets.only(left|right)` / `EdgeInsets.fromLTRB` / `Alignment.centerLeft|centerRight` / `Positioned(left|right` in UI. Handled if those screens use `EdgeInsetsDirectional` / `AlignmentDirectional` / `start`/`end`. Match user-visible padding/alignment that is still left/right.

**Fix:** `EdgeInsetsDirectional.only(start: …, end: …)`, `AlignmentDirectional`, and `Positioned.directional`. Keep physical left/right only for something that must not flip.

---

## 42. Don't hard-code light colors when a dark theme exists

**Link:** https://docs.flutter.dev/cookbook/design/themes

**Why:** `Colors.white` scaffolds and `Colors.black` text ignore `darkTheme`. Dark mode looks broken.

**Detect:** the app has `darkTheme` / `ThemeMode.system` / `ThemeMode.dark`, and widgets paint backgrounds or text with `Colors.white`, `Colors.black`, or a raw `Color(0xFF…)` instead of `Theme.of(context).colorScheme`. A brand accent hex in the theme file is N/A. N/A if brightness is locked to light and there is no dark theme. Debug-only colors are N/A.

**Hunt:** grep `darkTheme|ThemeMode.system|ThemeMode.dark`. N/A if none. Then grep `Colors.white|Colors.black|Color(0x` in widget files (not the theme definition). Match background/text that should follow `colorScheme`. Do not flag every hex in `ThemeData`.

**Fix:** `Theme.of(context).colorScheme` (`surface`, `onSurface`, `primary`, …) or the project's theme tokens. Leave brand accents that are meant to stay the same in both modes.

---

## 43. Match status-bar icons to the background

**Link:** https://api.flutter.dev/flutter/services/SystemUiOverlayStyle-class.html

**Why:** light status-bar icons on a light background (or the reverse) are unreadable next to the clock and battery.

**Detect:** a page whose status-bar band clashes with icon brightness: no `AppBar.systemOverlayStyle`, no `AnnotatedRegion<SystemUiOverlayStyle>`, no `SystemChrome.setSystemUIOverlayStyle`. A Material `AppBar` whose `foregroundColor` / theme already drives overlay style is handled. Match `extendBodyBehindAppBar` or no-app-bar pages where icons disappear into the background.

**Hunt:** grep `SystemUiOverlayStyle|systemOverlayStyle|AnnotatedRegion`. Handled if overlay style follows page brightness (dark icons on light, light icons on dark). Match the first clashing no-app-bar / behind-app-bar page.

**Fix:** `AnnotatedRegion<SystemUiOverlayStyle>` or `AppBar(systemOverlayStyle: …)` from the page brightness (`SystemUiOverlayStyle.dark` on a light band, `.light` on a dark band). Reuse the project's overlay helper.
