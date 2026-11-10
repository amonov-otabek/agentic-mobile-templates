# UI Context — Design System

Machine-readable design system: tokens, theme, and the living component registry.
`context/ui-rules.md` says how to use these values; this file IS the values. Lock this
before the first UI task is built — retrofitting a design system is how features get
rebuilt (lesson learned from previous projects).

## Theme

- Mode: **light + dark, both** — selected automatically from the OS
  (`app.json` `userInterfaceStyle: "automatic"`, CSS `prefers-color-scheme`).
- Personality: clean / neutral / professional — a grayscale base with a single
  high-contrast primary and a red destructive accent. No decorative color.
- Rendering note: tokens are authored in `oklch`; `metro.config.js`
  (`withNativewind(..., { hexColors: true })`) compiles them to hex at build time
  because React Native cannot parse `oklch()` at runtime.

## Tokens

Single source of truth: **`src/global.css`** — Tailwind v4 `@theme inline` maps
NativeWind color names to CSS variables, and `:root` / `@media (prefers-color-scheme: dark)`
hold the actual values. Components reference tokens by class name only
(`bg-primary`, `text-muted-foreground`, `rounded-lg`) and **never restate values**.

```css
/* :root (light)                          dark override */
--background:           oklch(1 0 0)          /* → oklch(0.145 0 0)   near-black   */
--foreground:           oklch(0.145 0 0)      /* → oklch(0.985 0 0)   near-white   */
--card:                 oklch(1 0 0)          /* → oklch(0.205 0 0)                */
--card-foreground:      oklch(0.145 0 0)      /* → oklch(0.985 0 0)                */
--popover:              oklch(1 0 0)          /* → oklch(0.205 0 0)                */
--popover-foreground:   oklch(0.145 0 0)      /* → oklch(0.985 0 0)                */
--primary:              oklch(0.205 0 0)      /* → oklch(0.922 0 0)   inverts      */
--primary-foreground:   oklch(0.985 0 0)      /* → oklch(0.205 0 0)                */
--secondary:            oklch(0.97 0 0)       /* → oklch(0.269 0 0)                */
--secondary-foreground: oklch(0.205 0 0)      /* → oklch(0.985 0 0)                */
--muted:                oklch(0.97 0 0)       /* → oklch(0.269 0 0)                */
--muted-foreground:     oklch(0.556 0 0)      /* → oklch(0.708 0 0)               */
--accent:               oklch(0.97 0 0)       /* → oklch(0.269 0 0)                */
--accent-foreground:    oklch(0.205 0 0)      /* → oklch(0.985 0 0)                */
--destructive:          oklch(0.577 0.245 27.325) /* → oklch(0.704 0.191 22.216)  red */
--border:               oklch(0.922 0 0)      /* → oklch(1 0 0 / 10%)              */
--input:                oklch(0.922 0 0)      /* → oklch(1 0 0 / 15%)              */
--ring:                 oklch(0.708 0 0)      /* → oklch(0.556 0 0)   focus ring   */
--chart-1..5:           grayscale ramp        /* data viz                          */
--sidebar*:             sidebar surface set   /* reserved for nav surfaces         */

/* Radii — scaled off --radius: 0.625rem (10px) */
--radius-sm: calc(var(--radius) * 0.6)   /* ~6px  */
--radius-md: calc(var(--radius) * 0.8)   /* ~8px  */
--radius-lg: var(--radius)               /* 10px  */
--radius-xl: calc(var(--radius) * 1.4)   /* ~14px */
```

### Color usage guide

| Use | Token (NativeWind class) |
| --- | --- |
| Screen background | `bg-background` |
| Primary text | `text-foreground` |
| Secondary / hint text | `text-muted-foreground` |
| Card surface / text | `bg-card` / `text-card-foreground` |
| Primary action (button, link) | `bg-primary` / `text-primary` / `text-primary-foreground` |
| Secondary action | `bg-secondary` / `text-secondary-foreground` |
| Borders / dividers / inputs | `border-border` / `border-input` |
| Focus ring | `ring` |
| Error / destructive state | `text-destructive` / `bg-destructive/10` / `border-destructive` |

