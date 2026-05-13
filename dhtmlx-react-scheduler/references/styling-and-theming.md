# Styling And Theming

Use this file when theming Scheduler, matching an app design system, or styling events, rows, scales, and dialogs.

## Contents
- Styling Priority
- Skins
- Theme Variables
- Use Config/XY For Layout And Geometry
- Data-Driven Styling
- Rows And Timeline Cells
- Common Selectors And Escape Hatches
- Practical Guardrails

## Styling Priority

Use this order by default:
1. **Skin selection** via the `theme` prop — the primary theme switch
2. CSS variables for fine adjustments on top of the chosen skin
3. templates and config/xy for semantic or data-driven styling
4. direct CSS selectors only for structural exceptions

## Skins

Built-in skins (pass the name through the `theme` prop, not `setSkin()`):
- `terrace` (default)
- `dark`
- `material`
- `flat`
- `contrast_black`
- `contrast_white`

```tsx
const schedulerTheme = appTheme === "dark" ? "dark" : "terrace";
<ReactScheduler theme={schedulerTheme} ... />
```

Choose the skin closest to the target visual, then adjust with CSS variables. Verify any skin name beyond this list with MCP — the React wrapper does not silently fall back.

## Theme Variables

After choosing a skin, override individual variables at `:root`:

```css
:root {
  --dhx-scheduler-font-family: Inter, sans-serif;
  --dhx-scheduler-font-size: 14px;

  --dhx-scheduler-base-colors-primary: #2563eb;
  --dhx-scheduler-container-background: #ffffff;
  --dhx-scheduler-container-color: #0f172a;

  --dhx-scheduler-event-background: #2563eb;
  --dhx-scheduler-event-color: #ffffff;
}
```

### High-Impact Variables

Typography:
- `--dhx-scheduler-font-family`
- `--dhx-scheduler-font-size`
- `--dhx-scheduler-heading-font-size`
- `--dhx-scheduler-heading-font-weight`
- `--dhx-scheduler-caption-font-size`

Base colors:
- `--dhx-scheduler-base-colors-primary`
- `--dhx-scheduler-base-colors-primary-hover`
- `--dhx-scheduler-base-colors-primary-active`
- `--dhx-scheduler-base-colors-warning`
- `--dhx-scheduler-base-colors-error`
- `--dhx-scheduler-base-colors-success`
- `--dhx-scheduler-base-colors-select`
- `--dhx-scheduler-base-colors-hover-color`
- `--dhx-scheduler-base-colors-border`
- `--dhx-scheduler-base-colors-text-base`
- `--dhx-scheduler-base-colors-text-light`
- `--dhx-scheduler-base-colors-background`
- `--dhx-scheduler-base-colors-disabled`

Container and popup surfaces:
- `--dhx-scheduler-container-background`
- `--dhx-scheduler-container-color`
- `--dhx-scheduler-popup-background`
- `--dhx-scheduler-popup-color`
- `--dhx-scheduler-popup-border`

Layout, borders, and shadows:
- `--dhx-scheduler-base-padding`
- `--dhx-scheduler-border-radius`
- `--dhx-scheduler-default-border`
- `--dhx-scheduler-box-shadow-s`
- `--dhx-scheduler-box-shadow-m`
- `--dhx-scheduler-box-shadow-l`
- `--dhx-scheduler-toolbar-height`

Events:
- `--dhx-scheduler-event-background`
- `--dhx-scheduler-event-color`
- `--dhx-scheduler-event-border`
- `--dhx-scheduler-event-text-primary`
- `--dhx-scheduler-event-marker-color`
- predefined palette: `--dhx-scheduler-event-blue`, `--dhx-scheduler-event-green`, `--dhx-scheduler-event-violet`, `--dhx-scheduler-event-yellow`

Scales and calendar:
- `--dhx-scheduler-scale-color`
- `--dhx-scheduler-timescale-background`
- `--dhx-scheduler-timescale-today-background`
- `--dhx-scheduler-month-header-color`
- `--dhx-scheduler-inactive-month-color`

