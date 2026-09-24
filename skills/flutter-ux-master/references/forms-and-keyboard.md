# Forms and keyboard

## 25. Autofocus the field when the page has only one

**Link:** https://api.flutter.dev/flutter/material/TextField/autofocus.html

**Why:** a page that exists to collect one value (OTP, phone, new email) should open with the field focused. Making the user tap first is extra work.

**Detect:** a page whose only interactive control is one text field, and that field is not `autofocus: true`. Login (or similar) with extra social buttons is N/A — do not flag those.

**Hunt:** grep OTP/pin pages (`Pinput|pin_code|otp|smsCode`) and single-purpose files (`change_email|change_phone|edit_name|forgot_password`). N/A if none of those exist. Handled if that field sets `autofocus: true`. Match the first single-field page that does not. Stop there; do not scan every form.

**Fix:** set `autofocus: true` on that field. If the field is built after the first frame, `FocusNode.requestFocus()` once it exists. Do not autofocus login screens that also have social buttons.

---

## 26. Give every text field the right keyboard action

**Link:** https://api.flutter.dev/flutter/services/TextInputAction.html

**Why:** a multi-field form should move to the next field on the keyboard action, and submit on the last one. The user should not have to tap each field.

**Detect:** a page with two or more single-line fields that omit `textInputAction` (`.next` on non-last, `.done` / `.send` / `.go` / `.search` on last, with submit in `onFieldSubmitted` / `onSubmitted`). Multiline fields are N/A — return inserts a newline. A single field is rule 25, not this one.

**Hunt:** find files with two or more `TextField|TextFormField`. N/A if none. Handled if those fields set `textInputAction` (and the last one submits). Match a multi-field form that sets none.

**Fix:** `textInputAction: TextInputAction.next` on non-last fields, moving focus in `onFieldSubmitted`. Last field: `.done` / `.send` / `.go` / `.search` and submit there. Wire `FocusNode`s. Leave multiline fields on default return.

---

## 27. Unfocus the text field before opening a modal

**Link:** https://api.flutter.dev/flutter/widgets/FocusManager-class.html

**Why:** opening a picker or sheet while a field is focused brings the keyboard back when the modal closes. Unfocus first and it stays away.

**Detect:** a page that can have a focused field and also opens a modal (sheet, dialog, date picker, popup) whose opener does not unfocus. `FocusManager.instance.primaryFocus?.unfocus()` is handled. `FocusScope.of(context).unfocus()` is also handled when called from the page that owns the field — do not fail it. Prefer `FocusManager` when writing new code.

**Hunt:** grep `showModalBottomSheet|showDialog|showDatePicker|showCupertinoModalPopup` in files that also have `TextField|TextFormField`. N/A if no overlap. Handled if the opener calls `FocusManager.instance.primaryFocus?.unfocus()` or `FocusScope.of(context).unfocus()` (or the project helper that does). Match the first opener in that overlap that does neither.

**Fix:** before the modal call, `FocusManager.instance.primaryFocus?.unfocus()`. Leave an existing `FocusScope.of(context).unfocus()` on that opener.

---

## 28. Format phone numbers

**Link:** https://api.flutter.dev/flutter/services/TextInputFormatter-class.html

**Why:** a phone number as one run of digits is hard to read and easy to mistype. Format it wherever the user sees or types one.

**Detect:** a user-visible phone value or a phone field with no formatting (`inputFormatters`, mask, or a phone parser). A string named `phone` that is only sent to an API is N/A.

**Hunt:** grep `phoneNumber|msisdn|TextInputType.phone|AutofillHints.telephoneNumber`. N/A if none. Where a phone value hits `Text(` or a `TextField`, look for `MaskTextInputFormatter|inputFormatters|phone_numbers_parser|maskPhoneNumber`. Handled if display and input are formatted. Match the first raw user-visible phone.