No raw hex in JSX. The one exception is an SVG icon `color` prop (NativeWind class
cannot drive SVG fill) — pass a token-derived color or the theme value, not an
arbitrary hex.

## Typography scale

Font: system default (RN platform font) via the `Text` primitive
(`shared/components/ui/Text.tsx`, base `text-base text-foreground`). Sizes/weights are
Tailwind classes:

| Style | Class | Use |
| --- | --- | --- |
| Title | `text-2xl font-semibold` | screen / page titles |
| Card title | `text-lg font-semibold` | card + section headers |
| Body | `text-base` (16px) | default text |
| Label / control | `text-sm font-medium` | input labels, buttons |
| Caption / hint | `text-sm text-muted-foreground` | subtitles, helper, errors |

## Spacing

Tailwind scale (4px unit): `gap-1 / 2 / 3 / 4 / 6` and `px-3 / px-4 / px-6` are the
common steps. Observed conventions: card padding `p-4`, form field group `gap-2`, form
sections `gap-4`, screen sections `gap-6`. Control heights: `h-9` (sm) / `h-11`
(default, ≥44px touch target) / `h-12` (lg). Radii: controls `rounded-lg`, cards
`rounded-xl`.

## Component registry (living document)

Updated by Builders after every new component; checked by Builders before every new
component; audited by QA and `/review`. One entry per component:

```markdown
### ComponentName
- Path: `src/shared/components/<area>/<Name>.tsx` or `src/features/<feature>/components/<Name>.tsx`
- Purpose: one line
- Variants/props: the public API surface
- Used by: screens/features using it
```

Existing shared primitives (reuse these — do not re-implement):

### Button
- Path: `src/shared/components/ui/button.tsx`
- Purpose: primary pressable action with loading state.
- Variants/props: `variant` (default | secondary | outline | ghost | destructive),
  `size` (default | sm | lg | icon), `label`, `loading`, `disabled`, `children`.
- Used by: all forms (sign-in, sign-up, etc.).

### Text
- Path: `src/shared/components/ui/Text.tsx`
- Purpose: typography primitive applying theme foreground + className.
- Variants/props: all `TextProps` + `className`.

### Input
- Path: `src/shared/components/ui/input.tsx`
- Purpose: themed text field with invalid state.
- Variants/props: all `TextInputProps` + `invalid`.

### Field
- Path: `src/shared/components/ui/field.tsx`
- Purpose: Label + control + error in a vertical group (exports `Field`, `FieldError`).
- Variants/props: `label`, `error?`, `children`.

### Label
- Path: `src/shared/components/ui/label.tsx`
- Purpose: form/control label text.

### Card
- Path: `src/shared/components/ui/card.tsx`
- Purpose: bordered surface container (exports `Card`, `CardHeader`, `CardTitle`,
  `CardDescription`, `CardContent`, `CardFooter`).

### Checkbox
- Path: `src/shared/components/ui/checkbox.tsx`
- Purpose: controlled checkbox with accessibility state.
- Variants/props: `checked`, `onCheckedChange`, `className?`.

### Toaster
- Path: `src/shared/components/ui/Toaster.tsx`
- Purpose: app-root toast host (driven by `shared/hooks/useToast`); tap to dismiss.

### Icons
- Path: `src/shared/components/widgets/Icons.tsx`
- Purpose: central `lucide-react-native` icon map; callers pass `color`/`size`.

### LocaleSwitch
- Path: `src/shared/components/widgets/LocaleSwitch.tsx`
- Purpose: language switcher (en / ru / uz).

_(feature components — sign-in-form, sign-up-form, etc. — registered by their feature)_

## Invariants

1. No visual value outside `src/global.css` (consumed as NativeWind classes).
2. New tokens are added to `global.css` first, then used — never inline-then-backfill.
3. The registry never goes stale: building a component and not registering it is a QA
   rejection. Check the registry before building — reuse beats re-create.