Markers and blocked time:
- `--dhx-scheduler-blocked-time-background`
- `--dhx-scheduler-today-marker-color`

Lightbox:
- `--dhx-scheduler-lightbox-background`
- `--dhx-scheduler-lightbox-color`
- `--dhx-scheduler-lightbox-title-background`
- `--dhx-scheduler-lightbox-title-color`
- `--dhx-scheduler-lightbox-max-width`

### Map App Tokens To Scheduler Tokens

Use the app theme as the source of truth instead of inventing Scheduler-only colors:
- app accent → `--dhx-scheduler-base-colors-primary`
- app surface → `--dhx-scheduler-container-background`
- app body text → `--dhx-scheduler-container-color` and `--dhx-scheduler-base-colors-text-base`
- app border → `--dhx-scheduler-base-colors-border`
- app selected state → `--dhx-scheduler-base-colors-select`
- app hover state → `--dhx-scheduler-base-colors-hover-color`

### Variable Inheritance

Many Scheduler variables default to other variables (the doc marks these as "Defaults to" or "Derived from"). For example:
- `--dhx-scheduler-container-background` defaults to `--dhx-scheduler-base-colors-background`
- `--dhx-scheduler-popup-background` defaults to `--dhx-scheduler-container-background`
- `--dhx-scheduler-event-color` defaults to `--dhx-scheduler-event-text-primary`
- `--dhx-scheduler-lightbox-background` defaults to `--dhx-scheduler-popup-background`

Override the broader token (e.g. `--dhx-scheduler-base-colors-primary`) and let derived variables inherit. Override a derived variable only when that part of Scheduler should intentionally differ from the rest of the theme.

### Predefined Event Color Tokens

The `--dhx-scheduler-event-blue/green/violet/yellow` variables back the classname trick used in the quick-start. Combine an `event_class` template with a CSS class that re-binds `--dhx-scheduler-event-background`:

```ts
const templates: SchedulerTemplates = {
  event_class: (_start, _end, event) => event.classname || "",
};
```

```css
.violet { --dhx-scheduler-event-background: var(--dhx-scheduler-event-violet); }
.green  { --dhx-scheduler-event-background: var(--dhx-scheduler-event-green);  }
.yellow { --dhx-scheduler-event-background: var(--dhx-scheduler-event-yellow); }
```