**Fix:** format on input (`inputFormatters` / mask / the project's phone parser) and on display. Do not add a package if a formatter or parser is already there. API-only strings stay unformatted.

---

## 29. Keep the dial code from doubling in phone fields

**Link:** https://api.flutter.dev/flutter/services/AutofillHints-class.html

**Why:** a static `+XXX` prefix plus paste/autofill of a full international number stacks the code twice. The user has to delete it by hand.

**Detect:** a phone field that shows the dial code as a separate prefix (`prefixText`, prefix widget, country picker) and nothing strips a pasted/autofilled leading code. No such field is N/A.

**Hunt:** grep `TextInputType.phone|AutofillHints.telephoneNumber` and find a field with a separated dial code. N/A if none. Handled if `inputFormatters` (or the picker package) strips a leading `+` / country code on paste. Match if the prefix is separate and no stripper exists.

**Fix:** on paste/change, if the value starts with `+` or the visible country code, strip that prefix before writing the controller. Prefer the picker package's built-in strip if it has one.

---

## 30. Don't let amount fields go over the limit

**Link:** https://api.flutter.dev/flutter/services/TextInputFormatter-class.html

**Why:** typing past a balance or limit and then seeing an error is extra friction. Block the extra digits as they are typed.

**Detect:** an amount field with a known maximum (balance, quota, order cap) that accepts a larger value and then errors (field error, snackbar, dialog, rejected submit). Blocking at input is handled. A generic numeric field with no maximum is N/A. `TextInputType.number` alone is not a match.

**Hunt:** grep amount fields (`numberWithOptions`, currency controllers) that compare input to a `balance|limit|maxAmount|quota`. N/A if no field has a max. Handled if a formatter / `inputFormatters` rejects values above the max. Match if the only response is an error after the value is already in the field.

**Fix:** add a `TextInputFormatter` that clamps or rejects parsed values above the max as the user types. Keep the existing submit-time check if you want; do not rely on it alone.

---

## 31. Don't let the keyboard cover the focused field

**Link:** https://api.flutter.dev/flutter/material/Scaffold/resizeToAvoidBottomInset.html

**Why:** the user taps a field and the keyboard hides it. They cannot see what they type.

**Detect:** a page with a text field where `Scaffold.resizeToAvoidBottomInset: false` without compensating `viewInsets` padding, or a field that is not inside a vertical scrollable so focus cannot bring it above the keyboard. Default `Scaffold` (resize true) with the field in a `ListView` / `SingleChildScrollView` / `CustomScrollView` is handled. Chat composers that pin a bar above `viewInsets` are handled.

**Hunt:** grep `resizeToAvoidBottomInset:\s*false` and `TextField|TextFormField|CupertinoTextField` that are not inside a vertical scrollable. N/A if there are no fields. Match a false resize with no `MediaQuery.viewInsetsOf` padding, or a pinned field the keyboard covers.

**Fix:** keep `resizeToAvoidBottomInset` true (the default) and put fields in a vertical scrollable. Set `scrollPadding` so the focused field sits above the keyboard. If resize must stay false (custom chat), pad the pinned composer with `MediaQuery.viewInsetsOf(context).bottom`.

---

## 32. Give each text field the matching keyboard

**Link:** https://api.flutter.dev/flutter/services/TextInputType-class.html

**Why:** an email field with a sentence keyboard, a password with no hide/show, or a name that does not capitalize feels unfinished.

**Detect:** a field whose purpose is email, URL, phone, number, name, or password and that still uses default `TextInputType.text` (or is missing the matching `textCapitalization` / `autofillHints` / `obscureText` + visibility toggle / `keyboardAppearance`). Do not require every property on every field — flag a clear mismatch. Multiline body copy is N/A.

**Hunt:** grep `TextField|TextFormField|CupertinoTextField`. For labels/hints/names that are email, password, phone, url, number, or name, check `keyboardType`, `textCapitalization`, `autofillHints`, `obscureText` plus a visibility control, and `keyboardAppearance` vs theme brightness. Match the first clear mismatch.

**Fix:** set `keyboardType` to the data (`emailAddress`, `phone`, `url`, `number`, `visiblePassword`). Names: `textCapitalization: TextCapitalization.words`. Email/password: `none`, password also `obscureText: true`, `autocorrect: false`, `enableSuggestions: false`, and a visibility `IconButton`. Set `autofillHints` and `keyboardAppearance` from `Theme.brightness`. Rule 26 still owns `textInputAction`.

---

## 33. Confirm before popping a dirty form

**Link:** https://api.flutter.dev/flutter/widgets/PopScope-class.html

**Why:** the system back gesture or Android back button pops a half-filled form and the draft is gone. App-wide Android back / predictive back is rule 40 — this rule is only dirty forms.

**Detect:** a pushed route whose job is editing (form, settings text, composer) with local unsaved state and no `PopScope` / `onPopInvokedWithResult` / confirm-discard when dirty. Screens that autosave on every change are N/A. Read-only details are N/A.

**Hunt:** grep `Form|TextEditingController` on pushed routes, then `PopScope|WillPopScope|onPopInvoked`. N/A if no editable pushed routes. Handled if a dirty flag gates `canPop` and shows a discard/save dialog. Match a form route that pops freely while controllers differ from the initial values.

**Fix:** `PopScope(canPop: !dirty, onPopInvokedWithResult: …)` and a discard/save dialog. Use the project's dialog helper. Do not block pops on autosave screens. Do not use `WillPopScope` — it breaks Android predictive back (rule 40).
