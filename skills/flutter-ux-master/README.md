# flutter-ux-master

Audits a Flutter app for easy-to-miss UX details, then implements the ones you pick. Empty/error/offline/loading states, keyboard, tap targets, haptics, native polish.

Not a redesign skill. Not an adaptive-layout skill.

## How to prompt

```
/flutter-ux-master audit the UX before launch
/flutter-ux-master polish this screen
/flutter-ux-master check empty, error, and loading states
/flutter-ux-master check my forms
/flutter-ux-master add haptics
```

After install, “audit / polish the Flutter UX” is enough — the agent can pick the skill up on its own.

## What to expect

1. Named while you are building a screen: it watches, then reports catalog violations on what shipped.
2. Asked to review existing code: same — report, then wait.
3. It never applies the catalog on invoke. You pick which violations to fix (or say fix everything after the report).
4. Fixes follow the project’s own widgets and state management.

Catalog: states, lists/scrolling, images/type, forms/keyboard, motion/chrome, platform, accessibility, first-run/recovery.
