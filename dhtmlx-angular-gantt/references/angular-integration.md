# Angular Integration

Use this file when setting up or modifying the Angular wrapper integration.

## Installation

Trial:
```bash
npm install @dhtmlx/trial-angular-gantt
```

Commercial (requires private registry setup and login):
```bash
npm config set @dhx:registry=https://npm.dhtmlx.com
npm login --registry=https://npm.dhtmlx.com --scope=@dhx --auth-type=legacy
npm install @dhx/angular-gantt
```
The user must generate credentials in the [Client's Area](https://dhtmlx.com/clients/) and run the registry config and login commands themselves before the package can be installed. See [installation docs](https://docs.dhtmlx.com/gantt/integrations/angular/installation/) for details.

Wrapper import and CSS import must match the installed package.

Trial:
```ts
import { DhxGanttComponent } from "@dhtmlx/trial-angular-gantt";
```

```css
@import "@dhtmlx/trial-angular-gantt/dist/angular-gantt.css";
```

Commercial:
```ts
import { DhxGanttComponent } from "@dhx/angular-gantt";
```

```css
@import "@dhx/angular-gantt/dist/angular-gantt.css";
```

For NgModule apps, import `DhxGanttModule` from the same package channel.

## CSS Placement Rule

Prefer global wrapper CSS import (for example in `src/styles.css`).

If wrapper CSS or `.dhx-*` overrides are placed in a component stylesheet, Angular style encapsulation can interfere with Gantt internals. In that case, use `ViewEncapsulation.None` in that component.

## Height Rule

The Gantt container must have explicit height via parent chain and chart block.

Valid:
```html
<div style="height: 600px;">
  <dhx-gantt [tasks]="tasks" [links]="links"></dhx-gantt>
</div>
```

## Documented Inputs

Use documented inputs only:

- `tasks`
- `links`
- `resources`
- `resourceAssignments`
- `baselines`
- `config`
- `templates`
- `plugins`
- `calendars`
- `markers`
- `locale`
- `theme`
- `data`
- `events`
- `customLightbox`
- `groupTasks`
- `filter`
- `resourceFilter`

## Events And Ready

Use `events` for interaction behavior and `(ready)` for one-time initialized instance access.

```html
<dhx-gantt [events]="events" (ready)="onReady($event)"></dhx-gantt>
```

```ts
events = {
  onTaskCreated: (task: any) => true,
  onBeforeLightbox: (taskId: string | number) => true,
};

onReady({ instance }: { instance: GanttStatic }): void {
  instance.showDate(new Date());
}
```

## Instance Access

Use `@ViewChild(DhxGanttComponent)` when direct instance access is required:

```ts
@ViewChild(DhxGanttComponent) ganttCmp?: DhxGanttComponent;

showToday(): void {
  this.ganttCmp?.instance?.showDate(new Date());
}
```

If you mutate data through the instance while also passing `tasks` and `links` as inputs, keep them synchronized. Otherwise Angular input updates can overwrite chart-side changes.

## Theme Rule

Use app theme as the single source of truth.

Preferred built-in themes:
- `"terrace"`
- `"dark"`

Typical mapping:
```ts
ganttTheme: "dark" | "terrace" = appTheme === "dark" ? "dark" : "terrace";
```

Template usage:
```html
<dhx-gantt
  [tasks]="tasks"
  [links]="links"
  [theme]="ganttTheme">
</dhx-gantt>
```

The `theme` input can be updated at runtime; keep it bound to app-level state/store so Gantt follows light/dark changes automatically.

Do not introduce separate Gantt-only theme state if the app already has a global theme source.

## Resources And Calendars

When implementing resources:
- pass the resource dataset through the `[resources]` input
- pair it with `[resourceAssignments]` when assignments live in a separate datastore
- keep resource IDs and task assignment fields consistent across both inputs
- derive display labels from the same persisted field used for editing
- include an explicit unassigned option when needed
- keep the resource grid, resource timeline, and workload views tied to the same source of truth

When implementing working time:
- keep one source of truth for working and non-working time rules
- pass calendar definitions through the `[calendars]` input; calendars are synchronized by `id`, so keep IDs stable across reloads
- use documented templates (`timeline_cell_class`, `task_cell_class`) for non-working-day styling
- do not infer calendar engine behavior — verify with MCP if a rule is unclear

## Plugins

Activate documented plugins through the `[plugins]` input. Each plugin is a key on the map; pass `true` to enable.

```html
<dhx-gantt
  [tasks]="tasks"
  [links]="links"
  [plugins]="{ auto_scheduling: true, critical_path: true }">
</dhx-gantt>
```

Common plugins: `auto_scheduling`, `critical_path`. Advanced plugin behavior (scheduling rules, workload calculation, overload thresholds) is not derivable from the input shape alone — verify expected behavior with MCP before relying on it.

## Markers, Baselines, Filter, Locale

Additional documented inputs for advanced timelines:

- `[markers]` (`Marker[] | null`): vertical timeline markers synchronized by `id`. Keep marker IDs stable so updates patch the existing marker rather than replacing the set.
- `[baselines]` (`any[] | null`): baseline dataset for planned-vs-actual comparison. Treat baselines as immutable alongside the live task set; do not mutate them through `data.save`.
- `[filter]` (`TaskFilter`): `(task) => boolean` predicate, or `null` to show all. Keep a stable reference when the filter logic has not changed — the wrapper compares by identity and re-renders only when the reference changes.
- `[locale]` (`string | null`): locale name passed to `gantt.i18n.setLocale(...)`. Bind it to app-level locale state so Gantt follows app i18n changes automatically.

## Config And Templates

Use `config` and `templates` for declarative setup.

Common documented config areas:
- `config.columns`
- `config.scales`
- `config.readonly`
- `config.drag_move`
- `config.drag_resize`
- `config.resource_store`
- `config.resource_property`
- `config.layout`

Typical config:
```ts
config = {
  date_format: "%Y-%m-%d %H:%i",
  row_height: 36,
  bar_height: 24,
  scales: [
    { unit: "month", step: 1, format: "%F %Y" },
    { unit: "day", step: 1, format: "%d %M" },
  ],
  columns: [
    { name: "text", label: "Task", tree: true, width: "*" },
    { name: "start_date", label: "Start", align: "center" },
    { name: "duration", label: "Duration", align: "center" },
    { name: "add", width: 44 },
  ],
};
```

Do not guess template callback signatures. Verify with MCP if there is any doubt.

Example:
```ts
task_text: (_start, _end, task) => `#${task.id}: ${task.text}`
```

Template-driven styling example:
```ts
templates = {
  task_class: (_start: Date, _end: Date, task: any) =>
    (task.progress ?? 0) >= 1 ? "completed-task" : "",
  resource_cell_value: (_start: Date, _end: Date, _resource: any, tasks: any[]) =>
    `<div>${tasks.length * 8}h</div>`,
};
```

Use `templateComponent(...)` only when Angular component rendering inside Gantt templates is required.
