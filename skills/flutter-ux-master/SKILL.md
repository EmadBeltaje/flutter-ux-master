---
name: flutter-ux-master
description: Audits a Flutter codebase for easy-to-overlook UX details — empty/error/offline/loading states, keyboard covering fields, pull-to-refresh, tap targets, semantics, missing haptics, SnackBars behind the nav bar, images that pop in, unformatted dates/numbers/phones, mechanical bottom sheets, iOS swipe-back, Android back, text fields without the right keyboard, and other native-feel polish. Every invoke ends on a violation report; the user picks what to fix. Use when they ask to audit, review, polish, or improve Flutter UX, ask what's missing before launch, or name this skill (flutter-ux-master / flutter UX master) — including while creating a screen (watch, then report).
paths:
  - "**/*.dart"
---

# Flutter UX Master

A checklist-driven auditor for the details that make a Flutter app feel finished. Each rule has **Detect** (what counts), **Hunt** (how to grep), and **Fix** (how to implement). A hunt hit is a candidate, not a finding — **Detect** decides.

This is polish, not redesign: skip new screens, visual restyles, and adaptive layout (breakpoints, `NavigationRail` vs `NavigationBar`, master-detail).

Skip `test/`, `*_test.dart`, demo/example apps, and generated files on every rule.

## Workflow

This skill reports. It does not apply the catalog until the user names findings from a report in this conversation.

**Watch** — the user named this skill while creating or changing a screen: finish that work without catalog edits, then audit what shipped, then report.
**Review** — the user asked to audit, review, or polish existing code: same end state. Audit, then report.

**1. Load the groups the work needs.**
Read the matching file(s) below (skill folder, not the app repo). Load every group that applies to this task or screen — often more than one. A checkout page can need Forms, Lists (keyboard dismiss, pull-to-refresh), Images and type (dates/numbers), Accessibility, and Motion (haptics, SnackBar). Skip groups that have nothing to do with the work. Full-app audit: every group. One named detail ("format this date"): that one group.

- States — [states.md](references/states.md)
- Lists and scrolling — [lists-and-scrolling.md](references/lists-and-scrolling.md)
- Images and type — [images-and-type.md](references/images-and-type.md)
- Forms and keyboard — [forms-and-keyboard.md](references/forms-and-keyboard.md)
- Motion and chrome — [motion-and-chrome.md](references/motion-and-chrome.md)
- Platform — [platform.md](references/platform.md)
- Accessibility — [accessibility.md](references/accessibility.md)
- First-run and recovery — [first-run-and-recovery.md](references/first-run-and-recovery.md)

Index: [ux-detail-rules.md](references/ux-detail-rules.md)

**2. Scope the audit.**
If the user wants everything, run every rule. If they name an area or a screen, run the groups that area needs — not one group by default, and not the full catalog unless they asked for it.

**3. Detect, per rule.**
Run the `Hunt` greps, then apply `Detect`. Use that rule's N/A / handled / skip conditions. Do not flag a keyword in an unrelated file.

**4. Report. Wait.**
One line per finding. Done for this invoke. Do not edit for catalog rules in the same turn as the report.

```
- **{rule title}** — {one-line why}
  `{file}:{line}` — {what you saw}
```

Mark scoped rules that do not apply as `N/A: {reason}`. Do not invent a priority order; group quick theme/one-liner wins above layout or state-machine work so they are not buried. Zero findings: say so and stop.

**5. Fix only what they pick.**
After a report in this conversation, the user names findings (or says fix everything on that report). Follow each picked rule's **Fix**, one at a time. **Link** is optional extra — do not block on fetching it. Prefer **Fix** if they disagree. Match the project's state management, routing, and widgets — do not paste a generic snippet.

## Notes

- Zero hunt hits usually means N/A (no `google_fonts`, no phone fields), not a miss. The catalog says which is which.
- "The app does this somewhere" is not enough to pass a rule that asks for a specific screen, and one uncovered moment is not enough to fail a rule that only requires the behavior to exist.