This is the canonical data-driven coloring path. See [Data-Driven Styling](#data-driven-styling) for the broader pattern.

For the full catalog (including all inheritance defaults), see `scheduler-docs/docs/guides/theme-css-variables.md`.

## Use Config/XY For Layout And Geometry

Some visible behavior belongs to Scheduler config/xy, not CSS.

Config (passed as the `config` prop):
- `config.first_hour` — first visible hour in day/week/units views
- `config.last_hour` — last visible hour in day/week/units views
- `config.hour_size_px` — pixel height of one hour on the time scale of day, week, and units views (the primary control for vertical density of the time grid)
- `config.time_step` — minutes per slot for create/drag snapping
- `config.readonly` — disables user edits
- `config.drag_move` / `config.drag_resize` / `config.drag_create` — drag-interaction toggles

XY (passed as the `xy` prop, all values are numbers in pixels):
- `xy.bar_height` — height of event bars in the **Month** view (default `20`)
- `xy.min_event_height` — minimum height of an event's box in **day / week / units** views (default `40`); raise it to keep short events readable
- `xy.scale_width` — width of the Y-Axis (the time column) in **day / week / timeline / units** views (default `50`)
- `xy.scale_height` — height of the X-Axis row across all views (default `20`)
- `xy.month_scale_height` — top offset of an event in a cell in the **Month** view (default `20`)
- `xy.editor_width` — inline editor width in **day / week / units** views (default `140`)
- `xy.menu_width` — selection-menu width in **day / week / units** views (default `25`)
- `xy.scroll_width` — width of the scrollbar area (default `18`)
- `xy.margin_left` / `xy.margin_top` — main area margins
- `xy.map_date_width` / `xy.map_description_width` — column widths in the **Map** view
- `xy.lightbox_additional_height` — extra height added to the built-in lightbox

Example:
```tsx
const config: SchedulerConfig = useMemo(() => ({
  first_hour: 8,
  last_hour: 20,
  hour_size_px: 44,
  time_step: 15,
}), []);

const xy: SchedulerSizes = useMemo(() => ({
  scale_width: 60,
  min_event_height: 32,
  bar_height: 24,
}), []);

<ReactScheduler config={config} xy={xy} ... />
```

Do not force these through CSS when a documented config/xy option exists. Verify any xy field not listed above with MCP (see `scheduler-docs/docs/api/other/xy.md`).

### Scheduler Geometry Bridge (theme-level only)

Themes can drive a subset of `xy` values through CSS variables. Scheduler reads these from computed styles and applies them to `scheduler.xy.*`:
- `--dhx-scheduler-xy-scale_width`
- `--dhx-scheduler-xy-bar_height`
- `--dhx-scheduler-xy-month_head_height`
- `--dhx-scheduler-xy-scale_height`
- `--dhx-scheduler-xy-scroll_width`
- `--dhx-scheduler-config-form_wide` (enables the wide lightbox form)

Rule: prefer the `xy` prop for app-level overrides — explicit, typed, and discoverable in code. Use the geometry-bridge variables only when shipping a CSS-only theme that has no React entry point to set the `xy` prop.

## Data-Driven Styling

When styling depends on event data, prefer templates.

Use `event_class` for semantic styling by status/priority/type:

```ts
const templates = {
  event_class: (_start: Date, _end: Date, event: Event) => {
    return event.status ? `event-${event.status}` : "";
  },
};
```

```css
.event-confirmed {
  --dhx-scheduler-event-background: #16a34a;
}
```

Supported event color shortcut fields:
- `color`
- `textColor`

Important caveat:
- per-event `color` sets `--dhx-scheduler-event-background` as an inline CSS variable on the event element, overriding the theme value
- per-event `textColor` sets `--dhx-scheduler-event-color` the same way
- inline variables win over class-based overrides defined elsewhere in CSS, so they can quietly defeat class-driven theming in complex themes

Use event-level color fields only when the app truly stores per-event visual values as data. For reusable semantic styling, `event_class` is usually safer.

Advanced pattern:
- if colors come from backend-driven entities such as owners, stages, or statuses, generate CSS classes from the loaded dataset and return those classes from `event_class`

## Rows And Timeline Cells

Use view-specific row/cell templates:

Timeline/Units hooks:
- `..._row_class` templates for row-level classes
- `..._cell_class` templates for cell-level classes
- `..._cell_value` templates for cell content/value rendering

Day/Week hooks:
- `time_slot_class` for slot-level classes
- `time_slot_text` for slot-level content

Month/Calendar hooks:
- calendar/month templates and documented CSS classes for month/day rendering

## Common Selectors And Escape Hatches

Use these when variables or templates are not enough:

Event elements:
- `.dhx_cal_event`
- `.dhx_cal_header`
- `.dhx_title`
- `.dhx_cal_data`

Day/Week grid and scale:
- `.dhx_scale_holder`

Timeline/Units cells:
- `.dhx_matrix_cell`
- `.dhx_matrix_scell`
- `.dhx_matrix_line`

Month/Calendar elements:
- `.dhx_month_head`
- `.dhx_mini_calendar` (mini-calendar plugin root)

Dialogs and overlays:
- `.dhx_cal_light`

Prefer scoped selectors under a local wrapper class to avoid global collisions.

## Practical Guardrails

- prefer CSS variables for theme-wide restyling before touching selectors
- prefer templates when styling depends on event, slot, or section data
- prefer scoped selectors over global overrides when changing only one view area
- do not force config/xy-controlled behavior through CSS
- keep theme source unified with the app-level theme state
