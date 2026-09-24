# States

## 1. Show an empty state when a list has nothing

**Why:** a screen whose data is an empty list paints a blank body. The user does not know if something broke or if there is just nothing here yet. Show a short empty view — icon or illustration, one-line title, optional action — in the same place the list would be.

**Detect:** a collection screen (inbox, search results, cart, feed, favorites) that builds a `ListView` / `GridView` / sliver list when `isEmpty` / `itemCount == 0` and has no dedicated empty child. A loading placeholder is not an empty state. A screen that cannot be empty (hard-coded items) is N/A.

**Hunt:** grep `ListView|GridView|SliverList|SliverGrid|itemCount`. N/A if the app has no collection screens. Handled if empty branches render a named empty widget (`EmptyView`, `EmptyState`, l10n `nothingHere`, icon + title column). Match if `itemCount` / `isEmpty` still builds an empty scrollable or `SizedBox.shrink` / `Container()`.

**Fix:** when the collection is empty, replace the scrollable with a centered column: optional icon, title, one-line explanation, optional button. Keep it in the same scaffold slot as the list so chrome does not jump. Reuse the project's empty widget if it has one.

---

## 2. Show an error with retry when a load fails

**Why:** a failed fetch that only prints to the console, swallows the error, or flashes a snackbar over a blank body leaves the user stuck. They need to see that it failed and a way to try again without leaving the screen.

**Detect:** a screen that loads remote data (`FutureBuilder`, `StreamBuilder`, bloc/cubit, repository call) and on error shows no in-place retry. A snackbar or dialog alone on an otherwise empty body does not count. A fire-and-forget analytics call is N/A. Widget-build crashes belong to rule 47, not this one.

**Hunt:** grep `FutureBuilder|StreamBuilder|snapshot.hasError|AsyncError|isError|LoadFailure|onError`. N/A if the app has no async loads. Handled if a failed load renders a body with a retry control that re-runs the same fetch. Match if `hasError` / failure state builds nothing, a blank scaffold, or only `SnackBar` / `showDialog`.

**Fix:** map the failure to a dedicated view in the same slot as the content: short message + retry that triggers the existing refresh/fetch. Reuse the project's error widget if it has one.

---

## 3. Tell the user when they're offline

**Why:** a network call that fails because there is no connection looks like a generic error, or the screen just sits on a spinner. If the device is offline, say so, and keep showing cached content when you have it.

**Detect:** the app fetches over the network and has no offline path — no connectivity listen, no "you're offline" banner/view, and failed calls are not distinguished from other errors. A one-off airplane-mode test screen is not enough. Local-only apps (no HTTP / no remote SDK) are N/A.

**Hunt:** grep `http.|Dio|chopper|graphql|supabase|firebase` for network use; then `connectivity_plus|InternetConnection|offline|isOffline|NetworkAware|hasConnection`. N/A if there is no network client. Handled if connectivity (or an equivalent reachability check) drives a visible banner or inline state, and cached data stays on screen when present. Match if network screens exist and nothing in the UI mentions offline / no connection.

**Fix:** listen to connectivity (or the project's existing reachability helper — do not add a package if one is already there). Show a compact banner or inline state. Prefer cached content plus the banner over a full-screen blocking error.

---

## 4. Don't blank the screen while loading

**Why:** replacing the whole body with nothing, or disabling a submit with no visual change, feels like the tap was ignored. Keep the layout and show progress in place — a skeleton, a centered indicator in the content slot, or a spinner on the button.

**Detect:** a load or submit that clears the body (`SizedBox.shrink`, empty `Container`, missing `waiting` branch) or a button that stays visually idle while `isLoading` / `isSubmitting` is true. A first-launch splash is not this rule (see rule 48 for Flutter web boot). An already-visible list that refreshes in the background is handled.

**Hunt:** grep `FutureBuilder|ConnectionState.waiting|isLoading|isSubmitting|Busy|Shimmer|Skeleton`. N/A if there are no async screens or submits. Handled if waiting builds a placeholder that occupies the same slot, or the submit button shows progress and ignores repeat taps. Match if `waiting` / `isLoading` returns an empty widget, or `onPressed` stays enabled with no indicator for the whole in-flight request.

**Fix:** `FutureBuilder`/`StreamBuilder`: handle `waiting` with a skeleton or progress in the content slot. Submits: disable `onPressed` and swap the label for a small `CircularProgressIndicator` (or the project's button-loading pattern) until the future settles.
