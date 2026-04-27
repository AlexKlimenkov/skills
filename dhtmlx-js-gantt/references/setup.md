# JavaScript Setup

Use this file when initializing or configuring a core DHTMLX JavaScript Gantt instance.

## Installation

Trial:
```bash
npm install @dhtmlx/trial-gantt
```

Commercial (requires private registry setup and login):
```bash
npm config set @dhx:registry=https://npm.dhtmlx.com
npm login --registry=https://npm.dhtmlx.com --scope=@dhx
npm install @dhx/gantt
```
The user must generate credentials in the [Client's Area](https://dhtmlx.com/clients/) and run the registry config and login commands themselves before the package can be installed. See [installation docs](https://docs.dhtmlx.com/gantt/guides/installation/#npmevaluationandproversions) for details.

CSS import must match the installed package and must be a separate import line:

```ts
import "@dhtmlx/trial-gantt/dist/react-gantt.css";
```

or

```ts
import "@dhx/gantt/dist/react-gantt.css";
```

## Imports

Use the import style that matches the installed package. Check `package.json`, existing imports, and lockfiles before changing package names.

Trial package (default singleton usage):

```ts
import { gantt } from "@dhx/trial-gantt";
import "@dhx/trial-gantt/codebase/dhtmlxgantt.css";
```

Commercial package:

```ts
import { gantt } from "@dhx/gantt";
import "@dhx/gantt/codebase/dhtmlxgantt.css";
```

Instance factory pattern (used in some examples):

```ts
import { Gantt } from "@dhx/trial-gantt";
import "@dhx/trial-gantt/codebase/dhtmlxgantt.css";

const gantt = Gantt.getGanttInstance();
```

Do not mix singleton and factory-instance patterns in the same feature unless the feature explicitly manages multiple instances.

## Vite

Include the installed Gantt package in `optimizeDeps` when Vite has trouble pre-bundling or the project already follows this pattern.

For trial package:

```ts
// vite.config.ts
export default {
  optimizeDeps: {
    include: ["@dhx/trial-gantt"],
  },
};
```

For commercial package:

```ts
export default {
  optimizeDeps: {
    include: ["@dhx/gantt"],
  },
};
```

## Height Rule

The Gantt container must have a defined height:

```html
<div id="gantt_here" style="height: 600px;"></div>
```

If using `height: 100%`, all parent elements must also have a defined height. Otherwise, the container will collapse.

## Initialization

Initialize Gantt after the container exists:

```ts
gantt.init("gantt_here");
```

or:

```ts
gantt.init(container);
```

Load client-side data with:

```ts
gantt.parse({ tasks, links });
```

Keep all Gantt-related logic (initialization, configuration, data loading, events, DataProcessor, cleanup) in a single module. Avoid scattering direct Gantt calls across the app.

## Cleanup

Dispose Gantt when the container is removed:

```ts
gantt.destructor();
container.innerHTML = "";
```

## Config And Templates

Common configuration areas:

- `gantt.config.columns` — grid columns and rendering
- `gantt.config.scales` — timeline scales and headers
- `gantt.config.scale_height`, `row_height`, `bar_height` — sizing
- `gantt.config.layout` — layout and view structure
- `gantt.config.lightbox` — task editing UI
- interaction settings (`readonly`, etc.)

Templates:

- `gantt.templates.*` — formatting of task bars, grid cells, scale labels, tooltips, and CSS classes

Example:

```ts
gantt.config.columns = [
  { name: "text", label: "Task name", tree: true, width: "*" },
  { name: "start_date", label: "Start", align: "center", width: 100 },
  { name: "duration", label: "Duration", align: "center", width: 80 },
  { name: "add", label: "", width: 44 },
];

gantt.config.scales = [
  { unit: "month", step: 1, format: "%F %Y" },
  { unit: "day", step: 1, format: "%d %M" },
];

// Task bar text
gantt.templates.task_text = (start, end, task) => {
  return task.text;
};

// Highlight timeline cells (e.g. weekends)
gantt.templates.timeline_cell_class = (task, date) => {
  const day = date.getDay();
  return day === 0 || day === 6 ? "weekend" : "";
};

// Scale styling
gantt.templates.scale_cell_class = (date) => {
  const day = date.getDay();
  return day === 0 || day === 6 ? "weekend" : "";
};

// Grid column template
gantt.config.columns[0].template = (task) => {
  return task.text.toUpperCase();
};
```

Do not guess template signatures. Verify unfamiliar templates with MCP.

## Read-Only Mode

To make the chart non-editable:

```ts
gantt.config.readonly = true;
```

This disables built-in editing (drag, resize, inline edit, lightbox).

Note that API methods still work. If needed, block mutations in your own event handlers or UI.

Also remove or hide app-level controls that mutate data (e.g. add buttons, undo/redo).
