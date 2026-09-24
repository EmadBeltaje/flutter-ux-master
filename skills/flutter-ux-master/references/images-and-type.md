# Images and type

## 18. Load network images smoothly

**Link:** https://api.flutter.dev/flutter/widgets/Image/Image.network.html

**Why:** a network image with no placeholder and no failure state pops in or leaves a hole. Show a quiet placeholder, fade the image in, and on failure show a quiet broken-image treatment — not a technical message.

**Detect:** a network image with no loading placeholder or no error widget. `Image.asset` / bundled images are N/A. `FadeInImage`, `CachedNetworkImage` (with `placeholder` + `errorWidget`), and `image_fade` count as handled.

**Hunt:** grep `Image.network|NetworkImage|CachedNetworkImage|ExtendedImage|FadeInImage|image_fade`. N/A if none. Handled if every user-visible network image has a placeholder and an error widget (or uses `FadeInImage` / a helper that does both). Match the first user-visible network image that has neither.

**Fix:** use the project's image helper if it has one. Otherwise `CachedNetworkImage` (only if already in pubspec) or `Image.network` / `FadeInImage` with `loadingBuilder`/`placeholder` and `errorBuilder`. Quiet placeholder and quiet broken state — no exception text.

---

## 19. Reserve space for images before they load

**Link:** https://api.flutter.dev/flutter/widgets/AspectRatio-class.html

**Why:** an unsized image snaps to its decoded height and pushes everything below it. Size the box before the bytes arrive.

**Detect:** a network image with no size before load: no `AspectRatio`, no width and height, no parent that fixes height. `fit:` alone does not count. Width without height does not count.

**Hunt:** grep `Image.network|CachedNetworkImage|NetworkImage`. N/A if none. Handled if the image (or a shared avatar/card wrapper) has `AspectRatio`, both width and height, or a height-bounded parent. Match the first user-visible network image that has none of those.

**Fix:** wrap in `AspectRatio` (known ratio) or give both width and height. Avatars: a square `SizedBox`. Prefer a shared card/avatar wrapper over sizing each call site.

---

## 20. Preload first-screen images and icons

**Link:** https://api.flutter.dev/flutter/widgets/precacheImage.html

**Why:** Flutter decodes assets on first paint, so splash logos and home icons flicker in a few frames late. Precache the first screen during startup — not every asset in the bundle.

**Detect:** the first route (splash, home, shell) shows `SvgPicture` / `Image.asset` / `AssetImage` and those first-screen assets are never precached. Precaching the whole `assets/` tree is not required. Screens the user can only reach later are N/A.

**Hunt:** grep `SvgPicture|Image.asset|AssetImage` on the first route / splash / home; then `precacheImage|svg.cache`. N/A if the first screen has no asset images/icons. Handled if those first-screen assets are precached during splash/startup. Match if the first screen uses asset images/icons and nothing precaches them.

**Fix:** during splash/startup, `precacheImage(AssetImage('…'), context)` for raster assets and `svg.cache.putIfAbsent` (or the project's SVG cache) for first-screen SVGs. Only the assets that first route paints.

---

## 21. Preload Google Fonts so text doesn't swap fonts

**Link:** https://pub.dev/packages/google_fonts

**Why:** `google_fonts` downloads on first use, so a fresh install flashes the default font. Await the fonts you actually use during splash.

**Detect:** `google_fonts` is in pubspec and nothing awaits `GoogleFonts.pendingFonts` at startup for the weights/styles the app uses. A bare `GoogleFonts.pendingFonts([GoogleFonts.inter()])` only loads 400 normal — that is not handled if the UI uses more. N/A if the package is absent (bundled fonts only).

**Hunt:** grep `google_fonts` in pubspec. N/A if missing. If present, grep `pendingFonts`. Match if the package is used and `pendingFonts` is missing, or it lists only a default weight the UI does not stop at.

**Fix:** on splash/startup, `await GoogleFonts.pendingFonts([…])` with every family and weight the UI actually uses (e.g. `GoogleFonts.inter()`, `GoogleFonts.inter(fontWeight: FontWeight.w600)`). Do not load the whole catalog.

---

## 22. Format numbers for user's locale

**Link:** https://pub.dev/documentation/intl/latest/intl/NumberFormat-class.html

**Why:** `1234567` is hard to read. Users expect `1,234,567` or `1.234.567` depending on locale.

**Detect:** a user-visible count, price, or amount rendered with raw interpolation / `.toString()` / `.toStringAsFixed` instead of `NumberFormat` (or an equivalent: `NumberFormat`, `intl` compact, a money widget). IDs, versions, and debug labels are N/A. Copyable IDs are rule 24.

**Hunt:** grep `NumberFormat|decimalPattern|simpleCurrency|compact(`. A shared display helper used for those values is handled. Otherwise grep `Text(` that interpolates `price|amount|total|count|balance|quantity` without a formatter. Match only user-visible amounts/counts. Skip IDs and `toString` in non-UI files.

**Fix:** format with `NumberFormat` (decimal / `simpleCurrency` / compact) using the project's locale (`intl`, l10n, or `Localizations.localeOf`). Reuse an existing money/count helper. Skip IDs and versions.

---

## 23. Format dates for user's locale

**Link:** https://pub.dev/documentation/intl/latest/intl/DateFormat-class.html

**Why:** `2016-06-24 14:44:00.000` is what `DateTime` prints. Show a locale date like `24 July 2016, 14:44`.

**Detect:** a date shown to the user without `DateFormat` / `intl` / `timeago` / a project date helper. Hand-built `'$day/$month/$year'` is a match. `DateTime` used only in logic, parsing, or logging is N/A.

**Hunt:** grep `DateFormat|timeago|DateTimeExtension|formatDate`. A shared helper used for display is handled. Otherwise grep `Text(` near `DateTime` or `.$day|$month|$year`. Match a user-visible date that is raw `.toString()` or hand-concatenated. Skip non-UI files.

**Fix:** display through `DateFormat` (or `timeago` / the project's date helper) with the app locale. Do not concatenate `day/month/year` by hand. Reuse the existing helper if the project has one.

---

## 24. Let users copy IDs and codes

**Link:** https://api.flutter.dev/flutter/material/SelectableText-class.html

**Why:** an order number, booking code, or IBAN in a plain `Text` cannot be selected. The user has to retype it or screenshot it.

**Detect:** a user-visible identifier (order id, confirmation / booking code, IBAN, SKU, invite / referral code, tracking number) rendered with `Text` and no select or copy. Names, titles, and body copy are N/A. A control that already copies is handled.

**Hunt:** grep `orderId|orderNumber|bookingCode|confirmationCode|iban|trackingNumber|referralCode|inviteCode|sku` near `Text(`. N/A if none. Handled if that value is `SelectableText`, or an onTap / icon calls `Clipboard.setData` with a short confirmation. Match the first such value in a plain `Text`.

**Fix:** `SelectableText` for short IDs, or tap-to-copy with `Clipboard.setData` plus the project's snackbar/toast. Do not wrap every `Text` in `SelectableText`.
