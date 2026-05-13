# Advanced Patterns

Use this file when implementing custom views, plugins, advanced filtering, undo/redo, or scheduling safeguards.

## Contents
- Views And Plugins
- Resource/Section Mapping
- Undo/Redo With State Management
- Conflict And Availability Guards
- Recurring Exceptions
- Custom Fields And Backend Mapping

## Views And Plugins

Activate every non-default view through the `plugins` prop and register its configuration through the `views` prop.

Units view (Pro):
- enable with `plugins={{ units: true }}`
- every event must carry the field named by the `property` config (for example `unit_id`)
- pass a `UnitsViewConfig` entry via the `views` prop with `name`, `property`, and `list` ({ key, label } items)
- `key` values in `list` and the event field type/format must match exactly

Timeline view (Pro):
- enable with `plugins={{ timeline: true }}` (and `treetimeline` or `daytimeline` for those render modes)
- every event must carry the field named by `y_property`
- pass a `TimelineViewConfig` entry via the `views` prop with `name`, `y_property`, `y_unit`, `x_unit`, `x_step`, plus `render` (`bar` | `cell` | `tree` | `days`)
- `days` mode requires the time scale to cover exactly one day

Grid view (core):
- `plugins={{ grid_view: true }}`
- pass a `GridViewConfig` via the `views` prop with `fields` (array of column definitions)

Feature-gate UI when the build is not Pro — `timeline`, `units`, `treetimeline`, `daytimeline` are not available in Individual/GPL editions.

## Resource/Section Mapping

- keep one canonical event field for section/resource assignment; the field name must match the view config:
  - units view → `property` (e.g. `unit_id`)
  - timeline view → `y_property` (e.g. `section_id`)
  - these are scheduler-specific names; do not borrow `resource_property` from Gantt
- ensure section keys and event assignment values use the same ID type/format
- map section labels from persisted data, not hardcoded display-only arrays
- if unassigned events are supported, keep their behavior explicit in UI and persistence
- keep row/section mapping deterministic across reloads and refetches

Guardrails:
- do not silently remap unknown section IDs
- verify timeline/units configuration details with MCP before implementation

## Undo/Redo With State Management

- use refs to access the latest state snapshot inside `data.save` callbacks — do not rely on render-time closures
- if using Redux or another external store for Scheduler history, normalize date values before writing to the store (state must be serializable)
- undo and redo actions must also update persistence — client-only history creates data gaps on reload
- persist history snapshots when rehydration across sessions is required
- save or refetch operations must not wipe the undo/redo history stack
- keep history refs and store state in sync after every mutation

## Conflict And Availability Guards

For simple non-overlap rules, prefer the built-in `collision` plugin instead of hand-rolled checks:

```ts
const plugins: SchedulerPlugins = { collision: true };
const config: SchedulerConfig = { collision_limit: 1 };
```

For app-specific rules (location scope, business hours, role-based limits) keep a custom check path:
- run checks before persistence
- typical checks: overlap on the same assignment field (for example `unit_id`), location scope, business-hours window
- keep the check path explicit: `candidate event -> checks -> decision -> persist`
- if conflicts need confirmation, require explicit user confirmation before save
- if conflicts are blocked, stop the write and keep UI state consistent with persisted state

## Recurring Exceptions

Editing a recurring series can produce three persistence outcomes depending on what the user picks in `modals.onRecurrenceConfirm`:

- `"series"` — update the series row in place
- `"occurrence"` — create a new exception row with `recurring_event_id` pointing at the series and `original_start` set to the original occurrence date
- `"following"` — split the series; verify the exact split semantics with MCP

Deleting a single occurrence inserts a new row with `recurring_event_id`, `original_start`, and `deleted: true` — do not remove the series row.

See [data-and-crud.md](./data-and-crud.md) for the full recurring data model.

## Custom Fields And Backend Mapping

- treat custom fields (for example `location_id`, `resource_id`, `service_id`, `status`, `notes`) as app-level schema and keep them explicit in all CRUD paths
- map backend rows to Scheduler events explicitly (including `start_date` / `end_date` conversion)
- map Scheduler write payloads to backend payloads explicitly; do not persist raw callback objects
- keep server-assigned IDs and client temporary IDs synchronized with a deterministic replacement flow
- if the app has scope constraints (for example location-scoped resources/services), validate them before persistence
