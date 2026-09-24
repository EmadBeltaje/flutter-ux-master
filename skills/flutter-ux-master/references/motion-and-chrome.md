# Motion and chrome

## 34. Make bottom sheets smooth and draggable

**Link:** https://api.flutter.dev/flutter/material/showModalBottomSheet.html

**Why:** the default `showModalBottomSheet` opens and closes like a mechanical lid. Small sheets should drag with a handle.

**Detect:** a small, non-scrolling sheet opened with default `showModalBottomSheet` — no drag handle, no custom transition, not a `DraggableScrollableSheet`, not a sheet package. Full-screen / `isScrollControlled` sheets are rule 35. `showModalBottomSheet` used only to host a `DraggableScrollableSheet`, `showDragHandle: true`, `enableDrag: true` plus a handle, or packages (`wolt_modal_sheet`, `modal_bottom_sheet`, `smooth_sheets`) are handled.

**Hunt:** grep `showModalBottomSheet`. N/A if none. Handled if each call uses `showDragHandle: true`, wraps `DraggableScrollableSheet`, sets a custom `transitionAnimationController`, or comes from a sheet package. Match only the default mechanical calls that have none of those. Do not flag every `showModalBottomSheet`.

**Fix:** for small sheets, `showModalBottomSheet(..., showDragHandle: true, enableDrag: true)` or the project's sheet helper. Do not turn a small sheet into a full-screen route.

---

## 35. Give full-screen modals the modern sheet look

**Link:** https://api.flutter.dev/flutter/widgets/DraggableScrollableSheet-class.html

**Why:** a full-height modal that just slides up feels like a second route with no grab point. It should read as a sheet and close on a pull down from the top.

**Detect:** a modal that is visually full-screen (child height is the screen, or `CupertinoSheetRoute`) and is not a pull-to-dismiss sheet. `isScrollControlled: true` alone is N/A — that flag is also used to lift a small sheet above the keyboard. Small sheets are rule 34.

**Hunt:** grep `CupertinoSheetRoute` and `isScrollControlled: true`. For the latter, open the builder: match only when the child is full height (`MediaQuery.size.height`, `height: double.infinity`, `Expanded` fill). Handled if that route is a modern sheet (drag handle + pull-to-dismiss, or the project's sheet helper). N/A if there are no full-height bottom modals.

**Fix:** use the project's sheet helper, or a pull-to-dismiss full-height sheet (`showDragHandle: true` + drag, `DraggableScrollableSheet`, or `CupertinoSheetRoute`). The child needs a grab point — not a second `Scaffold` with no handle.

---

## 36. Make the whole GestureDetector area tappable

**Link:** https://api.flutter.dev/flutter/rendering/HitTestBehavior.html

**Why:** `GestureDetector` defaults to hitting only what the child paints. Padding and gaps in a `Row` do nothing, so a row tap misses. The whole box should hit.

**Detect:** a `GestureDetector` whose `behavior` is not `opaque` or `translucent` and whose child has unpainted area (`Padding`, gaps in `Row`/`Column`, `Container` with no color). `InkWell` / `IconButton` / `TextButton` are N/A. A detector whose child is a fully painted box is handled even without `behavior`.

**Hunt:** grep `GestureDetector`. Skip those that already set `HitTestBehavior.opaque` or `.translucent`. Match one whose child is `Padding`, `Row`, `Column`, or a colorless `Container` and `behavior` is unset. Do not flag every `GestureDetector`.

**Fix:** set `behavior: HitTestBehavior.opaque` (or `translucent` if hits behind must still work). Prefer `InkWell` / `IconButton` when the parent is already Material.

---

## 37. Add haptic feedback to key moments

**Link:** https://api.flutter.dev/flutter/services/HapticFeedback-class.html

**Why:** taps and results that happen in silence feel flat. A light haptic on a tab change, a successful submit, or an error makes the app feel like it is in the hand.

**Detect:** for each of these moments that exist, that path has no haptic: (1) bottom-nav / tab change, (2) successful submit, (3) error shown to the user (snackbar, dialog, inline). Web-only apps are N/A. A haptic on some other control does not handle these — report each missing moment as its own finding. Toggles are extra, not a pass by themselves.

**Hunt:** grep `HapticFeedback|haptic_feedback|Haptics.|vibrate`. N/A if there is no `android/` or `ios/` folder. Then open tab `onTap` / `onDestinationSelected`, submit-success, and error UI. Match each of those three that has no haptic on that path — do not pass the rule because an unrelated widget buzzes.

**Fix:** `HapticFeedback.selectionClick()` or `lightImpact()` on tab change; `lightImpact()` / `mediumImpact()` on successful submit; `heavyImpact()` or `vibrate()` when showing an error. Use the project's haptic helper if it has one. Do not add a package.

---

## 38. Don't hide SnackBars behind the nav bar or FAB

**Link:** https://api.flutter.dev/flutter/material/SnackBar-class.html

**Why:** a `SnackBar` slides in behind `NavigationBar` or a FAB. The user never sees the error or the undo.

**Detect:** a `Scaffold` with `bottomNavigationBar` or `floatingActionButton` whose snackbars use the default fixed behavior with no margin/padding for that chrome. `SnackBarBehavior.floating` plus a bottom margin that clears the bar/FAB is handled. A nested `Scaffold` whose messenger already sits above the chrome is handled. N/A if the app never shows snackbars.

**Hunt:** grep `showSnackBar|SnackBar(`. N/A if none. Handled if `behavior: SnackBarBehavior.floating` (or `snackBarTheme`) and margin/padding clears the nav/FAB. Match a default fixed snackbar on a scaffold that has a bar or FAB.

**Fix:** `SnackBar(behavior: SnackBarBehavior.floating, margin: EdgeInsets.only(bottom: …))` so it sits above the bar/FAB, or set `snackBarTheme` once. Reuse the project's messenger helper.
