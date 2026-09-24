# Lists and scrolling

## 5. Don't let the system navigation bar cover the bottom of scrollable lists

**Link:** https://api.flutter.dev/flutter/widgets/MediaQuery/viewPaddingOf.html

**Why:** the last item of a full-screen list hides behind the system navigation bar. Leave space as tall as that bar at the end of the list so the last item has breathing room.

**Detect:** a page-level vertical scrollable that runs to the bottom of the screen and does not end with inset equal to `MediaQuery.viewPadding.bottom` (or `viewPaddingOf`). `SafeArea` around the list is the wrong fix: it shrinks the viewport, so items clip as they pass the bar. A fixed `24` padding is not a fix. Nested lists, dialogs, bottom sheets, and shrink-wrapped lists inside a parent scroll are N/A.

**Hunt:** grep `ListView|CustomScrollView|GridView|NestedScrollView` that are `Scaffold.body` (or the page's primary scroll). Skip `shrinkWrap: true`, horizontal axis, and anything inside `showDialog` / `showModalBottomSheet`. Handled if the scroll extent uses `viewPadding.bottom` / `viewPaddingOf` / `SliverPadding` with that inset. Match if a page-level vertical scrollable has neither.

**Fix:** pad the page-level scrollable with `MediaQuery.viewPaddingOf(context).bottom` (add to existing padding, do not replace it). On a `CustomScrollView`, a trailing `SliverPadding` with that inset. Do not wrap the list in `SafeArea`.

---

## 6. Make horizontal lists feel scrollable

**Link:** https://api.flutter.dev/flutter/widgets/ShaderMask-class.html

**Why:** a horizontal list that exactly fills the viewport looks like a static row. Users never swipe. A clipped next item, a page indicator, or a fade on the end edge tells them there is more.

**Detect:** a horizontal scrollable that (1) can overflow and (2) has no affordance — no end fade, no peek of the next item, no pager dots / "see all". A row that always fits (two or three fixed chips) is N/A.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`. N/A if none. Handled if that list has `ShaderMask` / edge `LinearGradient`, `padEnds` / trailing padding that clips a next item, `PageIndicator` / dots, or a visible "see all". Match only when a horizontal list of dynamic/long content has none of those.

**Fix:** give the list end padding so the next item peeks, or wrap it in a `ShaderMask` that fades the trailing edge. `PageView`: add dots. Do not add a fade to a row that already fits.

---

## 7. Scroll to top when the current bottom nav item is tapped again

**Link:** https://api.flutter.dev/flutter/material/NavigationBar-class.html

**Why:** tapping the bottom nav item you are already on should scroll that page to the top. Native apps do this; users will try it.

**Detect:** the app has a bottom nav and tapping the selected tab does nothing. Look at the tap handler (or the bloc/cubit behind it): no same-index branch means not handled.

**Hunt:** grep `BottomNavigationBar|NavigationBar|CupertinoTabBar`. N/A if none. Handled if the tap path has `index == currentIndex` (or equivalent) that scrolls / jumps to top. Match if a bar exists and the handler only switches index.

**Fix:** in the nav tap/destination handler, when `index == currentIndex`, animate that tab's `ScrollController` (or `PrimaryScrollController.of(context)`) to 0. Otherwise switch index as today.

---

## 8. Scroll the tapped tab fully into view

**Link:** https://api.flutter.dev/flutter/widgets/Scrollable/ensureVisible.html

**Why:** a half-visible chip or category tab should scroll fully on-screen when tapped.

**Detect:** a custom horizontal tab/chip/filter strip that tracks a selected index and does not scroll the tapped item into view. Material `TabBar` already does this — those are handled.

**Hunt:** grep `TabBar` first; those are fine. Then find a horizontal `ListView` / `SingleChildScrollView` whose children are tappable and share a selected index. Handled if the selection handler calls `Scrollable.ensureVisible` or scrolls a controller to that item. Match if a custom strip has neither.

**Fix:** on tap, `Scrollable.ensureVisible` the item (short duration, alignment ~0.5) or jump the strip's `ScrollController` to that child's offset. Leave stock `TabBar` alone.

---

## 9. Keep statusbar tap scrolling to top on iOS

**Link:** https://api.flutter.dev/flutter/widgets/PrimaryScrollController-class.html

**Why:** tapping the status bar scrolls to top on iOS. A custom `ScrollController` on the primary list silently breaks it unless that same instance is the `PrimaryScrollController`.

**Detect:** a vertical page scrollable with its own `ScrollController` inside a `Scaffold` / `CupertinoPageScaffold` that is not attached as the primary controller. Nested/inner controllers (a horizontal carousel, a tab view's inner list) are N/A. Scrollables with no custom controller are handled — Flutter wires those.

**Hunt:** grep `ScrollController(`. N/A if none. For each controller passed to a vertical page-level scrollable, check that an ancestor `PrimaryScrollController` holds that same instance. Do not treat `primary: true` as handled — a scrollable with a custom `controller` cannot also be `primary: true` (Flutter asserts). Match the first page-level vertical list that has a custom controller and no such ancestor.

**Fix:** wrap the page with `PrimaryScrollController(controller: myController, child: …)` and pass that same instance to the list. Nested/inner lists keep their own controllers and stay off the primary.

---

## 10. Let horizontal lists reach the screen edges

**Link:** https://api.flutter.dev/flutter/widgets/ListView-class.html

**Why:** when the page scrollable has horizontal padding, a nested horizontal list clips at that padding instead of sliding to the screen edge. The list should own its padding and reach the edges.

**Detect:** a horizontal list nested in a padded page scrollable, so items clip at the padding line. A horizontal list that is meant to be inset (a card carousel inside a padded card) is N/A.

**Hunt:** grep `Axis.horizontal|scrollDirection: Axis.horizontal`. N/A if none. For each, read the parent: if a page `ListView` / `SingleChildScrollView` / `Padding` applies horizontal padding around the list, that is a match. Skip lists whose parent is already edge-to-edge.

**Fix:** pull the nested list out of the parent's horizontal padding. Put that padding on the horizontal list's first/last items (or on the horizontal `ListView.padding`) so items can scroll to the screen edge.

---

## 11. Dismiss the keyboard when the user scrolls

**Link:** https://api.flutter.dev/flutter/widgets/ScrollViewKeyboardDismissBehavior.html

**Why:** the user finishes typing and scrolls, but the keyboard still covers half the screen. Scrolling means they are done — close it. Chat threads are the exception: they scroll while typing.

**Detect:** a form-style scrollable that contains a text field and does not dismiss on drag. Handled if `keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag` is set on that scrollable (or an ancestor `ScrollConfiguration`). Also handled if a parent `GestureDetector` unfocuses on vertical drag. Chat / composer screens stay on `manual`.

**Hunt:** grep `TextField|TextFormField|CupertinoTextField` and find the enclosing vertical `ListView` / `SingleChildScrollView` / `CustomScrollView`. N/A if no scrollable contains a field. Skip files that are chat / conversation / composer. Handled if that scrollable or its `ScrollConfiguration` sets `onDrag`, or a wrapper unfocuses on scroll. Match a form scrollable that has neither.

**Fix:** set `keyboardDismissBehavior: ScrollViewKeyboardDismissBehavior.onDrag` on the form's vertical scrollable (or an ancestor `ScrollConfiguration`). Leave chat/composer on `manual`.

---

## 12. Let the user pull to refresh collection screens

**Link:** https://api.flutter.dev/flutter/material/RefreshIndicator-class.html

**Why:** a remote list that never refreshes from a pull feels stuck. Native apps put pull-to-refresh on inboxes, feeds, and search results.

**Detect:** a collection screen that loads remote data and has no pull-to-refresh (`RefreshIndicator`, `CupertinoSliverRefreshControl`, or the project's refresh wrapper). Local-only / hard-coded lists are N/A. Load-more / end-of-list is rule 13, not this.

**Hunt:** grep `RefreshIndicator|CupertinoSliverRefreshControl`. N/A if the app has no remote collection screens. Handled if those lists wrap the scrollable (or a sliver) with refresh that re-runs the existing fetch. Match the first remote collection that has neither.

**Fix:** wrap the Material list in `RefreshIndicator` (or add `CupertinoSliverRefreshControl` to a `CustomScrollView`) and call the existing refresh/fetch. If the list can be shorter than the viewport, use `AlwaysScrollableScrollPhysics` so the pull still works.

---

## 13. Paginate long lists and show the end

**Link:** https://api.flutter.dev/flutter/widgets/ScrollController-class.html

**Why:** a feed that dumps the first page and stops feels broken. The user needs more as they near the bottom, and a clear end when there is nothing left — not a spinner that never dies.

**Detect:** a remote collection that can grow past one screen (feed, inbox, search results, orders, notifications) with no next-page load and no end-of-list treatment. A short list fetched in full in one call is N/A. An empty collection is rule 1. Pull-to-refresh is rule 12.

**Hunt:** grep `ListView.builder|GridView.builder|SliverList` on those collection screens. Look for `page|cursor|offset|loadMore|onEndReached|PagingController|infinite_scroll_pagination` and a footer (`CircularProgressIndicator` / "that's all" / l10n `noMore`). N/A if there are no remote collections. Handled if scrolling near the end loads the next page (or a "Load more" control does), and the last page replaces the footer spinner with an end mark — or the fetch is clearly unpaged and complete. Match the first long remote list that only renders the first response, or that keeps a spinner after the last page.

**Fix:** when the user is near `maxScrollExtent`, call the existing next-page/cursor fetch. Show a small footer spinner while that page loads. On the last page, swap it for a short end label (or nothing — but not a spinner). Reuse the project's paging helper / `infinite_scroll_pagination` if it is already there. Do not add a package otherwise.

---

## 14. Debounce search-as-you-type

**Link:** https://api.flutter.dev/flutter/dart-async/Timer-class.html

**Why:** firing a search on every keystroke makes the list flicker and hammers the network. Wait until the user pauses, then search once.

**Detect:** a search field whose `onChanged` (or the bloc/cubit event behind it) kicks off a network search with no debounce. Search that only runs on submit (`onSubmitted`, a search button, or `SearchAnchor` suggestions that are local) is handled. Filtering an in-memory list is N/A.

**Hunt:** grep `SearchBar|SearchAnchor|SearchDelegate|hintText:.*[Ss]earch|searchQuery|onSearch`. N/A if none. Handled if that path uses `Timer`, `debounce`, `debounceTime`, `easy_debounce`, or a project debounce helper before the fetch. Match `onChanged` (or every-keystroke events) that call the repository/API directly.

**Fix:** debounce ~300ms with `Timer` (cancel the previous timer on each change) or the project's existing debounce. Cancel the in-flight request when the query changes. Empty query: show the idle/recent state — do not fire. Do not add a package.

---

## 15. Keep tab and pager state when the user switches away

**Link:** https://api.flutter.dev/flutter/widgets/AutomaticKeepAliveClientMixin-mixin.html

**Why:** switching bottom-nav tabs or pager pages rebuilds the child from scratch. Scroll position and in-progress form input vanish.

**Detect:** a `TabBarView` / `PageView` / bottom-nav body that builds a new child on every index change with no `IndexedStack`, `AutomaticKeepAliveClientMixin`, or `PageStorageKey`. Nested inner lists are N/A. A tab that should reset (e.g. search) is N/A.

**Hunt:** grep `TabBarView|PageView|IndexedStack|BottomNavigationBar|NavigationBar`. Handled if the tab/pager body is an `IndexedStack`, uses `AutomaticKeepAliveClientMixin`, or has a `PageStorageKey` that restores scroll. Match a switch/`builder` that creates a fresh page each visit and drops scroll or field state.

**Fix:** `IndexedStack` for a small set of tabs, or `AutomaticKeepAliveClientMixin` / `PageStorageKey` on each tab body. Prefer `PageStorageKey` when you only need scroll restoration and the page is heavy.

---

## 16. Offer undo after a destructive swipe

**Link:** https://api.flutter.dev/flutter/material/SnackBarAction-class.html

**Why:** swipe-to-delete with no undo is a trap. Mail and messages let the user put the item back.

**Detect:** `Dismissible` / `Slidable` (or similar) that deletes on dismiss with no `SnackBarAction` undo and no `confirmDismiss` / confirm dialog. Archive-to-another-tab with an obvious reverse is handled. Non-destructive swipes are N/A.

**Hunt:** grep `Dismissible|Slidable|onDismissed|confirmDismiss`. N/A if none. Handled if that path shows undo (`SnackBarAction`) or confirms first. Match a destructive swipe that commits immediately with neither.

**Fix:** after dismiss, show a `SnackBar` with an Undo `SnackBarAction` that restores the item (or delay the delete until the bar closes). `confirmDismiss` is an acceptable alternative. Match the project's snackbar style.

---

## 17. Show a click cursor on tappable rows on desktop and web

**Link:** https://api.flutter.dev/flutter/widgets/MouseRegion-class.html

**Why:** on a pointer device, a `GestureDetector` row keeps the arrow cursor. It does not look tappable.

**Detect:** applies only if `web/`, `macos/`, `windows/`, or `linux/` exists. N/A for mobile-only. A `GestureDetector` / custom tap target with `onTap` and no `MouseRegion` / `mouseCursor: SystemMouseCursors.click`. `InkWell`, `IconButton`, `TextButton`, and other Material buttons already set a click cursor — those are handled.

**Hunt:** N/A if none of those folders exist. Grep `GestureDetector` with `onTap` and no `MouseRegion` / `SystemMouseCursors.click` / `mouseCursor`. Match the first desktop/web tap target that still uses the arrow.

**Fix:** wrap with `MouseRegion(cursor: SystemMouseCursors.click)` or switch to `InkWell` / a Material button. Do not wrap every detector in a mobile-only app.
