# Accessibility

## 44. Give icon-only controls a semantic label

**Why:** a back chevron, overflow `...`, or icon-only `GestureDetector` with no label is silent to TalkBack and VoiceOver. The control must speak its name.

**Detect:** an interactive control whose visible child is only an icon (no text) and that has no `tooltip`, `semanticsLabel`, `semanticLabel`, or wrapping `Semantics(label:)`. `IconButton` with `tooltip:` is handled. Decorative icons that are not tappable should be `excludeFromSemantics: true` (or sit under an already-labeled parent) — flag those only when they are announced as unnamed buttons. Text buttons and labeled chips are N/A.

**Hunt:** grep `IconButton|InkWell|GestureDetector|IconButton.icon`. For each icon-only control, check `tooltip:|semanticsLabel|semanticLabel|Semantics(`. Handled if present. Match an icon-only control with none. Skip icons inside a `ListTile` / `NavigationDestination` that already has a label.

**Fix:** add `tooltip:` on `IconButton`, or `Semantics(label: '...', button: true)` on custom detectors. Use the action name ("Close", "Search"), not the icon name ("X", "magnifying glass"). Mark decorative icons `excludeFromSemantics: true`.

**Link:** https://docs.flutter.dev/ui/accessibility-and-internationalization/accessibility

---

## 45. Keep tap targets at least 48dp

**Why:** a 24dp icon with zero padding is easy to miss and fails accessibility guidelines. Interactive targets need a minimum 48×48 logical-pixel hit area, even when the glyph is smaller.

**Detect:** a tappable widget whose hit box is under 48×48: `IconButton` with `visualDensity: VisualDensity.compact` / `padding: EdgeInsets.zero` / tight `constraints`, or `GestureDetector` / `InkWell` wrapping a 16–24 icon with no min size. `IconButton` at default density is handled (it is already 48). A dense data-table cell that is not the only way to do that action is a note, not a fail, if a 48dp control sits next to it.

**Hunt:** grep `IconButton|InkWell|GestureDetector` around `Icon(`. Match `visualDensity: VisualDensity.compact`, `padding: EdgeInsets.zero`, `constraints: BoxConstraints(` under 48, or an `Icon(size: 16|18|20|24)` as the only child of a detector/InkWell with no `minimumSize` / `SizedBox` 48. Handled if `styleFrom(minimumSize: Size(48, 48))`, `minimumSize: Size(48, 48)`, or a 48 `SizedBox` wraps the hit.

**Fix:** prefer `IconButton` defaults, or `styleFrom(minimumSize: const Size(48, 48))`. For custom detectors, wrap with `SizedBox(width: 48, height: 48)` (or `ConstrainedBox`) and center the icon. Do not enlarge the glyph just to hit 48.

**Link:** https://m3.material.io/foundations/accessible-design/accessibility-basics

---

## 46. Let text scale without breaking the layout

**Why:** users raise text size so they can read the app. A toolbar title, list title, or nav label that cannot reflow overflows at 1.5–2×. Reflow first — wrap, scroll, `Expanded`. Clamp only after you have measured a screen that cannot reflow, and never pin the whole app to 1.0.

**Detect:** (1) app-wide `textScaler` / `textScaleFactor` clamped to `1.0` or `disabled` — always a match. (2) An `AppBar` / `SliverAppBar` title, `ListTile` title/subtitle, or `NavigationBar` / `BottomNavigationBar` label whose text cannot wrap, flex, or ellipsize. Icon + short-label rows, `Chip`s, and buttons with a short string are N/A. A measured clamp around `1.3` on a screen that cannot reflow (fixed HUD, camera overlay) is handled for that screen only. Do not flag every `Row` that contains `Text`.

**Hunt:** grep `textScaler|textScaleFactor|MediaQuery.textScalerOf` for a `1.0` / `disabled` clamp. Then only `AppBar` / `SliverAppBar` titles, `ListTile` title/subtitle, and bottom-nav labels. Do not sweep the repo for `Row(` + `Text(`.

**Fix:** on those titles/labels, wrap or `Flexible` + ellipsis, and let the page scroll. If a specific overlay cannot reflow, clamp that subtree with `MediaQuery.withClampedTextScaling` (around 1.3), not `MaterialApp` at 1.0.

**Link:** https://api.flutter.dev/flutter/widgets/MediaQuery/textScalerOf.html
