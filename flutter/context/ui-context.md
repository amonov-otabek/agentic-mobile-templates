# UI Context — Design System

Machine-readable design system: tokens, theme, and the living widget registry.
`context/ui-rules.md` says how to use these values; this file IS the values. Lock this
before the first UI task is built — retrofitting a design system is how features get
rebuilt (lesson learned from previous projects). This is the primary design reference
for the app.

## Theme

- Mode: **light + dark, both** (default follows system; user can override). One
  `ColorScheme` per mode, both generated from the same seed/tokens.
- Engine: **Material 3** (`ThemeData(useMaterial3: true)`), built from a
  `ColorScheme.fromSeed` plus explicit token overrides. Theme is defined once in
  `core/theme/` (`app_theme.dart` for `ThemeData`, `tokens.dart` for raw values) and
  referenced everywhere via `Theme.of(context)` + a `BuildContext` tokens extension.
- Personality: clean, modern, content-first; generous spacing; rounded corners;
  restrained color with one accent. Mobile-native (Material on Android, Cupertino-aware
  affordances on iOS where it matters).

## Tokens

Single source of truth. Raw values live in `core/theme/tokens.dart`; semantic colors
are exposed through the Material 3 `ColorScheme` so widgets read
`Theme.of(context).colorScheme.*` and never restate hex.

```dart
// core/theme/tokens.dart — raw palette and scales (the ONLY place literals live)
abstract final class AppColors {
  // Brand / accent
  static const seed         = Color(0xFF4F46E5); // indigo — drives the M3 ColorScheme
  // Light
  static const lightSurface   = Color(0xFFFFFFFF);
  static const lightOnSurface = Color(0xFF111114);
  static const lightMuted     = Color(0xFF6B7280);
  static const lightBorder    = Color(0xFFE5E7EB);
  // Dark
  static const darkSurface    = Color(0xFF0E0E11);
  static const darkOnSurface  = Color(0xFFF3F4F6);
  static const darkMuted      = Color(0xFF9CA3AF);
  static const darkBorder     = Color(0xFF26262B);
  // Status (shared across modes)
  static const success = Color(0xFF16A34A);
  static const warning = Color(0xFFD97706);
  static const error   = Color(0xFFDC2626);
}

// Spacing scale (logical pixels)
abstract final class AppSpacing {
  static const xs = 4.0;
  static const sm = 8.0;
  static const md = 12.0;
  static const lg = 16.0;
  static const xl = 24.0;
  static const xxl = 32.0;
  static const xxxl = 48.0;
}

// Corner radii
abstract final class AppRadius {
  static const sm = 8.0;
  static const md = 12.0;
  static const lg = 16.0;
  static const pill = 999.0;
}

// Font family (loaded via pubspec assets or google_fonts if approved)
abstract final class AppFonts {
  static const sans = 'Inter';
}
```

### Color usage guide

Widgets reference the **semantic** `ColorScheme` role, not the raw token.

| Use | ColorScheme role (M3) |
| --- | --- |
| Screen background | `surface` |
| Primary text / icons | `onSurface` |
| Secondary / muted text | `onSurfaceVariant` |
| Primary actions, active nav, links | `primary` / `onPrimary` |
| Cards / elevated surfaces | `surfaceContainer` / `surfaceContainerHigh` |
| Borders / dividers | `outlineVariant` |
| Error states | `error` / `onError` |
| Success / warning | extend the scheme via a `ThemeExtension` (`AppStatusColors`) using `AppColors.success` / `AppColors.warning` |

Material 3's scheme does not ship success/warning roles — expose them through a
`ThemeExtension` so they remain token-driven and theme-aware (light/dark).

## Typography scale

Defined once as a Material 3 `TextTheme` in `core/theme`; widgets use
`Theme.of(context).textTheme.*` (or a small named alias). Sizes scale with the OS.

| Role (M3 slot) | Size / weight | Use |
| --- | --- | --- |
| `headlineSmall` | 24 / w700 | screen titles |
| `titleLarge` | 20 / w600 | section headers, app bar titles |
| `titleMedium` | 16 / w600 | list item titles, card headers |
| `bodyLarge` | 16 / w400 | default body text |
| `bodyMedium` | 14 / w400 | secondary body, descriptions |
| `labelLarge` | 14 / w600 | button labels |
| `bodySmall` / `labelSmall` | 12 / w400 | captions, metadata, hints |

Font family `Inter` (`AppFonts.sans`) applied across the `TextTheme`.

## Spacing & layout

- Scale: `4 / 8 / 12 / 16 / 24 / 32 / 48` (`AppSpacing.*`). Never hardcode a pixel gap.
- Screen edge padding: `AppSpacing.lg` (16). Card padding: `AppSpacing.lg` (16).
  Section gaps: `AppSpacing.xl` (24). Form field gaps: `AppSpacing.md` (12).
- Use `SafeArea` on every screen; respect notches/insets. Lists scroll; content never
  overflows horizontally.
- Corner radii from `AppRadius` (cards `md`, buttons `md`, chips/avatars `pill`).
- Elevation via M3 surface tints (`surfaceContainer*`), not hand-set shadows.

## States (required)

Anything interactive ships: default, pressed, focused, disabled, loading. Anything
async ships: loading, empty, error, success. Loading prefers skeletons or a centered
progress for first load; inline spinners for actions. Errors render a `Failure`-driven
message + retry; empty states render an explicit empty widget (no blank screen).

## Widget / component registry (living document)

Updated by Builders after every new shared widget; checked by Builders before every
new widget; audited by QA and `/review`. Feature-local widgets that are not reused
live under the feature and need not be registered; anything shared MUST be. One entry
per widget:

```markdown
### WidgetName
- Path: `lib/shared/widgets/widget_name.dart` (or `lib/features/<name>/presentation/widgets/...`)
- Purpose: one line
- Props: the public constructor surface (named, typed)
- States handled: default / pressed / disabled / loading (+ async states if applicable)
- Used by: screens/features using it
```

_(empty — first entry added by the first UI task)_

## Invariants

1. No visual value outside this file / `core/theme`. Zero inline `Color(0x...)`, no
   magic font sizes, no magic spacing — read tokens via `Theme.of(context)` or the
   `AppSpacing`/`AppRadius` scales.
2. New tokens are added here (and in `core/theme`) first, then used — never
   inline-then-backfill.
3. The registry never goes stale: building a shared widget and not registering it is a
   QA rejection.
4. Both themes stay in lockstep: any token that affects color must resolve correctly in
   light AND dark.
