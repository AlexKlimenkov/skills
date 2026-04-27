# Advanced Patterns

Use this file when implementing row reorder, resource panels, undo/redo, working calendars, zoom, or designing a backend schema for Gantt data.

## Contents
- Row Reorder With Sortorder Persistence
- Resource Panel And Workload Visualization
- Working Calendar
- Zoom
- Undo/Redo With State Management
- Backend Schema Template

## Row Reorder With Sortorder Persistence

- enable row ordering through documented Gantt config
- verify reorder event names and signatures with MCP before wiring handlers
- treat reorder as a dedicated flow
- do not route reorder through the normal single-task update path
- when a row moves, rebuild the full ordered task list in memory
- recompute `sortorder` for all affected tasks, not only the moved task
- persist the full reordered set
- if hierarchy changes during the move, persist `parent_id` alongside `sortorder`
- normal task updates must preserve current `sortorder` unless a dedicated reorder flow changes it
- load tasks with `ORDER BY sortorder` from the backend
- keep the reorder implementation targeted; do not refactor unrelated CRUD paths during this change
- if DataProcessor also marks the moved task as updated, suppress that single update when the batch reorder already persists sortorder and parent
- ignore reorder persistence for temporary task ids

## Resource Panel And Workload Visualization

- Gantt does not enforce a fixed task field for resource binding
- the relation is defined by `gantt.config.resource_property`, and resource features use `task[gantt.config.resource_property]`

Guardrails:

- use one consistent resource model across the feature
- do not mix task fields and assignment datasets
- ensure UI, Gantt config, and persistence use the same field or assignment structure

Product integration:

- build the resource list from real project data
- include an "Unassigned" option only if needed
- ensure lightbox selection maps to the same field or assignment structure
- if using assignments, drive both UI and persistence from the assignments dataset
- format workload values according to product rules (e.g. hours)

Layout:

- include `resourceGrid` and `resourceTimeline` or `resourceHistogram` when needed
- use shared horizontal scrolling for combined views

Notes:

- resource datastore may require initialization after `gantt.init`

### Supported Resource Binding Models

Single-resource field on task:

```ts
config.resource_property = "user_id";
// task.user_id <-> resource.id
```

Multiple resource IDs on task:

```ts
config.resource_property = "users";
// task.users = [2, 3]
```

Assignment objects on task:

```ts
config.resource_property = "users";
// task.users = [{ resource_id: 2, value: 8 }, { resource_id: 3, value: 4 }]
```

Separate assignments list:

```ts
{
  tasks: [...],
  links: [...],
  resources: [...],
  assignments: [{ id: 1, task_id: 5, resource_id: 2, value: 8 }]
}
```

## Working Calendar

- keep one source of truth for working and non-working time rules
- configure working time through documented Gantt APIs
- call `gantt.setWorkTime(...)` only with verified arguments for the installed version
- use templates for calendar-driven cell styling when appropriate
- re-render Gantt after changing calendar rules

Common pattern:

```ts
gantt.config.work_time = true;
gantt.config.correct_work_time = true;
gantt.setWorkTime({ day: 0, hours: false });
gantt.setWorkTime({ day: 6, hours: false });
```

Calendar-driven styling should use templates rather than global date math in CSS:

```ts
gantt.templates.timeline_cell_class = (_task, date) =>
  isNonWorking(date) ? "weekend-cell" : "";
gantt.templates.scale_cell_class = (date) =>
  isNonWorking(date) ? "weekend-cell" : "";
```

## Zoom

Use the built-in zoom extension for multi-level zoom:

```ts
gantt.ext.zoom.init({ levels: [...] });
gantt.ext.zoom.setLevel("day");
```

Common zoom levels:
- `hour`
- `day`
- `week`
- `month`
- `year`

For manual zoom (without the extension):
- update `gantt.config.scales`
- call `gantt.render()` after changes


## Undo/Redo With State Management

- decide whether undo/redo is Gantt-managed, app-managed, or both
- if using an app store, snapshot normalized task and link data only
- ensure undo/redo updates persistence if state must survive reload
- avoid wiping the undo/redo stack during save or refetch unless intended
- after app-managed undo/redo, reparse Gantt from the current store snapshot

## Backend Schema Template

Recommended practical schema for a writable Gantt-backed app with tasks, links, and optional resources. This is not a DHTMLX-mandated contract. Adjust the field names and tables to fit the app and backend.

Tasks table:
- `id` - UUID, primary key
- ID type must be consistent across tasks and links
- `project_id` - strongly recommended foreign key to a `projects` table for multi-project apps
- `text` - task name
- `start_date` - timestamp (nullable for projects that derive dates from children)
- `duration` - integer in the configured project duration unit (nullable, >= 0 when set)
- `progress` - numeric, constrained between 0 and 1
- `parent_id` - self-referencing foreign key to tasks (must not equal `id`), cascade on delete
- use a consistent root value for top-level tasks (e.g. `0` or `null`)
- `sortorder` - strongly recommended integer field for persistent row ordering
- `type` - one of `task`, `project`, `milestone`
- resource relation:
  - either a task field named to match `config.resource_property`
  - or no task field at all when the app uses separate resource assignments

Duration storage guardrail:
- treat `duration_unit` as a project-level storage decision, not just a display preference
- choose the smallest unit the product must support at project start, because changing from `day` to `hour` or `minute` later usually requires conversion of persisted task durations
- stored durations are integers, so `duration_unit="day"` supports whole-day tasks only and cannot represent a plain 4-hour task as `duration: 4`
- use `duration_unit="hour"` when tasks like 4 hours must be stored directly, and `duration_unit="minute"` when the product needs minute-level precision

Links table:
- `id` - UUID, primary key
- `project_id` - strongly recommended foreign key to a `projects` table for multi-project apps
- `source` - foreign key to task, cascade on delete
- `target` - foreign key to task, cascade on delete (must not equal `source`)
- `type` - one of `0`, `1`, `2`, `3`

Recommended constraints:
- unique constraint on `(project_id, source, target, type)` for links
- cascade on delete for project foreign keys in both tables

Recommended indexes:
- `project_id` on both tables
- `parent_id` on tasks
- `source` and `target` on links
