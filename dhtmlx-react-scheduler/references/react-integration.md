# React Integration

Use this file when setting up or modifying the React wrapper integration.

## Installation

Trial:
```bash
npm install @dhtmlx/trial-react-scheduler
```

Commercial (requires private registry setup and login):
```bash
npm config set @dhx:registry=https://npm.dhtmlx.com
npm login --registry=https://npm.dhtmlx.com --scope=@dhx
npm install @dhx/react-scheduler
```
The user must generate credentials in the [Client's Area](https://dhtmlx.com/clients/) and run the registry config and login commands themselves before the package can be installed. See [installation docs](https://docs.dhtmlx.com/scheduler/integrations/react/installation/) for details.

CSS import must match the installed package and must be a separate import line:

```ts
import "@dhtmlx/trial-react-scheduler/dist/react-scheduler.css";
```

or

```ts
import "@dhx/react-scheduler/dist/react-scheduler.css";
```

## Next.js

In Next.js (App Router), add `"use client"` at the top of the Scheduler component file. Next.js uses Server Components by default, but React Scheduler must render as a Client Component.

## Height Rule

The Scheduler container must have explicit height.

Valid:
```tsx
<div style={{ height: "600px" }}>
  <ReactScheduler ... />
</div>
```

## Documented Props

Use documented props only:

- `events`
- `view`
- `date`
- `markers`
- `plugins`
- `data`
- `locale`
- `theme`
- `templates`
- `config`
- `xy`
- `filter`
- `modals`
- `eventBoxRenderer`
- `views`
- `customViews`
- `customLightbox`
- `templateWrapper`
- `on<EventName>`
- `ref`

## Plugins

`plugins` is a **prop**. The core `scheduler.plugins({...})` instance method is still available through the ref and is fully supported, but the prop form is tidier and requires less code — prefer it from the React wrapper.

```tsx
import { type SchedulerPlugins } from "@dhx/react-scheduler";

const plugins: SchedulerPlugins = { recurring: true, collision: true };

<ReactScheduler events={events} plugins={plugins} />
```

Commonly used keys (grouped by purpose):
- features: `recurring`, `collision`, `limit`, `readonly`, `tooltip`, `quick_info`, `multiselect`, `multisection`, `drag_between`, `all_timed`, `active_links`, `container_autoresize`, `key_nav`, `editors`
- views (each requires its plugin): `agenda_view`, `daytimeline`, `grid_view`, `map_view`, `minical`, `timeline`, `treetimeline`, `units`, `week_agenda`, `year_view`

Plugin dependencies — verify with MCP before combining:
- `treetimeline` requires `timeline`
- `daytimeline` requires `timeline`
- recurring lightbox section requires `recurring`

## Views

Core views (work in all editions): `day`, `week`, `month`, `year`, `agenda`, `weekagenda`, `grid`, `map`.

Pro/Enterprise/Ultimate-only views: `timeline`, `units` (and the timeline variants `treetimeline`, `daytimeline`). Each Pro view requires its plugin enabled on the `plugins` prop and a configuration entry. Feature-gate UI when the build is not Pro.

- `view` prop selects the active view by name.
- `views` prop registers configurations for the available views (timeline/units/grid use the `TimelineViewConfig`, `UnitsViewConfig`, `GridViewConfig` shapes exported from the wrapper).
- `customViews` prop registers user-defined view definitions.

## Ref Access

Use `ref` with `ReactSchedulerRef` when direct instance access is required:

```tsx
const schedulerRef = useRef<ReactSchedulerRef>(null);
const scheduler = schedulerRef.current?.instance;
```

If you mutate data through the instance while also passing `events` as props, keep them synchronized. Otherwise React can overwrite those changes on re-render.

## Theme Rule

Use the app theme as the single source of truth. Pass the skin through the `theme` **prop**, not through `setSkin()` on the instance.

Built-in skins: `terrace` (default), `dark`, `material`, `flat`, `contrast_black`, `contrast_white`.

Typical mapping:
```tsx
const schedulerTheme = appTheme === "dark" ? "dark" : "terrace";
<ReactScheduler theme={schedulerTheme} ... />
```

Do not introduce separate Scheduler-only theme state if the app already has a global theme source.

## Config And Templates

Wrap `config` and `templates` in `useMemo` when they contain functions.

Typical:
```tsx
const config: SchedulerConfig = useMemo(() => ({ ... }), [deps]);
const templates: SchedulerTemplates = useMemo(() => ({ ... }), [deps]);
```

Common documented config areas:
- `config.first_hour`
- `config.last_hour`
- `config.hour_size_px`
- `config.time_step`
- `config.readonly`
- `config.drag_move`
- `config.drag_resize`
- `config.drag_create`
- `config.header`

Do not guess template callback signatures. Verify with MCP if there is any doubt.

## Event Handlers

When wiring `on<EventName>` callbacks:
- verify exact callback signatures with MCP
- keep handlers side-effect-safe and avoid stale closure state
- route writes through one persistence path (`data.save` or controlled app action), not both at once
