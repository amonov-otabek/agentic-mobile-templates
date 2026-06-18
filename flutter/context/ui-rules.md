# UI Rules

Human-readable constraints for everything user-facing. The machine-readable design
system (tokens, theme values, component registry) lives in `context/ui-context.md` — this
file says how to use it. Builders read both before any UI task; QA enforces both.

## The golden rules

1. **Tokens only.** Zero hardcoded hex colors, pixel font sizes, or magic spacing
   values. Every visual value comes from a token in `context/ui-context.md`. This is a QA
   rejection, not a style nit.
2. **No static data as props.** Nav items, tab labels, and other build-time
   lists live INSIDE the widget as explicit Dart — screens pass only dynamic data.
   No mapping over static literal arrays passed as props.
3. **Screens orchestrate, widgets render.** A screen composes sections; it never
   inlines section trees. Extract feature-local widgets to
   `lib/features/<name>/presentation/widgets/<concern>.dart` (shared ones to
   `lib/shared/widgets/`).
4. **Check the registry first.** Before building any component, check the registry
   in `context/ui-context.md`. If it exists, reuse it exactly; if you build it, register it.

## Layout

- Screen edge padding from spacing tokens (`AppSpacing.lg`); content adapts to phone
  and tablet widths (use `LayoutBuilder`/`MediaQuery` where a wider layout helps).
- `SafeArea` on every screen; respect notches/insets. No horizontal overflow; content
  wraps or scrolls. Respect the OS text-scale setting.

## Typography

- Font family `Inter` (`AppFonts.sans`) applied once across the Material 3 `TextTheme`
  in `core/theme`; widgets read `Theme.of(context).textTheme.*`.
- The scale (headline / title / body / label levels with weight + size) is defined in
  `context/ui-context.md` — never restate sizes inline.

## Components

For each core widget (buttons, cards, inputs, badges, list tiles, nav bars), follow
the spec in `context/ui-context.md` exactly. Required interaction states for anything
interactive: default, pressed, focused, disabled, loading. Required data
states for anything async: loading, empty, error, success.

## Accessibility (blocking)

- Text contrast ≥ 4.5:1; visible focus/selection states.
- `Semantics` labels on icon-only controls; semantic labels on meaningful images;
  labels on inputs. Never use color as the only signal.
- Touch targets ≥ 48×48 logical px; controls disabled while their async action runs.
- Text scales with the OS setting — no fixed-height containers that clip scaled text.

## Motion and performance

- Respect the OS reduce-motion / disable-animations setting (`MediaQuery.disableAnimations`).
- Prefer `const` widgets to cut rebuild cost; reserve space for async content to avoid jank.
- Images: cache and size appropriately; never block first paint on a network image.

## Do not

- Use a raw `Color(0x...)` or magic number where a token exists.
- Introduce a new visual pattern when the registry already has one for the job.
- Add a package or icon set not listed in `context/ui-context.md` / `context/architecture.md`.
