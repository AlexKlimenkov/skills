# Styling And Theming

Use this file when theming Scheduler, matching an app design system, or styling events, rows, scales, and dialogs.

## Contents
- Styling Priority
- Theme Variables First
- Use Config/XY For Layout And Geometry
- Data-Driven Styling
- Rows And Timeline Cells
- Common Selectors And Escape Hatches
- Practical Guardrails

## Styling Priority

Use this order by default:
1. CSS variables for global theme alignment
2. templates and config/xy for semantic or data-driven styling
3. direct CSS selectors only for structural exceptions

## Theme Variables First

CSS variables are the preferred customization path.

Define shared variables at `:root` so inheritance works correctly across the component:

```css
:root {
  --dhx-scheduler-font-family: Inter, sans-serif;
  --dhx-scheduler-font-size: 14px;

  --dhx-scheduler-container-background: #ffffff;
  --dhx-scheduler-container-color: #0f172a;

  --dhx-scheduler-event-background: #2563eb;
  --dhx-scheduler-event-color: #ffffff;
}
```

Map app design tokens to Scheduler variables instead of inventing Scheduler-only colors.

## Use Config/XY For Layout And Geometry

Some visible behavior belongs to Scheduler config/xy, not CSS:
- `config.first_hour`
- `config.last_hour`
- `config.hour_size_px`
- `config.time_step`
- `config.readonly`
- `xy` sizing values

Do not force these through CSS when a documented config/xy option exists.

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
- these fields are event-level visual overrides
- they can conflict with class-based styling expectations in complex themes

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
- `.dhx_cal_container .dhx_mini_calendar .dhx_month_head`

Dialogs and overlays:
- `.dhx_cal_light`

Prefer scoped selectors under a local wrapper class to avoid global collisions.

## Practical Guardrails

- prefer CSS variables for theme-wide restyling before touching selectors
- prefer templates when styling depends on event, slot, or section data
- prefer scoped selectors over global overrides when changing only one view area
- do not force config/xy-controlled behavior through CSS
- keep theme source unified with the app-level theme state
